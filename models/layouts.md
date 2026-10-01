# Layouts: Tree, Grid, Box, and Container

The `VisioAutomation.Models.Layouts` namespace adds **algorithmic placement** on top of the [DOM](dom.md). Instead of specifying X/Y for every shape, you describe the structure (a tree of nodes, a rows-by-columns grid, packed boxes, columns of container items) and the layout engine assigns coordinates and connects shapes for you.

There are four general-purpose layouts in this namespace plus a directed-graph layout that wraps Microsoft Automatic Graph Layout (MSAGL) and is documented separately on the [Directed graph](directed-graph.md) page.

| Layout | Namespace | Best for |
| :--- | :--- | :--- |
| Tree | `VisioAutomation.Models.Layouts.Tree` | Hierarchies with one root, parent-child edges only. |
| Grid | `VisioAutomation.Models.Layouts.Grid` | Uniform rows and columns of identical shapes (heatmaps, calendars, lattice diagrams). |
| Box | `VisioAutomation.Models.Layouts.Box` | Nested rectangles with directional packing. Geometry only. |
| Container | `VisioAutomation.Models.Layouts.Container` | Side-by-side columns of labelled items, each column wrapped in a Visio container shape. |
| Directed graph | `VisioAutomation.Models.Layouts.DirectedGraph` | General graphs with cycles, multiple roots, or non-tree edges. See [Directed graph](directed-graph.md). |

Tree, Grid, Container and directed graph each render a Visio drawing. Box is geometry only: it computes rectangles and leaves drawing to you. The differences are in the input data structure and the geometry algorithm.

## Tree layout

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

## Grid layout

Use when the data is a uniform rectangular grid of identical shapes. The layout takes a column count, a row count, a cell size and a master (an `IVisio.Master` you have already loaded), and drops that master at every grid cell.

```csharp
using GRID = VisioAutomation.Models.Layouts.Grid;
using VA = VisioAutomation;

int cols = 3;
int rows = 6;
var cellsize = new VA.Core.Size(0.5, 0.25);

var grid = new GRID.GridLayout(cols, rows, cellsize, rectMaster);
grid.Origin = new VA.Core.Point(0, 4);
grid.PerformLayout();
grid.Render(visioPage);
```

`PerformLayout()` must be called before `Render`, because it computes each node's rectangle and `Render` reads those rectangles. Each grid cell renders as one shape instance; the result is `cols * rows` shapes on the page. The `Rows` and `Columns` lists let you tweak per-row `Height` and per-column `Width` before layout if uniformity isn't quite enough (both throw if set to zero or less). `CellSpacing` (default 0.5 x 0.25) sets the gaps, `ColumnDirection` (`LeftToRight` by default, or `RightToLeft`) and `RowDirection` (`BottomToTop` by default, or `TopToBottom`) control growth direction, and `GetNode(col, row)` returns a node whose `Text`, `Cells` and `Draw` (default `true`) you can set.

## Box layout

Use when the data is a tree of rectangular regions packed in a particular direction (left-to-right, top-to-bottom, etc.) inside a parent rectangle. The output is positioned rectangles. The layout has no Visio rendering of its own: you walk its `Nodes` and emit DOM shapes (or anything else) from the rectangles yourself.

The model is a tree of `Container` nodes, where each container has a `Direction` (the axis along which its children pack) and a list of children. Each child is either another `Container` (for nesting) or a `Box` (a leaf rectangle of a given size). Each container has `PaddingLeft`, `PaddingRight`, `PaddingTop` and `PaddingBottom` (all 0.125 by default) and a `ChildSpacing` (also 0.125 by default) inserted between adjacent children.

`Direction` takes one of four values. It sets both the axis children pack along and the edge the first child starts against:

| Direction | Axis | First child is placed |
| --- | --- | --- |
| `LeftToRight` | horizontal | at the left edge, with later children to its right |
| `RightToLeft` | horizontal | at the right edge, with later children to its left |
| `BottomToTop` | vertical | at the bottom edge, with later children above it |
| `TopToBottom` | vertical | at the top edge, with later children below it |

