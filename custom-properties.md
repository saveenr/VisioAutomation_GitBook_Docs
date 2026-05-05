# Custom properties

Custom properties (also called "Shape Data" in modern Visio UI) are user-defined named fields on a shape, stored in the `Prop.` section of its ShapeSheet. VisioAutomation exposes them via `VisioAutomation.Shapes.CustomPropertyHelper`.

## Add a single custom property

`Set` takes a `CustomPropertyCells` object that lets you specify any combination of the property's cells (Value, Prompt, Format, Type, Label, etc.):

```csharp
var page = doc.Pages.Add();
var s1 = page.DrawRectangle(0, 0, 1, 1);

var cp = new VisioAutomation.Shapes.CustomPropertyCells();
cp.Value = "Hello World";
cp.Label = "Greeting";

VisioAutomation.Shapes.CustomPropertyHelper.Set(s1, "Greeting", cp);
```

`CustomPropertyCells` exposes the full set of property fields as `Core.CellValue` properties:

* `Value`: the property's value
* `Prompt`: tooltip / hint shown to the user
* `Label`: display label
* `Format`: formatting expression
* `Type`: data type (numeric, string, date, …)
* `Calendar`, `Invisible`, `LangID`, `SortKey`, `Ask`: additional control fields

All are `Core.CellValue`-typed and accept implicit conversions from strings, ints, doubles, and bools.

## Read all custom properties from a shape

`GetDictionary` returns the properties keyed by name. The `CellValueType` parameter chooses between formula text and evaluated result:

```csharp
var props = VisioAutomation.Shapes.CustomPropertyHelper.GetDictionary(
                s1,
                VisioAutomation.Core.CellValueType.Formula);

foreach (var kv in props)
{
    System.Console.WriteLine("{0} = {1}", kv.Key, kv.Value.Value.Value);
}
```

The returned dictionary's values are `CustomPropertyCells` objects, so you can inspect each property's full set of fields, not just the value.

## Read from multiple shapes

For multiple shapes on the same page, build a `ShapeIDPairs` and use the page-level overload, one batched Visio call per page:

```csharp
var pairs = VisioAutomation.Core.ShapeIDPairs.FromShapes(s1, s2);
var per_shape = VisioAutomation.Shapes.CustomPropertyHelper.GetDictionary(
                    page,
                    pairs,
                    VisioAutomation.Core.CellValueType.Formula);
// per_shape is a List<CustomPropertyDictionary>, one entry per shape
```

## Count and enumerate

```csharp
int count = VisioAutomation.Shapes.CustomPropertyHelper.GetCount(s1);

System.Collections.Generic.List<string> names =
    VisioAutomation.Shapes.CustomPropertyHelper.GetNames(s1);
```

## Check if a property exists

```csharp
bool exists = VisioAutomation.Shapes.CustomPropertyHelper.Contains(s1, "Greeting");
```

## Delete a property

```csharp
VisioAutomation.Shapes.CustomPropertyHelper.Delete(s1, "Greeting");
```

## Validating property names

```csharp
bool ok = VisioAutomation.Shapes.CustomPropertyHelper.IsValidName("MyProp");
```
