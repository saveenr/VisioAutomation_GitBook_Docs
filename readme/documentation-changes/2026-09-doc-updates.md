# 2026-09 doc updates

## 2026-09: Introduction to models page

Added an [Introduction to models](../../models/introduction.md) page at the top of the Diagram models section. It explains what a model is (an in-memory structure that you build and then render), why that keeps you close to the document you want, and that the [DOM](../../models/dom.md) is the foundation where the Visio-specific and performance work lives. It lists every model and says which ones do not go through the DOM: Box, Container and Form pages. The [Layout models](../../models/layouts.md) overview and the Container layout page no longer say that every layout draws through the DOM.

## 2026-09: Data models page

Added a [Data models](../../models/data.md) page for the two models that draw existing .NET data: the [Data table model](../../models/data-table.md) and the [XML model](../../models/xml-model.md). It compares them and says what they share, and the two pages are now nested beneath it in the table of contents. No existing page changed.

## 2026-09: Model pages renamed to say "model"

The pages in the Diagram models section are about the models, not about the things they draw, so their titles now say so: [Layout models](../../models/layouts.md) (was Layouts), [Tree](../../models/layouts-tree.md), [Grid](../../models/layouts-grid.md), [Box](../../models/layouts-box.md), [Container](../../models/layouts-container.md) and [Directed graph](../../models/directed-graph.md) layout model, [Org chart model](../../models/org-charts.md), [Form page model](../../models/forms.md) and [DOM](../../models/dom.md). Layout styles and the Directed graph XML format keep their names because they are a Visio feature and a file format rather than models. Page addresses did not change, so existing links still work.

## 2026-09: Where layout styles are used

[Layout styles](../../models/layout-styles.md) gained a "Where layout styles are used" section: a style is applied to a page, so it is used by the DOM `Page.Layout`, `client.Page.LayoutPage`, the `Format-VisioPage -LayoutStyle` cmdlet and the Visio-based directed graph renderer, and not only by a layout type. The [Directed graph](../../models/directed-graph.md) page gained a matching "Laying out with Visio instead of MSAGL" section that describes that renderer and what it does not apply. The [Layouts](../../models/layouts.md) overview no longer says layout styles are used by directed graph and other style-driven renderers, which overstated it.

## 2026-09: Document models page for Org charts and Form pages

Added a [Document models](../../models/documents.md) page for the two models that describe a whole Visio document and create it: [Org charts](../../models/org-charts.md) and [Form pages](../../models/forms.md). It explains what they have in common, compares them in a table, and the two pages are now nested beneath it in the table of contents. No existing page changed.

## 2026-09: Table of contents sections reorganized

Some section names in the table of contents did not match how Visio uses the terms, so the sidebar was regrouped. "Shape data" has a specific meaning in Visio (custom properties), so that section is now **Shapes**, and the [Geometry](../../geometry.md) page moved into it. [Connection points](../../connection-points.md) and [Connectors](../../connectors.md) moved to a new **Connections** section. "Formatting and layout" is now **Formatting**. [Analyzers](../../analyzers.md) moved out of Diagnostics into a new **Diagram analysis** section. Only the grouping changed: no page was added or removed.

## 2026-09: Layouts page split into an overview and one page per layout

[Layouts](../../models/layouts.md) is now a short overview with the comparison table. [Tree layout](../../models/layouts-tree.md), [Grid layout](../../models/layouts-grid.md), [Box layout](../../models/layouts-box.md) and [Container layout](../../models/layouts-container.md) each have their own page, nested under it in the table of contents. The text of each section moved unchanged, the overview is still the page at `models/layouts.md`, and links that pointed at the old `#tree-layout` and `#grid-layout` sections now go to the new pages. [Directed graph](../../models/directed-graph.md), with its [XML format](../../directed-graph-xml.md) page nested beneath it, also moved under Layouts so every layout is in one place.

## 2026-09: Models documentation accuracy pass

