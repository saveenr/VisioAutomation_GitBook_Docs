# Data table model

`VisioAutomation.Models.Data.DataTableModel` draws a `System.Data.DataTable` as a grid of rectangles, one per cell, with the cell value as the shape text. Use it to get tabular data onto a Visio page quickly: a lookup table beside a diagram, a legend, or a quick look at a query result.

It is deliberately minimal. There is no header row and no per-cell formatting. If you need those, build a [`GridLayout`](layouts-grid.md) directly.

## Hello-world

```csharp
using System.Data;
using VAMODELS = VisioAutomation.Models;

var dt = new DataTable();
dt.Columns.Add("Name");
dt.Columns.Add("Role");
dt.Rows.Add("Alice", "Owner");
dt.Rows.Add("Bob", "Reviewer");

var model = new VAMODELS.Data.DataTableModel();
model.DataTable = dt;
model.CellWidth = 1.5;
model.CellHeight = 0.5;
model.CellSpacing = 0.1;

client.Model.DrawDataTableModel(VisioScripting.TargetPage.Auto, model);
```

This puts four rectangles on the active page, each 1.5 inches wide and 0.5 inches high: `Alice` and `Owner` on the first row, `Bob` and `Reviewer` on the second.

## The model

`DataTableModel` has four properties:

| Property | Type | What it does |
| --- | --- | --- |
| `DataTable` | `System.Data.DataTable` | The data to draw. Must have at least one row. |
| `CellSpacing` | `double` | Gap in inches between cells, applied both horizontally and vertically. |
| `CellWidth` | `double` | Width in inches of every column. Default `1.0`. Must be greater than zero. |
| `CellHeight` | `double` | Height in inches of every row. Default `1.0`. Must be greater than zero. |

The model gives every column the same width and every row the same height. For different sizes per column or row, call `DrawDataTable` with lists (see [Two ways to draw](#two-ways-to-draw)).

`CellWidth` and `CellHeight` take effect in current source, an unreleased change after NuGet 3.1.0 ([#206](https://github.com/saveenr/VisioAutomation/issues/206)). In 3.1.0 and earlier they have no effect: every cell is drawn 1 x 1 inch and only `CellSpacing` is honored.

## Two ways to draw

`client.Model` offers two entry points:

* **`DrawDataTableModel(TargetPage, DataTableModel)`** reads the model and draws it on the target page. It returns nothing.
* **`DrawDataTable(TargetPage, DataTable, IList<double> widths, IList<double> heights, Size cellspacing)`** takes the pieces directly and returns the `List<IVisio.Shape>` it drew, which is useful when you want to format the shapes afterwards.

```csharp
var widths = new[] { 2.0, 1.5 };      // inches, one per column
var heights = new[] { 0.5, 0.25 };    // inches, one per row
var spacing = new VisioAutomation.Core.Size(0.1, 0.1);

var shapes = client.Model.DrawDataTable(VisioScripting.TargetPage.Auto, dt, widths, heights, spacing);

foreach (var shape in shapes)
{
    shape.CellsU["FillForegnd"].FormulaU = "RGB(230,230,230)";
}
```

A column or row beyond the end of its list keeps the 1 inch default, extra entries are ignored, and a zero or negative size throws `ArgumentOutOfRangeException`.

`DrawDataTableModel` draws on the `TargetPage` you pass in current source, an unreleased change after NuGet 3.1.0 ([#207](https://github.com/saveenr/VisioAutomation/issues/207)). In 3.1.0 and earlier it resolved that argument and then always drew on the active page, so pass `TargetPage.Auto` and make the page you want active first.

## What gets drawn

* **One rectangle per cell**, using the `Rectangle` master from `basic_u.vss`, so a table of `rows x columns` produces that many shapes.
* **Text** is `value.ToString()`. A null or `DBNull` value becomes empty text.
* **Layout** starts at the top left of the page and runs row by row, top to bottom.
* **No header row.** Column names are not drawn. To show headers, add them as the first data row.
* **The page** is made a foreground page and resized to fit its contents afterwards. The whole draw is one undo step.

Calling `DrawDataTable` with a null `DataTable`, `widths` or `heights` throws `ArgumentNullException`, and a table with no rows throws `ArgumentOutOfRangeException`.

## Cell sizes

Every cell is 1 x 1 inch unless you set a size. `DataTableModel` applies `CellWidth` and `CellHeight` to every column and row; `DrawDataTable` applies the entries of its `widths` and `heights` lists to individual columns and rows. For per-cell text formatting or other control, use `GridLayout` directly; see [Grid layout model](layouts-grid.md).

In NuGet 3.1.0 and earlier, sizes are not applied: every cell is 1 x 1 inch, the `widths` and `heights` arguments to `DrawDataTable` must be non-null but their values are not read, and `CellWidth` and `CellHeight` have no effect. To size columns and rows in those versions, use `GridLayout`, whose `Columns[i].Width` and `Rows[i].Height` do take effect.

## From PowerShell

`Out-VisioApplication` accepts a `DataTableModel` from the pipeline and draws it on the active page:

```powershell
Import-Module Visio
New-VisioDocument | Out-Null

$dt = New-Object System.Data.DataTable
[void]$dt.Columns.Add('Name')
[void]$dt.Columns.Add('Role')
[void]$dt.Rows.Add('Alice', 'Owner')
[void]$dt.Rows.Add('Bob', 'Reviewer')

$model = New-Object VisioAutomation.Models.Data.DataTableModel
$model.DataTable = $dt
$model.CellWidth = 1.5
$model.CellHeight = 0.5
$model.CellSpacing = 0.1

$model | Out-VisioApplication
```

See the [Out-VisioApplication cmdlet page](https://saveenr.gitbook.io/visiopowershell/cmdlets/visioapplication/out-visioapplication) for the other model types it accepts.

## See also

* [client.Model](../visio-scripting/model.md) (the `DrawDataTable` and `DrawDataTableModel` methods)
* [Grid layout model](layouts-grid.md) (the `GridLayout` that draws the table, with per-row and per-column sizing)
* [XML model](xml-model.md) (the other `Models.Data` type)
