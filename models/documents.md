# Document models

The `VisioAutomation.Models.Documents` namespace holds models that describe a **whole Visio document** and then create it. You build a document object (an org chart), call `Render` with the Visio application, and get a new Visio document back, with its pages, shapes and template already in place. This differs from a [layout](layouts.md), which mostly arranges shapes on a page you give it.

There is one document model:

| Model | Namespace | Best for | Entry point |
| :--- | :--- | :--- | :--- |
| [Org chart model](org-charts.md) | `VisioAutomation.Models.Documents.OrgCharts` | A reporting structure: a tree of people drawn with Visio's org chart template. | `OrgChartDocument.Render(app)`, or `Client.Model.DrawOrgChartDocument(...)` from VisioScripting. Can be loaded from `<orgchart>` XML. |

## What they have in common

* **A document object is the root.** `OrgChartDocument` holds everything that goes into the document: the org chart's roots.
* **`Render` takes the Visio application.** It does not take a page or an existing document, because it creates a new document every time. The output never goes onto a page that is already open.
* **The result is an ordinary Visio document.** After it renders you can edit it, save it or export it like any other.

## See also

* [Org chart model](org-charts.md) (a tree of people, with the org chart template, XML loading and styling options)
* [Layout models](layouts.md) (arranging shapes on a page; the [directed graph](directed-graph.md) layout has its own `DirectedGraphDocument`, which also renders into a new document)
* [DOM](dom.md) (the underlying shape model)
* [client.Model](../visio-scripting/model.md) (the `DrawOrgChartDocument` and `LoadOrgChartFromXml` methods)
