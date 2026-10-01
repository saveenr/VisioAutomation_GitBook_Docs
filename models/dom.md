# DOM

The **Document Object Model** under `VisioAutomation.Models.Dom` is the highest-level authoring API in the library. Build an in-memory tree of plain objects describing the diagram you want, then call `Render()` to materialize it as actual Visio shapes in one batch. The model decouples diagram authoring from per-shape COM bookkeeping, and makes diagrams composable from helpers and loops.

If you are looking for the imperative, COM-style equivalent, see the [Drawing primitives](../extensions/drawing.md) page; the DOM is built on the same primitives but stitches them together into a tree.

## The shape of the tree

```
Document  (not a Node; the root)
   |- Pages : PageList
            |- Page
                  |- Shapes : ShapeList
                            |- BaseShape (base class of everything below)
                                  |- Shape  (instance of a master)
                                  |     |- Connector
                                  |- Rectangle
                                  |- Oval
                                  |- Line
                                  |- PolyLine
                                  |- BezierCurve
```

Each node carries the data needed to materialize itself. `Document`, `Page`, `PageList` and `ShapeList` have a `Render()` method; the individual shape nodes do not, and are drawn when their containing `ShapeList` renders. `Shape` instances reference a master by name or by `IVisio.Master`. The geometric primitives (`Rectangle`, `Oval`, `Line`, `PolyLine`, `BezierCurve`) carry their own coordinates and don't need a master.

## Where the output goes

You can render from any level of the tree, and whichever level you start from, its children render too. What differs is **where the output lands**: a new document, new pages in a document you already have, or an existing page.

| Desired output | How to get it | Notes |
| :--- | :--- | :--- |
| A **new Visio document**. | `Document.Render(app)` | Creates the document, blank or from a template. The first `Page` node is rendered into the document's initial page, and each remaining `Page` node is added as a new page. |
| **New pages in an existing document.** | `Page.Render(doc)` or `PageList.Render(doc)` | Adds one new page per `Page` node, after the pages that are already there. Nothing existing is changed. |
| **An existing page, then new pages.** | `PageList.Render(startPage)` | Renders the first `Page` node into `startPage` and adds a new page to its document for each of the others. `Document.Render` uses this. |
| **An existing page**, filled in. | `Page.Render(visioPage)` | Draws the shapes and also applies the page-level settings: the page's name and size, its page and layout cells, the optional layout style, and the optional resize to fit. |
| **Shapes only, on an existing page.** | `ShapeList.Render(visioPage)` | Draws the shapes and changes nothing about the page itself. |

```csharp
// New document (blank here; the constructor can also take a template)
var doc_node = new VADOM.Document();
doc_node.Pages.Add(page_node);
IVisio.Document newDoc = doc_node.Render(visioApp);

// New page in a document you already have
IVisio.Page newPage = page_node.Render(visioDoc);

// An existing page, including its page-level settings
page_node.Render(visioPage);

// Only the shapes, onto an existing page
shape_list.Render(visioPage);
```

A few details:

