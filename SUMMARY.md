# Table of contents

* [Introduction](README.md)
  * [Getting started](readme/getting-started.md)
  * [Related projects](readme/related-projects.md)
  * [Documentation updates](readme/documentation-changes/README.md)
    * [2026-09 doc updates](readme/documentation-changes/2026-09-doc-updates.md)
    * [2026-05 doc updates](readme/documentation-changes/2026-05-doc-updates.md)

## Core APIs

* [Stencils and masters](stencils-and-masters.md)
* [Extension methods](extension-methods.md)
  * [LINQ bridges](extensions/linq.md)
  * [Drawing primitives](extensions/drawing.md)
  * [Master dropping](extensions/dropping.md)
  * [Typed ShapeSheet I/O](extensions/shapesheet.md)
  * [Coordinates and bounding boxes](extensions/coordinates.md)
  * [One-off extensions](extensions/misc.md)
* [ShapeSheet](shapesheet/README.md)
  * [SRC structs](shapesheet/cells.md)
  * [Query the ShapeSheet](shapesheet/query-the-shapesheet.md)
  * [Modify the ShapeSheet](shapesheet/modify-the-shapesheet.md)
* [Application](application.md)
* [Undo scope](undo-scope.md)

## VisioScripting

* [VisioScripting.Client](visio-scripting.md)
  * [client.Application](visio-scripting/application.md)
  * [client.Arrange](visio-scripting/arrange.md)
  * [client.Connection](visio-scripting/connection.md)
  * [client.ConnectionPoint](visio-scripting/connection-point.md)
  * [client.Container](visio-scripting/container.md)
  * [client.Control](visio-scripting/control.md)
  * [client.CustomProperty](visio-scripting/custom-property.md)
  * [client.Developer](visio-scripting/developer.md)
  * [client.Document](visio-scripting/document.md)
  * [client.Draw](visio-scripting/draw.md)
  * [client.Export](visio-scripting/export.md)
  * [client.Grouping](visio-scripting/grouping.md)
  * [client.Hyperlink](visio-scripting/hyperlink.md)
  * [client.Layer](visio-scripting/layer.md)
  * [client.Lock](visio-scripting/lock.md)
  * [client.Master](visio-scripting/master.md)
  * [client.Model](visio-scripting/model.md)
  * [client.Output](visio-scripting/output.md)
  * [client.Page](visio-scripting/page.md)
  * [client.Selection](visio-scripting/selection.md)
  * [client.ShapeSheet](visio-scripting/shape-sheet.md)
  * [client.Text](visio-scripting/text.md)
  * [client.Undo](visio-scripting/undo.md)
  * [client.UserDefinedCell](visio-scripting/user-defined-cell.md)
  * [client.View](visio-scripting/view.md)

## Shapes

* [User-defined cells](user-defined-cells.md)
* [Custom properties](custom-properties.md)
* [Hyperlinks](hyperlinks.md)
* [Lock cells](lock-cells.md)
* [Control handles](control-handles.md)
* [Geometry](geometry.md)

## Connections

* [Connection points](connection-points.md)
* [Connectors](connectors.md)

## Formatting

* [Shape cells](shape-cells.md)
  * [Shape XForm cells](shape-xform-cells.md)
  * [Shape format cells](shape-format-cells.md)
  * [Shape layout cells](shape-layout-cells.md)
* [Page cells](page-cells.md)
  * [Page format cells](pages/format.md)
  * [Page layout cells](pages/layout.md)
  * [Page print cells](pages/print.md)
  * [Page ruler and grid cells](pages/ruler-grid.md)
  * [Page helper](pages/helper.md)
* [Text formatting](text-formatting.md)
  * [Character cells](text/character.md)
  * [Paragraph cells](text/paragraph.md)
  * [Text-block cells](text/block.md)
  * [Tab stops](text/tab-stops.md)

## Diagram models

* [Introduction to models](models/introduction.md)
* [DOM](models/dom.md)
* [Layout models](models/layouts.md)
  * [Tree layout model](models/layouts-tree.md)
  * [Grid layout model](models/layouts-grid.md)
  * [Box layout model](models/layouts-box.md)
  * [Container layout model](models/layouts-container.md)
  * [Directed graph layout model](models/directed-graph.md)
    * [Directed graph XML format](directed-graph-xml.md)
* [Document models](models/documents.md)
  * [Org chart model](models/org-charts.md)
  * [Form page model](models/forms.md)
* [Data models](models/data.md)
  * [Data table model](models/data-table.md)
  * [XML model](models/xml-model.md)
* [Layout styles](models/layout-styles.md)

## Diagram analysis

* [Analyzers](analyzers.md)

## Diagnostics

* [Visio error log](logging.md)
* [Exception types](exceptions.md)

## Reference

* [Namespaces](namespaces.md)
* [Classes](classes.md)
* [Compiling](compiling.md)
* [Version compatibility](version-compatibility.md)
* [Resources](resources/README.md)
