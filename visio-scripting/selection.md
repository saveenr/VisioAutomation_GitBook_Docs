# client.Selection

`client.Selection` selects, deselects, and acts on shapes via the active window's selection. Every method ultimately reads or writes an `IVisio.Selection`; some take an explicit `TargetWindow`, others take a `TargetSelection` that wraps a selection directly.

## Methods

| Method                                                                | Returns                          | Notes                                                                                |
| --------------------------------------------------------------------- | -------------------------------- | ------------------------------------------------------------------------------------ |
| `GetSelection(TargetWindow)`                                          | `IVisio.Selection`               | Returns the resolved window's current `Selection` object.                            |
| `SelectShapeOperation(TargetWindow, ShapeSelectionOperation)`         | `void`                           | Dispatches to `SelectAllShapes`, `InvertSelection`, or `SelectNone`.                 |
| `SelectAllShapes(TargetWindow)`                                       | `void`                           | Calls `Window.SelectAll()`.                                                          |
| `SelectNone(TargetWindow)`                                            | `void`                           | Clears the selection.                                                                |
| `InvertSelection(TargetWindow)`                                       | `void`                           | Selects every shape on the page that is not currently selected.                      |
| `SelectShapesById(TargetWindow, IVisio.Shape shape)`                  | `void`                           | Adds a single shape to the selection.                                                |
| `SelectShapes(TargetWindow, IEnumerable<IVisio.Shape> shapes)`        | `void`                           | Adds a sequence of shapes to the selection.                                          |
| `GetSelectedShapes(TargetWindow)`                                     | `IList<IVisio.Shape>`            | Snapshots the resolved window's selection as a `List<Shape>`.                        |
| `DeleteShapes(TargetSelection)`                                       | `void`                           | Deletes everything in the resolved selection; no-op when empty.                      |
| `DuplicateShapes(TargetSelection)`                                    | `void`                           | Calls `Selection.Duplicate()` on the resolved selection.                             |
| `ContainsShapes(TargetSelection)`                                     | `bool`                           | True when at least one shape is selected.                                            |
| `GetShapeDimensions(TargetShapes)`                                    | `List<ShapeDimensions>`          | Reads each resolved shape's bounding-box-style dimension record.                     |

`TargetWindow.Auto` resolves to the active window. `TargetSelection.Auto` resolves to that window's current selection. `TargetShapes.Auto` resolves to the currently selected shapes. None of these write methods open undo scopes (the underlying `IVisio` calls are themselves undoable).

## Removal candidates (CY27)

These methods are currently `public` but have no known consumers as of 2026-05. They are candidates for removal in CY27 (see [issue #183](https://github.com/saveenr/VisioAutomation/issues/183)). New code should not depend on them.

| Method                                                          | Notes                                                                              |
| --------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| `SubSelectShapes(IList<IVisio.Shape> shapes)`                   | Sub-selects shapes inside the active group; rarely needed at the facade layer.     |
| `SelectShapesByMaster(TargetPage, IVisio.Master)`                | Creates a selection by-master then discards it; effectively a no-op.               |
| `SelectShapesByLayer(TargetPage, string layername)`              | Creates a selection by-layer then discards it; effectively a no-op.                |
| `GetSelectedShapes(TargetSelection)`                             | Duplicate of the `TargetWindow` overload; both delegate to `SelectionHelper`.      |
| `GetShapesRecursive(TargetSelection)`                            | Deep traversal that the rest of the library does not exercise.                     |
| `GetShapeCount(TargetSelection)`                                 | Equivalent to `Selection.Count`.                                                   |
| `GetSubSelectedShapes(TargetSelection)`                          | Reads `IterationMode`-toggled sub-selection; rarely needed.                        |
| `CopySelectedShapes(TargetSelection)`                            | Wraps `Selection.Copy()`; not used inside the library.                             |
| `ContainsShapes(TargetSelection, int min_items)`                 | Threshold variant of `ContainsShapes`; the no-arg overload covers the common case. |

## Example

```csharp
using IVisio = Microsoft.Office.Interop.Visio;

// Select everything on the active page, snapshot it, then delete a subset
client.Selection.SelectAllShapes(VisioScripting.TargetWindow.Auto);

var selected = client.Selection.GetSelectedShapes(VisioScripting.TargetWindow.Auto);
System.Console.WriteLine("Page has {0} shapes", selected.Count);

// Narrow the selection back down to two specific shapes
client.Selection.SelectNone(VisioScripting.TargetWindow.Auto);
client.Selection.SelectShapes(VisioScripting.TargetWindow.Auto,
                                new[] { selected[0], selected[1] });

// Then duplicate them
client.Selection.DuplicateShapes(VisioScripting.TargetSelection.Auto);
```

## See also

- [VisioScripting.Client overview](../visio-scripting.md): how `client.Selection` fits into the facade.
- [Arrange](arrange.md): align, distribute, and nudge the selection that `client.Selection` builds.
- [Grouping](grouping.md): group or ungroup the current selection.
- [Application](application.md): `client.Application.DeleteShapes` falls back to `client.Selection.DeleteShapes` when its `TargetShapes` argument is unresolved.
