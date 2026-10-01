# Layout styles

`VisioAutomation.Models.LayoutStyles` is a thin wrapper over Visio's built-in **page-level layout** feature: the same one accessible from Visio's *Design* tab as "Re-Layout Page" with its choice of style. The wrapper exposes the layouts as typed C# classes so you can configure them programmatically and apply them to a page in one call.

This is distinct from the algorithmic [`Layouts/`](layouts.md) namespace. The `Layouts/` types compute coordinates client-side and emit shapes; `LayoutStyles/` instead **configures Visio** to do the layout itself by writing the relevant cells on the page sheet (`AvenueSizeX`, `AvenueSizeY`, `LineRouteExt`, `PlaceStyle`, and in most cases `RouteStyle`) and calling `Page.Layout()` once. The result is Visio's own layout of the page rather than coordinates computed by this library.

When in doubt: use `Layouts/` when you want the library to compute and emit the shapes; use `LayoutStyles/` when you want Visio's own layout engine to place existing shapes.

## The base class

Every layout style inherits from `LayoutStyleBase`, which exposes four cross-cutting properties. Each subclass also writes the `PlaceStyle` cell, which selects the layout type, and that is why the styles differ.

| Property | Type | Maps to | Default |
| :--- | :--- | :--- | :--- |
| `ConnectorStyle` | `ConnectorStyle` | `RouteStyle` cell (see limits below) | (style-specific, see the styles table) |
| `ConnectorAppearance` | `ConnectorAppearance` | `LineRouteExt` cell | `Default` |
| `AvenueSizeX` | `double` | `AvenueSizeX` cell | `0.375` |
| `AvenueSizeY` | `double` | `AvenueSizeY` cell | `0.375` |

`Apply(page)` is the single entry point: it writes the cells to the target page's `PageSheet` and calls `page.Layout()` once. After `Apply` returns, Visio has laid out the page using the configured style.

## The styles

Five concrete styles ship in the box. Each maps to one of Visio's named auto-layout modes.

