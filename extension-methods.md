# Extension methods

`VisioAutomation.Extensions` is a namespace of extension methods that fill in gaps in Visio's COM API &mdash; LINQ-friendly enumeration over collection types, drawing primitives that take typed `Core.Rectangle` / `Core.Point` arguments, ShapeSheet I/O that takes typed `Src` / `SidSrc` arguments, and a handful of one-off conveniences.

To use them, add a `using` directive at the top of your file:

```csharp
using VisioAutomation.Extensions;
```

## LINQ bridges over the COM collection types

Visio's COM collection types (`IVisio.Pages`, `IVisio.Shapes`, etc.) don't implement `IEnumerable<T>` &mdash; they predate generics. The extensions add `ToEnumerable()` and `ToList()` overloads so you can use LINQ against them:

| Collection | Extensions |
| --- | --- |
| `Pages` | `ToEnumerable() : IEnumerable<Page>`, `ToList() : List<Page>`, `GetNamesU() : string[]` |
| `Shapes` | `ToEnumerable() : IEnumerable<Shape>`, `ToList() : List<Shape>` |
| `Masters` | `ToEnumerable() : IEnumerable<Master>`, `ToList() : List<Master>` |
| `Documents` | `ToEnumerable() : IEnumerable<Document>`, `ToList() : List<Document>` |
| `Fonts` | `ToEnumerable() : IEnumerable<Font>`, `ToList() : List<Font>` |
| `Layers` | `ToEnumerable() : IEnumerable<Layer>`, `ToList() : List<Layer>` |
| `Colors` | `ToEnumerable() : IEnumerable<Color>`, `ToList() : List<Color>` |
| `Connects` | `ToEnumerable() : IEnumerable<Connect>`, `ToList() : List<Connect>` |
| `Styles` | `ToEnumerable() : IEnumerable<Style>`, `ToList() : List<Style>`, `GetNamesU() : string[]` |
| `Selection` | `ToEnumerable() : IEnumerable<Shape>`, `ToList() : List<Shape>`, `GetIDs() : int[]`, `GetBoundingBox(args) : Rectangle` |
| `Section` | `ToEnumerable() : IEnumerable<Row>`, `ToList() : List<Row>` |
| `Windows` | `ToEnumerable() : IEnumerable<Window>`, `ToList() : List<Window>` |

Example:

```csharp
// Build a font-name to font-ID dictionary
var fontname_to_id = doc.Fonts.ToEnumerable()
    .ToDictionary(f => f.Name, f => f.ID);

// Find every page whose name starts with "Backup"
var backup_pages = doc.Pages.ToEnumerable()
    .Where(p => p.NameU.StartsWith("Backup"))
    .ToList();
```

## Drawing primitives

Visio's `IVisio.Page.DrawRectangle(x1, y1, x2, y2)` takes four loose `double`s. The extensions add overloads that accept a typed `Core.Rectangle` (or `Core.Point` for line endpoints), and add a few drawing primitives that the raw API doesn't expose at all (`DrawBezier`, `DrawPolyline`, `DrawQuarterArc`, `DrawNurbs`).

The same set is available on `IVisio.Page`, `IVisio.Master`, and `IVisio.Shape` (for drawing inside a group).

| Method | Returns | Notes |
| --- | --- | --- |
| `DrawRectangle(rect)` | `Shape` | Bounding box. |
| `DrawOval(rect)` | `Shape` | Bounding box. |
| `DrawLine(p1, p2)` | `Shape` | Endpoints. |
| `DrawBezier(points)` | `Shape` | Cubic Bezier; control + endpoint vertices. |
| `DrawPolyline(points)` | `Shape` | Open polyline through the given points. |
| `DrawQuarterArc(p1, p2, axis)` | `Shape` | Arc swept from `p1` to `p2`. |
| `DrawNurbs(...)` | `Shape` | NURBS curve. (Page only.) |

```csharp
var rect = new VisioAutomation.Core.Rectangle(0, 0, 4, 3);
var shape = page.DrawRectangle(rect);

var pts = new[] {
    new VisioAutomation.Core.Point(0, 0),
    new VisioAutomation.Core.Point(2, 1),
    new VisioAutomation.Core.Point(4, 0),
};
var poly = page.DrawPolyline(pts);
```

