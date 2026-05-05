# User-defined cells

User-defined cells are named cells under the `User.` section of a shape's ShapeSheet, used to hold expressions that other formulas on the same shape can reference. VisioAutomation exposes them via `VisioAutomation.Shapes.UserDefinedCellHelper`.

## Read user-defined cells from a shape

`GetDictionary` returns a dictionary keyed by the cell name. The `CellValueType` parameter chooses between formula text and evaluated result:

```csharp
var udcells = VisioAutomation.Shapes.UserDefinedCellHelper.GetDictionary(
                  shape1,
                  VisioAutomation.Core.CellValueType.Formula);

foreach (var kv in udcells)
{
    System.Console.WriteLine("{0} = {1}", kv.Key, kv.Value.Value);
}
```

To read from multiple shapes on the same page in one batched call, build a `ShapeIDPairs` and use the page-level overload:

```csharp
var pairs = VisioAutomation.Core.ShapeIDPairs.FromShapes(shape1, shape2, shape3);
var per_shape = VisioAutomation.Shapes.UserDefinedCellHelper.GetDictionary(
                    page1,
                    pairs,
                    VisioAutomation.Core.CellValueType.Formula);
// per_shape is a List<UserDefinedCellDictionary>, one entry per shape
```

## Add or update a cell

`Set` takes a `UserDefinedCellCells` object holding the value and prompt:

```csharp
var cells = new VisioAutomation.Shapes.UserDefinedCellCells();
cells.Value = "42";
cells.Prompt = "The answer";

VisioAutomation.Shapes.UserDefinedCellHelper.Set(shape1, "MyCell", cells);
```

Both `Value` and `Prompt` are `Core.CellValue` properties; implicit conversions let you assign strings, ints, doubles, or bools directly without constructing a `CellValue`.

## Count and enumerate cells

```csharp
int count = VisioAutomation.Shapes.UserDefinedCellHelper.GetCount(shape1);

System.Collections.Generic.List<string> names =
    VisioAutomation.Shapes.UserDefinedCellHelper.GetNames(shape1);
```

## Check if a cell exists

```csharp
bool exists = VisioAutomation.Shapes.UserDefinedCellHelper.Contains(shape1, "MyCell");
```

## Delete a cell

```csharp
VisioAutomation.Shapes.UserDefinedCellHelper.Delete(shape1, "MyCell");
```

## Validating cell names

```csharp
bool ok = VisioAutomation.Shapes.UserDefinedCellHelper.IsValidName("MyCell");
// or, throw on invalid:
VisioAutomation.Shapes.UserDefinedCellHelper.CheckValidName("MyCell");
```
