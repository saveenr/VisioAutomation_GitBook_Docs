# client.Grouping

`client.Grouping` groups and ungroups shapes. Grouping makes a single composite shape out of a selection; ungrouping breaks one back into its parts.

## Methods

| Method                                          | Returns         | Notes                                                                                 |
| ----------------------------------------------- | --------------- | ------------------------------------------------------------------------------------- |
| `Group(TargetSelection)`                        | `IVisio.Shape`  | Groups the resolved selection and returns the new group shape.                        |
| `Ungroup(TargetShapes)`                         | `void`          | Calls `Ungroup()` on every resolved shape.                                            |

`TargetSelection.Auto` resolves to the active window's selection at call time; `TargetShapes.Auto` resolves to the currently selected shapes. Neither method opens an undo scope of its own (the underlying `IVisio` calls are themselves undoable).

## Removal candidates (CY27)

These methods are currently `public` but have no known consumers as of 2026-05. They are candidates for removal in CY27 (see [issue #183](https://github.com/saveenr/VisioAutomation/issues/183)). New code should not depend on them.

| Method                                  | Notes                                                                                  |
| --------------------------------------- | -------------------------------------------------------------------------------------- |
| `Ungroup(TargetSelection)`              | UI-command-based ungroup that ignores its argument; `Ungroup(TargetShapes)` is the working form. |

## Example

```csharp
using IVisio = Microsoft.Office.Interop.Visio;

// Select some shapes, then group them
client.Selection.SelectShapes(VisioScripting.TargetWindow.Auto,
                                new[] { shapeA, shapeB, shapeC });
var group = client.Grouping.Group(VisioScripting.TargetSelection.Auto);

System.Console.WriteLine("Created group {0} with {1} child shapes",
    group.Name, group.Shapes.Count);

// And later, ungroup it
client.Grouping.Ungroup(new VisioScripting.TargetShapes(group));
```

## See also

- [VisioScripting.Client overview](../visio-scripting.md): how `client.Grouping` fits into the facade.
- [Selection](selection.md): the selection that `Group` consumes comes from `client.Selection`.
- [Container](container.md): related "wrap a set of shapes" operation, but as a container shape.
- [Master](master.md): drop a master to create a single shape rather than grouping existing ones.
