# Lock cells

The `Protection.` section of a shape's ShapeSheet holds 20 named **lock cells** (`LockMoveX`, `LockSelect`, `LockTextEdit`, &hellip;) that tell Visio to refuse certain user operations on the shape. VisioAutomation exposes them via the `VisioAutomation.Shapes.LockCells` record. Set a cell to `1` to enable the lock, `0` to clear it.

There is no separate "helper" class &mdash; you just construct a `LockCells` and write it through the standard ShapeSheet writer.

## The `LockCells` record

| Field | ShapeSheet cell | Effect |
| --- | --- | --- |
| `Aspect` | `LockAspect` | Preserve aspect ratio when sized. |
| `Begin` | `LockBegin` | Begin endpoint cannot be moved. |
| `CalcWH` | `LockCalcWH` | Width / height formulas can't be recalculated. |
| `Crop` | `LockCrop` | Cropping is disabled. |
| `CustProp` | `LockCustomProp` | Custom (shape-data) properties cannot be edited. |
| `Delete` | `LockDelete` | Shape cannot be deleted. |
| `End` | `LockEnd` | End endpoint cannot be moved. |
| `Format` | `LockFormat` | Formatting is locked. |
| `FromGroupFormat` | `LockFromGroupFormat` | Group formatting cannot reach the shape. |
| `Group` | `LockGroup` | Cannot be grouped or ungrouped. |
| `Height` | `LockHeight` | Height cannot be changed. |
| `MoveX` | `LockMoveX` | Cannot be moved horizontally. |
| `MoveY` | `LockMoveY` | Cannot be moved vertically. |
| `Rotate` | `LockRotate` | Cannot be rotated. |
| `Select` | `LockSelect` | Cannot be selected. |
| `TextEdit` | `LockTextEdit` | Text cannot be edited. |
| `ThemeColors` | `LockThemeColors` | Theme color changes don't apply. |
| `ThemeEffects` | `LockThemeEffects` | Theme effect changes don't apply. |
| `VertexEdit` | `LockVertexEdit` | Geometry vertices cannot be edited. |
| `Width` | `LockWidth` | Width cannot be changed. |

All fields are `Core.CellValue`-typed.

## Apply locks to a shape

Construct a `LockCells` with the locks you want, hand it to a `SrcWriter`, commit. Cells you don't set are left untouched.

```csharp
var locks = new VisioAutomation.Shapes.LockCells();
locks.MoveX  = 1;
locks.MoveY  = 1;
locks.Width  = 1;
locks.Height = 1;

var writer = new VisioAutomation.ShapeSheet.Writers.SrcWriter();
writer.SetValues(locks);
writer.Commit(shape, VisioAutomation.Core.CellValueType.Formula);
```

To clear a lock, set the field to `0` instead of `1`.

## Read the current lock state

`GetCells` reads the lock cells off a single shape:

```csharp
var locks = VisioAutomation.Shapes.LockCells.GetCells(
    shape,
    VisioAutomation.Core.CellValueType.Formula);

System.Console.WriteLine($"Width-locked: {locks.Width.Value}");
```

The multi-shape overload reads from a page in one batched call:

```csharp
var locks_per_shape = VisioAutomation.Shapes.LockCells.GetCells(
    page, shapeids, VisioAutomation.Core.CellValueType.Formula);
```

## See also

* [Custom properties](custom-properties.md) &mdash; the read/write pattern is the same shape (cell record + helper).
* [ShapeSheet &mdash; modify](shapesheet/modify-the-shapesheet.md) &mdash; the underlying `SrcWriter` flow.
