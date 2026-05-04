# Geometry

Most Visio shapes are *masters* dropped onto a page &mdash; their geometry comes pre-baked. When you need to draw a custom path (a rectangle that's not a master, a polyline, an arc), you build it as one or more **geometry sections** in the shape's ShapeSheet. VisioAutomation exposes this via `VisioAutomation.Shapes.GeometryHelper`, `GeometrySection`, and `GeometryRow`.

## Concepts

A shape can have multiple geometry sections (`Geometry1`, `Geometry2`, ...). Each section has section-level properties (`NoFill`, `NoLine`, `NoShow`, `NoSnap`, `NoQuickDrag`) and a list of **rows**. Each row is a drawing instruction with a tag (`MoveTo`, `LineTo`, `ArcTo`, etc.) and up to seven cell values (`X`, `Y`, `A`, `B`, `C`, `D`, `E`). Most rows use just `X` and `Y`.

## Build a geometry section

Construct a `GeometrySection`, add rows describing the path, then call `Render(shape)` to commit it:

```csharp
var page  = doc.Pages.Add();
var shape = page.DrawRectangle(0, 0, 4, 4);

// Replace the rectangle's default geometry with a custom triangle
VisioAutomation.Shapes.GeometryHelper.Delete(shape);

var geom = new VisioAutomation.Shapes.GeometrySection();
geom.AddMoveTo(0, 0);
geom.AddLineTo(4, 0);
geom.AddLineTo(2, 3);
geom.AddLineTo(0, 0);

geom.Render(shape);
```

`Render` returns the section index Visio assigned and commits all rows in a single writer transaction.

## Section-level options

The five flags on a `GeometrySection` control how the section behaves:

| Field | Effect when set to `1` |
| --- | --- |
| `NoFill` | Don't fill the path even if the shape has a fill. |
| `NoLine` | Don't stroke the path even if the shape has a line. |
| `NoShow` | Hide the section (still participates in calculations). |
| `NoSnap` | Geometry vertices don't act as snap targets. |
| `NoQuickDrag` | Disable click-and-drag editing of vertices. |

```csharp
var geom = new VisioAutomation.Shapes.GeometrySection();
geom.NoLine = 1;     // fill only, no outline
geom.AddMoveTo(0, 0);
geom.AddLineTo(2, 0);
geom.AddLineTo(2, 2);
geom.AddLineTo(0, 0);
geom.Render(shape);
```

## Row factory methods

`GeometryRow` has static factories for each Visio row type. The most common ones:

```csharp
GeometryRow.CreateMoveTo(x, y);
GeometryRow.CreateLineTo(x, y);
GeometryRow.CreateArcTo(x, y, a);          // a = arc bulge
```

`GeometrySection` exposes the matching `AddMoveTo` / `AddLineTo` / `AddArcTo` shortcuts that build the row and append it in one step.

## Add another section to an existing shape

`GeometryHelper.AddSection(shape)` adds a new (empty) geometry section and returns its section index. `GeometrySection.Render` already calls this internally, so the typical pattern is to build a `GeometrySection`, set its rows, and call `Render` &mdash; you don't normally call `AddSection` directly.

## Delete all geometry sections

```csharp
VisioAutomation.Shapes.GeometryHelper.Delete(shape);
```

This removes every `Geometry1`, `Geometry2`, ... section from the shape.

## See also

* [Stencils and masters](stencils-and-masters.md) &mdash; for non-custom geometry, drop a master instead.
* [Modify the ShapeSheet](shapesheet/modify-the-shapesheet.md) &mdash; geometry rows are committed through the same `SrcWriter` infrastructure.
