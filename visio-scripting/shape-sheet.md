# client.ShapeSheet

`client.ShapeSheet` exposes a page-scoped batch reader and writer for ShapeSheet cells, plus a small shape-naming helper. The reader and writer are the high-level wrappers around the lower-level [ShapeSheet](../shapesheet/README.md) APIs, batched per page so a single `Commit()` posts every queued cell change.

## Methods

| Method                                              | Returns                  | Notes                                                                                |
| --------------------------------------------------- | ------------------------ | ------------------------------------------------------------------------------------ |
| `GetWriterForPage(TargetPage)`                      | `ShapeSheetWriter`       | Returns a per-page writer; queue cell sets, then call `.Commit()` to post.           |
| `GetReaderForPage(TargetPage)`                      | `ShapeSheetReader`       | Returns a per-page reader; queue cell queries, then call `.Read()` to fetch.         |
| `SetShapeName(TargetShapes, IList<string> names)`   | `void`                   | Sets `Shape.Name` per shape; reuses last name if `names` is shorter than the shape list. |

`TargetPage.Auto` resolves to the active page; `TargetShapes.Auto` resolves to the currently selected shapes. `SetShapeName` opens an undo scope so the whole batch reverses together. The reader and writer the other two methods produce are themselves the entry points for any non-trivial ShapeSheet work.

## Example

```csharp
using IVisio = Microsoft.Office.Interop.Visio;

// Use the writer to set Width and Height on every selected shape in one commit
var writer = client.ShapeSheet.GetWriterForPage(VisioScripting.TargetPage.Auto);
foreach (var shape in client.Selection.GetSelectedShapes(VisioScripting.TargetWindow.Auto))
{
    writer.SetFormula((short)shape.ID, VisioAutomation.Core.SrcConstants.XFormWidth, "2.0 in");
    writer.SetFormula((short)shape.ID, VisioAutomation.Core.SrcConstants.XFormHeight, "1.0 in");
}
writer.Commit();

// Use the reader to fetch the resolved values back
var reader = client.ShapeSheet.GetReaderForPage(VisioScripting.TargetPage.Auto);
// ... (set up queries on `reader`, call reader.Read())

// Rename the same selection
client.ShapeSheet.SetShapeName(VisioScripting.TargetShapes.Auto,
                                 new[] { "node-a", "node-b", "node-c" });
```

## See also

- [ShapeSheet](../shapesheet/README.md): the lower-level cell, query, and writer surface.
- [Query the ShapeSheet](../shapesheet/query-the-shapesheet.md): the read side of the cell model.
- [Modify the ShapeSheet](../shapesheet/modify-the-shapesheet.md): the write side of the cell model.
- [VisioScripting.Client overview](../visio-scripting.md): how `client.ShapeSheet` fits into the facade.
