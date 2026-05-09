# client.Lock

`client.Lock` reads and writes the protection cells on a shape: `LockCalcWH`, `LockMoveX`, `LockMoveY`, `LockWidth`, `LockHeight`, and the rest of the `Protection` section. The lower-level cell-level API for these is on the [Lock cells](../lock-cells.md) page.

## Methods

| Method                                                        | Returns                          | Notes                                                                                |
| ------------------------------------------------------------- | -------------------------------- | ------------------------------------------------------------------------------------ |
| `SetLockCells(TargetShapes, LockCells lockcells)`             | `void`                           | Writes every non-null field of `lockcells` to every resolved shape, in one batch.    |
| `GetLockCells(TargetShapes, CellValueType cvt)`               | `Dictionary<int, LockCells>`     | Keys are Visio shape IDs; `cvt` selects formula or resolved value.                   |

`TargetShapes.Auto` resolves to the currently selected shapes at call time. `SetLockCells` opens an undo scope so the whole batch reverses together.

## Example

```csharp
var locks = new VisioAutomation.Shapes.LockCells();
locks.MoveX = 1;     // disallow horizontal move
locks.MoveY = 1;     // disallow vertical move
locks.Width = 1;     // disallow resize-width
locks.Height = 1;    // disallow resize-height

// Pin every selected shape in place
client.Lock.SetLockCells(VisioScripting.TargetShapes.Auto, locks);

// Read them back as resolved values
var dict = client.Lock.GetLockCells(VisioScripting.TargetShapes.Auto,
                                      VisioAutomation.Core.CellValueType.Result);
foreach (var kv in dict)
{
    System.Console.WriteLine("Shape #{0}: MoveX={1} MoveY={2}",
        kv.Key, kv.Value.MoveX, kv.Value.MoveY);
}
```

## See also

- [Lock cells](../lock-cells.md): the lower-level `LockCells` API and the underlying `Protection` section.
- [Custom property](custom-property.md): related per-shape cells with the same `Get*`/`Set*` shape.
- [VisioScripting.Client overview](../visio-scripting.md): how `client.Lock` fits into the facade.
- [Undo scope](../undo-scope.md): the scope that `SetLockCells` opens.
