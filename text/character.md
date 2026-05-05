# Character cells

`VisioAutomation.Text.CharacterCells` covers the per-run character properties stored in a shape's **Character** ShapeSheet section &mdash; font, size, color, weight / italic / underline, language, etc. It is one of the three text-formatting cell records; the others are [Paragraph cells](paragraph.md) and [Text-block cells](block.md). Tab stops are handled separately by [`TextHelper`](tab-stops.md).

A shape can have multiple character rows, one per text run. See [Multiple rows](#multiple-rows) below.

## The `CharacterCells` record

### Common formatting

| Field   | ShapeSheet cell | Purpose                                                                                  |
| ------- | --------------- | ---------------------------------------------------------------------------------------- |
| `Font`  | `Font`          | Font family name or font ID. Use `doc.Fonts["..."].ID` for a fast integer lookup.        |
| `Size`  | `Size`          | Font size (e.g. `"14pt"`).                                                               |
| `Color` | `Color`         | Foreground color (e.g. `"rgb(0,0,128)"`, a theme-color index, or a palette index).       |
| `Style` | `Style`         | Bitmask: `1` = bold, `2` = italic, `4` = underline, `8` = small caps. Combine by adding. |
| `Case`  | `Case`          | Force case: `0` = as typed, `1` = upper, `2` = initial caps.                              |
| `Pos`   | `Pos`           | Sub/super-script: `0` = normal, `1` = superscript, `2` = subscript.                       |

### Decorations and spacing

| Field                 | ShapeSheet cell      | Purpose                                                                |
| --------------------- | -------------------- | ---------------------------------------------------------------------- |
| `Strikethru`          | `Strikethru`         | If `1`, draw a strike-through line.                                     |
| `Overline`            | `Overline`           | If `1`, draw an overline.                                              |
| `DoubleStrikethrough` | `DoubleStrikethrough`| If `1`, draw a double strike-through.                                  |
| `DoubleUnderline`     | `DoubleUnderline`    | If `1`, draw a double underline.                                       |
| `FontScale`           | `FontScale`          | Horizontal stretch factor (`1.0` = no stretch).                         |
| `Letterspace`         | `Letterspace`        | Inter-character spacing.                                               |
| `Perpendicular`       | `Perpendicular`      | If `1`, render the run perpendicular to its baseline.                  |
| `ColorTransparency`   | `ColorTrans`         | Transparency of `Color` (`0.0` = opaque, `1.0` = invisible).            |
| `UseVertical`         | `UseVertical`        | If `1`, lay the run out vertically (CJK).                              |

### Localization

| Field               | ShapeSheet cell | Purpose                                                                          |
| ------------------- | --------------- | -------------------------------------------------------------------------------- |
| `LangID`            | `LangID`        | Locale identifier (LCID).                                                        |
| `Locale`            | `Locale`        | Locale value used by `Color` resolution.                                         |
| `LocalizeFont`      | `LocalizeFont`  | If `1`, Visio may swap the font for a localized equivalent.                     |
| `RTLText`           | `RTLText`       | If `1`, render right-to-left.                                                    |
| `AsianFont`         | `AsianFont`     | Asian-text font name / ID.                                                      |
| `ComplexScriptFont` | `CompleScript`  | Complex-script font name / ID.                                                   |
| `ComplexScriptSize` | `CompleScriptSize` | Complex-script font size.                                                    |

All fields are `Core.CellValue`-typed and accept implicit conversions from strings, ints, doubles, and bools.

## Set character formatting

Construct the record, set the fields you want, hand it to a `SrcWriter`, commit. Cells you don't set are left untouched.

```csharp
var ch = new VisioAutomation.Text.CharacterCells();
ch.Font  = "Segoe UI";
ch.Size  = "14pt";
ch.Color = "rgb(0,0,128)";
ch.Style = 1;                 // bold

var writer = new VisioAutomation.ShapeSheet.Writers.SrcWriter();
writer.SetValues(ch);
writer.Commit(shape, VisioAutomation.Core.CellValueType.Formula);
```

## Multiple rows

The Character section can hold multiple rows &mdash; one per text run. Pass a row index to `SetValues` to target a specific row:

```csharp
var run0 = new VisioAutomation.Text.CharacterCells();
run0.Font  = "Segoe UI";
run0.Size  = "10pt";

var run1 = new VisioAutomation.Text.CharacterCells();
run1.Font  = "Segoe UI";
run1.Size  = "10pt";
run1.Style = 1;                // bold for the second run

var writer = new VisioAutomation.ShapeSheet.Writers.SrcWriter();
writer.SetValues(run0, 0);
writer.SetValues(run1, 1);
writer.Commit(shape, VisioAutomation.Core.CellValueType.Formula);
```

The same multi-row pattern applies to `ParagraphCells` (one row per paragraph).

## Read existing values

`GetCells(shape, CellValueType)` returns a `CellRecords<CharacterCells>` &mdash; one entry per character row on the shape. The page-level overload `GetCells(page, ShapeIDPairs, CellValueType)` reads many shapes in one batched call.

```csharp
// All character rows on one shape:
var rows = VisioAutomation.Text.CharacterCells.GetCells(
    shape,
    VisioAutomation.Core.CellValueType.Result);

foreach (var row in rows)
{
    System.Console.WriteLine($"Row: font={row.Font.Value} size={row.Size.Value}");
}
```

Pass `CellValueType.Formula` to read the underlying formulas instead of evaluated results.

## See also

* [Text formatting overview](../text-formatting.md) &mdash; the three text-formatting records at a glance.
* [Paragraph cells](paragraph.md), [Text-block cells](block.md), [Tab stops](tab-stops.md) &mdash; the other text-formatting records.
* [Modify the ShapeSheet](../shapesheet/modify-the-shapesheet.md) &mdash; the underlying `SrcWriter` API.