## Master dropping

Drop a master at a typed `Core.Point` instead of two loose doubles. The same set is available on `IVisio.Page`, `IVisio.Master`, and `IVisio.Shape`.

| Method | Returns |
| --- | --- |
| `Drop(master, point)` | `Shape` |
| `DropManyU(masters, points)` | `short[]` (the new shapes' IDs) |

```csharp
var stencil    = doc.Application.Documents.OpenStencil("basic_u.vss");
var rectmaster = stencil.Masters["Rectangle"];

var s = page.Drop(rectmaster, new VisioAutomation.Core.Point(2.0, 1.0));
```

`DropManyU` is the same operation in batch &mdash; one `IVisio.DropManyU` COM call for many shapes, much faster than calling `Drop` in a loop. `DropManyU` returns shape IDs; turn them back into objects with `Shapes.GetShapesFromIDs` (see below).

## ShapeSheet I/O on shapes / pages / masters

Typed accessors that take `Core.Src` / `Core.SidSrc` instead of raw section/row/cell shorts. The same set is available on `Shape`, `Page`, and `Master`.

| Method | Returns | Purpose |
| --- | --- | --- |
| `GetFormulasU(streamarray)` | `string[]` | Read formulas. |
| `GetResults<TResult>(streamarray, unitcodes)` | `TResult[]` | Read evaluated results in the requested type. |
| `SetFormulas(streamarray, formulas, flags)` | `int` | Bulk-write formulas. |
| `SetResults(streamarray, unitcodes, results, flags)` | `int` | Bulk-write evaluated values. |

In practice you'll usually go through the higher-level `CellQuery` / `SectionQuery` ([Query the ShapeSheet](shapesheet/query-the-shapesheet.md)) and `SrcWriter` / `SidSrcWriter` ([Modify the ShapeSheet](shapesheet/modify-the-shapesheet.md)), which wrap these.

## Geometry / coordinates

| Method | Receiver | Purpose |
| --- | --- | --- |
| `GetBoundingBox(args)` | `Page`, `Shape`, `Selection`, `Master` | Returns a `Core.Rectangle` for the requested bounding-box flavour. |
| `XYFromPage(point)` | `Shape` | Convert a page-coord `Point` into the shape's local coords. |
| `XYToPage(point)` | `Shape` | Convert a shape-local `Point` into page coords. |
| `ResizeToFitContents(padding)` | `Page` | Resize the page to its contents and add the given padding. |

```csharp
// Bounding box of every selected shape, on the page
var rect = window.Selection.GetBoundingBox(IVisio.VisBoundingBoxArgs.visBBoxUprightWH);

// Convert a click in page coords into the shape's local coords
var local = shape.XYFromPage(new VisioAutomation.Core.Point(5.0, 4.0));
```

## Other one-offs

| Method | Receiver | Purpose |
| --- | --- | --- |
| `Quit(force_close)` | `Application` | `Application.Quit()`, but with the option to suppress the unsaved-changes prompt (sets `AlertResponse` to "No" first). |
| `Close(force_close)` | `Document` | Same idea for closing a single document. |
| `OpenStencil(filename)` | `Documents` | Opens a file as a docked, read-only stencil; throws `VisioOperationException` if Visio refuses. |
| `Select(shapes)` | `Window` | Replace the current selection with the given shapes. |
| `GetViewRect()` / `SetViewRect(rect)` | `Window` | View-rectangle accessors. |
| `GetWindowRect()` / `SetWindowRect(rect)` | `Window` | Window-rectangle accessors (uses `System.Drawing.Rectangle`). |
| `Shapes.GetShapesFromIDs(ids)` | `IVisio.Shapes` | Turn an `IList<int>` (e.g., from `DropManyU`) back into `Shape` objects. |

## See also

* [Stencils and masters](stencils-and-masters.md) &mdash; uses `Drop` / `DropManyU` / `Documents.OpenStencil`.
* [Query the ShapeSheet](shapesheet/query-the-shapesheet.md), [Modify the ShapeSheet](shapesheet/modify-the-shapesheet.md) &mdash; the high-level wrappers around the typed ShapeSheet I/O extensions.
* [Application](application.md) &mdash; `ApplicationHelper` covers a few application-level operations not exposed as extension methods.
