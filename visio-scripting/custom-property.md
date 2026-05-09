# client.CustomProperty

`client.CustomProperty` reads, writes, and deletes custom properties (Shape Data) on shapes. Custom properties are named, typed values stored in each shape's `Prop` section. The lower-level cell-level API for these is on the [Custom properties](../custom-properties.md) page.

## Methods

| Method                                                                            | Returns                                                | Notes                                                                                |
| --------------------------------------------------------------------------------- | ------------------------------------------------------ | ------------------------------------------------------------------------------------ |
| `GetCustomPropertiesAsShapeDictionary(TargetShapes, CellValueType type)`          | `IDictionary<Shape, CustomPropertyDictionary>`         | One dictionary of `(name, cells)` per shape, keyed by `Shape`.                       |
| `ContainCustomPropertyWithName(TargetShapes, string name)`                        | `List<bool>`                                           | One boolean per shape: does the shape have a property of that name.                  |
| `SetCustomProperty(TargetShapes, string name, CustomPropertyCells)`               | `void`                                                 | Writes (creates or updates) the named property on every resolved shape.              |
| `DeleteCustomPropertyWithName(TargetShapes, string name)`                         | `void`                                                 | Removes the property from every resolved shape; no-op when the name is missing.      |

`TargetShapes.Auto` resolves to the currently selected shapes at call time. The write methods open an undo scope so the whole batch reverses together.

## Removal candidates (CY27)

These methods are currently `public` but have no known consumers as of 2026-05. They are candidates for removal in CY27 (see [issue #183](https://github.com/saveenr/VisioAutomation/issues/183)). New code should not depend on them.

| Method                                                                  | Notes                                                                       |
| ----------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| `GetCustomProperties(TargetShapes, CellValueType type)`                 | Bare `List<CustomPropertyDictionary>` form; `GetCustomPropertiesAsShapeDictionary` is the keyed-by-shape version most callers want. |

## Example

```csharp
var cp = new VisioAutomation.Shapes.CustomPropertyCells();
cp.Value = "\"Hello\"";       // values are formulas, hence the escaped quotes
cp.Label = "\"Greeting\"";
cp.Type = "0";                 // string

// Set property "greeting" on every selected shape
client.CustomProperty.SetCustomProperty(VisioScripting.TargetShapes.Auto, "greeting", cp);

// Read every property back as resolved values
var dict = client.CustomProperty.GetCustomPropertiesAsShapeDictionary(
    VisioScripting.TargetShapes.Auto,
    VisioAutomation.Core.CellValueType.Result);

foreach (var kv in dict)
{
    foreach (var entry in kv.Value)
    {
        System.Console.WriteLine("{0}.{1} = {2}", kv.Key.Name, entry.Key, entry.Value.Value);
    }
}
```

`CustomPropertyCells` fields are *formulas*, not literals, which is why `Value` and `Label` are wrapped in escaped double-quotes. See [Custom properties: values are formulas, not literals](../custom-properties.md) for more on this convention.

## See also

- [Custom properties](../custom-properties.md): the lower-level `CustomPropertyCells` and `CustomPropertyHelper` APIs.
- [User-defined cell](user-defined-cell.md): the unlabeled-row analog of custom properties.
- [Hyperlink](hyperlink.md): related per-shape data with the same "value is a formula" convention.
- [VisioScripting.Client overview](../visio-scripting.md): how `client.CustomProperty` fits into the facade.
