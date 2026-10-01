# Data table model

`VisioAutomation.Models.Data.DataTableModel` draws a `System.Data.DataTable` as a grid of rectangles, one per cell, with the cell value as the shape text. Use it to get tabular data onto a Visio page quickly: a lookup table beside a diagram, a legend, or a quick look at a query result.

It is deliberately minimal. There is no header row, no per-column formatting and no per-column sizing. If you need those, build a [`GridLayout`](layouts.md) directly.

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
model.CellSpacing = 0.1;

client.Model.DrawDataTableModel(VisioScripting.TargetPage.Auto, model);
```

This puts four rectangles on the active page: `Alice` and `Owner` on the first row, `Bob` and `Reviewer` on the second.

## The model

`DataTableModel` has four properties:

| Property | Type | What it does |
| --- | --- | --- |
| `DataTable` | `System.Data.DataTable` | The data to draw. Must have at least one row. |
| `CellSpacing` | `double` | Gap in inches between cells, applied both horizontally and vertically. |
| `CellWidth` | `double` | Not used by the renderer; see [Cell sizes](#cell-sizes). |
| `CellHeight` | `double` | Not used by the renderer; see [Cell sizes](#cell-sizes). |

## Two ways to draw

`client.Model` offers two entry points:

* **`DrawDataTableModel(TargetPage, DataTableModel)`** reads the model and draws it. It returns nothing.
* **`DrawDataTable(TargetPage, DataTable, IList<double> widths, IList<double> heights, Size cellspacing)`** takes the pieces directly and returns the `List<IVisio.Shape>` it drew, which is useful when you want to format the shapes afterwards.

```csharp
var widths = new[] { 1.0, 1.0 };      // required, but not used for sizing
var heights = new[] { 1.0, 1.0 };     // required, but not used for sizing
var spacing = new VisioAutomation.Core.Size(0.1, 0.1);

var shapes = client.Model.DrawDataTable(VisioScripting.TargetPage.Auto, dt, widths, heights, spacing);

foreach (var shape in shapes)
{
    shape.CellsU["FillForegnd"].FormulaU = "RGB(230,230,230)";
}
```

`DrawDataTableModel` always draws on the active page. It resolves its `TargetPage` argument but then calls `DrawDataTable` with `TargetPage.Auto`, so pass `TargetPage.Auto` and make the page you want active first.

## What gets drawn

* **One rectangle per cell**, using the `Rectangle` master from `basic_u.vss`, so a table of `rows x columns` produces that many shapes.
* **Text** is `value.ToString()`. A null or `DBNull` value becomes empty text.
* **Layout** starts at the top left of the page and runs row by row, top to bottom.
* **No header row.** Column names are not drawn. To show headers, add them as the first data row.
* **The page** is made a foreground page and resized to fit its contents afterwards. The whole draw is one undo step.

Calling `DrawDataTable` with a null `DataTable`, `widths` or `heights` throws `ArgumentNullException`, and a table with no rows throws `ArgumentOutOfRangeException`.

## Cell sizes

Every cell is drawn at 1 x 1 inch. The `widths` and `heights` arguments to `DrawDataTable` must be non-null, but the renderer does not read their values, and the `CellWidth` and `CellHeight` properties on `DataTableModel` have no effect for the same reason. Only the spacing is honored. To size columns and rows individually, use `GridLayout`, whose `Columns[i].Width` and `Rows[i].Height` do take effect; see [Layouts](layouts.md#grid-layout).

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
$model.CellSpacing = 0.1

$model | Out-VisioApplication
```

See the [Out-VisioApplication cmdlet page](https://saveenr.gitbook.io/visiopowershell/cmdlets/visioapplication/out-visioapplication) for the other model types it accepts.

## See also

* [client.Model](../visio-scripting/model.md) (the `DrawDataTable` and `DrawDataTableModel` methods)
* [Layouts](layouts.md) (the `GridLayout` that draws the table, with per-row and per-column sizing)
* [XML model](xml-model.md) (the other `Models.Data` type)
