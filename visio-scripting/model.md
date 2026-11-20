# client.Model

`client.Model` renders the higher-level diagram models onto a page: data tables, grids, tree-shaped XML, org charts, and MSAGL-laid-out directed graphs. The model objects themselves live in `VisioAutomation.Models`; see [DOM](../models/dom.md), [Directed graph layout model](../models/directed-graph.md), [Org chart model](../models/org-charts.md), [Data table model](../models/data-table.md) and [XML model](../models/xml-model.md) for the model surface.

## Methods

| Method                                                                                                          | Returns                          | Notes                                                                                |
| --------------------------------------------------------------------------------------------------------------- | -------------------------------- | ------------------------------------------------------------------------------------ |
| `DrawDataTableModelIntoPage(TargetPage, DataTableModel)`                                                                | `void`                           | Draws the model's `DataTable` on the target page using its `CellWidth`, `CellHeight` and `CellSpacing`, then resizes the page to fit its contents. |
| `DrawGridIntoPage(TargetPage, GridLayout)`                                                                              | `void`                           | Renders an arbitrary `GridLayout`. Shapes only: the page is not resized.                                                   |
| `DrawXmlModelIntoPage(TargetPage, XmlModel)`                                                                            | `void`                           | Renders the element structure of an XML document as a tree (one node per element). The target page is resized to the tree.   |
| `DrawOrgChartDocument(OrgChartDocument)`                                                                 | `void`                           | Renders an org-chart model into a **new document**. No existing page is changed.                       |
| `DrawDirectedGraphDocument(DirectedGraphDocument, DirectedGraphStyling)`                                        | `void`                           | Creates a new document, renders one page per layout via the MSAGL bridge.            |
| `LoadDirectedGraphFromXml(XDocument)`                                                                           | `DirectedGraphDocument`          | Parses XML into a `DirectedGraphDocument` ready for `DrawDirectedGraphDocument`.     |
| `LoadOrgChartFromXml(XDocument)`                                                                                | `OrgChartDocument`               | Parses XML into an `OrgChartDocument` ready for `DrawOrgChartDocument`.                      |

`TargetPage.Auto` resolves to the active page at call time. The grid and table renderers wrap their writes in an undo scope; `DrawDirectedGraphDocument` creates its own document and so manages page lifetime itself.

## Renamed in NuGet 5.0.0

Each draw method now says where its output goes: `IntoPage` draws onto a page you give it, `Document` creates a new document. The old names still work and are marked `[Obsolete]`; they will be removed in a later release.

| Old name | New name |
| --- | --- |
| `DrawGrid(TargetPage, GridLayout)` | `DrawGridIntoPage(TargetPage, GridLayout)` |
| `DrawDataTableModel(TargetPage, DataTableModel)` | `DrawDataTableModelIntoPage(TargetPage, DataTableModel)` |
| `DrawXmlModel(TargetPage, XmlModel)` | `DrawXmlModelIntoPage(TargetPage, XmlModel)` |
| `DrawOrgChart(TargetPage, OrgChartDocument)` | `DrawOrgChartDocument(OrgChartDocument)` (no target page) |

`DrawDataTable(TargetPage, DataTable, widths, heights, cellspacing)` was removed from the public surface in 5.0.0 and is internal. Use `DrawDataTableModelIntoPage`, or a `GridLayout` with `DrawGridIntoPage`.

## Example

```csharp
using IVisio = Microsoft.Office.Interop.Visio;

// Render a small table on the active page
var dt = new System.Data.DataTable();
dt.Columns.Add("Name");
dt.Columns.Add("Role");
dt.Rows.Add("Alice", "Owner");
dt.Rows.Add("Bob", "Reviewer");

var model = new VisioAutomation.Models.Data.DataTableModel();
model.DataTable = dt;
model.CellWidth = 1.5;     // inches
model.CellHeight = 0.5;
client.Model.DrawDataTableModelIntoPage(VisioScripting.TargetPage.Auto, model);

// Load a directed-graph XML and render it onto fresh pages
var dgxml = System.Xml.Linq.XDocument.Load(@"C:\models\graph.xml");
var dgdoc = client.Model.LoadDirectedGraphFromXml(dgxml);
client.Model.DrawDirectedGraphDocument(dgdoc, new VisioAutomation.Models.Layouts.DirectedGraph.DirectedGraphStyling());
```

## See also

- [DOM](../models/dom.md): the building blocks for arbitrary shape models.
- [Directed graph layout model](../models/directed-graph.md): the directed-graph model type.
- [Org chart model](../models/org-charts.md): the org-chart model type.
- [Data table model](../models/data-table.md): the `DataTableModel` type.
- [XML model](../models/xml-model.md): the `XmlModel` type.
- [VisioScripting.Client overview](../visio-scripting.md): how `client.Model` fits into the facade.
