# Directed graph XML format

VisioAutomation can render a directed graph (nodes connected by edges, automatically laid out via MSAGL's Sugiyama layered algorithm) from a compact XML format. The same XML is consumed by two entry points:

* The PowerShell pipeline `Import-VisioModel some.xml | Out-VisioApplication`, where the `Import-VisioModel` cmdlet returns a `DirectedGraphDocument`.
* The .NET method `client.Model.LoadDirectedGraphFromXml(xmldoc)`, called through the public `VisioScripting.Client` facade (NuGet 3.1.0 and later).

The facade loader, `Client.Model.LoadDirectedGraphFromXml`, was added in NuGet 3.1.0. With 3.0.0, use `VisioScripting.Loaders.DirectedGraphDocumentLoader.LoadFromXml(client, xmldoc)`. That loader class is `internal` from 3.1.0 on, so new code should use the facade.

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

* one `<renderoptions>` element (see below),
* one `<shapes>` container with `<shape>` children,
* one `<connectors>` container with `<connector>` children.

All three child elements are required. A missing one makes the loader throw a `NullReferenceException`; an empty `<connectors></connectors>` is fine. Element and attribute names are lowercase and case-sensitive. Only the enum values of `direction`, `connectortype` and `layout` are case-insensitive.

## `<renderoptions>`

Per-page rendering knobs.

| Attribute | Type | Required? | Default | Notes |
| --- | --- | --- | --- | --- |
| `usedynamicconnectors` | `bool` | yes | none | If `true`, connectors are dropped from the page's "Dynamic Connector" master and route around shapes. If `false`, MSAGL routes the edge geometry directly. The programmatic default does not apply to XML: omitting the attribute throws `ArgumentException`. |
| `scalingfactor` | `double` | yes | none | Scale applied around the MSAGL layout. Node sizes are multiplied by this value before layout and the results are divided by it afterward, so node sizes are unchanged in inches. MSAGL's own separation constants (node and layer separation, margins) are fixed in MSAGL units and are not scaled, so after the division they shrink to roughly (fixed value / factor) inches. A larger value therefore gives tighter gaps relative to node size and a smaller page; a smaller value gives looser spacing. Most in-repo fixtures use `20`; one uses `5`. Omitting the attribute throws `ArgumentException` (the programmatic default of `14` does not apply to XML). |
| `direction` | `TopToBottom` \| `BottomToTop` \| `LeftToRight` \| `RightToLeft` | no | `TopToBottom` | Which way the graph flows. Case-insensitive. |
| `connectortype` | `Curved` \| `Straight` \| `RightAngle` | no | `Curved` | Connector style applied to every edge on the page. Case-insensitive. Per-edge override is not supported. |
| `layerseparation` | `double` | no | none (MSAGL's default) | Minimum distance in inches between layers (rows for `TopToBottom`, columns for `LeftToRight`). A non-numeric value throws `FormatException`. |
| `edgelabelboxwidth` | `double` | no | `1.0` | Width in inches reserved for each edge's label, for every edge whether or not it has a label. |
| `edgelabelboxheight` | `double` | no | `0.5` | Height in inches reserved for each edge's label. Smaller values give tighter gaps between layers. |
| `layout` | `Sugiyama` | no | none | Parsed only if present, then discarded. Currently only `Sugiyama` is accepted; any other value raises `ArgumentException`. The attribute exists so that future layout algorithms can be opted into without breaking existing XML. |

`layerseparation`, `edgelabelboxwidth` and `edgelabelboxheight` were added in NuGet 3.1.0; older packages ignore them. If you set only one of the two label box attributes, the other keeps its default. For what they do, see [Tightening the layout](models/directed-graph.md#tightening-the-layout).

```xml
<renderoptions usedynamicconnectors="false" scalingfactor="20"
               direction="LeftToRight" layerseparation="0.25"
               edgelabelboxwidth="0.8" edgelabelboxheight="0.12" />
```

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

Each `<customprop>` is added to the dropped shape's custom-properties section using `name` as the row name and `value` as the value. The loader stores it as a string-typed custom property (the value is quoted in the cell). Only a value that begins with `=` is kept as a raw formula. Both `name` and `value` are required.

## `<connector>` (under `<connectors>`)

Each connector becomes one edge in the graph.

| Attribute | Required? | Notes |
| --- | --- | --- |
| `id` | yes | Unique within the page. |
| `from` | yes | Source shape's `id`. Must match a `<shape>` in the same page (see the failure modes below). |
| `to` | yes | Destination shape's `id`. Must match a `<shape>` in the same page (see the failure modes below). |
| `label` | yes | Text rendered on the connector. May be empty. |
| `color` | no | Web color (for example `#ff0000`). Defaults to black. An invalid value throws `FormatException`. |
| `weight` | no | Line weight in points (the loader converts to inches). Defaults to `1`. |

Every connector drawn from XML gets an end arrow; there is no attribute to change this.

### Failure modes

Validation happens at render time, not at load time:

* A `from` or `to` that matches no shape in the page loads without error, but rendering throws `ArgumentException` ("Connector's From/To node is null"). The loader notes such problems only in verbose output.
* Duplicate shape ids throw `ArgumentException` while the document is loaded.
* Duplicate connector ids throw when the edge is added to the graph.

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
