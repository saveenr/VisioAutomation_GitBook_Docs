# client.Arrange

`client.Arrange` covers spatial operations on a selection of shapes: nudging by an offset, aligning along an axis, and distributing evenly. Each write is wrapped in an undo scope so a single Undo reverses the whole operation.

## Methods

| Method                                                                       | Returns | Notes                                                                                |
| ---------------------------------------------------------------------------- | ------- | ------------------------------------------------------------------------------------ |
| `Nudge(TargetSelection, double dx, double dy)`                               | `void`  | Moves the resolved selection by `(dx, dy)` inches; no-op when both deltas are zero.  |
| `AlignHorizontal(TargetSelection, AlignmentHorizontal)`                      | `void`  | Aligns shapes to `Left`, `Center`, or `Right`.                                       |
| `AlignVertical(TargetSelection, AlignmentVertical)`                          | `void`  | Aligns shapes to `Top`, `Center`, or `Bottom`.                                       |
| `DistributeOnAxis(TargetSelection, Axis)`                                    | `void`  | Even space distribution along `XAxis` or `YAxis`; needs at least 2 shapes.           |
| `DistributeHorizontal(TargetSelection, AlignmentHorizontal)`                 | `void`  | Distribute by left edge, center, or right edge; needs at least 2 shapes.             |

`TargetSelection.Auto` resolves to the active window's current selection at call time. Distribute methods silently no-op if fewer than 2 shapes are selected.

## Removal candidates (CY27)

These methods are currently `public` but have no known consumers as of 2026-05. They are candidates for removal in CY27 (see [issue #183](https://github.com/saveenr/VisioAutomation/issues/183)). New code should not depend on them.

| Method                                                       | Notes                                                                |
| ------------------------------------------------------------ | -------------------------------------------------------------------- |
| `Send(ShapeSendDirection)`                                   | Send-to-back / forward / etc; reads the active window directly, ignoring `TargetSelection`. |
| `DistributeVertical(TargetSelection, AlignmentVertical)`     | Vertical-axis counterpart to `DistributeHorizontal`; redundant with `DistributeOnAxis(YAxis)`. |

## Example

```csharp
// Align the current selection to the left edge, then distribute vertically
client.Arrange.AlignHorizontal(VisioScripting.TargetSelection.Auto,
                                VisioScripting.Models.AlignmentHorizontal.Left);

client.Arrange.DistributeOnAxis(VisioScripting.TargetSelection.Auto,
                                 VisioScripting.Models.Axis.YAxis);

// Nudge the result one quarter inch to the right
client.Arrange.Nudge(VisioScripting.TargetSelection.Auto, 0.25, 0.0);
```

## See also

- [VisioScripting.Client overview](../visio-scripting.md): how `client.Arrange` fits into the facade.
- [Selection](selection.md): operate on or change the selection that `client.Arrange` consumes.
- [Page](page.md): page-level layout operations live on `client.Page.LayoutPage`.
- [Undo scope](../undo-scope.md): every `Arrange` write opens an undo scope.
