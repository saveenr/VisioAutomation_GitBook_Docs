# Org charts

`VisioAutomation.Models.Documents.OrgCharts` is a turn-key org-chart generator. Build a tree of `Node`s, add the root to an `OrgChartDocument`, call `Render(app)`, and you get a Visio document with the org-chart template applied, position-shape masters dropped per node, dynamic connectors between parent and child, and per-node text labels.

The generator is built on top of the [DOM](dom.md) and an internal tree layout, so the result is a real, editable Visio document, not a static export. After render the user can move shapes around and Visio's auto-layout will keep the connections tidy..

## Hello-world

A single-node "org chart":

```csharp
using VAORGCHART = VisioAutomation.Models.Documents.OrgCharts;
using VA = VisioAutomation;

var orgchart = new VAORGCHART.OrgChartDocument();

var ceo = new VAORGCHART.Node("Alex (CEO)");
ceo.Size = new VA.Core.Size(4, 2);
orgchart.OrgCharts.Add(ceo);

orgchart.Render(visioApp);
```

`OrgCharts` is a `List<Node>`; the API supports multiple roots in one document (each renders to its own page). The render call requires an `IVisio.Application`, not a page or document, because it creates a new document from the org-chart template every time.

## Building a tree

Each `Node` has a `Children` list; nesting is done by adding to it. Names go into the constructor:

```csharp
var ceo = new VAORGCHART.Node("Alex (CEO)");
ceo.Size = new VA.Core.Size(4, 2);

var cto = new VAORGCHART.Node("Sam (CTO)");
var cfo = new VAORGCHART.Node("Robin (CFO)");

var lead1 = new VAORGCHART.Node("Casey (Lead)");
var lead2 = new VAORGCHART.Node("Pat (Lead)");

ceo.Children.Add(cto);
ceo.Children.Add(cfo);
cto.Children.Add(lead1);
cto.Children.Add(lead2);

orgchart.OrgCharts.Add(ceo);
orgchart.Render(visioApp);
```

The renderer drops the org-chart template's "Position" master per node (or "Position Belt" on Visio 2013+, see _Styling_ below), sizes each shape, lays them out top-down by default, and connects parent to child with the dynamic connector master.

After render every `Node` has its `VisioShape` populated for follow-up work.

## Per-node URL hyperlinks

Set `Node.Url` before rendering and the renderer adds a Visio Hyperlink (named `Row_1`) on the shape with that address, so users can right-click and follow it.

```csharp
var lead1 = new VAORGCHART.Node("Casey (Lead)");
lead1.Url = "https://example.internal/people/casey";
```

## Layout direction

`OrgChartLayoutOptions.Direction` is an `OrgChartLayoutDirection` enum: `Down` (default), `Up`, `Left`, `Right`.

```csharp
orgchart.OrgChartLayoutOptions.Direction = VAORGCHART.OrgChartLayoutDirection.Right;
```

`Down` (top-down) is the conventional org-chart shape and what most renders should pick.

## Connector style

`OrgChartLayoutOptions.UseDynamicConnectors` toggles between two connector strategies:

* **`true` (default)** drops Visio's dynamic connector master between parent and child. Visio routes the connector and re-routes when shapes move; this is what users typically expect.
* **`false`** draws Bezier curves directly from the layout's geometry. The drawing is fixed at render time and won't re-route.

For interactive org charts, leave it at the default. For one-shot exports where you'll save to PNG and never edit, `false` produces tighter geometry.

## Styling: templates, masters, fonts

`OrgChartStyling` (settable on `OrgChartDocument.Styling`) exposes the per-Visio-version names the renderer uses. The defaults match the Visio install layout for both 2010 and 2013+:

| Setting                                                 | Visio 2010 default  | Visio 2013+ default |
| ------------------------------------------------------- | ------------------- | ------------------- |
| `Visio2010Template` / `Visio2013Template`               | `orgch_u.vst`       | `orgch_u.vstx`      |
| `Visio2010NodeMaster` / `Visio2013NodeMaster`           | `Position`          | `Position Belt`     |
| `Visio2010ConnectorMaster` / `Visio2013ConnectorMaster` | `Dynamic connector` | `Dynamic connector` |

The renderer auto-picks the right pair based on the running Visio's major version (≥15 means 2013+). Override either set if your install ships a different stencil or template, or if you want a custom node master.

## Page sizing

`OrgChartLayoutOptions.PageBorderWidth` controls the border around the chart. The renderer sizes the page to fit the laid-out tree plus this border on every side, and sets `ResizeToFit` so Visio will keep the page sized to contents as the user adds shapes.

`DefaultNodeSize` controls the size of any `Node` that doesn't have a `Size` set explicitly. The first child without a size will be drawn at this size; setting `Size` on the root (or on each node) overrides it per-shape.

## Multiple charts in one document

`OrgChartDocument.OrgCharts` is a list; adding more than one root produces a multi-page document, one chart per page. This is useful for "departments side by side" or "before / after" comparisons in a single deliverable.

```csharp
orgchart.OrgCharts.Add(eng_root);
orgchart.OrgCharts.Add(sales_root);
orgchart.Render(app);   // produces a 2-page document
```

## Loading from XML

The same `OrgChartDocument` can be built from XML via `VisioScripting.Loaders.OrgChartDocumentLoader.LoadFromXml(client, xml)`, mirroring the directed-graph XML loader. The XML schema is documented in the source-side data files (`VTest.Models/datafiles/orgchart_*.xml`); inline build is more common for programmatic use.

## See also

* [Declarative DOM](dom.md) (the underlying shape model the renderer emits into)
* [Layouts](layouts.md) (Tree, Grid, and Box layouts; the org-chart renderer uses an internal tree layout under the hood)
* [Layout styles](layout-styles.md) (Visio's page-level auto-layout, which can be applied on top of an org chart for re-flow on edit)
