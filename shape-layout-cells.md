# Shape layout cells

`VisioAutomation.Shapes.ShapeLayoutCells` covers the per-shape cells in the **Shape Layout** section &mdash; placement, routing, and line-jump behavior used by Visio's auto-layout engine. It is one of the three `Shape*Cells` records; the others are [Shape XForm cells](shape-xform-cells.md) and [Shape format cells](shape-format-cells.md).

These cells are mostly relevant when you're letting Visio re-arrange shapes for you (e.g. via [`Format-VisioPage -LayoutStyle`](https://saveenr.gitbook.io/visiopowershell/cmdlets/pages/format-visiopage) on the PowerShell side, or by passing a `LayoutStyle` to the page-level layout commands on the .NET side). For purely manual placement they can usually be left at their defaults.

## The `ShapeLayoutCells` record

### Placement and routing

| Field                 | ShapeSheet cell        | Purpose                                                                                                               |
| --------------------- | ---------------------- | --------------------------------------------------------------------------------------------------------------------- |
| `ShapePlaceStyle`     | `ShapePlaceStyle`      | Placement style for the auto-layout engine (e.g. center / north / south / radial). `0` = `PlaceCenter`.                |
| `ShapePlaceFlip`      | `ShapePlaceFlip`       | Whether the layout engine may flip this shape when placing it.                                                        |
| `ShapeRouteStyle`     | `ShapeRouteStyle`      | Routing style for connectors that touch this shape.                                                                    |
| `ShapeFixedCode`      | `ShapeFixedCode`       | Bitmask: which dimensions are pinned during layout (a "fixed" shape isn't moved).                                     |
| `ShapePermeableX`     | `ShapePermeableX`      | If `1`, connectors may route through the shape horizontally.                                                           |
| `ShapePermeableY`     | `ShapePermeableY`      | If `1`, connectors may route through the shape vertically.                                                             |
| `ShapePermeablePlace` | `ShapePermeablePlace`  | If `1`, the layout engine may place other shapes on top of this one.                                                  |
| `ShapePlowCode`       | `ShapePlowCode`        | Whether this shape "plows" others out of its way during layout.                                                       |
| `ShapeSplit`          | `ShapeSplit`           | Whether this shape is a split-target (a connector dropped on it splits into two).                                     |
| `ShapeSplittable`     | `ShapeSplittable`      | Whether this shape may be split when a connector is dropped on it.                                                    |
| `ShapeDisplayLevel`   | `DisplayLevel`         | Z-order layer used by layout. *(Visio 2010+.)*                                                                        |
| `Relationships`       | `Relationships`        | Relationship hint for the layout engine. *(Visio 2010+.)*                                                             |

### Connector and line-jump behavior

| Field                 | ShapeSheet cell        | Purpose                                                                                                               |
| --------------------- | ---------------------- | --------------------------------------------------------------------------------------------------------------------- |
| `ConnectorFixedCode`  | `ConFixedCode`         | Bitmask: which routing decisions are pinned for this connector.                                                       |
| `LineRouteExt`        | `LineRouteExt`         | Routing-extent hint (none / straight / curved).                                                                       |
| `LineJumpCode`        | `LineJumpCode`         | When this line should jump over other lines (`0` = page default).                                                     |
| `LineJumpStyle`       | `LineJumpStyle`        | Visual style for jumps drawn on this line (arc / gap / square / ...).                                                  |
| `LineJumpDirX`        | `LineJumpDirX`         | Horizontal jump direction.                                                                                            |
| `LineJumpDirY`        | `LineJumpDirY`         | Vertical jump direction.                                                                                              |

All fields are `Core.CellValue`-typed and accept implicit conversions from strings, ints, doubles, and bools.

## Set layout / routing behavior

Construct the record, set the fields you want, hand it to a `SrcWriter`, commit. Cells you don't set are left untouched.

```csharp
var layout = new VisioAutomation.Shapes.ShapeLayoutCells();
layout.ShapePlaceStyle = 0;          // PlaceCenter
layout.ShapePermeableX = 1;
layout.ShapePermeableY = 1;

var writer = new VisioAutomation.ShapeSheet.Writers.SrcWriter();
writer.SetValues(layout);
writer.Commit(shape, VisioAutomation.Core.CellValueType.Formula);
```

## Read existing values

`GetCells` returns a single `ShapeLayoutCells` instance for one shape, or a `CellRecords<ShapeLayoutCells>` for many shapes on a page in one batched call.

```csharp
var layout = VisioAutomation.Shapes.ShapeLayoutCells.GetCells(
    shape,
    VisioAutomation.Core.CellValueType.Formula);

System.Console.WriteLine($"PlaceStyle = {layout.ShapePlaceStyle.Value}");
```

Pass `CellValueType.Result` to read evaluated values instead of formulas.

## See also

* [Shape cells overview](shape-cells.md) &mdash; the three `Shape*Cells` records at a glance.
* [Shape XForm cells](shape-xform-cells.md), [Shape format cells](shape-format-cells.md) &mdash; the other two records; combine multiple records in one writer.
* [Modify the ShapeSheet](shapesheet/modify-the-shapesheet.md) &mdash; the underlying `SrcWriter` / `SidSrcWriter` API.
* [Page cells](page-cells.md) &mdash; the page-level `PageLayoutCells` record holds the matching layout-engine settings for the whole page.
