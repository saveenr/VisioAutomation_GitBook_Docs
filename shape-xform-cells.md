# Shape XForm cells

`VisioAutomation.Shapes.ShapeXFormCells` covers the seven cells in a shape's `Shape Transform` section &mdash; size, position, rotation, and the local "pin" (anchor) point. It is one of the three `Shape*Cells` records; the others are [Shape format cells](shape-format-cells.md) and [Shape layout cells](shape-layout-cells.md).

## The `ShapeXFormCells` record

| Field             | ShapeSheet cell | Purpose                                                                                          |
| ----------------- | --------------- | ------------------------------------------------------------------------------------------------ |
| `PinX`            | `PinX`          | On-page X coordinate of the shape's pin (its rotation / anchor point).                           |
| `PinY`            | `PinY`          | On-page Y coordinate of the shape's pin.                                                         |
| `LocPinX`         | `LocPinX`       | X coordinate of the pin relative to the shape's local coordinate space. Defaults to `Width*0.5`.  |
| `LocPinY`         | `LocPinY`       | Y coordinate of the pin relative to the shape's local coordinate space. Defaults to `Height*0.5`. |
| `Width`           | `Width`         | Shape width.                                                                                     |
| `Height`          | `Height`        | Shape height.                                                                                    |
| `Angle`           | `Angle`         | Rotation angle around the pin (in radians by default; pass an explicit unit to override).        |

All fields are `Core.CellValue`-typed and accept implicit conversions from strings, ints, doubles, and bools.

## Set position and size

Construct the record, set the fields you want, hand it to a `SrcWriter`, commit. Cells you don't set are left untouched.

```csharp
var xform = new VisioAutomation.Shapes.ShapeXFormCells();
xform.PinX   = 4.0;
xform.PinY   = 2.0;
xform.Width  = 1.5;
xform.Height = 1.0;

var writer = new VisioAutomation.ShapeSheet.Writers.SrcWriter();
writer.SetValues(xform);
writer.Commit(shape, VisioAutomation.Core.CellValueType.Formula);
```

`PinX`/`PinY` are the on-page coordinates of the shape's pin. `Width`/`Height` are the shape's size. `Angle` is rotation around the pin.

## Read existing values

`GetCells` returns a single `ShapeXFormCells` instance for one shape, or a `CellRecords<ShapeXFormCells>` for many shapes on a page in one batched call.

```csharp
var xform = VisioAutomation.Shapes.ShapeXFormCells.GetCells(
    shape,
    VisioAutomation.Core.CellValueType.Result);

System.Console.WriteLine($"Width = {xform.Width.Value}");
```

Pass `CellValueType.Formula` to read the underlying formulas instead of evaluated results.

## See also

* [Shape cells overview](shape-cells.md) &mdash; the three `Shape*Cells` records at a glance.
* [Shape format cells](shape-format-cells.md), [Shape layout cells](shape-layout-cells.md) &mdash; the other two records; combine multiple records in one writer.
* [Modify the ShapeSheet](shapesheet/modify-the-shapesheet.md) &mdash; the underlying `SrcWriter` / `SidSrcWriter` API.
