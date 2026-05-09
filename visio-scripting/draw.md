# client.Draw

`client.Draw` draws geometry primitives directly on a page: rectangles, ovals, lines, beziers, polylines. Each method wraps the underlying `IVisio.Page.Draw*` call in an undo scope. The lower-level extension equivalents are on [Drawing primitives](../extensions/drawing.md).

## Methods

| Method                                                                                  | Returns         | Notes                                                                              |
| --------------------------------------------------------------------------------------- | --------------- | ---------------------------------------------------------------------------------- |
| `DrawRectangle(TargetPage, Rectangle)`                                                  | `IVisio.Shape`  | Rectangle by `VisioAutomation.Core.Rectangle`.                                     |
| `DrawRectangle(TargetPage, double x0, double y0, double x1, double y1)`                 | `IVisio.Shape`  | Rectangle by lower-left and upper-right.                                           |
| `DrawOval(TargetPage, Rectangle)`                                                       | `IVisio.Shape`  | Oval inscribed in the bounding rect.                                               |
| `DrawOval(TargetPage, double x0, double y0, double x1, double y1)`                      | `IVisio.Shape`  | Oval by bounding-rect coordinates.                                                 |
| `DrawLine(TargetPage, Point, Point)`                                                    | `IVisio.Shape`  | Line from `p0` to `p1`.                                                            |
| `DrawLine(TargetPage, double x0, double y0, double x1, double y1)`                      | `IVisio.Shape`  | Line by endpoint coordinates.                                                      |
| `DrawBezier(TargetPage, IEnumerable<Point>)`                                            | `IVisio.Shape`  | Bezier through the given control points.                                           |
| `DrawPolyLine(TargetPage, IList<Point>)`                                                | `IVisio.Shape`  | Polyline through the given points.                                                 |

`TargetPage.Auto` resolves to the active page at call time. Every method opens an undo scope so a single Undo reverses the draw.

## Removal candidates (CY27)

These methods are currently `public` but have no known consumers as of 2026-05. They are candidates for removal in CY27 (see [issue #183](https://github.com/saveenr/VisioAutomation/issues/183)). New code should not depend on them.

| Method                                              | Notes                                                                          |
| --------------------------------------------------- | ------------------------------------------------------------------------------ |
| `Duplicate(TargetSelection, int n)`                 | Duplicates the first selected shape `n` times; `client.Selection.DuplicateShapes` covers the common case. |

## Example

```csharp
using IVisio = Microsoft.Office.Interop.Visio;

// Rectangle, oval, line, polyline on the active page
var rect = client.Draw.DrawRectangle(VisioScripting.TargetPage.Auto, 0, 0, 4, 2);
var oval = client.Draw.DrawOval(VisioScripting.TargetPage.Auto, 5, 0, 9, 2);
var line = client.Draw.DrawLine(VisioScripting.TargetPage.Auto, 0, 3, 9, 3);

var points = new[]
{
    new VisioAutomation.Core.Point(0, 4),
    new VisioAutomation.Core.Point(2, 5),
    new VisioAutomation.Core.Point(4, 4),
    new VisioAutomation.Core.Point(6, 5),
};
var poly = client.Draw.DrawPolyLine(VisioScripting.TargetPage.Auto, points);
```

## See also

- [Drawing primitives](../extensions/drawing.md): the lower-level `Page.Draw*` extension methods.
- [Master](master.md): drop pre-built masters instead of drawing primitives.
- [Model](model.md): higher-level layouts (grid, tree, directed graph) that produce many shapes at once.
- [VisioScripting.Client overview](../visio-scripting.md): how `client.Draw` fits into the facade.
