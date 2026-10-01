# Tree layout model

Use when the data is a true tree: one root, no cycles, every node has at most one parent. Build a `Drawing` whose `Root` is a `Node`, recursively attach `Children`, then `Render(page)`. The layout decides positions; you only specify size (and only if you don't want the default).

```csharp
using VATREE = VisioAutomation.Models.Layouts.Tree;
using VA = VisioAutomation;

var t = new VATREE.Drawing();
t.Root = new VATREE.Node("Root");

var na = new VATREE.Node("A");
var nb = new VATREE.Node("B");
var na1 = new VATREE.Node("A1");
var na2 = new VATREE.Node("A2");

t.Root.Children.Add(na);
t.Root.Children.Add(nb);
na.Children.Add(na1);
na.Children.Add(na2);

t.LayoutOptions.DefaultNodeSize = new VA.Core.Size(1, 1);
t.Render(visioPage);
```

Each `Node` exposes `VisioShape` after rendering, so you can pull the COM object back out for follow-up formatting. Per-node size is set with `Node.Size` (a nullable `Size`; when null the layout uses `LayoutOptions.DefaultNodeSize`, 2 x 0.5 by default). `LayoutOptions` also holds `Direction` (`Up`, `Down`, `Left`, `Right`; default `Down`), `ConnectorType` (`DynamicConnector`, `CurvedBezier` or `PolyLine`; default `DynamicConnector`) and `ConnectorCells` for formatting the connectors. The spacing between levels, siblings and subtrees is fixed inside the layout (1, 0.25 and 1 respectively).

The layout always uses Visio's "Rectangle" master from `basic_u.vss` for nodes and "Dynamic Connector" from `connec_u.vss` for edges; these are not configurable on the tree layout.

## See also

* [Layout models](layouts.md) (the overview and comparison of all the layouts)
* [Declarative DOM model](dom.md) (the underlying shape model the layouts emit into)
* [Org chart model](org-charts.md) (shares the internal tree-layout engine rather than this public Tree API)
* [XML model](xml-model.md) (draws an XML document's structure with the tree layout)
