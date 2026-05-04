# Analyzers

`VisioAutomation.Analyzers` provides a small set of utilities for **inspecting** an existing Visio diagram &mdash; reading its structure rather than building it. The current public surface is focused on connector analysis: extracting the directed graph of how shapes are connected on a page.

## `ConnectionAnalyzer.GetDirectedEdges`

Returns the list of directed edges (`from-shape`, `to-shape`, `connector-shape`) implied by the connectors on a page.

```csharp
var options = new VisioAutomation.Analyzers.ConnectionAnalyzerOptions();
var edges = VisioAutomation.Analyzers.ConnectionAnalyzer.GetDirectedEdges(page, options);

foreach (var edge in edges)
{
    System.Console.WriteLine(
        "{0} -> {1} via {2}",
        edge.From.NameU,
        edge.To.NameU,
        edge.Connector.NameU);
}
```

The returned `ConnectorEdge` has `From`, `To`, and `Connector` properties.

## How direction is determined

`ConnectionAnalyzerOptions` controls how the analyzer chooses which end of a connector is "from" and which is "to":

| `EdgeDirectionSource` | Meaning |
| --- | --- |
| `UseConnectionOrder` | Trust the order Visio recorded the glue connections (the connector's begin endpoint is "from", end is "to"). |
| `UseConnectorArrows` *(default)* | Look at the connector's `LineBeginArrow` / `LineEndArrow` cells to decide direction based on which end has an arrowhead. |

When `UseConnectorArrows` is in effect and a connector has no arrowheads at either end, `EdgeNoArrowsHandling` decides what to do:

| `EdgeNoArrowsHandling` | Meaning |
| --- | --- |
| `ExcludeEdge` | Drop the edge from the result. |
| `IncludeEdgesForBothDirections` *(default)* | Emit two edges (one in each direction). |

```csharp
var options = new VisioAutomation.Analyzers.ConnectionAnalyzerOptions();
options.EdgeDirectionSource    = VisioAutomation.Analyzers.EdgeDirectionSource.UseConnectorArrows;
options.EdgeNoArrowsHandling   = VisioAutomation.Analyzers.EdgeNoArrowsHandling.ExcludeEdge;

var edges = VisioAutomation.Analyzers.ConnectionAnalyzer.GetDirectedEdges(page, options);
```

## Generic `DirectedEdge<TNode, TData>`

For your own graph-building code on top of an analysis result, the namespace also exposes a small generic struct:

```csharp
public struct DirectedEdge<TNode, TData>
{
    public readonly TNode From;
    public readonly TNode To;
    public readonly TData Data;
    // ...
}
```

It's used internally by `GetDirectedEdges` and is reusable when you need to build your own typed edge collection.

## See also

* [Connectors](connectors.md) &mdash; building connections programmatically (the inverse direction).
