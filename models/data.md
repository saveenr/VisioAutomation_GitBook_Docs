# Data models

The `VisioAutomation.Models.Data` namespace holds models that take **existing .NET data** and draw a picture of it on a Visio page. You hand over a `System.Data.DataTable` or a `System.Xml.XmlDocument`, and the model turns it into shapes. This differs from a [layout model](layouts.md), where you describe the nodes yourself, and from a [document model](documents.md), which creates a whole new Visio document.

There are two data models:

| Model | Namespace | Best for | Entry point |
| :--- | :--- | :--- | :--- |
| [Data table model](data-table.md) | `VisioAutomation.Models.Data` | Tabular data: a lookup table beside a diagram, a legend, or a quick look at a query result. | `client.Model.DrawDataTableModel(...)` or `DrawDataTable(...)` from VisioScripting, or `Out-VisioApplication` from PowerShell. |
| [XML model](xml-model.md) | `VisioAutomation.Models.Data` | The nesting of an XML document, drawn as a tree of element names. | `client.Model.DrawXmlModel(...)` from VisioScripting, or `Out-VisioApplication` from PowerShell. |

## What they have in common

* **The input is data you already have.** A `DataTable` or an `XmlDocument`, not a description of shapes.
* **They draw onto a page.** You pass a `TargetPage` (or use the active page), and the shapes land there. Nothing creates a new document.
* **They are deliberately minimal.** The data table has no header row or per-cell formatting, and the XML model shows element names only. Both are for a quick picture, not a finished report.
* **They are built on a layout.** The data table is drawn with a [grid layout](layouts-grid.md) and the XML model with a [tree layout](layouts-tree.md), so for more control you can use that layout directly.

## See also

* [Data table model](data-table.md) (a `DataTable` as a grid of rectangles)
* [XML model](xml-model.md) (the element structure of an `XmlDocument` as a tree)
* [Layout models](layouts.md) (the grid and tree layouts that draw these)
* [Document models](documents.md) (models that create a whole Visio document)
* [client.Model](../visio-scripting/model.md) (the VisioScripting methods that draw these models)
