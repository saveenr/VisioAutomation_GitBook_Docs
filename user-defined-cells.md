# User-defined cells

## Get the user-defined cells

```
// For a Single shape
var udcell1s = VA.Shapes.UserDefinedCellHelper.Get(shape1);

// For multiple shapes
var props = VA.Shapes.UserDefinedCellHelper.Get (page1, shapes);
```

## Add a new cell

```
VA.Shapes.UserDefinedCellHelper.Set (s1, prop.Name, prop.Value, prop.Prompt);
```

## Find out how many user defined cells exist

```
int count = VA.Shapes.UserDefinedCellHelper.GetCount(s1);
```

## Check if a cell exists

```
bool exists = VA.Shapes.UserDefinedCellHelper.Contains(s1, "FOO1");
```

## Delete a cell

```
VA.Shapes.UserDefinedCellHelper.Delete(s1, "FOO3");
```
