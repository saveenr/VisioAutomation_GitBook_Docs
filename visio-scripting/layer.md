# client.Layer

`client.Layer` looks up layers on a page. Visio layers are page-scoped named groupings that shapes can opt into for visibility, printability, and selection control.

> All methods on `client.Layer` are removal candidates (see below). Layer creation, member-add/remove, and visibility management are already covered by `IVisio.Page.Layers` directly; the helpers here provide nothing on top.

## Methods

This group has no methods that are not removal candidates.

## Removal candidates (CY27)

These methods are currently `public` but have no known consumers as of 2026-05. They are candidates for removal in CY27 (see [issue #183](https://github.com/saveenr/VisioAutomation/issues/183)). New code should not depend on them.

| Method                                                 | Notes                                                                       |
| ------------------------------------------------------ | --------------------------------------------------------------------------- |
| `FindLayersOnPageByName(TargetPage, string name)`      | Single-name lookup on `Page.Layers.ItemU[name]`.                            |
| `GetLayersOnPage(TargetPage)`                          | Wraps `Page.Layers.ToList()`.                                               |

## Example

For now, prefer the underlying `IVisio.Layers` collection directly:

```csharp
using IVisio = Microsoft.Office.Interop.Visio;
using VisioAutomation.Extensions;

var page = client.Page.GetActivePage();
var layers = page.Layers.ToList();
foreach (var layer in layers)
{
    System.Console.WriteLine("Layer: {0}", layer.Name);
}
```

## See also

- [VisioScripting.Client overview](../visio-scripting.md): how `client.Layer` fits into the facade.
- [Selection](selection.md): historical "select shapes by layer" call (also a removal candidate).
- [Page](page.md): page-level operations including page format and layout.
