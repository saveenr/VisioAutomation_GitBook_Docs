# Layouts: Tree, Grid, and Box

The `VisioAutomation.Models.Layouts` namespace adds **algorithmic placement** on top of the [DOM](dom.md). Instead of specifying X/Y for every shape, you describe the structure (a tree of nodes, a rows-by-columns grid, a packed box layout) and the layout engine assigns coordinates and connects shapes for you.

There are three general-purpose layouts in this namespace plus a directed-graph layout that wraps Microsoft Automatic Graph Layout (MSAGL) and is documented separately on the [Directed graph](directed-graph.md) page.

| Layout | Namespace | Best for |
| :--- | :--- | :--- |
| Tree | `VisioAutomation.Models.Layouts.Tree` | Hierarchies with one root, parent-child edges only. |
| Grid | `VisioAutomation.Models.Layouts.Grid` | Uniform rows and columns of identical shapes (heatmaps, calendars, lattice diagrams). |
| Box | `VisioAutomation.Models.Layouts.Box` | Nested rectangles with directional packing (UML-style box diagrams, treemaps). |
| Directed graph | `VisioAutomation.Models.Layouts.DirectedGraph` | General graphs with cycles, multiple roots, or non-tree edges. See [Directed graph](directed-graph.md). |

All four ultimately produce a Visio drawing on a target page. The differences are in the input data structure and the geometry algorithm.

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

Each `Node` exposes `VisioShape` after rendering, so you can pull the COM object back out for follow-up formatting. Per-node size, custom master, and direction are all configurable through `LayoutOptions`.

By default the layout uses Visio's "Rectangle" master from `basic_u.vss` for nodes and "Dynamic Connector" from `connec_u.vss` for edges; both are overridable.

## Grid layout

Use when the data is a uniform rectangular grid of identical shapes. The layout takes a master, a cell count, and a cell size, and drops the same master at every grid cell.

```csharp
using GRID = VisioAutomation.Models.Layouts.Grid;
using VA = VisioAutomation;

int cols = 3;
int rows = 6;
var cellsize = new VA.Core.Size(0.5, 0.25);

var grid = new GRID.GridLayout(cols, rows, cellsize, rectMaster);
grid.Origin = new VA.Core.Point(0, 4);
grid.Render(visioPage);
```

Each grid cell renders as one shape instance; the result is `cols * rows` shapes on the page. `Row` and `Column` accessors let you tweak per-row or per-column properties before render if uniformity isn't quite enough; see the source for the row-direction and column-direction enums controlling growth direction.

## Box layout

Use when the data is a tree of rectangular regions packed in a particular direction (left-to-right, top-to-bottom, etc.) inside a parent rectangle. The output is positioned rectangles; you can render them as Visio shapes or use the resulting geometry directly.

The model is a tree of `Container` nodes, where each container has a `Direction` (the axis along which its children pack) and a list of children. Each child is either another `Container` (for nesting) or a `Box` (a leaf rectangle of a given size).

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
// n1.Rectangle = (0.5, 0.5, 2.5, 1.5)
// n2.Rectangle = (2.5, 0.5, 5.5, 1.5)
```

Containers can nest: a child container packs its own children along its own direction, and the parent treats it as a single rectangle whose size is the bounding box of its packed contents. This is what makes the layout treemap-style.

`PerformLayout()` is computational only; it doesn't talk to Visio. To render, walk the tree and emit DOM shapes (or use the rectangles for any other purpose, e.g. a JPEG or SVG). The separation makes Box layout useful for non-Visio output too.

The companion [`Container`](https://github.com/saveenr/VisioAutomation/tree/master/VisioAutomation_2010/VisioAutomation.Models/Layouts/Container) layout (under `VisioAutomation.Models.Layouts.Container`) is a thin wrapper specifically for laying out Visio's container shapes; for general use the `Box` family is the right entry point.

## Default masters and stencils

All three layouts default to opening `basic_u.vss` for shape masters and `connec_u.vss` for connector masters. If your Visio install ships a different stencil set (e.g. localised Visio or a non-standard install path) the layouts let you override the stencil and master names per layout.

## See also

* [Declarative DOM](dom.md) (the underlying shape model the layouts emit into)
* [Layout styles](layout-styles.md) (`LayoutStyleBase` and its subclasses, used by directed graph and other style-driven renderers)
* [Directed graph](directed-graph.md) (general graphs via MSAGL, for non-tree edges)
* [Org charts](org-charts.md) (turn-key org-chart generator built on top of these layouts)
