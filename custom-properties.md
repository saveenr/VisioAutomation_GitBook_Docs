# Custom properties

Custom properties (also called "Shape Data" in modern Visio UI) are user-defined named fields on a shape, stored in the `Prop.` section of its ShapeSheet. VisioAutomation exposes them via `VisioAutomation.Shapes.CustomPropertyHelper`.

## Add a single custom property

`Set` takes a `CustomPropertyCells` object. The recommended way to populate it is via the typed setters, which take care of Visio's formula encoding for you:

```csharp
var page = doc.Pages.Add();
var s1 = page.DrawRectangle(0, 0, 1, 1);

var cp = new VisioAutomation.Shapes.CustomPropertyCells();
cp.SetString("Hello World");
cp.Label = "\"Greeting\"";

VisioAutomation.Shapes.CustomPropertyHelper.Set(s1, "Greeting", cp);
```

Each typed setter sets the `Formula` field to a correctly-encoded Visio expression and the `Type` field to match. See [Setting values](#setting-values) below for the full list.

`CustomPropertyCells` exposes the full set of property fields:

| Field | Purpose |
| --- | --- |
| `Formula` | The property's value, expressed as a Visio formula. (Renamed from `Value` in 2026-05; the old name is preserved as an `[Obsolete]` alias.) |
| `Prompt` | Tooltip / hint shown to the user. |
| `Label` | Display label. |
| `Format` | Formatting expression. |
| `Type` | Data type code (0=string, 2=number, 3=boolean, 5=date, …). |
| `Calendar`, `Invisible`, `LangID`, `SortKey`, `Ask` | Additional control fields. |

`Formula`, `Label`, `Format`, and `Prompt` are all Visio formulas. Use a typed setter for `Formula`; for `Label` / `Format` / `Prompt`, either pre-quote the literal (`cp.Label = "\"Greeting\""`) or call `cp.EncodeValues()` before `Set` to encode all four fields together. The remaining fields are numeric codes and accept ordinary integer assignment.

## Setting values

Each setter writes a correctly-encoded formula and sets `Type` to match:

| Setter | What it does | Resulting `Type` |
| --- | --- | --- |
| `cp.SetString("hello")` | Encodes the string as a Visio string literal (`"hello"`). | `0` (String) |
| `cp.SetNumber(42)` / `cp.SetNumber(3.14)` | Writes a numeric formula. | `2` (Number) |
| `cp.SetBool(true)` | Writes `TRUE` or `FALSE`. | `3` (Boolean) |
| `cp.SetDate(DateTime.Now)` | Wraps in `DATETIME("…")`. | `5` (Date) |
| `cp.SetFormula("=…")` | Raw escape hatch. Writes the formula verbatim, leaves `Type` untouched. Use when you have a hand-crafted Visio formula. |  |

Direct assignment to `cp.Formula` works too, but you take responsibility for valid Visio formula syntax. A bare identifier (`cp.Formula = "testVal"`) is parsed by Visio as a name reference, fails to resolve, and raises a friendly `ArgumentException` with a diagnostic from `Set`. Calling `EncodeValues()` before `Set` is the older alternative; the typed setters are the recommended path.

## What Visio actually stores

The `Type` cell is metadata, not enforcement. Visio doesn't validate that the formula matches the declared type, so a few combinations are surprising. Things to know:

* **Numeric strings sneak through Type=String.** `cp.Formula = "42"` with `Type=String` succeeds and renders as `42.0000`, not the string `"42"`. Use `SetString("42")` to force the string interpretation.
* **Type=Boolean accepts `"1"` / `"0"`.** Visio stores them as numeric formulas, and the rendered Result is `1.0000` / `0.0000`, not `TRUE` / `FALSE`. Use `SetBool(true)` / `SetBool(false)` to get the expected boolean rendering.
* **Type=Date accepts arbitrary quoted strings.** `cp.Formula = "\"2017-03-31\""` with `Type=Date` stores the literal string, not a parsed date. Use `SetDate(...)` to get a real `DATETIME` formula.
* **Empty / `null` / whitespace defaults to `0`.** All four cases (`""`, `null`, `" "`, missing-write) leave the cell at Visio's default value, which renders as `0.0000`. If you observe a property that reads back as `0` and you didn't set `0`, this is the path to check.

The full per-Type characterization (driving the test suite) lives in [`docs/internal/custom-property-encoding.md`](https://github.com/saveenr/VisioAutomation/blob/master/docs/internal/custom-property-encoding.md) on the source repo.

## Read all custom properties from a shape

`GetDictionary` returns the properties keyed by name. The `CellValueType` parameter chooses between formula text and evaluated result:

```csharp
var props = VisioAutomation.Shapes.CustomPropertyHelper.GetDictionary(
                s1,
                VisioAutomation.Core.CellValueType.Formula);

foreach (var kv in props)
{
    System.Console.WriteLine("{0} = {1}", kv.Key, kv.Value.Formula.Value);
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
