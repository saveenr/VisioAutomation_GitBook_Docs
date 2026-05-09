# client.Text

`client.Text` reads and writes the plain-text content of shapes. For *formatted* text (font, color, paragraph, character cells), see [Text formatting](../text-formatting.md) and the cell-level helpers it links to.

## Methods

| Method                                            | Returns         | Notes                                                                                |
| ------------------------------------------------- | --------------- | ------------------------------------------------------------------------------------ |
| `SetShapeText(TargetShapes, IList<string> texts)` | `void`          | Sets `Shape.Text` per shape; reuses texts cyclically when fewer texts than shapes.   |
| `GetShapeText(TargetShapes)`                      | `List<string>`  | One string per resolved shape.                                                       |

`TargetShapes.Auto` resolves to the currently selected shapes at call time. `SetShapeText` opens an undo scope so the whole batch reverses together; a `null` element in `texts` skips that shape.

## Removal candidates (CY27)

These methods are currently `public` but have no known consumers as of 2026-05. They are candidates for removal in CY27 (see [issue #183](https://github.com/saveenr/VisioAutomation/issues/183)). New code should not depend on them.

| Method                                          | Notes                                                                                |
| ----------------------------------------------- | ------------------------------------------------------------------------------------ |
| `GetShapeTextFormat(TargetShapes)`              | Returns `TextFormat` per shape via `VisioAutomation.Text.TextFormat.GetFormat`.      |

## Example

```csharp
// Set distinct labels on three selected shapes
client.Text.SetShapeText(VisioScripting.TargetShapes.Auto,
                           new[] { "Start", "Middle", "End" });

// Set one label across all selected shapes (cyclic reuse)
client.Text.SetShapeText(VisioScripting.TargetShapes.Auto,
                           new[] { "Step" });

// Read every selected shape's text back
var texts = client.Text.GetShapeText(VisioScripting.TargetShapes.Auto);
foreach (var t in texts)
{
    System.Console.WriteLine(t);
}
```

## See also

- [Text formatting](../text-formatting.md): character, paragraph, and text-block cells (font, color, alignment).
- [Custom property](custom-property.md): name / label / type metadata on each shape's data rows.
- [VisioScripting.Client overview](../visio-scripting.md): how `client.Text` fits into the facade.
- [Undo scope](../undo-scope.md): the scope that `SetShapeText` opens.
