# Page print cells

`VisioAutomation.Pages.PagePrintCells` covers the cells in a page's **Print Properties** section &mdash; margins, paper kind, orientation, scale, the centering / fit-to-N-pages flags, and the printer source. It is one of the four PageSheet records; the others are [Page format cells](format.md), [Page layout cells](layout.md), and [Page ruler and grid cells](ruler-grid.md).

## The `PagePrintCells` record

### Margins

| Field          | ShapeSheet cell      | Purpose                |
| -------------- | -------------------- | ---------------------- |
| `LeftMargin`   | `PrintLeftMargin`    | Left margin width.     |
| `RightMargin`  | `PrintRightMargin`   | Right margin width.    |
| `TopMargin`    | `PrintTopMargin`     | Top margin height.     |
| `BottomMargin` | `PrintBottomMargin`  | Bottom margin height.  |

### Paper and orientation

| Field          | ShapeSheet cell        | Purpose                                                                            |
| -------------- | ---------------------- | ---------------------------------------------------------------------------------- |
| `PaperKind`    | `PaperKind`            | Paper-size code (Letter / Legal / A4 / ...).                                        |
| `PaperSource`  | `PaperSource`          | Printer paper-source / tray code.                                                  |
| `Orientation`  | `PrintPageOrientation` | `1` = portrait, `2` = landscape.                                                    |

### Scale and tiling

| Field      | ShapeSheet cell  | Purpose                                                                          |
| ---------- | ---------------- | -------------------------------------------------------------------------------- |
| `ScaleX`   | `ScaleX`         | Horizontal print scale.                                                          |
| `ScaleY`   | `ScaleY`         | Vertical print scale.                                                            |
| `PagesX`   | `PagesX`         | When tiling, the number of printer pages used along the X axis.                   |
| `PagesY`   | `PagesY`         | When tiling, the number of printer pages used along the Y axis.                   |
| `OnPage`   | `OnPage`         | If `1`, scale to fit on `PagesX` x `PagesY` pages.                                 |
| `CenterX`  | `CenterX`        | If `1`, center horizontally on the printer page.                                  |
| `CenterY`  | `CenterY`        | If `1`, center vertically on the printer page.                                    |
| `Grid`     | `PrintGrid`      | If `1`, print the grid lines.                                                    |

All fields are `Core.CellValue`-typed and accept implicit conversions from strings, ints, doubles, and bools.

## Configure print margins

Construct the record, set the fields you want, hand it to a `SrcWriter`, commit. Cells you don't set are left untouched.

```csharp
var pp = new VisioAutomation.Pages.PagePrintCells();
pp.LeftMargin   = "0.5 in";
pp.RightMargin  = "0.5 in";
pp.TopMargin    = "0.75 in";
pp.BottomMargin = "0.75 in";

var writer = new VisioAutomation.ShapeSheet.Writers.SrcWriter();
writer.SetValues(pp);
writer.Commit(page.PageSheet, VisioAutomation.Core.CellValueType.Formula);
```

## Read existing values

```csharp
var pp = VisioAutomation.Pages.PagePrintCells.GetCells(
    page.PageSheet,
    VisioAutomation.Core.CellValueType.Result);

System.Console.WriteLine($"LeftMargin = {pp.LeftMargin.Value}");
```

Pass `CellValueType.Formula` to read the underlying formulas instead of evaluated results.

## See also

* [Page cells overview](../page-cells.md) &mdash; the four PageSheet records at a glance.
* [Page format cells](format.md), [Page layout cells](layout.md), [Page ruler and grid cells](ruler-grid.md) &mdash; the other PageSheet records.
* [Modify the ShapeSheet](../shapesheet/modify-the-shapesheet.md) &mdash; the underlying `SrcWriter` API.
