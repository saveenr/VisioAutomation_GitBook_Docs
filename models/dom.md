# Declarative DOM

The **Document Object Model** under `VisioAutomation.Models.Dom` is the highest-level authoring API in the library. Build an in-memory tree of plain objects describing the diagram you want, then call `Render()` to materialize it as actual Visio shapes in one batch. The model decouples diagram authoring from per-shape COM bookkeeping, makes diagrams composable from helpers and loops, and produces a single undo step rather than dozens.

If you are looking for the imperative, COM-style equivalent, see the [Drawing primitives](../extensions/drawing.md) page; the DOM is built on the same primitives but stitches them together into a tree.

## The shape of the tree

```
Document
   |- Pages : PageList
            |- Page
                  |- Shapes : ShapeList
                            |- Shape  (instance of a master)
                            |- Rectangle
                            |- Oval
                            |- Line
                            |- PolyLine
                            |- BezierCurve
                            |- Connector
```

Each node carries the data needed to materialize itself plus a `Render()` method. `Shape` instances reference a master by name or by `IVisio.Master`. The geometric primitives (`Rectangle`, `Oval`, `Line`, `PolyLine`, `BezierCurve`) carry their own coordinates and don't need a master.

A render happens at any level: rendering a `Document` creates a new Visio document; rendering a `Page` adds a page to an existing document; rendering a `ShapeList` drops shapes into an already-open page. Whichever level you start from, the children render too.

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
var page_node = new VADOM.Page();

// By master object (master already resolved)
var stencil = visioDoc.Application.Documents.OpenStencil("basic_u.vss");
var rectMaster = stencil.Masters["Rectangle"];
page_node.Shapes.Drop(rectMaster, 3, 3);

// By name (resolved at render time)
page_node.Shapes.Drop("Rectangle", "basic_u.vss", 5, 5);

page_node.Render(visioDoc);
```

`ShapeList` exposes convenience helpers (`Drop`, `DrawRectangle`, `DrawLine`, `DrawBezier`, `Connect`) that mirror the most common imperative drawing operations from [Drawing primitives](../extensions/drawing.md).

## Connectors

Edges between shapes become `Connector` nodes inside the same `ShapeList`. The shorthand `Shapes.Connect(masterName, stencilName, fromShape, toShape)` packages the three steps (create connector, glue from end, glue to end) into one call.

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

The connector's source and target are tracked as references to the DOM `Shape` nodes, not their Visio IDs, so the connector can be specified before either endpoint has been rendered.

## Custom properties on DOM shapes

Every shape node exposes a `CustomProperties` dictionary that's applied during render. Each entry is a [`CustomPropertyCells`](../custom-properties.md), so the same typed-setter idioms apply.

```csharp
var rect = new VADOM.Rectangle(1, 1, 4, 4);
rect.CustomProperties = new VisioAutomation.Shapes.CustomPropertyDictionary();

var owner = new VisioAutomation.Shapes.CustomPropertyCells();
owner.SetString("Alex");          // typed setter handles formula encoding
owner.Label = "\"Owner\"";        // pre-encoded literal

rect.CustomProperties["Owner"] = owner;
```

For the formula-vs-literal distinction and the typed setters, see the [Custom properties](../custom-properties.md) page.

## Rendering at three levels

`Render()` is overloaded so the same node tree can target different containers:

```csharp
// Render a ShapeList directly into an already-open page
shape_list.Render(visioPage);

// Render a Page node, creating a new page in an existing document
var newPage = page_node.Render(visioDoc);

// Render a Document node, creating a brand-new Visio document
var doc_node = new VADOM.Document();
doc_node.Pages.Add(page_node);
var newDoc = doc_node.Render(visioApp);
```

`VADOM.Document` also accepts a template filename and measurement system in its constructor, so a render can start from a Visio template (`.vst` / `.vstx`) instead of a blank document.

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

DOM rendering batches the COM operations behind a single undo scope, which is already a substantial speedup over per-shape imperative drops. For large diagrams the `RenderPerformanceSettings` and `RenderPerformanceScope` types let you opt into additional optimisations (suppressed redraw, batched cell writes). They are off by default; turn them on if you observe rendering taking noticeable time on your dataset.

## See also

* [Drawing primitives](../extensions/drawing.md) (the imperative-style equivalent)
* [Custom properties](../custom-properties.md) (formula-vs-literal, typed setters)
* [Shape cells](../shape-cells.md) (the cell vocabulary used by the `Cells` property on each node)
* [Layouts](layouts.md) (algorithmic placement on top of the DOM)
* [Directed graph](directed-graph.md) (graph-shaped diagrams via MSAGL)
* [Org charts](org-charts.md) (turn-key org-chart generator over the DOM)
