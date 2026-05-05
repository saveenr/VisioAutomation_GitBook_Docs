# Page layout cells

`VisioAutomation.Pages.PageLayoutCells` covers the page-level cells that drive Visio's auto-layout engine: spacing between shapes ("avenues"), block sizes, line routing, line-jump style, placement direction, and the page-level resize / split / avoid-page-breaks flags. It is one of the four PageSheet records; the others are [Page format cells](format.md), [Page print cells](print.md), and [Page ruler and grid cells](ruler-grid.md).

These cells are mostly relevant when you're letting Visio re-arrange shapes for you (e.g. via [`Format-VisioPage -LayoutStyle`](https://saveenr.gitbook.io/visiopowershell/cmdlets/pages/format-visiopage) on the PowerShell side, or by passing a `LayoutStyle` to the page-level layout commands on the .NET side). The matching per-shape cells live in [`ShapeLayoutCells`](../shape-layout-cells.md).

## The `PageLayoutCells` record

### Spacing

| Field           | ShapeSheet cell  | Purpose                                                                               |
| --------------- | ---------------- | ------------------------------------------------------------------------------------- |
| `AvenueSizeX`   | `AvenueSizeX`    | Horizontal spacing between placed shapes.                                             |
| `AvenueSizeY`   | `AvenueSizeY`    | Vertical spacing between placed shapes.                                               |
| `BlockSizeX`    | `BlockSizeX`     | Horizontal block size used by the placement algorithm.                                |
| `BlockSizeY`    | `BlockSizeY`     | Vertical block size used by the placement algorithm.                                  |
| `LineToLineX`   | `LineToLineX`    | Minimum horizontal gap between parallel connectors.                                   |
| `LineToLineY`   | `LineToLineY`    | Minimum vertical gap between parallel connectors.                                     |
| `LineToNodeX`   | `LineToNodeX`    | Minimum horizontal gap between a connector and a placed shape.                        |
| `LineToNodeY`   | `LineToNodeY`    | Minimum vertical gap between a connector and a placed shape.                          |

### Placement and routing

| Field             | ShapeSheet cell      | Purpose                                                                              |
| ----------------- | -------------------- | ------------------------------------------------------------------------------------ |
| `PlaceStyle`      | `PlaceStyle`         | Placement style (top-to-bottom / left-to-right / radial / etc.).                     |
| `PlaceDepth`      | `PlaceDepth`         | Placement direction (top down vs. bottom up).                                        |
| `PlaceFlip`       | `PlaceFlip`          | Whether shapes can be flipped during placement.                                       |
| `PlowCode`        | `PlowCode`           | How the layout engine "plows" overlapping shapes out of the way.                     |
| `RouteStyle`      | `RouteStyle`         | Connector-routing style (right-angle, straight, curved, ...).                         |
| `LineRouteExt`    | `LineRouteExt`       | Routing-extent hint (none / straight / curved).                                      |
| `LineAdjustFrom`  | `LineAdjustFrom`     | Which connectors get adjusted to avoid overlaps (no lines / connectors only / all).   |
| `LineAdjustTo`    | `LineAdjustTo`       | What connectors are routed *to* (no shapes / shapes only / shapes and lines).         |
| `CtrlAsInput`     | `CtrlAsInput`        | If `1`, control-point handles are treated as routing input.                          |
| `DynamicsOff`     | `DynamicsOff`        | If `1`, layout dynamics (interactive re-routing) are disabled.                       |
| `EnableGrid`      | `EnableGrid`         | If `1`, layout snaps results to the grid.                                            |

### Line-jump behavior

| Field            | ShapeSheet cell  | Purpose                                                  |
| ---------------- | ---------------- | -------------------------------------------------------- |
| `LineJumpCode`   | `LineJumpCode`   | When connectors should jump (`0` = none, `1` = horizontal, `2` = vertical, `3` = both). |
| `LineJumpStyle`  | `LineJumpStyle`  | Visual style for jumps (arc / gap / square / ...).        |
| `LineJumpFactorX`| `LineJumpFactorX`| Horizontal jump-size multiplier.                         |
| `LineJumpFactorY`| `LineJumpFactorY`| Vertical jump-size multiplier.                            |
| `LineJumpDirX`   | `LineJumpDirX`   | Horizontal jump direction.                               |
| `LineJumpDirY`   | `LineJumpDirY`   | Vertical jump direction.                                 |

### Page-level placement options

| Field             | ShapeSheet cell    | Purpose                                                     |
| ----------------- | ------------------ | ----------------------------------------------------------- |
| `PageShapeSplit`  | `PageShapeSplit`   | Whether the page lets connectors split shapes when dropped on them. |
| `ResizePage`      | `ResizePage`       | Whether the page auto-resizes to fit its contents.          |
| `AvoidPageBreaks` | `AvoidPageBreaks`  | If `1`, layout avoids placing shapes across page breaks. *(Visio 2010+.)* |

All fields are `Core.CellValue`-typed and accept implicit conversions from strings, ints, doubles, and bools.

## Configure auto-layout parameters

Construct the record, set the fields you want, hand it to a `SrcWriter`, commit. Cells you don't set are left untouched.

```csharp
var pl = new VisioAutomation.Pages.PageLayoutCells();
pl.AvenueSizeX = "0.5 in";
pl.AvenueSizeY = "0.25 in";
pl.EnableGrid  = 0;

var writer = new VisioAutomation.ShapeSheet.Writers.SrcWriter();
writer.SetValues(pl);
writer.Commit(page.PageSheet, VisioAutomation.Core.CellValueType.Formula);
```

## Read existing values

```csharp
var pl = VisioAutomation.Pages.PageLayoutCells.GetCells(
    page.PageSheet,
    VisioAutomation.Core.CellValueType.Formula);

System.Console.WriteLine($"AvenueSizeX = {pl.AvenueSizeX.Value}");
```

Pass `CellValueType.Result` to read evaluated values instead of formulas.

## See also

* [Page cells overview](../page-cells.md): the four PageSheet records at a glance.
* [Shape layout cells](../shape-layout-cells.md): the matching per-shape record.
* [Page format cells](format.md), [Page print cells](print.md), [Page ruler and grid cells](ruler-grid.md): the other PageSheet records.
* [Modify the ShapeSheet](../shapesheet/modify-the-shapesheet.md): the underlying `SrcWriter` API.
