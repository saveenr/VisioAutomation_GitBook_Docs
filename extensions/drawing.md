# Drawing primitives

Visio's `IVisio.Page.DrawRectangle(x1, y1, x2, y2)` takes four loose `double`s. The extensions add overloads that accept a typed `Core.Rectangle` (or `Core.Point` for line endpoints), and add a few drawing primitives that the raw API doesn't expose at all (`DrawBezier`, `DrawPolyline`, `DrawQuarterArc`, `DrawNurbs`).

The same set is available on `IVisio.Page`, `IVisio.Master`, and `IVisio.Shape` (for drawing inside a group).

To use the extensions, add a `using` directive at the top of your file:

```csharp
using VisioAutomation.Extensions;
```

## Available methods

| Method                          | Returns | Notes                                                       |
| ------------------------------- | ------- | ----------------------------------------------------------- |
| `DrawRectangle(rect)`           | `Shape` | Bounding box.                                               |
| `DrawOval(rect)`                | `Shape` | Bounding box.                                               |
| `DrawLine(p1, p2)`              | `Shape` | Endpoints.                                                  |
| `DrawBezier(points)`            | `Shape` | Cubic Bezier; control + endpoint vertices.                  |
| `DrawPolyline(points)`          | `Shape` | Open polyline through the given points.                     |
| `DrawQuarterArc(p1, p2, flags)` | `Shape` | Arc swept from `p1` to `p2`; `flags` is `VisArcSweepFlags`. |
| `DrawNurbs(...)`                | `Shape` | NURBS curve. (Page only.)                                   |

## Example

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

## See also

* [Master dropping](dropping.md) &mdash; the other way to get shapes onto a page.
* [Geometry](../geometry.md) &mdash; building custom paths via `GeometrySection` / `GeometryRow` once a shape exists.
* [Extension methods overview](../extension-methods.md).
