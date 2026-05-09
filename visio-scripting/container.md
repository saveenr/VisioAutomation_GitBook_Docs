# client.Container

`client.Container` drops a Visio container shape around the current selection. Containers come from container masters; the related stencil API lives on the [Stencils and masters](../stencils-and-masters.md) page.

## Methods

| Method                                                | Returns         | Notes                                                                                |
| ----------------------------------------------------- | --------------- | ------------------------------------------------------------------------------------ |
| `DropContainer(TargetPage, string masterName)`        | `IVisio.Shape`  | Opens the built-in Containers stencil, looks up `masterName`, drops it around the current selection. |
| `DropContainerMaster(TargetPage, IVisio.Master)`      | `IVisio.Shape`  | As above, but uses a `Master` you already have (no stencil load).                    |

`TargetPage.Auto` resolves to the active page. Both methods read the active window's selection to determine which shapes the new container wraps.

## Example

```csharp
using IVisio = Microsoft.Office.Interop.Visio;

// Select some shapes first via client.Selection, then drop a container around them
var selectionWindow = VisioScripting.TargetWindow.Auto;
client.Selection.SelectAllShapes(selectionWindow);

// Use a built-in container by name
var container = client.Container.DropContainer(VisioScripting.TargetPage.Auto,
                                                "Plain");

System.Console.WriteLine("Container {0} now wraps the selection", container.Name);
```

## See also

- [Stencils and masters](../stencils-and-masters.md): the broader stencil and master concepts.
- [Master](master.md): finding and dropping arbitrary masters (not just containers).
- [Selection](selection.md): the selection that the container wraps comes from `client.Selection`.
- [VisioScripting.Client overview](../visio-scripting.md): how `client.Container` fits into the facade.
