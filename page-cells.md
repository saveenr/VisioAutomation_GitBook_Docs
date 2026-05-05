# Page cells

A Visio **page** has its own ShapeSheet (the *PageSheet*) holding settings that apply to the page as a whole: size, scale, drawing units, print margins, ruler / grid configuration, and auto-layout parameters. VisioAutomation exposes these as **four cell records** under `VisioAutomation.Pages` plus a few helper methods on `PageHelper`, each documented on its own page.

## The four PageSheet records

| Record                  | Covers                                                                                                | Page                                            |
| ----------------------- | ----------------------------------------------------------------------------------------------------- | ----------------------------------------------- |
| `PageFormatCells`       | Page size, drawing scale, drawing-size type, page-shadow cells.                                       | [Page format cells](pages/format.md)            |
| `PageLayoutCells`       | Auto-layout configuration: `AvenueSizeX/Y`, `BlockSizeX/Y`, `LineJump*`, `Place*`, `RouteStyle`, etc. | [Page layout cells](pages/layout.md)            |
| `PagePrintCells`        | Print setup: margins, paper kind, orientation, scale, pages-X/Y, centering, fit-to-N-pages.          | [Page print cells](pages/print.md)              |
| `PageRulerAndGridCells` | Ruler density / origin and grid density / origin / spacing along each axis.                          | [Page ruler and grid cells](pages/ruler-grid.md) |

All four follow the same cell-record pattern as the [Shape cells](shape-cells.md) records: set the fields you care about, hand the record to a `SrcWriter`, commit. PageSheets are written using the same writers as Shape ShapeSheets; just commit against `page.PageSheet` instead of a shape.

## Page helper methods

A separate static class wraps the most common one-line operations:

* [Page helper](pages/helper.md): `SetSize` / `GetSize`, `ResizeToFitContents` (with optional padding), and `Duplicate` (copy PageSheet cells + shapes between pages).

## Combine multiple records in one writer

A `SrcWriter` accepts as many `SetValues` calls as you want. Mix records in a single transaction so all the changes commit atomically:

```csharp
var writer = new VisioAutomation.ShapeSheet.Writers.SrcWriter();
writer.SetValues(fmt);
writer.SetValues(pp);
writer.SetValues(rg);
writer.Commit(page.PageSheet, VisioAutomation.Core.CellValueType.Formula);
```

## See also

* [Page format cells](pages/format.md), [Page layout cells](pages/layout.md), [Page print cells](pages/print.md), [Page ruler and grid cells](pages/ruler-grid.md): the per-record details.
* [Page helper](pages/helper.md): one-line shortcuts.
* [Shape cells](shape-cells.md): the matching per-shape records.
* [Modify the ShapeSheet](shapesheet/modify-the-shapesheet.md): the underlying `SrcWriter` / `SidSrcWriter` API.
* [Stencils and masters](stencils-and-masters.md): loading masters into a page.