In a horizontal container, a child shorter than the container is positioned by its `VAlignToParent` (`Top`, `Center` or `Bottom`; default `Top`). In a vertical container, a child narrower than the container is positioned by its `HAlignToParent` (`Left`, `Center` or `Right`; default `Left`).


```csharp
using VABOX = VisioAutomation.Models.Layouts.Box;

var layout = new VABOX.BoxLayout();
layout.Root = new VABOX.Container(VABOX.Direction.LeftToRight);

var n1 = layout.Root.AddBox(2, 1);
var n2 = layout.Root.AddBox(3, 1);

layout.Root.PaddingLeft = 0.5;
layout.Root.PaddingRight = 0.5;
layout.Root.PaddingTop = 0.5;
layout.Root.PaddingBottom = 0.5;

layout.PerformLayout();

// After PerformLayout, every node has a populated Rectangle.
// Rectangles are (left, bottom, right, top).
// With the default ChildSpacing of 0.125 between the two boxes:
// n1.Rectangle = (0.5, 0.5, 2.5, 1.5)
// n2.Rectangle = (2.625, 0.5, 5.625, 1.5)
// layout.Root.Rectangle = (0, 0, 6.125, 2.0)
```

Containers can nest: a child container packs its own children along its own direction, and the parent treats it as a single rectangle whose size is the bounding box of its packed contents. The layout is a nested stack-and-pad packer; it does not size boxes proportionally the way a treemap does.

Nested `RightToLeft` containers were fixed in NuGet 3.1.0 ([#202](https://github.com/saveenr/VisioAutomation/issues/202)). In 3.0.0 and earlier, a `RightToLeft` container placed anywhere other than the root misplaces its children whenever its origin Y differs from its X, for example one nested inside a vertical container. The root container is always placed at (0, 0), so it was not affected.

`PerformLayout()` is computational only; it doesn't talk to Visio. To render, walk the tree and emit DOM shapes (or use the rectangles for any other purpose, e.g. a JPEG or SVG). The separation makes Box layout useful for non-Visio output too.

## Container layout

Use when you want several labelled columns of items, each column wrapped in a Visio container shape. `ContainerLayout` is independent of the Box layout: it arranges one column per container, with the container's items stacked top to bottom inside it. (`Layouts.Container.Container` and `Layouts.Box.Container` are unrelated types that happen to share a name.)

```csharp
using VACONT = VisioAutomation.Models.Layouts.Container;
using IVisio = Microsoft.Office.Interop.Visio;

var layout = new VACONT.ContainerLayout();
var c1 = layout.AddContainer("Fruit");
c1.Add("Apple");
c1.Add("Pear");
var c2 = layout.AddContainer("Vegetables");
c2.Add("Carrot");

layout.PerformLayout();
IVisio.Page page = layout.Render(visioDoc);
```

`Render` takes an `IVisio.Document`, adds a new page to it and returns that page. Calling `Render` before `PerformLayout()` throws an `ArgumentException`.

`layout.LayoutOptions` controls the geometry: `ItemWidth` (2.0), `ItemHeight` (0.25), `Padding` (0.125), `ContainerHeaderHeight` (0.25), `ContainerHorizontalDistance` (1.0) and `ItemVerticalSpacing` (0.125). The masters are public fields: `ManualItemMaster` (default "Rounded Rectangle") and `ManualContainerMaster` (default "Rectangle") are the ones `Render` drops, and `ContainerMaster` (default "Container 1") is also exposed.

## Default masters and stencils

The tree layout opens `basic_u.vss` for shape masters and `connec_u.vss` for connector masters, and neither is configurable. The grid layout uses whatever `IVisio.Master` you pass to its constructor. The box layout draws nothing. Only the container layout lets you override master names, through the public fields on its `LayoutOptions` (plus `ManualItemStencil`, which defaults to `basic_u.vss`).

## See also

* [Declarative DOM](dom.md) (the underlying shape model the layouts emit into)
* [Layout styles](layout-styles.md) (`LayoutStyleBase` and its subclasses, used by directed graph and other style-driven renderers)
* [Directed graph](directed-graph.md) (general graphs via MSAGL, for non-tree edges)
* [Org charts](org-charts.md) (turn-key org-chart generator that shares the internal tree-layout engine rather than the public Tree API)
