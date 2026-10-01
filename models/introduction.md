# Introduction to models

A **model** in VisioAutomation is an in-memory structure that describes the document you want, built entirely in your own code. Nothing touches Visio while you build it. When the model is complete you invoke a renderer (usually a `Render` method), and the library turns it into shapes, connectors and pages.

```csharp
// 1. Build the model: a table of rows and columns, a tree of people, a graph.
var model = new VisioAutomation.Models.Data.DataTableModel();
model.DataTable = dataTable;

// 2. Render it.
client.Model.DrawDataTableModel(VisioScripting.TargetPage.Auto, model);
```

## Why work this way

* **You work close to the document you want.** You describe an org chart as people and reporting lines, a table as rows and columns, a graph as nodes and edges. You do not describe it as a list of master drops, connector glue operations and cell writes.
* **The library handles the Visio-specific work.** Placing shapes, connecting them, setting cells and keeping the drawing fast are done for you, including the more esoteric Visio features and the performance techniques that make large drawings render in reasonable time.
* **You still need to know some Visio.** You choose stencils and masters, and you need to understand pages and documents. But that is the basic vocabulary, not the advanced techniques.

If you need something a model does not cover, you can still reach into the shapes it created afterward and use the [imperative API](../extensions/drawing.md).

## The DOM is the foundation

The [DOM](dom.md) is the fundamental model. It describes pages, shapes, connectors and their cell values as a tree of plain objects, and its `Render` is where the Visio-specific techniques live:

* **Batched drawing.** Shapes are dropped onto a page in bulk, and their cell values are written in bulk, instead of one COM call at a time.
* **Render performance settings.** While a page renders, the DOM temporarily changes Visio application settings (deferred recalculation, auto-connect, live dynamics, screen updating) and restores the originals afterward.

Most of the other models build on the DOM. They work out *what* should be drawn and *where*, produce a DOM page, and hand it to the DOM to render, so they share its shape dropping and cell writing without repeating it. Most of them also get its render performance settings. The exception is the Grid layout, and so the data table model that uses it: they render only the shapes, so they get the batching but not the settings. The [layout models](layouts.md) [Tree](layouts-tree.md), [Grid](layouts-grid.md) and [Directed graph](directed-graph.md) do this, and so do the [Org chart model](org-charts.md) and both [data models](data.md) (through the grid and tree layouts).

Three models do **not** go through the DOM:

* **[Box geometry](box-geometry.md)** draws nothing. It only computes rectangles, and you draw them yourself, with the DOM or otherwise. It is not one of the layout models.
* **[Container layout](layouts-container.md)** drops its shapes and writes their formatting directly, so it does not get the DOM's render performance settings.
* **[Form page model](forms.md)** builds pages and text blocks directly, also without the DOM.

## The models

| Model | What you describe | What it produces |
| :--- | :--- | :--- |
| [DOM](dom.md) | Pages, shapes, connectors and cells. | Any drawing. The foundation the others build on. |
| [Layout models](layouts.md) | Structure only: a tree, a grid, columns or a graph. | Shapes placed by an algorithm. See [Tree](layouts-tree.md), [Grid](layouts-grid.md), [Container](layouts-container.md) and [Directed graph](directed-graph.md). |
| [Document models](documents.md) | A whole document: an org chart or a set of form pages. | A new Visio document. See [Org chart](org-charts.md) and [Form page](forms.md). |
| [Data models](data.md) | Data you already have: a `DataTable` or an `XmlDocument`. | A picture of that data on a page. See [Data table](data-table.md) and [XML](xml-model.md). |
| [Box geometry](box-geometry.md) | Nested rectangles packed in a direction. | Rectangles only. It draws nothing, and nothing else in the library uses it. |
| [Layout styles](layout-styles.md) | Not a model. Visio's own page-level layout feature. | A re-arranged page, applied after or instead of a layout model. |

Each model's page has a "Where the output goes" section that says what the model produces (a new document, new pages, or shapes on an existing page) and what it does to the page you give it.

## See also

* [DOM](dom.md) (the foundation, including its render performance settings)
* [client.Model](../visio-scripting/model.md) (the VisioScripting methods that draw the models)
