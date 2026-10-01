# Org chart model

`VisioAutomation.Models.Documents.OrgCharts` is a turn-key org-chart generator. Build a tree of `Node`s, add the root to an `OrgChartDocument`, call `Render(app)` (or `Client.Model.DrawOrgChart(VisioScripting.TargetPage.Auto, orgChartDocument)` from VisioScripting), and you get a new Visio document with the org-chart template applied, position-shape masters dropped per node, dynamic connectors between parent and child, and per-node text labels.

The generator is built on top of the [DOM](dom.md) and an internal tree layout, so the result is a real, editable Visio document, not a static export. After render the user can move shapes around, and the dynamic connectors stay glued to their shapes and re-route when shapes are moved.

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

`OrgCharts` is a `List<Node>`; add more than one root to get one chart per page (see _Multiple charts in one document_ below). The render call requires an `IVisio.Application`, not a page or document, because it creates a new document from the org-chart template every time. Output always goes to that new document, never onto an existing page.

From VisioScripting, `Client.Model.DrawOrgChart(VisioScripting.TargetPage.Auto, orgChartDocument)` does the same thing. The `TargetPage` only supplies the `Application`; the chart is rendered into a new document, and the target page is then resized to fit its own contents.

## Building a tree

Each `Node` has a `Children` list; nesting is done by adding to it. Names go into the constructor (the text can also be changed afterwards through the settable `Node.Text` property):

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

`Down` (top-down) is the conventional org-chart shape and what most renders should pick. Note that only `Down` is exercised by the test suite and known to work; the source carries a TODO for the other directions, so treat `Up`, `Left` and `Right` as unverified.

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

`OrgChartLayoutOptions.PageBorderWidth` controls the border around the chart. The renderer first sizes the page to the laid-out tree plus twice this border, then performs a one-off resize-to-fit of the page contents at render time using a margin of twice this value. The page is not kept resized as the user adds shapes later.

`DefaultNodeSize` (default `Size(2, 0.5)`) controls the size of every `Node` that doesn't have a `Size` set explicitly. Setting `Size` on a node overrides it for that shape.

## Multiple charts in one document

`OrgChartDocument.OrgCharts` is a list and the renderer creates one page per root, each showing that root's own tree:

```csharp
var orgchart = new VAORGCHART.OrgChartDocument();

var team_a = new VAORGCHART.Node("A");
team_a.Children.Add(new VAORGCHART.Node("B"));

var team_x = new VAORGCHART.Node("X");
team_x.Children.Add(new VAORGCHART.Node("Y"));

orgchart.OrgCharts.Add(team_a);   // page 1
orgchart.OrgCharts.Add(team_x);   // page 2

orgchart.Render(visioApp);
```

Rendering one chart per root was fixed in NuGet 3.1.0 ([#201](https://github.com/saveenr/VisioAutomation/issues/201)). In 3.0.0 and earlier every page is built from the first root's tree, so extra roots do not produce distinct charts; use a single root per document there. Loading from XML only ever produces one root.

## Loading from XML

Build an `OrgChartDocument` from XML with `client.Model.LoadOrgChartFromXml(xml)`, where `xml` is an `XDocument`. This public facade method was added in NuGet 3.1.0. With 3.0.0, use `VisioScripting.Loaders.OrgChartDocumentLoader.LoadFromXml(client, xml)` instead; that loader class is `internal` from 3.1.0 on. Then draw the result with `client.Model.DrawOrgChart(VisioScripting.TargetPage.Auto, orgChartDocument)`.

The schema is illustrated by the fixture `VTest/datafiles/orgchart_1.xml`; inline construction is more common for programmatic use.

```xml
<orgchart>
  <shape id="0" name="Akuma" />
  <shape id="1" name="Ryu" parentid="0"/>
  <shape id="2" name="Ken" parentid="0"/>
  <shape id="3" name="Chun-Li" parentid="2"/>
</orgchart>
```

The rules the loader applies:

* The root element is `<orgchart>`; its children are `<shape>` elements. Elements with any other name are ignored.
* `id` and `name` are required (a missing attribute throws). `parentid` is optional.
* The first `<shape>` becomes the sole root of the chart.
* A shape is attached to its parent only if the `parentid` has already appeared earlier in the file. Parents must be listed before their children; otherwise the shape is silently left unattached.
* There are no `url` or `size` attributes.
* A document with zero `<shape>` elements loads, but `Render` then throws because the chart has no root.

## See also

* [DOM model](dom.md) (the underlying shape model the renderer emits into)
* [Layout models](layouts.md) (Tree, Grid, and Box layouts; the org-chart renderer uses an internal tree layout under the hood)
* [Layout styles](layout-styles.md) (Visio's page-level auto-layout, which can be applied on top of an org chart for re-flow on edit)
