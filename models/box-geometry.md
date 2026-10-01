# Box geometry

`VisioAutomation.Models.Layouts.Box` computes **rectangles**. It is not one of the [layout models](layouts.md): it draws nothing, there is no VisioScripting method or cmdlet that draws it, and nothing else in the library uses it. Reach for it only when you want positioned rectangles to draw yourself (with the [DOM](dom.md), the [imperative API](../extensions/drawing.md), or even a non-Visio output). If you want shapes placed on a page for you, use the [Tree](layouts-tree.md), [Grid](layouts-grid.md), [Container](layouts-container.md) or [Directed graph](directed-graph.md) layout instead. What the Box layout is for is still being discussed in [#218](https://github.com/saveenr/VisioAutomation/issues/218).

Use it when the data is a tree of rectangular regions packed in a particular direction (left-to-right, top-to-bottom, etc.) inside a parent rectangle. The output is positioned rectangles. The class has no Visio rendering of its own: you walk its `Nodes` and emit shapes from the rectangles yourself.

The model is a tree of `Container` nodes, where each container has a `Direction` (the axis along which its children pack) and a list of children. Each child is either another `Container` (for nesting) or a `Box` (a leaf rectangle of a given size). Each container has `PaddingLeft`, `PaddingRight`, `PaddingTop` and `PaddingBottom` (all 0.125 by default) and a `ChildSpacing` (also 0.125 by default) inserted between adjacent children.

`Direction` takes one of four values. It sets both the axis children pack along and the edge the first child starts against:

| Direction | Axis | First child is placed |
| --- | --- | --- |
| `LeftToRight` | horizontal | at the left edge, with later children to its right |
| `RightToLeft` | horizontal | at the right edge, with later children to its left |
| `BottomToTop` | vertical | at the bottom edge, with later children above it |
| `TopToBottom` | vertical | at the top edge, with later children below it |

In a horizontal container, a child shorter than the container is positioned by its `VAlignToParent` (`Top`, `Center` or `Bottom`; default `Top`). In a vertical container, a child narrower than the container is positioned by its `HAlignToParent` (`Left`, `Center` or `Right`; default `Left`).


```csharp
using VABOX = VisioAutomation.Models.Layouts.Box;

var layout = new VABOX.BoxLayout();
layout.Root = new VABOX.Container(VABOX.Direction.LeftToRight);

var n1 = layout.Root.AddBox(2, 1);
var n2 = layout.Root.AddBox(3, 1);

layout.Root.PaddingLeft = 0.5;
layout.Root.PaddingRight = 0.5;
layout.Root.PaddingTop = 0.5;
layout.Root.PaddingBottom = 0.5;

layout.PerformLayout();

// After PerformLayout, every node has a populated Rectangle.
// Rectangles are (left, bottom, right, top).
// With the default ChildSpacing of 0.125 between the two boxes:
// n1.Rectangle = (0.5, 0.5, 2.5, 1.5)
// n2.Rectangle = (2.625, 0.5, 5.625, 1.5)
// layout.Root.Rectangle = (0, 0, 6.125, 2.0)
```

Containers can nest: a child container packs its own children along its own direction, and the parent treats it as a single rectangle whose size is the bounding box of its packed contents. The layout is a nested stack-and-pad packer; it does not size boxes proportionally the way a treemap does.

Nested `RightToLeft` containers were fixed in NuGet 3.1.0 ([#202](https://github.com/saveenr/VisioAutomation/issues/202)). In 3.0.0 and earlier, a `RightToLeft` container placed anywhere other than the root misplaces its children whenever its origin Y differs from its X, for example one nested inside a vertical container. The root container is always placed at (0, 0), so it was not affected.

`PerformLayout()` is computational only; it doesn't talk to Visio. To render, walk the tree and emit DOM shapes (or use the rectangles for any other purpose, e.g. a JPEG or SVG). The separation makes it useful for non-Visio output too.

## See also

* [Layout models](layouts.md) (the layouts that do draw shapes)
* [DOM](dom.md) (emit DOM shapes from the rectangles this layout computes)
