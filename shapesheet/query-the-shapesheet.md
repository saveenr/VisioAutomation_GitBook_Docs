# Query the ShapeSheet

## Query the ShapeSheet

A ShapeSheet Query is a high-performance way of retrieving data from shapes.

You can retrieve:

* Formulas
* Results
* Formulas and Results simultaneously

You can query

* A single shape
* Multiple shapes on a page

There are two kinds of queries:

* A CellQuery - used for retrieving specifically identified cells
* A SectionQuery - an optimized way of retrieving all the cells across multiple rows

## Building a query

Queries are simple objects. Create one, identify which cells you want by adding a column for each cell. And then ask the query to get formulas, results, or both. The query will return a Table object with one or more rows.

## Examples

Getting a formula from a single cell from a shape

```
var query = new VA.ShapeSheet.Query.CellQuery();
var col_fg = query.AddColumn(src_fg);
var formulas = query.GetFormulas(shape);
```

formulas will be Table object - with one column and with one row

To get the retrieved value

```
string f_fg = formulas[0,0]
string f_fg = formulas[0,col_fg]
```

Getting a formula from multiple cells from a shape

```
var query = new VA.ShapeSheet.Query.CellQuery();
var col_fg = query.AddColumn(src_fg);
var col_bg = query.AddColumn(src_bg);
var col_filpat = query.AddColumn(src_filpat);
var formulas = query.GetFormulas(shape);
```

You can retrieve the values like this

```
string f_fg = formulas[0,col_fg]
string f_bg = formulas[0,col_bg]
string f_filpat = formulas[0,col_filpat]
```

## Return results from a single shape

Now let's learn how to return results.

The first thing you should realize is that you can specify what datatype to return the results in. Your choices are double, bool, intm string. To specify the type you will use a generic parameter on the GetResults() method.

```
double [] results_double = query.GetResults<double>(shape);
int [] results_int = query.GetResults<int>(shape);
int [] results_bool = query.GetResults<bool>(shape);
string [] results_string = query.GetResults<string>(shape);
```

## A Note about grouping

You'll notice that tables that come back from these queries always have groups. Their usage will not be clear until later, but the intention is that the groups identify which rows in the table belong to which shape.

When perfoming a CellQuery on a single shape, there is only ever one group that identifies exactly one row.

More on this later

Querying a Section on a single Shape

Imagine you want to query all the cells in a specific section. For example maybe you want to find all the character formatting cells. A section query makes it easy to query cell. The because all the rows will be fetched we only need to know the section index and the cell indices.

```
var query = new VA.ShapeSheet.Query.SectionQuery((short)IVisio.VisSectionIndices.visSectionProp);
var query.AddColumn(VA.ShapeSheet.SRCConstants.Char_Color.Cell);
var formulas = query.GetFormulas(shape1);
```

The returned table contains more than one row. But it contains exactly one group - each group is mapped to a specific shape - that contains all the rows.

NOTE: A group could contain zero rows because some sections may not exist on the shape or that section may contain no rows

Retieving cells from multiple shapes You know how to get shapesheet data from a single shape. What about multiple shapes? The process is very similar: construct the same query but instead of asking the query to run against a specific shape, you identify the page that contains all the shapes and provide the shapeids for those shapes.

```
var query = new VA.ShapeSheet.Query.CellQuery();
var col_fg = query.AddColumn(src_fg);
var col_bg = query.AddColumn(src_bg);
var shapeids = new int { shape1.ID, shape2.ID }
var formulas = query.GetFormulas(page, shapeids);
```

Now the formulas table that is returned will have exactly two rows and twoc olumns. It will also have two groups. Each group identifies one row.

What if we wanted the all character colors for each shape?

```
var query = new VA.ShapeSheet.Query.SectionQuery((short)IVisio.VisSectionIndices.visSectionProp);
var query.AddColumn(VA.ShapeSheet.SRCConstants.Char_Color.Cell);
var shapeids = new[] { s1.ID, s2.ID, s3.ID, s4.ID };
var formulas = query.GetFormulas(page1, shapeids);
```

The formulas table will have multiple rows, but will have exactly two groups. Each group will contain 1 or more rows.
