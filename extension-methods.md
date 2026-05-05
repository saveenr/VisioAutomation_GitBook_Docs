# Extension methods

`VisioAutomation.Extensions` is a namespace of extension methods that fill in gaps in Visio's COM API &mdash; LINQ-friendly enumeration over collection types, drawing primitives that take typed `Core.Rectangle` / `Core.Point` arguments, ShapeSheet I/O that takes typed `Src` / `SidSrc` arguments, and a handful of one-off conveniences.

To use any of them, add a `using` directive at the top of your file:

```csharp
using VisioAutomation.Extensions;
```

## Categories

The extensions are grouped into six categories, each documented on its own page:

| Category                            | Covers                                                                                                  | Page                                                |
| ----------------------------------- | ------------------------------------------------------------------------------------------------------- | --------------------------------------------------- |
| LINQ bridges over COM collections   | `ToEnumerable()` / `ToList()` over `Pages`, `Shapes`, `Masters`, `Fonts`, etc.                          | [LINQ bridges](extensions/linq.md)                  |
| Drawing primitives                  | `DrawRectangle(rect)`, `DrawLine(p1, p2)`, `DrawPolyline`, `DrawBezier`, `DrawQuarterArc`, `DrawNurbs`. | [Drawing primitives](extensions/drawing.md)         |
| Master dropping                     | `Drop(master, point)`, `DropManyU(masters, points)`.                                                    | [Master dropping](extensions/dropping.md)           |
| Typed ShapeSheet I/O                | `GetFormulasU` / `GetResults<T>` / `SetFormulas` / `SetResults` keyed by `Src` / `SidSrc`.              | [Typed ShapeSheet I/O](extensions/shapesheet.md)    |
| Coordinates and bounding boxes      | `GetBoundingBox(args)`, `XYFromPage`, `XYToPage`, `ResizeToFitContents`.                                | [Coordinates and bounding boxes](extensions/coordinates.md) |
| One-offs                            | `Quit(force_close)`, `Close(force_close)`, `OpenStencil`, `Select`, view / window rects, `GetShapesFromIDs`. | [One-off extensions](extensions/misc.md)            |

## See also

* [Stencils and masters](stencils-and-masters.md) &mdash; uses `Drop` / `DropManyU` / `Documents.OpenStencil`.
* [Query the ShapeSheet](shapesheet/query-the-shapesheet.md), [Modify the ShapeSheet](shapesheet/modify-the-shapesheet.md) &mdash; the high-level wrappers around the typed ShapeSheet I/O extensions.
* [Application](application.md) &mdash; `ApplicationHelper` covers a few application-level operations not exposed as extension methods.
