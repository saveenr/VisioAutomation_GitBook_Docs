# Page ruler and grid cells

`VisioAutomation.Pages.PageRulerAndGridCells` covers the ten cells in a page's **Ruler & Grid** section: ruler density and origin per axis, grid density / origin / spacing per axis. It is one of the four PageSheet records; the others are [Page format cells](format.md), [Page layout cells](layout.md), and [Page print cells](print.md).

## The `PageRulerAndGridCells` record

### Grid

| Field          | ShapeSheet cell | Purpose                                                                                  |
| -------------- | --------------- | ---------------------------------------------------------------------------------------- |
| `XGridDensity` | `XGridDensity`  | Horizontal grid density (fixed / fine / normal / coarse).                                 |
| `YGridDensity` | `YGridDensity`  | Vertical grid density.                                                                   |
| `XGridSpacing` | `XGridSpacing`  | Distance between vertical grid lines.                                                    |
| `YGridSpacing` | `YGridSpacing`  | Distance between horizontal grid lines.                                                  |
| `XGridOrigin`  | `XGridOrigin`   | X coordinate of the grid origin.                                                         |
| `YGridOrigin`  | `YGridOrigin`   | Y coordinate of the grid origin.                                                         |

### Ruler

| Field           | ShapeSheet cell  | Purpose                                                                                  |
| --------------- | ---------------- | ---------------------------------------------------------------------------------------- |
| `XRulerDensity` | `XRulerDensity`  | Horizontal ruler subdivision density.                                                    |
| `YRulerDensity` | `YRulerDensity`  | Vertical ruler subdivision density.                                                      |
| `XRulerOrigin`  | `XRulerOrigin`   | X coordinate of the ruler origin (offsets the "0" mark on the horizontal ruler).         |
| `YRulerOrigin`  | `YRulerOrigin`   | Y coordinate of the ruler origin.                                                        |

All fields are `Core.CellValue`-typed and accept implicit conversions from strings, ints, doubles, and bools.

## Configure ruler and grid

Construct the record, set the fields you want, hand it to a `SrcWriter`, commit. Cells you don't set are left untouched.

```csharp
var rg = new VisioAutomation.Pages.PageRulerAndGridCells();
rg.XGridSpacing = "0.25 in";
rg.YGridSpacing = "0.25 in";

var writer = new VisioAutomation.ShapeSheet.Writers.SrcWriter();
writer.SetValues(rg);
writer.Commit(page.PageSheet, VisioAutomation.Core.CellValueType.Formula);
```

## Read existing values

```csharp
var rg = VisioAutomation.Pages.PageRulerAndGridCells.GetCells(
    page.PageSheet,
    VisioAutomation.Core.CellValueType.Result);

System.Console.WriteLine($"XGridSpacing = {rg.XGridSpacing.Value}");
```

Pass `CellValueType.Formula` to read the underlying formulas instead of evaluated results.

## See also

* [Page cells overview](../page-cells.md): the four PageSheet records at a glance.
* [Page format cells](format.md), [Page layout cells](layout.md), [Page print cells](print.md): the other PageSheet records.
* [Modify the ShapeSheet](../shapesheet/modify-the-shapesheet.md): the underlying `SrcWriter` API.
