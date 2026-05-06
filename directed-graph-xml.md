# Directed graph XML format

VisioAutomation can render a directed graph (nodes connected by edges, automatically laid out via MSAGL's Sugiyama layered algorithm) from a compact XML format. The same XML is consumed by two entry points:

* The PowerShell pipeline `Import-VisioModel some.xml | Out-VisioApplication`, where the `Import-VisioModel` cmdlet returns a `DirectedGraphDocument`.
* The .NET method `VisioScripting.Loaders.DirectedGraphDocumentLoader.LoadFromXml(client, xmldoc)`, called directly from C#.

The format is small enough to write by hand and is intentionally a separate file format from `.vsd` / `.vsdx`. Use it to declare a graph in source-controlled XML, then have VisioAutomation lay it out and emit the Visio drawing.

## Minimal example

```xml
<directedgraph>
  <page>
    <renderoptions usedynamicconnectors="true" scalingfactor="20" />
    <shapes>
      <shape id="n1" label="A" stencil="basic_u.vss" master="Rectangle" />
      <shape id="n2" label="B" stencil="basic_u.vss" master="Rectangle" />
      <shape id="n3" label="C" stencil="basic_u.vss" master="Rectangle" />
    </shapes>
    <connectors>
      <connector id="c1" from="n1" to="n2" label="" />
      <connector id="c2" from="n2" to="n3" label="" />
    </connectors>
  </page>
</directedgraph>
```

A handful of in-repo fixtures live at [`VisioAutomation_2010/VTest/datafiles/directed_graph_*.xml`](https://github.com/saveenr/VisioAutomation/tree/master/VisioAutomation_2010/VTest/datafiles).

## Root element: `<directedgraph>`

The root element must be `<directedgraph>`. Other root names raise `ArgumentException`. (Older releases accepted any root name and silently parsed only the `<page>` children, which is what bit the user in [issue #105](https://github.com/saveenr/VisioAutomation/issues/105).)

The root element takes no attributes. Layout settings live on each page's `<renderoptions>` element (see below).

## `<page>`

A `<directedgraph>` can contain one or more `<page>` elements. Each `<page>` becomes one Visio page in the output document.

A `<page>` contains exactly:

* one `<renderoptions>` element (optional content, see below),
* one `<shapes>` container with `<shape>` children,
* one `<connectors>` container with `<connector>` children.

## `<renderoptions>`

Per-page rendering knobs.

| Attribute | Type | Default | Notes |
| --- | --- | --- | --- |
| `usedynamicconnectors` | `bool` | required | If `true`, connectors are dropped from the page's "Dynamic Connector" master and route around shapes. If `false`, MSAGL routes the edge geometry directly. |
| `scalingfactor` | `double` | required | Multiplier on MSAGL's coordinates. Larger values produce more spread-out layouts. The in-repo fixtures use `20`. |
| `direction` | `TopToBottom` \| `BottomToTop` \| `LeftToRight` \| `RightToLeft` | `TopToBottom` | Which way the graph flows. Case-insensitive. |
| `connectortype` | `Curved` \| `Straight` \| `RightAngle` | `Curved` | Connector style applied to every edge on the page. Case-insensitive. Per-edge override is not supported. |
| `layout` | `Sugiyama` | (optional) | Currently only `Sugiyama` is accepted; any other value raises `ArgumentException`. The attribute exists so that future layout algorithms can be opted into without breaking existing XML. |

## `<shape>` (under `<shapes>`)

Each shape becomes one node in the directed graph.

| Attribute | Required? | Notes |
| --- | --- | --- |
| `id` | yes | Unique within the page. Connectors reference this id via `from` / `to`. |
| `label` | yes | Text rendered on the dropped shape. |
| `stencil` | yes | Filename of the stencil to load (for example `basic_u.vss`, `basflo_u.vss`, `server_u.vss`). |
| `master` | yes | Master name within that stencil (for example `Rectangle`, `Process`, `Server`). |
| `url` | no | If set, the dropped shape gets a hyperlink with this address. |

A `<shape>` can also contain `<customprop>` children:

```xml
<shape id="n4" label="Foo" stencil="server_u.vss" master="Web Server">
  <customprop name="prop1" value="value1" />
  <customprop name="prop2" value="value2" />
</shape>
```

Each `<customprop>` is added to the dropped shape's custom-properties section using `name` as the row name and `value` as the value formula.

## `<connector>` (under `<connectors>`)

Each connector becomes one edge in the graph.

| Attribute | Required? | Notes |
| --- | --- | --- |
| `id` | yes | Unique within the page. |
| `from` | yes | Source shape's `id`. Must match a `<shape>` in the same page. |
| `to` | yes | Destination shape's `id`. Must match a `<shape>` in the same page. |
| `label` | yes | Text rendered on the connector. May be empty. |
| `color` | no | Web color (for example `#ff0000`). Defaults to black. |
| `weight` | no | Line weight in points (the loader converts to inches). Defaults to `1`. |

## What's not supported in XML

The following can be set programmatically on the in-memory `DirectedGraphLayout` model but are not exposed in the XML schema:

* Per-edge `connectortype` override. The page-level setting applies to every edge.
* Per-shape size or styling beyond the master template.
* Layout algorithms other than Sugiyama.
* MSAGL's finer-grained layered-layout knobs (layer separation, edge routing strategy, etc.).

If you need any of these, load the XML, mutate the resulting `DirectedGraphDocument` from C#, then render via `Client.Model.DrawDirectedGraphDocument`.

## See also

* [Connectors](connectors.md): the lower-level helpers for connecting two shapes when you don't need automated layout.
* [Stencils and masters](stencils-and-masters.md): how the `stencil` and `master` attributes resolve at runtime.
