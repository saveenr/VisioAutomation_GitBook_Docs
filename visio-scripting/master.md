# client.Master

`client.Master` lists, finds, and drops masters from a document's stencils. A master is the reusable shape definition stored in a stencil; dropping a master onto a page produces a regular shape that inherits the master's geometry, formatting, and ShapeSheet rows. Background and full reference: [Stencils and masters](../stencils-and-masters.md).

## Methods

| Method                                                                                              | Returns                  | Notes                                                                              |
| --------------------------------------------------------------------------------------------------- | ------------------------ | ---------------------------------------------------------------------------------- |
| `GetMasters(TargetDocument)`                                                                        | `List<IVisio.Master>`    | All masters in the document's `Masters` collection.                                |
| `GetMaster(TargetDocument, string name)`                                                            | `IVisio.Master`          | Looks up a master by exact name; throws if not found.                              |
| `FindMasters(TargetDocument, string name)`                                                          | `List<IVisio.Master>`    | Wildcard match on master name; `null` or `"*"` returns every master.               |
| `DropMaster(TargetPage, IVisio.Master master, Point p)`                                             | `IVisio.Shape`           | Drops a single master at point `p`.                                                |
| `DropMasters(TargetPage, IList<IVisio.Master> masters, IList<Point> points)`                        | `short[]`                | Bulk drop; returns the new shape IDs in the same order as the input.               |

`TargetDocument.Auto` resolves to the active document; `TargetPage.Auto` resolves to the active page. `DropMasters` is the form to prefer when dropping more than a couple of shapes (it's a single COM call versus N).

## Example

```csharp
using IVisio = Microsoft.Office.Interop.Visio;

// Open a stencil document, then drop one of its masters
var stencil = client.Document.OpenStencilDocument("basic_u.vss");
var rectMaster = client.Master.GetMaster(new VisioScripting.TargetDocument(stencil), "Rectangle");

var s1 = client.Master.DropMaster(VisioScripting.TargetPage.Auto,
                                   rectMaster,
                                   new VisioAutomation.Core.Point(2, 2));

// Bulk-drop a whole row
var masters = new[] { rectMaster, rectMaster, rectMaster };
var points = new[]
{
    new VisioAutomation.Core.Point(0, 0),
    new VisioAutomation.Core.Point(2, 0),
    new VisioAutomation.Core.Point(4, 0),
};
short[] ids = client.Master.DropMasters(VisioScripting.TargetPage.Auto, masters, points);
```

## See also

- [Stencils and masters](../stencils-and-masters.md): the lower-level master and stencil APIs.
- [Document](document.md): `client.Document.OpenStencilDocument` opens the stencils that `client.Master` reads from.
- [Container](container.md): `DropContainer` is a specialized master-drop for container shapes.
- [VisioScripting.Client overview](../visio-scripting.md): how `client.Master` fits into the facade.