Every page covering the [`VisioAutomation.Models`](https://github.com/saveenr/VisioAutomation/tree/master/VisioAutomation_2010/VisioAutomation.Models) project was reviewed against the source, and the claims that no longer matched were corrected. All C# snippets on the changed pages were compile-checked afterwards, and the PowerShell examples added in this period were run against a live Visio. The main corrections:

* **[Directed graph](../../models/directed-graph.md)** and **[Directed graph XML format](../../directed-graph-xml.md)**: the note on `scalingfactor` had the spacing backwards (a larger value gives tighter gaps relative to node size). `usedynamicconnectors` and `scalingfactor` are required in XML, as are the `<renderoptions>`, `<shapes>` and `<connectors>` elements. `MsaglRenderer.Render` uses the renderer's own options and does not read the ones stored on the layout. A node with no `Size` takes the size of its master, so `DefaultShapeSize` does not apply in practice. The Directed graph page also gained a layout options reference and a "How the layout works" section, and the XML page gained a "Failure modes" section.
* **[Declarative DOM](../../models/dom.md)**: removed an unsupported claim that rendering is a single undo step; corrected the descriptions of `RenderPerformanceSettings`, the node hierarchy, `Connect` and connector endpoints; added the `using VisioAutomation.Extensions;` that the `OpenStencil` snippet needs.
* **[Layouts](../../models/layouts.md)** ([Tree](../../models/layouts-tree.md), [Grid](../../models/layouts-grid.md), [Box](../../models/layouts-box.md) and [Container](../../models/layouts-container.md)): added the **Container layout**, which had no coverage; corrected the Box example coordinates, the Grid example (it needs `PerformLayout()`) and the claims about which layout settings and masters are configurable; documented the four Box `Direction` values and the default child alignment.
* **[Layout styles](../../models/layout-styles.md)**: corrected the `CompactTreeDirection` and `ConnectorStyle` value lists, which cells `Apply` writes, and the per-style defaults.
* **[Org charts](../../models/org-charts.md)** and **[Form pages](../../models/forms.md)**: an org chart is always drawn into a new document, its XML schema is now documented, and a form page has two text blocks.

Two source bugs found during the review were filed and fixed: the org chart renderer drew the first root on every page ([#201](https://github.com/saveenr/VisioAutomation/issues/201)), and Box layouts placed nested `RightToLeft` containers wrongly ([#202](https://github.com/saveenr/VisioAutomation/issues/202)). The pages describe both the fixed behavior and the earlier behavior.

## 2026-09: New pages for the data table and XML models

Added [Data table model](../../models/data-table.md) and [XML model](../../models/xml-model.md) under **Diagram models**. These two renderable models had appeared only as one-line rows in the [client.Model](../../visio-scripting/model.md) table, where three rows and an example comment were also corrected. Each page has C# and PowerShell examples.

Writing them exposed three behaviors that looked unintended, so each was filed and fixed in the source: data table cell sizes were ignored ([#206](https://github.com/saveenr/VisioAutomation/issues/206)), `DrawDataTableModel` always drew on the active page ([#207](https://github.com/saveenr/VisioAutomation/issues/207)), and the XML tree showed `#document` instead of the document element's name ([#208](https://github.com/saveenr/VisioAutomation/issues/208)). The pages describe the fixed behavior, which is in current source and unreleased after NuGet 3.1.0, together with what 3.1.0 and earlier do.

## 2026-09: Directed graph spacing options

Documented the two new options for tightening directed-graph layouts, `EdgeLabelBoxSize` and `LayerSeparation`, and the matching XML attributes `layerseparation`, `edgelabelboxwidth` and `edgelabelboxheight`. They are in the options table and a new "Tightening the layout" section on [Directed graph](../../models/directed-graph.md), and in the `<renderoptions>` table of the [XML format](../../directed-graph-xml.md) page. They were added in NuGet 3.1.0.

## 2026-09: Post-release sweep for VisioAutomation 3.1.0

VisioAutomation2010 3.1.0 was published on 2026-09-30. Notes that called a change "unreleased after NuGet 3.0.0" or "in current source" now say which version it landed in, and keep the 3.0.0 workaround for readers who are still on that package. This covers the facade loaders, the spacing options and XML attributes, and the org chart and Box fixes above. [Version compatibility](../../version-compatibility.md) gained a `3.1.0` row, and 3.1.0 is now named as the recommended starting point.

## 2026-09: Compiling page matches the current build baseline

[Compiling](../../compiling.md) now describes the current toolchain: Visual Studio 2026, the .NET 10 SDK selected by `global.json`, an explicit C# 14 language version, and the SLNX solution. It no longer says Visual Studio 2026 is unsupported. Reference assemblies and the Visio interop assembly come from NuGet, so no Developer Pack install is needed, and Visio itself is required only to run the tests.