| Style | Class | Extra properties | Default `ConnectorStyle` |
| :--- | :--- | :--- | :--- |
| Hierarchy | `HierarchyLayoutStyle` | `LayoutDirection` (default `BottomToTop`, the enum's first value), `HorizontalAlignment` (default `Center`), `VerticalAlignment` (default `Middle`) | `OrganizationChart` |
| Flowchart | `FlowchartLayoutStyle` | `LayoutDirection` (default `TopToBottom`) | `Flowchart` |
| Compact tree | `CompactTreeLayout` | `Direction` (a `CompactTreeDirection` enum, default `DownThenRight`) | `OrganizationChart` |
| Circular | `CircularLayoutStyle` | (base properties only) | `CenterToCenter` |
| Radial | `RadialLayoutStyle` | (base properties only) | `RightAngle` |

`LayoutDirection` is a four-value enum: `BottomToTop`, `TopToBottom`, `LeftToRight`, `RightToLeft`. `CompactTreeDirection` is a different, eight-value enum that names a primary and a secondary direction: `DownThenLeft`, `DownThenRight`, `UpThenLeft`, `UpThenRigtht` (the misspelling is in the source), `LeftThenDown`, `LeftThenUp`, `RightThenDown` and `RightThenUp`.

## Hello-world

Apply a hierarchy layout to the active page, top-to-bottom, with org-chart connector routing:

```csharp
using VALAY = VisioAutomation.Models.LayoutStyles;

var style = new VALAY.HierarchyLayoutStyle();
style.LayoutDirection = VALAY.LayoutDirection.TopToBottom;
style.HorizontalAlignment = VALAY.HorizontalAlignment.Center;
style.VerticalAlignment = VALAY.VerticalAlignment.Middle;
style.ConnectorStyle = VALAY.ConnectorStyle.OrganizationChart;
style.ConnectorAppearance = VALAY.ConnectorAppearance.Default;

style.Apply(visioPage);
```

`ConnectorStyle.OrganizationChart` plus `LayoutDirection.TopToBottom` together produce Visio's classic top-down org-chart routing.

## Picking a `ConnectorStyle`

`ConnectorStyle` controls the abstract routing logic Visio uses; it determines the `RouteStyle` cell value. The ten values are:

| Value | When to use |
| :--- | :--- |
| `Flowchart` | Flowchart-style routing (turns at axis-aligned angles, packs flow direction). |
| `OrganizationChart` | Org-chart-style routing (vertical mainlines with horizontal sibling spurs). |
| `Simple` | Simple axis-aligned routing without flowchart-specific quirks. |
| `RightAngle` | Force right-angle (Manhattan) routing regardless of layout. |
| `Straight` | Force straight lines. |
| `CenterToCenter` | Connect shape centers; ignores edge points. |
| `Network` | Mesh-network-style routing. |
| `SimpleHorizontalVertical` | Simple routing that runs horizontal then vertical. |
| `SimpleVerticalHorizontal` | Simple routing that runs vertical then horizontal. |
| `Tree` | Tree-style routing. |

For `Flowchart`, `OrganizationChart`, and `Simple` on the hierarchy and flowchart styles, the chosen `LayoutDirection` further specialises the cell value (Visio has separate cell values for each direction, e.g. `visLORouteFlowchartNS` vs. `visLORouteFlowchartWE`).

There are limits on what is written:

* `RightAngle`, `Straight`, `CenterToCenter` and `Network` write `RouteStyle` on every style.
* On `CompactTreeLayout`, `CircularLayoutStyle` and `RadialLayoutStyle`, the other values (`OrganizationChart`, `Flowchart`, `Simple`, `SimpleHorizontalVertical`, `SimpleVerticalHorizontal`, `Tree`) map to nothing, so `RouteStyle` is simply not written.
* On `HierarchyLayoutStyle` and `FlowchartLayoutStyle`, `SimpleHorizontalVertical`, `SimpleVerticalHorizontal` and `Tree` throw `ArgumentOutOfRangeException`.

## `ConnectorAppearance`

This controls what existing connectors look like after re-layout. Values: `Default` (sets the default routing, `visLORouteExtDefault`), `Straight` (force straight segments), `Curved` (force NURBS-style curves). It maps to the `LineRouteExt` page cell.

## When to apply

`Apply()` overwrites the layout cells (`PlaceStyle`, `LineRouteExt`, the avenue cells and, where applicable, `RouteStyle`) on the page sheet and then calls `Page.Layout()` once, which is what moves the shapes.

The typical pattern is:

1. Drop or generate shapes (e.g. via [DOM](dom.md) or [Layouts](layouts.md) helpers).
2. Choose and configure a `LayoutStyleBase` subclass.
3. Call `Apply(page)`.

If you re-apply with a different style later, the cells written by both are overwritten with the new values. A cell the new style doesn't write (for example `RouteStyle` on a compact tree) keeps its earlier value.

## Where layout styles are used

A layout style is not tied to a particular layout type. It is applied to a page, so anything that works with a page can use one:

* **[Declarative DOM](dom.md):** set `Layout` on a DOM `Page` node. `Render` draws the shapes, then applies the style, then resizes the page if you asked it to.
* **VisioScripting:** `client.Page.LayoutPage(TargetPages, LayoutStyleBase)` applies a style to the resolved pages in one undo step. See [client.Page](../visio-scripting/page.md).
* **PowerShell:** the [`Format-VisioPage`](https://saveenr.gitbook.io/visiopowershell/cmdlets/pages/format-visiopage) cmdlet takes a style object through its `-LayoutStyle` parameter.
* **Directed graph, Visio-based renderer:** `VisioLayoutRenderer` lays out a [directed graph](directed-graph.md#laying-out-with-visio-instead-of-msagl) with a layout style instead of MSAGL. Its `VisioLayoutOptions.VisioLayoutStyle` defaults to a top-to-bottom flowchart. The `MsaglRenderer` that VisioScripting and PowerShell use does not use layout styles, so this is the only layout type that does.

## See also

* [Layouts](layouts.md) (algorithmic placement: Tree, Grid, Box, Container and directed graph)
* [Directed graph](directed-graph.md) (graph-shaped layout via MSAGL)
* [Declarative DOM](dom.md) (`Page.Layout` applies a style when a DOM page renders)
* [client.Page](../visio-scripting/page.md) (`LayoutPage` applies a style to pages)
* [Page cells](../page-cells.md) (the `PageLayoutCells` and related cell groups that this namespace writes to)
