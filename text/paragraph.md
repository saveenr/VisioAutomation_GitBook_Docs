# Paragraph cells

`VisioAutomation.Text.ParagraphCells` covers the per-paragraph properties stored in a shape's **Paragraph** ShapeSheet section: alignment, indent, spacing, and bullet settings. It is one of the three text-formatting cell records; the others are [Character cells](character.md) and [Text-block cells](block.md). Tab stops are handled separately by [`TextHelper`](tab-stops.md).

A shape can have multiple paragraph rows, one per paragraph. See the [multi-row pattern](character.md#multiple-rows) on the character-cells page; the same `SetValues(record, rowIndex)` overload applies here.

## The `ParagraphCells` record

### Alignment, indent, and spacing

| Field             | ShapeSheet cell     | Purpose                                                                              |
| ----------------- | ------------------- | ------------------------------------------------------------------------------------ |
| `HorizontalAlign` | `HorzAlign`         | `0` = left, `1` = center, `2` = right, `3` = justify, `4` = force-justify.            |
| `IndentLeft`      | `IndLeft`           | Left indent.                                                                         |
| `IndentRight`     | `IndRight`          | Right indent.                                                                        |
| `IndentFirst`     | `IndFirst`          | First-line indent (positive = indented, negative = hanging).                          |
| `SpacingBefore`   | `SpBefore`          | Space above the paragraph.                                                           |
| `SpacingAfter`    | `SpAfter`           | Space below the paragraph.                                                           |
| `SpacingLine`     | `SpLine`            | Line spacing within the paragraph.                                                    |
| `Flags`           | `Flags`             | Bitmask of additional paragraph flags.                                                |

### Bullets

| Field                | ShapeSheet cell      | Purpose                                                                              |
| -------------------- | -------------------- | ------------------------------------------------------------------------------------ |
| `Bullet`             | `Bullet`             | Bullet style index (`0` = none, then a built-in catalogue).                          |
| `BulletString`       | `BulletStr`          | Custom bullet character / string when `Bullet` selects "custom".                     |
| `BulletFont`         | `BulletFont`         | Font used for the bullet character.                                                  |
| `BulletFontSize`     | `BulletFontSize`     | Bullet font size.                                                                    |
| `LocalizeBulletFont` | `LocalizeBulletFont` | If `1`, Visio may swap the bullet font for a localized equivalent.                  |
| `TextPosAfterBullet` | `TextPosAfterBullet` | Distance from the bullet to the start of paragraph text.                              |

All fields are `Core.CellValue`-typed and accept implicit conversions from strings, ints, doubles, and bools.

## Set paragraph formatting

Construct the record, set the fields you want, hand it to a `SrcWriter`, commit. Cells you don't set are left untouched.

```csharp
var pg = new VisioAutomation.Text.ParagraphCells();
pg.HorizontalAlign = 1;       // center
pg.SpacingBefore   = "6 pt";
pg.SpacingAfter    = "6 pt";

var writer = new VisioAutomation.ShapeSheet.Writers.SrcWriter();
writer.SetValues(pg);
writer.Commit(shape, VisioAutomation.Core.CellValueType.Formula);
```

## Read existing values

`GetCells(shape, CellValueType)` returns a `CellRecords<ParagraphCells>`: one entry per paragraph row on the shape. The page-level overload `GetCells(page, ShapeIDPairs, CellValueType)` reads many shapes in one batched call.

```csharp
var rows = VisioAutomation.Text.ParagraphCells.GetCells(
    shape,
    VisioAutomation.Core.CellValueType.Result);

foreach (var row in rows)
{
    System.Console.WriteLine($"Align = {row.HorizontalAlign.Value}");
}
```

Pass `CellValueType.Formula` to read the underlying formulas instead of evaluated results.

## See also

* [Text formatting overview](../text-formatting.md): the three text-formatting records at a glance.
* [Character cells](character.md), [Text-block cells](block.md), [Tab stops](tab-stops.md): the other text-formatting records.
* [Modify the ShapeSheet](../shapesheet/modify-the-shapesheet.md): the underlying `SrcWriter` API.
