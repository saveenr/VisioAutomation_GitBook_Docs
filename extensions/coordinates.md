# Coordinates and bounding boxes

Geometry helpers for the common coordinate-system conversions and bounding-box queries that the raw COM API exposes either awkwardly (out-parameters) or not at all.

To use the extensions, add a `using` directive at the top of your file:

```csharp
using VisioAutomation.Extensions;
```

## Available methods

| Method                          | Receiver                            | Purpose                                                                                  |
| ------------------------------- | ----------------------------------- | ---------------------------------------------------------------------------------------- |
| `GetBoundingBox(args)`          | `Page`, `Shape`, `Selection`, `Master` | Returns a `Core.Rectangle` for the requested bounding-box flavour.                       |
| `XYFromPage(point)`             | `Shape`                             | Convert a page-coord `Point` into the shape's local coords.                              |
| `XYToPage(point)`               | `Shape`                             | Convert a shape-local `Point` into page coords.                                          |
| `ResizeToFitContents(padding)`  | `Page`                              | Resize the page to its contents and add the given padding.                               |

## Examples

```csharp
// Bounding box of every selected shape, on the page
var rect = window.Selection.GetBoundingBox(IVisio.VisBoundingBoxArgs.visBBoxUprightWH);

// Convert a click in page coords into the shape's local coords
var local = shape.XYFromPage(new VisioAutomation.Core.Point(5.0, 4.0));
```

## See also

* [Page cells](../page-cells.md) &mdash; `PageHelper.ResizeToFitContents` is the underlying implementation, and is also reachable directly.
* [Geometry](../geometry.md) &mdash; the geometry-section model for shapes that need a custom path rather than just bounding-box / coordinate work.
* [Extension methods overview](../extension-methods.md).
