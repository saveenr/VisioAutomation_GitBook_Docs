# client.Hyperlink

`client.Hyperlink` adds, reads, and removes hyperlinks on shapes. Each shape can carry multiple hyperlinks, indexed by row in the shape's `Hyperlink` section. The lower-level cell-level API for these is on the [Hyperlinks](../hyperlinks.md) page; the `client.Hyperlink` group adds shape-set-aware bulk operations and undo-scoped writes.

## Methods

| Method                                                | Returns                                                            | Notes                                                                          |
| ----------------------------------------------------- | ------------------------------------------------------------------ | ------------------------------------------------------------------------------ |
| `AddHyperlink(TargetShapes, HyperlinkCells)`          | `List<int>`                                                        | Appends one hyperlink per shape, returns the index of each new hyperlink row.  |
| `DeleteHyperlinkAtIndex(TargetShapes, int n)`         | `void`                                                             | Skips shapes with fewer than `n+1` hyperlinks.                                 |
| `GetHyperlinks(TargetShapes, CellValueType cvt)`      | `Dictionary<Shape, CellRecords<HyperlinkCells>>`                   | Returns one record-set per shape; `cvt` selects formula or resolved value.     |

All three methods resolve `TargetShapes.Auto` to the currently selected shapes at call time, run no-ops if the resolved set is empty, and wrap the write methods in an undo scope.

## Example

```csharp
var hlink = new VisioAutomation.Shapes.HyperlinkCells();
hlink.Address = "\"https://example.com\"";
hlink.Description = "\"Example\"";

// Operate on whatever is currently selected
var added = client.Hyperlink.AddHyperlink(VisioScripting.TargetShapes.Auto, hlink);

var dict = client.Hyperlink.GetHyperlinks(VisioScripting.TargetShapes.Auto,
                                           VisioAutomation.Core.CellValueType.Formula);
foreach (var kv in dict)
{
    foreach (var record in kv.Value)
    {
        System.Console.WriteLine("{0}: {1}", kv.Key.Name, record.Cells.Address);
    }
}

client.Hyperlink.DeleteHyperlinkAtIndex(VisioScripting.TargetShapes.Auto, 0);
```

`HyperlinkCells.Address` and `HyperlinkCells.Description` are ShapeSheet *formulas*, not literal strings, which is why the example wraps the URL and label in escaped double-quotes. See [Custom properties: values are formulas, not literals](../custom-properties.md) for the same convention applied to a different cell.

## See also

- [Hyperlinks](../hyperlinks.md): the lower-level `VisioAutomation.Shapes.HyperlinkHelper` API and `HyperlinkCells` cell-level surface.
- [VisioScripting.Client overview](../visio-scripting.md): how `client.Hyperlink` fits into the facade.
- [Custom properties](../custom-properties.md): related per-shape data with the same "value is a formula" convention.
