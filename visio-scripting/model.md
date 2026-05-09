# client.Model

`client.Model` renders the higher-level diagram models onto a page: data tables, grids, tree-shaped XML, org charts, and MSAGL-laid-out directed graphs. The model objects themselves live in `VisioAutomation.Models`; see [Declarative DOM](../models/dom.md), [Directed graph](../models/directed-graph.md), and [Org charts](../models/org-charts.md) for the model surface.

## Methods

| Method                                                                                                          | Returns                          | Notes                                                                                |
| --------------------------------------------------------------------------------------------------------------- | -------------------------------- | ------------------------------------------------------------------------------------ |
| `DrawDataTable(TargetPage, DataTable, IList<double> widths, IList<double> heights, Size cellspacing)`           | `List<IVisio.Shape>`             | Renders a `System.Data.DataTable` as a grid of rectangles, one cell per value.       |
| `DrawDataTableModel(TargetPage, DataTableModel)`                                                                | `void`                           | Convenience wrapper around `DrawDataTable` using sizes baked into the model.         |
| `DrawGrid(TargetPage, GridLayout)`                                                                              | `void`                           | Renders an arbitrary `GridLayout`.                                                   |
| `DrawXmlModel(TargetPage, XmlModel)`                                                                            | `void`                           | Renders an XML document as a tree of nodes (one node per `XmlElement`).              |
| `DrawOrgChart(TargetPage, OrgChartDocument)`                                                                    | `void`                           | Renders an org-chart model; resizes the page to fit when done.                       |
| `DrawDirectedGraphDocument(DirectedGraphDocument, DirectedGraphStyling)`                                        | `void`                           | Creates a new document, renders one page per layout via the MSAGL bridge.            |
| `LoadDirectedGraphFromXml(XDocument)`                                                                           | `DirectedGraphDocument`          | Parses XML into a `DirectedGraphDocument` ready for `DrawDirectedGraphDocument`.     |
| `LoadOrgChartFromXml(XDocument)`                                                                                | `OrgChartDocument`               | Parses XML into an `OrgChartDocument` ready for `DrawOrgChart`.                      |

`TargetPage.Auto` resolves to the active page at call time. The grid and table renderers wrap their writes in an undo scope; `DrawDirectedGraphDocument` creates its own document and so manages page lifetime itself.

## Example

```csharp
using IVisio = Microsoft.Office.Interop.Visio;

// Render a small table on the active page
var dt = new System.Data.DataTable();
dt.Columns.Add("Name");
dt.Columns.Add("Role");
dt.Rows.Add("Alice", "Owner");
dt.Rows.Add("Bob", "Reviewer");

var widths = new[] { 1.5, 1.5 };
var heights = new[] { 0.5, 0.5 };
var spacing = new VisioAutomation.Core.Size(0.05, 0.05);
client.Model.DrawDataTable(VisioScripting.TargetPage.Auto, dt, widths, heights, spacing);

// Load a directed-graph XML and render it onto fresh pages
var dgxml = System.Xml.Linq.XDocument.Load(@"C:\models\graph.xml");
var dgdoc = client.Model.LoadDirectedGraphFromXml(dgxml);
client.Model.DrawDirectedGraphDocument(dgdoc, new VisioAutomation.Models.Layouts.DirectedGraph.DirectedGraphStyling());
```

## See also

- [Declarative DOM](../models/dom.md): the building blocks for arbitrary shape models.
- [Directed graph](../models/directed-graph.md): the directed-graph model type.
- [Org charts](../models/org-charts.md): the org-chart model type.
- [VisioScripting.Client overview](../visio-scripting.md): how `client.Model` fits into the facade.
