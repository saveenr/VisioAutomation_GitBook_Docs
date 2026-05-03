# User-defined cells

## Get the user-defined cells

```
// For a Single shape
var udcell1s = VA.Shapes.UserDefinedCellsHelper.Get(shape1);

// For multiple shapes
var props = VA.Shapes.UserDefinedCellsHelper.Get (page1, shapes);
```

## Add a new cell

```
VA.Shapes.UserDefinedCellsHelper.Set (s1, prop.Name, prop.Value, prop.Prompt);
```

## Find out how many user defined cells exist

```
int count = VA.Shapes.UserDefinedCellsHelper.GetCount(s1);
```

## Check if a cell exists

```
bool exists = VA.Shapes.UserDefinedCellsHelper.Contains(s1, "FOO1");
```

## Delete a cell

```
VA.Shapes.UserDefinedCellsHelper.Delete(s1, "FOO3");
```
