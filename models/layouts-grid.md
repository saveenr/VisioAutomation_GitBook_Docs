# Grid layout model

Use when the data is a uniform rectangular grid of identical shapes. The layout takes a column count, a row count, a cell size and a master (an `IVisio.Master` you have already loaded), and drops that master at every grid cell.

```csharp
using GRID = VisioAutomation.Models.Layouts.Grid;
using VA = VisioAutomation;

int cols = 3;
int rows = 6;
var cellsize = new VA.Core.Size(0.5, 0.25);

var grid = new GRID.GridLayout(cols, rows, cellsize, rectMaster);
grid.Origin = new VA.Core.Point(0, 4);
grid.PerformLayout();
grid.Render(visioPage);
```

`PerformLayout()` must be called before `Render`, because it computes each node's rectangle and `Render` reads those rectangles. Each grid cell renders as one shape instance; the result is `cols * rows` shapes on the page. The `Rows` and `Columns` lists let you tweak per-row `Height` and per-column `Width` before layout if uniformity isn't quite enough (both throw if set to zero or less). `CellSpacing` (default 0.5 x 0.25) sets the gaps, `ColumnDirection` (`LeftToRight` by default, or `RightToLeft`) and `RowDirection` (`BottomToTop` by default, or `TopToBottom`) control growth direction, and `GetNode(col, row)` returns a node whose `Text`, `Cells` and `Draw` (default `true`) you can set.

## See also

* [Layout models](layouts.md) (the overview and comparison of all the layouts)
* [DOM](dom.md) (the underlying shape model the layouts emit into)
* [Data table model](data-table.md) (draws a `DataTable` with a grid layout)
