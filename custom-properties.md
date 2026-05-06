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
cp.EncodeValues();

VisioAutomation.Shapes.CustomPropertyHelper.Set(s1, "Greeting", cp);
```

The `cp.EncodeValues()` call is important: see [String values are formulas, not literals](#string-values-are-formulas-not-literals) below.

`CustomPropertyCells` exposes the full set of property fields as `Core.CellValue` properties:

* `Value`: the property's value
* `Prompt`: tooltip / hint shown to the user
* `Label`: display label
* `Format`: formatting expression
* `Type`: data type (numeric, string, date, …)
* `Calendar`, `Invisible`, `LangID`, `SortKey`, `Ask`: additional control fields

All are `Core.CellValue`-typed and accept implicit conversions from strings, ints, doubles, and bools.

### String values are formulas, not literals

Each `Core.CellValue` field on `CustomPropertyCells` is a Visio *formula*, not a literal value. Setting `cp.Value = "Hello World"` stores the formula `Hello World` (no quotes), which Visio evaluates as an expression: it tries to parse `Hello` as a name reference, fails, and silently substitutes a default (typically `0`). The property is created on the shape, but the value reads back wrong.

To store a string literal, the formula has to include the quotes. Two ways to do that:

```csharp
// Option A: call EncodeValues() before passing to Set.
// Quotes Value, Label, Format, and Prompt as needed.
var cp = new VisioAutomation.Shapes.CustomPropertyCells();
cp.Value = "Hello World";
cp.EncodeValues();
VisioAutomation.Shapes.CustomPropertyHelper.Set(s1, "Greeting", cp);

// Option B: pre-quote the formula yourself.
var cp = new VisioAutomation.Shapes.CustomPropertyCells();
cp.Value = "\"Hello World\"";
VisioAutomation.Shapes.CustomPropertyHelper.Set(s1, "Greeting", cp);
```

Numeric, boolean, and date values are not strings and don't need quoting; pass them as ordinary literals (`cp.Value = 42`, `cp.Value = true`, etc.). The dedicated typed constructors (`new CustomPropertyCells(42)`, `new CustomPropertyCells(true)`, `new CustomPropertyCells(DateTime.Now)`) handle the formatting for you.

The PowerShell `Set-VisioCustomProperty` cmdlet calls `EncodeValues()` internally, so its callers don't have to think about this. The .NET-level API leaves it to the caller; [issue #144](https://github.com/saveenr/VisioAutomation/issues/144) tracks options for making this more ergonomic.

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
