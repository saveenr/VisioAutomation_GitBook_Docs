# Query the ShapeSheet

A ShapeSheet query is a high-performance way of retrieving cell data from one or more shapes: VisioAutomation batches the work into a single COM call into Visio, much faster than reading cells one at a time through the `Cells[…]` indexer.

You can retrieve:

* **Formulas** (the textual expression in each cell)
* **Results** (the evaluated value, returnable as `double`, `int`, `bool`, or `string`)

Choose between formulas and results per call: `GetFormulas(...)` vs `GetResults<T>(...)`.

You can query:

* A single shape
* Multiple shapes on a page

There are two query types:

* **`CellQuery`**: for retrieving specifically identified cells
* **`SectionQuery`**: an optimized way of retrieving all rows across one section (e.g., every custom property, every connection point)

## Building a query

Create a query, add one column for each cell you want, then call `GetFormulas` or `GetResults<T>`. The return value's exact type depends on which query and which overload. See the table below.

| Method | Return type |
|---|---|
| `CellQuery.GetFormulas(shape)` | `Data.DataRows<string>` |
| `CellQuery.GetResults<T>(shape)` | `Data.DataRows<T>` |
| `CellQuery.GetFormulas(page, shapeids)` | `Data.DataRows<string>` |
| `CellQuery.GetResults<T>(page, shapeids)` | `Data.DataRows<T>` |
| `SectionQuery.GetFormulas(shape)` | `Data.DataRowGroup<string>` |
| `SectionQuery.GetResults<T>(shape)` | `Data.DataRowGroup<T>` |
| `SectionQuery.GetFormulas(page, shapeidpairs)` | `Data.DataRowGroups<string>` |
| `SectionQuery.GetResults<T>(page, shapeidpairs)` | `Data.DataRowGroups<T>` |

`DataRows<T>` is a list of `DataRow<T>`; access a value with `result[rowIndex][columnIndex]`. `DataRowGroup<T>` and `DataRowGroups<T>` add the section/per-shape grouping needed by `SectionQuery` (covered below).

## Examples

Get a formula from a single cell from a shape:

```csharp
var query = new VA.ShapeSheet.Query.CellQuery();
var col_fg = query.Columns.Add(src_fg);
var formulas = query.GetFormulas(shape);     // Data.DataRows<string>
```

Read the value:

```csharp
string f_fg = formulas[0][0];
string f_fg_alt = formulas[0][col_fg.Ordinal];
```

`formulas` has one row (`formulas.Count == 1`); each row has one entry per added column.

Get formulas from multiple cells from a shape:

```csharp
var query = new VA.ShapeSheet.Query.CellQuery();
var col_fg = query.Columns.Add(src_fg);
var col_bg = query.Columns.Add(src_bg);
var col_filpat = query.Columns.Add(src_filpat);
var formulas = query.GetFormulas(shape);

string f_fg = formulas[0][col_fg.Ordinal];
string f_bg = formulas[0][col_bg.Ordinal];
string f_filpat = formulas[0][col_filpat.Ordinal];
```

## Returning results from a single shape

Specify the result data type with the generic parameter on `GetResults<T>`. Choices are `double`, `bool`, `int`, `string`:

```csharp
VA.ShapeSheet.Data.DataRows<double> results_double = query.GetResults<double>(shape);
VA.ShapeSheet.Data.DataRows<int>    results_int    = query.GetResults<int>(shape);
VA.ShapeSheet.Data.DataRows<bool>   results_bool   = query.GetResults<bool>(shape);
VA.ShapeSheet.Data.DataRows<string> results_string = query.GetResults<string>(shape);
```

## Querying a section on a single shape

Imagine you want every row in a particular section, for example, every custom property of a shape. A `SectionQuery` is the optimized path for that. You build it by adding one or more sections (each call returns a `DataColumns` you then add cell columns to); all rows in those sections are returned.

```csharp
var query = new VA.ShapeSheet.Query.SectionQuery();
var sec_cols = query.Add(IVisio.VisSectionIndices.visSectionProp);
sec_cols.Add(VA.Core.SrcConstants.CharColor);
var formulas = query.GetFormulas(shape1);     // Data.DataRowGroup<string>
```

The return type is `DataRowGroup<T>`: a single group (one shape) containing one `DataRows<T>` per section that was added to the query. Each `DataRows<T>` then contains one `DataRow<T>` per row in that section, and a row's cells are indexed by column. So iterating goes section → row → cell:

```csharp
foreach (var section in formulas)
{
    foreach (var row in section)
    {
        System.Console.WriteLine(row[0]);    // first column of this row
    }
}
```

If a queried section doesn't exist on the shape or is empty, that section's `DataRows<T>` will contain zero rows.

## Retrieving cells from multiple shapes

For multi-shape queries, build the same query but pass a page plus the set of shape IDs (`CellQuery`) or shape-ID-pairs (`SectionQuery`).

`CellQuery` accepts an `IList<int>` of shape IDs:

```csharp
var query = new VA.ShapeSheet.Query.CellQuery();
var col_fg = query.Columns.Add(src_fg);
var col_bg = query.Columns.Add(src_bg);
var shapeids = new[] { shape1.ID, shape2.ID };
var formulas = query.GetFormulas(page, shapeids);     // Data.DataRows<string>
```

The returned `DataRows<string>` has one row per shape (in the same order as the input shape IDs). Each row holds the requested columns for that shape.

`SectionQuery` requires `Core.ShapeIDPairs` for the page-level overload:

```csharp
var query = new VA.ShapeSheet.Query.SectionQuery();
var sec_cols = query.Add(IVisio.VisSectionIndices.visSectionProp);
sec_cols.Add(VA.Core.SrcConstants.CharColor);
var pairs = VA.Core.ShapeIDPairs.FromShapes(s1, s2, s3, s4);
var formulas = query.GetFormulas(page1, pairs);       // Data.DataRowGroups<string>
```

`DataRowGroups<T>` is a list of `DataRowGroup<T>`: one group per shape, each group containing the rows from that shape's section. A group can contain zero rows if the section doesn't exist on that shape or has no rows.
