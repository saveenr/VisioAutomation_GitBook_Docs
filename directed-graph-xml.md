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

The root element takes no attributes. Document-wide settings go in an optional `<documentoptions>` element (see below), and layout settings live on each page's `<renderoptions>` element.

## `<documentoptions>`

An optional element, placed directly under `<directedgraph>` before the `<page>` elements, that holds settings for the whole document. Every attribute is optional. Added in NuGet 3.2.0 ([#225](https://github.com/saveenr/VisioAutomation/issues/225)).

| Attribute | Type | Default | Notes |
| --- | --- | --- | --- |
| `template` | string | none | A Visio template file name (for example `basflo_u.vstx`; use `.vst` before Visio 2013). The drawing is created from it. See the note below. |
| `borderwidth` | `double` | `1.0` | Margin in inches on each of the left and right sides of the drawing, on every page. |
| `borderheight` | `double` | `1.0` | Margin in inches above and below the drawing, on every page. |

If you set only one of the two border attributes, the other keeps its default. The border is applied last, when each page is resized to fit its contents, so it decides the final margin around the drawing.

```xml
<directedgraph>
  <documentoptions borderwidth="0.5" borderheight="0.5" />
  <page> ... </page>
</directedgraph>
```

**About `template`.** The value is handed to `NewDocumentFromTemplate`, the same code as `New-VisioDocument -Template`. From NuGet 3.2.0 it creates the drawing from the template, so the drawing gets the template's page setup, styles and settings, and the stencils in the template's workspace open and dock. A stencil file (`.vss` or `.vssx`) is not a template and raises `ArgumentException`. In NuGet 3.1.0 and earlier the file was instead opened as a separate docked stencil (an empty one in Visio 2013 and later) beside a blank drawing, so the drawing was not based on the template ([#229](https://github.com/saveenr/VisioAutomation/issues/229)).

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
| `connectortype` | `Curved` \| `Straight` \| `RightAngle` | no | `Curved` | Connector style applied to every edge on the page, unless a `<connector>` sets its own `connectortype`. Case-insensitive. |
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
| `width` | no | Width of the shape in inches. Set both `width` and `height`, or neither: setting only one raises `ArgumentException`. With neither, the shape keeps its master's size. Added in NuGet 3.2.0 ([#225](https://github.com/saveenr/VisioAutomation/issues/225)). |
| `height` | no | Height of the shape in inches. See `width`. |

A `<shape>` can also contain `<customprop>` children:

```xml
<shape id="n4" label="Foo" stencil="server_u.vss" master="Web Server">
  <customprop name="prop1" value="value1" />
  <customprop name="prop2" value="value2" />
</shape>
```

Each `<customprop>` is added to the dropped shape's custom-properties section using `name` as the row name and `value` as the value. Without a `type` (see below) the loader stores it as a string-typed custom property (the value is quoted in the cell). Only a value that begins with `=` is kept as a raw formula. Both `name` and `value` are required.

### Custom property types

`<customprop>` also takes these optional attributes, added in NuGet 3.2.0 ([#225](https://github.com/saveenr/VisioAutomation/issues/225)):

| Attribute | Notes |
| --- | --- |
| `type` | `string` (the default), `number`, `boolean` (or `bool`) or `date`, case-insensitive. The value is parsed for its type, using the invariant culture for numbers and dates. Any other type raises `ArgumentException`. |
| `label` | The property's label. |
| `prompt` | The property's prompt text. |
| `format` | The property's format string. |

```xml
<customprop name="cost" value="2.5" type="number" label="Cost" />
<customprop name="active" value="true" type="boolean" />
```

A `<customprop>` works the same way under a `<connector>`.

### `<hyperlink>`

A `<shape>` can contain any number of `<hyperlink>` children, in addition to the `url` attribute. Added in NuGet 3.2.0 ([#225](https://github.com/saveenr/VisioAutomation/issues/225)).

| Attribute | Required? | Notes |
| --- | --- | --- |
| `name` | yes | Row name of the hyperlink. |
| `address` | yes | The address. |
| `subaddress` | no | A location inside the address, for example a page name. |
| `description` | no | Description text for the link. |

If the shape also has a `url` attribute, that link comes first (as `Row_1`), followed by the `<hyperlink>` links in order. Connectors do not support hyperlinks.

### `<cells>`

A `<shape>` or a `<connector>` can contain one `<cells>` element that sets ShapeSheet cells, for example fill, line and text formatting. Added in NuGet 3.2.0 ([#225](https://github.com/saveenr/VisioAutomation/issues/225)).

```xml
<shape id="n1" label="Server" stencil="server_u.vss" master="Web Server">
  <cells>
    <cell name="FillForeground" value="RGB(255,200,0)" />
    <cell name="CharSize" value="12 pt" />
  </cells>
</shape>
```

Each `<cell>` needs a `name` and a `value`. The `name` is the property name of [`VisioAutomation.Models.Dom.ShapeCells`](https://github.com/saveenr/VisioAutomation/blob/master/VisioAutomation_2010/VisioAutomation.Models/DOM/ShapeCells.cs), matched without regard to case (for example `FillForeground`, `LineColor`, `LinePattern`, `CharSize` or `XFormWidth`). The `value` is a Visio formula or literal, written as it would be in the ShapeSheet. An unknown name raises `ArgumentException`.

Cells set here win over the `width` and `height` attributes when both set the size; see [Setting `Size` and `Cells` together](models/directed-graph.md#setting-size-and-cells-together).

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
| `connectortype` | no | `Curved`, `Straight` or `RightAngle`, case-insensitive. Overrides the page's `connectortype` for this connector only. Added in NuGet 3.2.0 ([#225](https://github.com/saveenr/VisioAutomation/issues/225)). |

A `<connector>` can also contain a `<cells>` element (see above) and `<customprop>` children, which work as they do on a `<shape>`. Cells set explicitly in `<cells>` win over the `color`, `weight` and arrow defaults, and custom properties on a connector are applied to the drawn connector. Both were added in NuGet 3.2.0 ([#225](https://github.com/saveenr/VisioAutomation/issues/225)).

Every connector drawn from XML gets an end arrow by default; to change it, set the `LineEndArrow` cell in `<cells>`.

### Failure modes

Validation happens at render time, not at load time:

* A `from` or `to` that matches no shape in the page loads without error, but rendering throws `ArgumentException` ("Connector's From/To node is null"). The loader notes such problems only in verbose output.
* Duplicate shape ids throw `ArgumentException` while the document is loaded.
* Duplicate connector ids throw when the edge is added to the graph.

## What's not supported in XML

The following can be set programmatically on the in-memory model but are not exposed in the XML schema:

* Per-edge master and stencil. Every connector uses the page-wide connector master (`DirectedGraphStyling.EdgeMasterName` and `EdgeStencilName`).
* `DirectedGraphStyling` itself, and choosing the Visio-based renderer (`VisioLayoutRenderer`). Drawing from XML always uses the MSAGL renderer with the default styling.
* Cells that are not properties of `ShapeCells`, such as `User.` and `Prop.` rows. (Custom properties have their own `<customprop>` element.)
* Hyperlink properties other than `name`, `address`, `subaddress` and `description` (for example frame, sort key or new window), and hyperlinks on connectors.
* Layout algorithms other than Sugiyama.
* The page border (`PageBorderWidth`) and the default node size (`DefaultShapeSize`). The page border is overridden by the document's `borderwidth` and `borderheight` when a directed graph document is drawn, and the default node size is never used because node sizes come from their masters, so exposing them would have no effect.

The remaining gaps are tracked in [#225](https://github.com/saveenr/VisioAutomation/issues/225).

If you need any of these, load the XML, mutate the resulting `DirectedGraphDocument` from C#, then render via `Client.Model.DrawDirectedGraphDocument`.

## See also

* [Connectors](connectors.md): the lower-level helpers for connecting two shapes when you don't need automated layout.
* [Stencils and masters](stencils-and-masters.md): how the `stencil` and `master` attributes resolve at runtime.
