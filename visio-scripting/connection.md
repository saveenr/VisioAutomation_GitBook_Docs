# client.Connection

`client.Connection` connects shapes with connector shapes and analyzes the connector graph already on a page. The lower-level analyzer types live behind [Connectors](../connectors.md) and the analysis output appears under [Analyzers](../analyzers.md).

## Methods

| Method                                                                                                         | Returns                          | Notes                                                                                       |
| -------------------------------------------------------------------------------------------------------------- | -------------------------------- | ------------------------------------------------------------------------------------------- |
| `GetDirectedEdgesOnPage(TargetPage, ConnectionAnalyzerOptions)`                                                | `List<ConnectorEdge>`            | Direct from-shape / to-shape edges traced by following connector glue.                      |
| `ConnectShapes(TargetPage, IList<IVisio.Shape> from, IList<IVisio.Shape> to, IVisio.Master connectorMaster)`   | `List<IVisio.Shape>`             | Drops one connector per pair; uses a default connector when `connectorMaster` is `null`.    |

`TargetPage.Auto` resolves to the active page at call time. `ConnectShapes` runs in an undo scope so the entire batch reverses with a single Undo.

## Removal candidates (CY27)

These methods are currently `public` but have no known consumers as of 2026-05. They are candidates for removal in CY27 (see [issue #183](https://github.com/saveenr/VisioAutomation/issues/183)). New code should not depend on them.

| Method                                                                       | Notes                                                                 |
| ---------------------------------------------------------------------------- | --------------------------------------------------------------------- |
| `GetTransitiveClosureOnActivePage(TargetPage, ConnectionAnalyzerOptions)`    | Computes the transitive closure of the directed-edge graph on a page. |

## Example

```csharp
using IVisio = Microsoft.Office.Interop.Visio;

// Three shapes already on the active page: a, b, c
var a = client.Draw.DrawRectangle(VisioScripting.TargetPage.Auto, 0, 0, 1, 1);
var b = client.Draw.DrawRectangle(VisioScripting.TargetPage.Auto, 3, 0, 4, 1);
var c = client.Draw.DrawRectangle(VisioScripting.TargetPage.Auto, 6, 0, 7, 1);

// Connect a -> b and b -> c with the default connector
client.Connection.ConnectShapes(VisioScripting.TargetPage.Auto,
                                  new[] { a, b },
                                  new[] { b, c },
                                  null);

// Inspect the resulting directed-edge graph
var edges = client.Connection.GetDirectedEdgesOnPage(
    VisioScripting.TargetPage.Auto,
    VisioAutomation.Analyzers.ConnectionAnalyzerOptions.None);

foreach (var e in edges)
{
    System.Console.WriteLine("{0} -> {1}", e.From.Name, e.To.Name);
}
```

## See also

- [Connectors](../connectors.md): the connector cells and helper API.
- [Connection points](../connection-points.md): the cells that connectors glue to (`client.ConnectionPoint`).
- [Analyzers](../analyzers.md): `ConnectionAnalyzer` and the edge / closure data shapes.
- [VisioScripting.Client overview](../visio-scripting.md): how `client.Connection` fits into the facade.
