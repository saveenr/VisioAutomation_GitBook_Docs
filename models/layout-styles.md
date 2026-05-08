# Layout styles

`VisioAutomation.Models.LayoutStyles` is a thin wrapper over Visio's built-in **page-level layout** feature: the same one accessible from Visio's *Design* tab as "Re-Layout Page" with its choice of style. The wrapper exposes the layouts as typed C# classes so you can configure them programmatically and apply them to a page in one call.

This is distinct from the algorithmic [`Layouts/`](layouts.md) namespace. The `Layouts/` types compute coordinates client-side and emit shapes; `LayoutStyles/` instead **configures Visio** to do the layout itself by writing the relevant cells on the page sheet (`AvenueSizeX`, `AvenueSizeY`, `RouteStyle`, `LineRouteExt`) and calling `Page.Layout()`. The result is a page that Visio re-flows whenever the user moves shapes around, not a one-shot static rendering.

When in doubt: use `Layouts/` for one-shot output; use `LayoutStyles/` when you want Visio's interactive auto-layout behavior to keep working post-render.

## The base class

Every layout style inherits from `LayoutStyleBase`, which exposes four cross-cutting properties:

| Property | Type | Maps to | Default |
| :--- | :--- | :--- | :--- |
| `ConnectorStyle` | `ConnectorStyle` | `RouteStyle` cell | (style-specific) |
| `ConnectorAppearance` | `ConnectorAppearance` | `LineRouteExt` cell | `Default` |
| `AvenueSizeX` | `double` | `AvenueSizeX` cell | `0.375` |
| `AvenueSizeY` | `double` | `AvenueSizeY` cell | `0.375` |

`Apply(page)` is the single entry point: it writes the cells to the target page's `PageSheet` and triggers a `Layout()` call. After `Apply` returns, Visio has re-flowed the page using the configured style.

## The styles

Five concrete styles ship in the box. Each maps to one of Visio's named auto-layout modes.

| Style | Class | Extra properties |
| :--- | :--- | :--- |
| Hierarchy | `HierarchyLayoutStyle` | `LayoutDirection`, `HorizontalAlignment`, `VerticalAlignment` |
| Flowchart | `FlowchartLayoutStyle` | `LayoutDirection` |
| Compact tree | `CompactTreeLayout` | `Direction` (a `CompactTreeDirection` enum, distinct from `LayoutDirection`) |
| Circular | `CircularLayoutStyle` | (base properties only) |
| Radial | `RadialLayoutStyle` | (base properties only) |

`LayoutDirection` is a four-value enum: `TopToBottom`, `BottomToTop`, `LeftToRight`, `RightToLeft`. `CompactTreeDirection` is structurally similar but typed separately because Visio treats compact-tree direction as a different enum internally.

## Hello-world

Apply a hierarchy layout to the active page, top-to-bottom, with right-angle connectors:

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

`ConnectorStyle` controls the abstract routing logic Visio uses; it determines the `RouteStyle` cell value. Common values:

| Value | When to use |
| :--- | :--- |
| `Flowchart` | Flowchart-style routing (turns at axis-aligned angles, packs flow direction). |
| `OrganizationChart` | Org-chart-style routing (vertical mainlines with horizontal sibling spurs). |
| `Simple` | Simple axis-aligned routing without flowchart-specific quirks. |
| `RightAngle` | Force right-angle (Manhattan) routing regardless of layout. |
| `Straight` | Force straight lines. |
| `CenterToCenter` | Connect shape centers; ignores edge points. |
| `Network` | Mesh-network-style routing. |

For `Flowchart`, `OrganizationChart`, and `Simple`, the chosen `LayoutDirection` further specialises the cell value (Visio has separate cell values for each direction, e.g. `visLORouteFlowchartNS` vs. `visLORouteFlowchartWE`).

## `ConnectorAppearance`

This controls what existing connectors look like after re-layout. Values: `Default` (leave alone), `Straight` (force straight segments), `Curved` (force NURBS-style curves). It maps to the `LineRouteExt` page cell.

## When to apply

`Apply()` is destructive in the sense that it overwrites `RouteStyle` / `LineRouteExt` / avenue cells on the page sheet. The shapes themselves don't move until you call `Page.Layout()` (which `Apply` does for you), but the page-sheet cells change immediately.

The typical pattern is:

1. Drop or generate shapes (e.g. via [DOM](dom.md) or [Layouts](layouts.md) helpers).
2. Choose and configure a `LayoutStyleBase` subclass.
3. Call `Apply(page)`.

If you re-apply with a different style later, the previous style's cells are overwritten, so styles don't accumulate.

## See also

* [Layouts](layouts.md) (algorithmic placement: Tree, Grid, Box)
* [Directed graph](directed-graph.md) (graph-shaped layout via MSAGL)
* [Page cells](../page-cells.md) (the `PageLayoutCells` and related cell groups that this namespace writes to)
