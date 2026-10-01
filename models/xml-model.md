# XML model

`VisioAutomation.Models.Data.XmlModel` draws the **structure** of a `System.Xml.XmlDocument` as a tree: one node per element, with parent-child connectors. Use it to get a quick picture of how an XML document is nested.

It shows element names only. Attributes, text content and comments are not drawn, so it is a structure viewer, not a data viewer. For XML that describes a diagram in its own right, see [Directed graph XML format](../directed-graph-xml.md) or [Org chart model](org-charts.md).

## Where the output goes

The XML model draws onto a page that already exists, and it **resizes that page**.

| Desired output | How to get it | Notes |
| :--- | :--- | :--- |
| Shapes on an existing page, with the page resized | `client.Model.DrawXmlModel(targetPage, model)`, or `Out-VisioApplication` from PowerShell | Draws the element tree with the [tree layout](layouts-tree.md), which sets the page's size to the tree's bounds plus a 0.5 inch border. Shapes already on the page are not moved, so the page is sized to the tree and not to them. |

The tree is drawn through the [DOM](dom.md), so its [render performance settings](dom.md#render-performance) apply.

## Hello-world

```csharp
using VAMODELS = VisioAutomation.Models;

var xml = new System.Xml.XmlDocument();
xml.LoadXml("<root><child1><leaf/></child1><child2/></root>");

var model = new VAMODELS.Data.XmlModel();
model.XmlDocument = xml;

client.Model.DrawXmlModel(VisioScripting.TargetPage.Auto, model);
```

This draws a tree on the active page: a top node labelled `root` with `child1` and `child2` below it, and `leaf` below `child1`, joined by dynamic connectors.

## The model

`XmlModel` has a single property, `XmlDocument` (a `System.Xml.XmlDocument`). `client.Model.DrawXmlModel(TargetPage, XmlModel)` builds a [tree layout](layouts-tree.md) from it and renders it onto the target page.

## What gets drawn

* **Element names only.** Each node is labelled with the element's name.
* **The top node is the document element.** It is labelled with the document element's name (`root` above), and the nodes below it are its child elements. An `XmlDocument` with no document element throws `ArgumentException`.

That is how it works from NuGet 3.2.0 ([#208](https://github.com/saveenr/VisioAutomation/issues/208)). In 3.1.0 and earlier the top node was labelled `#document`, standing in for the document element: the document element's own name (`root` above) was never drawn, and a document with no document element threw `NullReferenceException`.
* **Nested elements nest.** Each element's child elements become its child nodes, recursively.
* **Not drawn:** attributes, text nodes, comments and processing instructions. An element with only text content appears as a leaf node, without the text.

Layout is the default tree layout (top to bottom, with fixed separations); see [Tree layout model](layouts-tree.md).

## From PowerShell

`Out-VisioApplication` accepts an `XmlModel` from the pipeline and draws it on the active page:

```powershell
Import-Module Visio
New-VisioDocument | Out-Null

$xml = New-Object System.Xml.XmlDocument
$xml.LoadXml('<root><child1><leaf/></child1><child2/></root>')

$model = New-Object VisioAutomation.Models.Data.XmlModel
$model.XmlDocument = $xml

$model | Out-VisioApplication
```

See the [Out-VisioApplication cmdlet page](https://saveenr.gitbook.io/visiopowershell/cmdlets/visioapplication/out-visioapplication) for the other model types it accepts.

## See also

* [client.Model](../visio-scripting/model.md) (the `DrawXmlModel` method)
* [Tree layout model](layouts-tree.md) (the tree layout that draws the structure)
* [Data table model](data-table.md) (the other `Models.Data` type)
* [Directed graph XML format](../directed-graph-xml.md) and [Org chart model](org-charts.md) (XML formats that describe a diagram)
