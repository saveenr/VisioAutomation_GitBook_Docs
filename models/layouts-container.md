# Container layout model

Use when you want several labelled columns of items, each column wrapped in a Visio container shape. `ContainerLayout` is independent of the [Box geometry](box-geometry.md) type: it arranges one column per container, with the container's items stacked top to bottom inside it. (`Layouts.Container.Container` and `Layouts.Box.Container` are unrelated types that happen to share a name.)

## Where the output goes

The container layout is the only layout that takes a **document** and not a page. It always adds a page of its own.

| Desired output | How to get it | Notes |
| :--- | :--- | :--- |
| One new page in an existing document | `ContainerLayout.Render(visioDoc)` | Adds a page to the document, draws the containers and items on it, resizes the page to fit its contents, and changes the active window's zoom to fit the page. Returns the new page. It cannot draw onto a page that already exists. |

It does not use the [DOM](dom.md), so the [render performance settings](dom.md#render-performance) do not apply.

## Example

```csharp
using VACONT = VisioAutomation.Models.Layouts.Container;
using IVisio = Microsoft.Office.Interop.Visio;

var layout = new VACONT.ContainerLayout();
var c1 = layout.AddContainer("Fruit");
c1.Add("Apple");
c1.Add("Pear");
var c2 = layout.AddContainer("Vegetables");
c2.Add("Carrot");

layout.PerformLayout();
IVisio.Page page = layout.Render(visioDoc);
```

`Render` takes an `IVisio.Document`, adds a new page to it and returns that page. Calling `Render` before `PerformLayout()` throws an `ArgumentException`.

`layout.LayoutOptions` controls the geometry: `ItemWidth` (2.0), `ItemHeight` (0.25), `Padding` (0.125), `ContainerHeaderHeight` (0.25), `ContainerHorizontalDistance` (1.0) and `ItemVerticalSpacing` (0.125). The masters are public fields: `ManualItemMaster` (default "Rounded Rectangle") and `ManualContainerMaster` (default "Rectangle") are the ones `Render` drops, and `ContainerMaster` (default "Container 1") is also exposed.

## See also

* [Layout models](layouts.md) (the overview and comparison of all the layouts)
* [DOM](dom.md) (the general-purpose shape model; the container layout does not use it and drops its shapes directly)
