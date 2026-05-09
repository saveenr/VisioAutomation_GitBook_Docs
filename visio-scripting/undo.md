# client.Undo

`client.Undo` opens undo scopes and triggers Undo or Redo on the attached Visio. Most write methods on the rest of `client` already wrap themselves in a scope; this group is the entry point when you need to wrap a batch of mixed operations into a single Undo step. Background: [Undo scope](../undo-scope.md).

## Methods

| Method                          | Returns                       | Notes                                                                                |
| ------------------------------- | ----------------------------- | ------------------------------------------------------------------------------------ |
| `NewUndoScope(string name)`     | `UndoScope` (`IDisposable`)   | Opens a named scope; commits on `Dispose`. Throws if no application is attached.     |
| `UndoLastAction()`              | `void`                        | Triggers `Application.Undo()`.                                                       |
| `RedoLastAction()`              | `void`                        | Triggers `Application.Redo()`.                                                       |

`NewUndoScope` is the form to use around a custom batch of Visio writes (drop a master, set custom properties, change layout) so that a single Undo reverses the whole sequence. The returned `UndoScope` is `IDisposable`; use a `using` block.

## Example

```csharp
using IVisio = Microsoft.Office.Interop.Visio;

// Wrap a custom batch into a single Undo step
using (var scope = client.Undo.NewUndoScope("Add labelled rectangle"))
{
    var rect = client.Draw.DrawRectangle(VisioScripting.TargetPage.Auto, 0, 0, 4, 2);
    client.Selection.SelectShapesById(VisioScripting.TargetWindow.Auto, rect);
    client.Text.SetShapeText(VisioScripting.TargetShapes.Auto, new[] { "Hello" });
}

// Now a single Ctrl+Z (or this call) reverses the whole batch
client.Undo.UndoLastAction();
client.Undo.RedoLastAction();
```

## See also

- [Undo scope](../undo-scope.md): the lower-level `VisioAutomation.Application.UndoScope` type.
- [VisioScripting.Client overview](../visio-scripting.md): how `client.Undo` fits into the facade.
- [Application](application.md): the attached `Application` that `Undo` / `Redo` are dispatched on.
