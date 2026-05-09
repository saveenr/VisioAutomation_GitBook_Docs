# client.UserDefinedCell

`client.UserDefinedCell` reads, writes, and deletes user-defined cells on shapes. User-defined cells are the unlabeled name-value rows in each shape's `User` section, the lower-level cousin of custom properties. The lower-level cell-level API for these is on the [User-defined cells](../user-defined-cells.md) page.

## Methods

| Method                                                                                | Returns                                                  | Notes                                                                              |
| ------------------------------------------------------------------------------------- | -------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| `GetUserDefinedCellsAsShapeDictionary(TargetShapes, CellValueType cvt)`               | `Dictionary<Shape, UserDefinedCellDictionary>`           | One dictionary of `(name, cells)` per shape, keyed by `Shape`.                     |
| `SetUserDefinedCell(TargetShapes, string name, UserDefinedCellCells)`                 | `void`                                                   | Writes (creates or updates) the named cell on every resolved shape.                |
| `DeleteUserDefinedCellsByName(TargetShapes, string name)`                             | `void`                                                   | Removes the named user-defined cell from every resolved shape.                     |

`TargetShapes.Auto` resolves to the currently selected shapes at call time. The write methods open an undo scope so the whole batch reverses together.

## Removal candidates (CY27)

These methods are currently `public` but have no known consumers as of 2026-05. They are candidates for removal in CY27 (see [issue #183](https://github.com/saveenr/VisioAutomation/issues/183)). New code should not depend on them.

| Method                                                                  | Notes                                                                       |
| ----------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| `GetUserDefinedCells(TargetShapes, CellValueType cvt)`                  | Bare `List<UserDefinedCellDictionary>` form; `GetUserDefinedCellsAsShapeDictionary` is the keyed-by-shape version most callers want. |
| `ContainsUserDefinedCellsWithName(TargetShapes, string name)`           | Per-shape boolean; `GetUserDefinedCellsAsShapeDictionary` plus a `ContainsKey` is the more direct test. |

## Example

```csharp
var udc = new VisioAutomation.Shapes.UserDefinedCellCells();
udc.Value = "\"42\"";          // formula, hence escaped quotes
udc.Prompt = "\"Answer\"";

// Set user-defined cell "answer" on every selected shape
client.UserDefinedCell.SetUserDefinedCell(VisioScripting.TargetShapes.Auto, "answer", udc);

// Read every cell back as resolved values
var dict = client.UserDefinedCell.GetUserDefinedCellsAsShapeDictionary(
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

## See also

- [User-defined cells](../user-defined-cells.md): the lower-level `UserDefinedCellCells` and `UserDefinedCellHelper` APIs.
- [Custom property](custom-property.md): the labeled-row sibling of user-defined cells.
- [VisioScripting.Client overview](../visio-scripting.md): how `client.UserDefinedCell` fits into the facade.
- [Undo scope](../undo-scope.md): the scope that the write methods open.
