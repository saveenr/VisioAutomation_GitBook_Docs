# Shape format cells

`VisioAutomation.Shapes.ShapeFormatCells` covers the visual-appearance cells from three ShapeSheet sections (**Fill Format**, **Line Format**, and **Fill Shadow**) in a single record. It is one of the three `Shape*Cells` records; the others are [Shape XForm cells](shape-xform-cells.md) and [Shape layout cells](shape-layout-cells.md).

## The `ShapeFormatCells` record

### Fill cells

| Field                        | ShapeSheet cell              | Purpose                                    |
| ---------------------------- | ---------------------------- | ------------------------------------------ |
| `FillForeground`             | `FillForegnd`                | Fill foreground color.                     |
| `FillForegroundTransparency` | `FillForegndTrans`           | Fill foreground transparency.              |
| `FillBackground`             | `FillBkgnd`                  | Fill background color (used by patterns).  |
| `FillBackgroundTransparency` | `FillBkgndTrans`             | Fill background transparency.              |
| `FillPattern`                | `FillPattern`                | Fill pattern index (`0` = none, `1` = solid, ...). |

### Shadow cells

| Field                              | ShapeSheet cell        | Purpose                                       |
| ---------------------------------- | ---------------------- | --------------------------------------------- |
| `FillShadowForeground`             | `ShdwForegnd`          | Shadow foreground color.                      |
| `FillShadowForegroundTransparency` | `ShdwForegndTrans`     | Shadow foreground transparency.               |
| `FillShadowBackground`             | `ShdwBkgnd`            | Shadow background color.                      |
| `FillShadowBackgroundTransparency` | `ShdwBkgndTrans`       | Shadow background transparency.               |
| `FillShadowPattern`                | `ShdwPattern`          | Shadow fill-pattern index.                    |
| `FillShadowOffsetX`                | `ShapeShdwOffsetX`     | Shadow horizontal offset.                     |
| `FillShadowOffsetY`                | `ShapeShdwOffsetY`     | Shadow vertical offset.                       |
| `FillShadowObliqueAngle`           | `ShapeShdwObliqueAngle`| Shadow oblique (skew) angle.                  |
| `FillShadowScaleFactor`            | `ShapeShdwScaleFactor` | Shadow scale factor.                          |
| `FillShadowType`                   | `ShapeShdwType`        | Shadow type (drop / oblique / inner / etc.).  |

### Line cells

| Field                   | ShapeSheet cell      | Purpose                                                       |
| ----------------------- | -------------------- | ------------------------------------------------------------- |
| `LineColor`             | `LineColor`          | Line color.                                                   |
| `LineColorTransparency` | `LineColorTrans`     | Line color transparency.                                      |
| `LinePattern`           | `LinePattern`        | Line pattern index (`0` = none, `1` = solid, dashed, ...).    |
| `LineWeight`            | `LineWeight`         | Line thickness.                                               |
| `LineCap`               | `LineCap`            | End-cap style (round / square / extended).                    |
| `LineRounding`          | `Rounding`           | Corner-rounding radius for the line.                          |
| `LineBeginArrow`        | `BeginArrow`         | Begin-endpoint arrow style.                                   |
| `LineBeginArrowSize`    | `BeginArrowSize`     | Begin-endpoint arrow size.                                    |
| `LineEndArrow`          | `EndArrow`           | End-endpoint arrow style.                                     |
| `LineEndArrowSize`      | `EndArrowSize`       | End-endpoint arrow size.                                      |

All fields are `Core.CellValue`-typed and accept implicit conversions from strings, ints, doubles, and bools.

## Set fill and line formatting

Construct the record, set the fields you want, hand it to a `SrcWriter`, commit. Cells you don't set are left untouched.

```csharp
var fmt = new VisioAutomation.Shapes.ShapeFormatCells();
fmt.FillForeground = "rgb(255,128,0)";
fmt.FillPattern    = 1;
fmt.LineColor      = "rgb(0,0,0)";
fmt.LineWeight     = "0.5 pt";
fmt.LinePattern    = 1;

var writer = new VisioAutomation.ShapeSheet.Writers.SrcWriter();
writer.SetValues(fmt);
writer.Commit(shape, VisioAutomation.Core.CellValueType.Formula);
```

See the `SrcConstants.Fill*` / `SrcConstants.Line*` / `SrcConstants.FillShadow*` constants for the underlying cell list.

## Read existing values

`GetCells` returns a single `ShapeFormatCells` instance for one shape, or a `CellRecords<ShapeFormatCells>` for many shapes on a page in one batched call.

```csharp
var fmt = VisioAutomation.Shapes.ShapeFormatCells.GetCells(
    shape,
    VisioAutomation.Core.CellValueType.Result);

System.Console.WriteLine($"Fill foreground = {fmt.FillForeground.Value}");
```

Pass `CellValueType.Formula` to read the underlying formulas instead of evaluated results.

## See also

* [Shape cells overview](shape-cells.md): the three `Shape*Cells` records at a glance.
* [Shape XForm cells](shape-xform-cells.md), [Shape layout cells](shape-layout-cells.md): the other two records; combine multiple records in one writer.
* [Modify the ShapeSheet](shapesheet/modify-the-shapesheet.md): the underlying `SrcWriter` / `SidSrcWriter` API.
