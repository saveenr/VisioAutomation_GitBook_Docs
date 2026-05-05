# Shape cells

A shape's ShapeSheet has many sections. The ones a typical script touches most often are split across **three records** under `VisioAutomation.Shapes`, each documented on its own page:

| Record              | Covers                                                                                                  | Page                                              |
| ------------------- | ------------------------------------------------------------------------------------------------------- | ------------------------------------------------- |
| `ShapeXFormCells`   | Size and position: `PinX`, `PinY`, `Width`, `Height`, `Angle`, `LocPinX`, `LocPinY`.                    | [Shape XForm cells](shape-xform-cells.md)         |
| `ShapeFormatCells`  | Visual appearance: fill colors / patterns, line color / weight / pattern, line arrows, fill shadow.     | [Shape format cells](shape-format-cells.md)       |
| `ShapeLayoutCells`  | Layout / routing behavior used by Visio's auto-layout engine: `ShapePlaceStyle`, `LineJump*`, etc.      | [Shape layout cells](shape-layout-cells.md)       |

All three follow the same cell-record pattern as [Custom properties](custom-properties.md), [User-defined cells](user-defined-cells.md), and [Lock cells](lock-cells.md): set the fields you care about, hand the record to a `SrcWriter`, commit. Cells you don't set are left untouched.

## Combine multiple records in one writer

A `SrcWriter` accepts as many `SetValues` calls as you want. Mix records in a single transaction so all the changes commit atomically:

```csharp
var writer = new VisioAutomation.ShapeSheet.Writers.SrcWriter();
writer.SetValues(xform);
writer.SetValues(fmt);
writer.SetValues(layout);
writer.Commit(shape, VisioAutomation.Core.CellValueType.Formula);
```

## See also

* [Shape XForm cells](shape-xform-cells.md), [Shape format cells](shape-format-cells.md), [Shape layout cells](shape-layout-cells.md) &mdash; the per-record details.
* [Modify the ShapeSheet](shapesheet/modify-the-shapesheet.md) &mdash; the underlying `SrcWriter` / `SidSrcWriter` API.
* [Lock cells](lock-cells.md), [Custom properties](custom-properties.md), [User-defined cells](user-defined-cells.md) &mdash; other per-shape ShapeSheet records.
