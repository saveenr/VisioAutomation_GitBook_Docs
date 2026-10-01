# Container layout

Use when you want several labelled columns of items, each column wrapped in a Visio container shape. `ContainerLayout` is independent of the Box layout: it arranges one column per container, with the container's items stacked top to bottom inside it. (`Layouts.Container.Container` and `Layouts.Box.Container` are unrelated types that happen to share a name.)

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

* [Layouts](layouts.md) (the overview and comparison of all the layouts)
* [Declarative DOM](dom.md) (the underlying shape model the layouts emit into)
