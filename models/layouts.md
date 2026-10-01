# Layout models

The `VisioAutomation.Models.Layouts` namespace adds **algorithmic placement**. Most of the layouts draw through the [DOM](dom.md); see [Introduction to models](introduction.md) for the exceptions. Instead of specifying X/Y for every shape, you describe the structure (a tree of nodes, a rows-by-columns grid, packed boxes, columns of container items) and the layout engine assigns coordinates and connects shapes for you.

There are three general-purpose layouts here plus a directed-graph layout that wraps Microsoft Automatic Graph Layout (MSAGL). The namespace also holds a Box type, but it only computes rectangles and draws nothing, so it is documented separately as [Box geometry](box-geometry.md).

| Layout | Namespace | Best for |
| :--- | :--- | :--- |
| [Tree](layouts-tree.md) | `VisioAutomation.Models.Layouts.Tree` | Hierarchies with one root, parent-child edges only. |
| [Grid](layouts-grid.md) | `VisioAutomation.Models.Layouts.Grid` | Uniform rows and columns of identical shapes (heatmaps, calendars, lattice diagrams). |
| [Container](layouts-container.md) | `VisioAutomation.Models.Layouts.Container` | Side-by-side columns of labelled items, each column drawn on a plain rectangle (not a Visio container shape). |
| [Directed graph layout model](directed-graph.md) | `VisioAutomation.Models.Layouts.DirectedGraph` | General graphs with cycles, multiple roots, or non-tree edges. |

Each of these renders a Visio drawing. The differences are in the input data structure and the geometry algorithm.

Each layout has its own page: [Tree](layouts-tree.md), [Grid](layouts-grid.md), [Container](layouts-container.md) and [Directed graph layout model](directed-graph.md).

## Default masters and stencils

The tree layout opens `basic_u.vss` for shape masters and `connec_u.vss` for connector masters, and neither is configurable. The grid layout uses whatever `IVisio.Master` you pass to its constructor. Only the container layout lets you override master names, through the public fields on its `LayoutOptions` (plus `ManualItemStencil`, which defaults to `basic_u.vss`).

## See also

* [Introduction to models](introduction.md) (what a model is, and which ones build on the DOM)
* [DOM](dom.md) (the underlying shape model that the tree, grid and directed graph layouts draw through)
* [Layout styles](layout-styles.md) (Visio's own page-level layout feature, applied to any page and separate from the layouts described here)
* [Directed graph layout model](directed-graph.md) (general graphs via MSAGL, for non-tree edges)
* [Org chart model](org-charts.md) (turn-key org-chart generator that shares the internal tree-layout engine rather than the public Tree API)
