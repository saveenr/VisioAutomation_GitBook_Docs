# Directed graph layout

`VisioAutomation.Models.Layouts.DirectedGraph` is a graph-shaped layout built on top of [Microsoft Automatic Graph Layout (MSAGL)](https://www.nuget.org/packages/Microsoft.Automatic.Graph.Layout). Use it for diagrams that don't fit a tree shape: graphs with cycles, multiple roots, parallel edges, or arbitrary connectivity. The layout engine assigns coordinates; you only describe nodes and edges.

For the XML wire format that the directed-graph loader accepts, see the separate [Directed graph XML format](../directed-graph-xml.md) page. This page covers the .NET object model.

## Building a graph in code

The entry point is `DirectedGraphLayout`. Add nodes by ID, label, stencil, and master; add edges between nodes with a connector type.

```csharp
using VADG = VisioAutomation.Models.Layouts.DirectedGraph;
using VA = VisioAutomation;

var d = new VADG.DirectedGraphLayout();

const string basic = "basic_u.vss";
var n0 = d.AddNode("n0", "Node 0", basic, "Rectangle");
n0.Size = new VA.Core.Size(3, 2);

var n1 = d.AddNode("n1", "Node 1", basic, "Rectangle");
var n2 = d.AddNode("n2", "Node 2", basic, "Rectangle");
var n3 = d.AddNode("n3", "Node 3", basic, "Rectangle");
d.AddNode("n4", "Node 4 (unconnected)", basic, "Rectangle");

d.AddEdge("c0", n0, n1, "0 -> 1", VA.Models.ConnectorType.Curved);
d.AddEdge("c1", n1, n2, "1 -> 2", VA.Models.ConnectorType.RightAngle);
d.AddEdge("c2", n1, n0, "0 -> 1", VA.Models.ConnectorType.Curved);     // back-edge: cycle
d.AddEdge("c3", n0, n2, "0 -> 2", VA.Models.ConnectorType.Straight);
d.AddEdge("c4", n2, n3, "2 -> 3", VA.Models.ConnectorType.Curved);
d.AddEdge("c5", n3, n0, "3 -> 0", VA.Models.ConnectorType.Curved);     // closes the cycle

var renderer = new VADG.MsaglRenderer();
renderer.Render(visioPage, d);
```

`AddNode` returns a `Node` whose properties (`Size`, `Cells`, `CustomProperties`) you set before render. Unconnected nodes are valid and appear in the layout.

`AddEdge` returns an `Edge`. Each edge carries a `ConnectorType`: `Curved` (the default), `Straight`, or `RightAngle`.

## Connector style: dynamic vs. routed

The renderer offers two connector styles, controlled by `LayoutOptions.UseDynamicConnectors`:

* **`UseDynamicConnectors = true`** drops Visio's built-in dynamic connector at each edge. Visio re-routes the connector when shapes move, but the initial path is whatever Visio computes, not what MSAGL computed. Best for diagrams the user will edit interactively after generation.
* **`UseDynamicConnectors = false`** uses the geometry MSAGL produced (Bezier curves or straight segments). The connector geometry is fixed at render time and won't re-route on shape moves. Best for one-shot output (export to image, save then close).

```csharp
var renderer = new VADG.MsaglRenderer();
renderer.LayoutOptions.UseDynamicConnectors = false;   // honor MSAGL's geometry
renderer.Render(visioPage, d);
```

## Layout direction

Set `LayoutOptions.Direction` on the `DirectedGraphLayout` to control the flow:

```csharp
d.LayoutOptions.Direction = VADG.MsaglDirection.LeftToRight;
```

Supported values: `TopToBottom` (default), `BottomToTop`, `LeftToRight`, `RightToLeft`.

The renderer copies its options from the layout when rendering, so passing the same `LayoutOptions` object to both is the typical pattern. If you construct the renderer's options separately, copy `LayoutOptions` over:

```csharp
var renderer = new VADG.MsaglRenderer();
renderer.LayoutOptions = d.LayoutOptions;
renderer.Render(visioPage, d);
```

## Custom properties on nodes

Each `Node` exposes `CustomProperties`, applied during render. This is the same `CustomPropertyDictionary` / `CustomPropertyCells` pair documented under [Custom properties](../custom-properties.md), so the formula-vs-literal rules apply: pre-encode literal values or use the typed setters.

```csharp
var n0 = d.AddNode("n0", "Untitled Node", "basflo_u.vss", "Decision");
n0.Size = new VA.Core.Size(3, 2);
n0.CustomProperties = new VisioAutomation.Shapes.CustomPropertyDictionary();
n0.CustomProperties["p1"] = new VisioAutomation.Shapes.CustomPropertyCells("\"v1\"");
n0.CustomProperties["p2"] = new VisioAutomation.Shapes.CustomPropertyCells("\"v2\"");
```

`CustomPropertyCells.SetString("v1")` is the modern equivalent and skips the manual quoting; see the [Custom properties](../custom-properties.md) page for the full set of typed setters.

## Setting `Size` and `Cells` together

If you want both a size *and* additional shapesheet styling on the same node, set them as two separate operations on the same `Cells`:

```csharp
var n = d.AddNode("n", "Sized + styled", "basic_u.vss", "Rectangle");
n.Size = new VA.Core.Size(1.25, 0.25);

n.Cells = new VA.Models.Dom.ShapeCells();
n.Cells.FillForeground = "rgb(0,255,0)";
n.Cells.FillPattern = 1;
```

Both the `Size`-derived width/height and the `Cells`-driven fill are honored at render. (This is the behavior fixed in [#82](https://github.com/saveenr/VisioAutomation/issues/82); pre-fix, setting `Cells` overwrote the `Cells` populated from `Size` and the shape rendered at master defaults.)

## Loading from XML

The same `DirectedGraphLayout` graph can be built from an XML document via `VisioScripting.Loaders.DirectedGraphDocumentLoader.LoadFromXml`. The XML schema is documented on the [Directed graph XML format](../directed-graph-xml.md) page; the call returns a `DirectedGraphDocument` (one or more `DirectedGraphLayout` per `<page>`):

```csharp
using SXL = System.Xml.Linq;

var xml = SXL.XDocument.Parse(xml_string);
var dg_doc = VisioScripting.Loaders.DirectedGraphDocumentLoader.LoadFromXml(client, xml);

var styling = new VA.Models.Layouts.DirectedGraph.DirectedGraphStyling();
client.Model.DrawDirectedGraphDocument(dg_doc, styling);
```

`<renderoptions>` attributes on each `<page>` map to `LayoutOptions` properties (`direction`, `usedynamicconnectors`, `scalingfactor`, `connectortype`, `layout`). The `layout` attribute currently accepts `Sugiyama`-only; any other value raises `ArgumentException` at load time.

## Multi-page directed-graph documents

A `DirectedGraphDocument` holds multiple `DirectedGraphLayout` instances, one per page. Each gets its own MSAGL run, its own `LayoutOptions`, and renders to its own Visio page. This is what the XML loader produces when given a multi-`<page>` document.

For one-page programmatic graphs the `MsaglRenderer.Render(page, layout)` overload above is enough; for multi-page documents the path through `Client.Model.DrawDirectedGraphDocument(dg_doc, styling)` handles page creation.

## See also

* [Directed graph XML format](../directed-graph-xml.md) (the XML wire format and its render-options schema)
* [Layout styles](layout-styles.md) (additional styling primitives layered on top of layouts)
* [Layouts](layouts.md) (Tree, Grid, and Box layouts for non-graph data)
* [Declarative DOM](dom.md) (the underlying shape model the renderer emits into)
* [Custom properties](../custom-properties.md) (formula-vs-literal, typed setters)
