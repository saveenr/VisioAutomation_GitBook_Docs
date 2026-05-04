# Page cells

A Visio **page** has its own ShapeSheet (the *PageSheet*) holding settings that apply to the page as a whole &mdash; size, scale, drawing units, print margins, ruler / grid configuration, and auto-layout parameters. VisioAutomation exposes these as four cell records under `VisioAutomation.Pages` plus a few helper methods on `PageHelper`.

## The four PageSheet records

* **`PageFormatCells`** &mdash; page size, drawing scale, drawing-size type, page-shadow cells.
* **`PageLayoutCells`** &mdash; auto-layout configuration: `AvenueSizeX/Y`, `BlockSizeX/Y`, `LineJump*`, `LineRouteExt`, `Place*`, `RouteStyle`, etc.
* **`PagePrintCells`** &mdash; print setup: margins, paper kind, orientation, scale, pages-X/Y.
* **`PageRulerAndGridCells`** &mdash; ruler density / origin and grid density / origin / spacing along each axis.

All four follow the same cell-record pattern: set fields, write through `SrcWriter`, commit.

## Set page size

```csharp
var fmt = new VisioAutomation.Pages.PageFormatCells();
fmt.Width  = 11;
fmt.Height = 8.5;

var writer = new VisioAutomation.ShapeSheet.Writers.SrcWriter();
writer.SetValues(fmt);
writer.Commit(page.PageSheet, VisioAutomation.Core.CellValueType.Formula);
```

`PageHelper.SetSize(page, size)` is a one-liner shortcut for the same operation:

```csharp
VisioAutomation.Pages.PageHelper.SetSize(page, new VisioAutomation.Core.Size(11, 8.5));
```

`PageHelper.GetSize(page)` reads it back.

## Configure print margins

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

## Configure auto-layout parameters

`PageLayoutCells` controls the spacing and routing behavior used when Visio re-flows shapes (e.g. via the layout engine).

```csharp
var pl = new VisioAutomation.Pages.PageLayoutCells();
pl.AvenueSizeX = "0.5 in";
pl.AvenueSizeY = "0.25 in";
pl.EnableGrid  = 0;

var writer = new VisioAutomation.ShapeSheet.Writers.SrcWriter();
writer.SetValues(pl);
writer.Commit(page.PageSheet, VisioAutomation.Core.CellValueType.Formula);
```

## Configure ruler and grid

```csharp
var rg = new VisioAutomation.Pages.PageRulerAndGridCells();
rg.XGridSpacing = "0.25 in";
rg.YGridSpacing = "0.25 in";

var writer = new VisioAutomation.ShapeSheet.Writers.SrcWriter();
writer.SetValues(rg);
writer.Commit(page.PageSheet, VisioAutomation.Core.CellValueType.Formula);
```

## Resize a page to fit its contents

`PageHelper.ResizeToFitContents` runs Visio's native resize-to-fit and then optionally adds padding around the result.

```csharp
var padding = new VisioAutomation.Core.Size(0.5, 0.5);
VisioAutomation.Pages.PageHelper.ResizeToFitContents(page, padding);
```

When the padding is non-zero the helper resizes the page to (resize-to-fit + 2 &times; padding) on each axis and re-centers the contents.

## Duplicate a page

`PageHelper.Duplicate(src, dest)` copies all PageSheet cells and all shapes from `src_page` to `dest_page`. The source page must be the active page; the destination must be a different page.

```csharp
var src  = doc.Pages[1];
var dest = doc.Pages.Add();
VisioAutomation.Pages.PageHelper.Duplicate(src, dest);
```

## See also

* [Modify the ShapeSheet](shapesheet/modify-the-shapesheet.md) &mdash; the underlying `SrcWriter` / `SidSrcWriter` API. PageSheets are written using the same writers as Shape ShapeSheets &mdash; just commit against `page.PageSheet` instead of a shape.
* [Stencils and masters](stencils-and-masters.md) &mdash; loading masters into a page.