* **Starting from a template.** `VADOM.Document` also accepts a template filename and measurement system in its constructor, so a render can start from a Visio template (`.vst` / `.vstx`) instead of a blank document.
* **A document with no pages.** Rendering a `Document` that has no `Page` nodes still creates the document, with its one empty initial page.
* **Existing pages are not cleared.** Rendering into an existing page adds shapes alongside whatever is already there.
* **Performance settings.** The temporary Visio settings described under [Render performance](#render-performance) are applied by `Page.Render`, so they apply to every call above except `ShapeList.Render`. A `ShapeList` render still drops shapes and writes cell values in bulk, but it does not change the application settings.

After rendering, each DOM node has its `VisioShape` (or `VisioPage`) property populated, so you can pull the underlying COM object out for further work.

## Hello-world

Drop a single rectangle with text:

```csharp
using VADOM = VisioAutomation.Models.Dom;
using VATEXT = VisioAutomation.Models.Text;

var page_node = new VADOM.Page();
var rect = new VADOM.Rectangle(1, 1, 9, 9);
rect.Text = new VATEXT.Element("Hello, Visio");
rect.Cells.FillForeground = "rgb(255,0,0)";
page_node.Shapes.Add(rect);

page_node.Render(visioDoc);   // visioDoc is an IVisio.Document
```

The `Cells` property on each node mirrors the Visio ShapeSheet structure, so the same shapesheet vocabulary documented under [Shape cells](../shape-cells.md) works here.

## Building from masters

Master-based shapes accept either a master object or a master name plus stencil name. Both styles are interchangeable; the name-based form looks the master up at render time.

```csharp
using VisioAutomation.Extensions;   // OpenStencil is an extension method on IVisio.Documents

var page_node = new VADOM.Page();

// By master object (master already resolved)
var stencil = visioDoc.Application.Documents.OpenStencil("basic_u.vss");
var rectMaster = stencil.Masters["Rectangle"];
page_node.Shapes.Drop(rectMaster, 3, 3);

// By name (resolved at render time)
page_node.Shapes.Drop("Rectangle", "basic_u.vss", 5, 5);

page_node.Render(visioDoc);
```

`ShapeList` exposes convenience helpers (`Drop`, `DrawRectangle`, `DrawOval`, `DrawLine`, `DrawPolyLine`, `DrawBezier`, `Connect`) that mirror the most common imperative drawing operations from [Drawing primitives](../extensions/drawing.md).

## Connectors

Edges between shapes become `Connector` nodes inside the same `ShapeList`. The shorthand `Shapes.Connect(masterName, stencilName, fromShape, toShape)` creates a `Connector` node and adds it to the list. Nothing is glued at that point; the glue happens when the `ShapeList` is rendered.

```csharp
var page_node = new VADOM.Page();
var rectMaster = "Rectangle";
var rectStencil = "basic_u.vss";

var s1 = new VADOM.Shape(rectMaster, rectStencil, new VisioAutomation.Core.Point(2, 2));
var s2 = new VADOM.Shape(rectMaster, rectStencil, new VisioAutomation.Core.Point(6, 6));
page_node.Shapes.Add(s1);
page_node.Shapes.Add(s2);

page_node.Shapes.Connect("Dynamic Connector", "connec_u.vss", s1, s2);

page_node.Render(visioDoc);
```

The connector's `From` and `To` are typed `BaseShape`, are read-only, and are set through the constructor (or `Connect`). They are references to DOM nodes, not Visio IDs, so the connector can be specified before either endpoint has been rendered. Both endpoints must be in the same `ShapeList` that is being rendered.

## Custom properties on DOM shapes

Every shape node exposes a `CustomProperties` dictionary (null by default; assign one) that's applied during render. Each entry is a [`CustomPropertyCells`](../custom-properties.md), so the same typed-setter idioms apply.

```csharp
var rect = new VADOM.Rectangle(1, 1, 4, 4);
rect.CustomProperties = new VisioAutomation.Shapes.CustomPropertyDictionary();

var owner = new VisioAutomation.Shapes.CustomPropertyCells();
owner.SetString("Alex");          // typed setter handles formula encoding
owner.Label = "\"Owner\"";        // pre-encoded literal

rect.CustomProperties["Owner"] = owner;
```

For the formula-vs-literal distinction and the typed setters, see the [Custom properties](../custom-properties.md) page.

## After render: round-tripping

After `Render()` returns, every DOM node has its corresponding `VisioShape` / `VisioPage` populated. You can use it for follow-up work that the DOM doesn't model directly (custom selection, advanced formatting, ShapeSheet queries):

```csharp
page_node.Render(visioDoc);

foreach (var s in page_node.Shapes)
{
    if (s is VADOM.Rectangle r && r.VisioShape != null)
    {
        // Drop into the imperative API for anything DOM doesn't cover.
        r.VisioShape.Text = "post-render touch-up";
    }
}
```

## Render performance

While a `Page` renders, it temporarily changes four Visio application settings to make rendering faster, then restores your original values afterward.

The settings are on each `Page`'s read-only `RenderPerformanceSettings` property. Each one is nullable, and `null` means leave that setting alone. A new `Page` starts with these values:

* **`DeferRecalc`**: Visio's `Application.DeferRecalc`, which decides whether Visio recalculates cell formulas during a series of actions.
  * Type: `short?`
  * Default: `0`
  * `0`: Visio recalculates formulas as needed.
  * Any nonzero value: Visio defers recalculating formulas until the render is finished.
* **`ScreenUpdating`**: Visio's `Application.ScreenUpdating`, which decides whether the window is redrawn during a series of actions.
  * Type: `short?`
  * Default: `1`
  * `1` (any nonzero value): Visio redraws the window as normal. This is the default because turning screen updating off can break page resizing.
  * `0`: Visio does not redraw the window while the page renders.
* **`EnableAutoConnect`**: Visio's `Application.Settings.EnableAutoConnect`, which turns Visio's AutoConnect feature on or off. Visio itself has it on by default.
  * Type: `bool?`
  * Default: `false`
  * `false`: AutoConnect is off while the page renders.
  * `true`: AutoConnect is on.
* **`LiveDynamics`**: Visio's `Application.LiveDynamics`, which decides how often Visio recalculates shape properties during drag operations.
  * Type: `bool?`
  * Default: `false`
  * `false`: Visio recalculates only after the mouse button is released.
  * `true`: Visio recalculates on every mouse move, which raises more events such as `CellChanged`. Add-ins that respond to those events can run faster with `false`.

Each of these is documented in Microsoft's Visio reference: [`DeferRecalc`](https://learn.microsoft.com/en-us/office/vba/api/Visio.Application.DeferRecalc), [`ScreenUpdating`](https://learn.microsoft.com/en-us/office/vba/api/Visio.Application.ScreenUpdating), [`EnableAutoConnect`](https://learn.microsoft.com/en-us/office/vba/api/Visio.ApplicationSettings.EnableAutoConnect) and [`LiveDynamics`](https://learn.microsoft.com/en-us/office/vba/api/visio.application.livedynamics).

To change one, set it before you call `Render`:

```csharp
var page_node = new VADOM.Page();
page_node.RenderPerformanceSettings.DeferRecalc = 1;   // adjust before Render
```

Only `Page.Render` applies these settings, so [`ShapeList.Render`](#where-the-output-goes) does not.

## See also

* [Drawing primitives](../extensions/drawing.md) (the imperative-style equivalent)
* [Custom properties](../custom-properties.md) (formula-vs-literal, typed setters)
* [Shape cells](../shape-cells.md) (the cell vocabulary used by the `Cells` property on each node)
* [Layout models](layouts.md) (algorithmic placement on top of the DOM)
* [Directed graph layout model](directed-graph.md) (graph-shaped diagrams via MSAGL)
* [Org chart model](org-charts.md) (turn-key org-chart generator over the DOM)
