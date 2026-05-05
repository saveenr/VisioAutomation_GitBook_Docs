# Text formatting

Visio stores text formatting in three ShapeSheet sections, exposed by VisioAutomation as **three cell records** under `VisioAutomation.Text`, each documented on its own page. Tab stops live in a fourth section with an unusual layout, and are wrapped by a separate helper.

## The three text-formatting records

| Record           | ShapeSheet section | Covers                                                                                              | Page                                          |
| ---------------- | ------------------ | --------------------------------------------------------------------------------------------------- | --------------------------------------------- |
| `CharacterCells` | Character          | Per-run character properties: font, size, color, weight / italic / underline, language, etc.        | [Character cells](text/character.md)          |
| `ParagraphCells` | Paragraph          | Per-paragraph properties: alignment, indent, spacing, bullets.                                      | [Paragraph cells](text/paragraph.md)          |
| `TextBlockCells` | Text Block         | Box-level settings: margins, vertical alignment, default tab stop, background, writing direction.   | [Text-block cells](text/block.md)             |

The Character and Paragraph sections can hold **multiple rows** &mdash; one row per text run / paragraph. The Text Block section has exactly one row.

## Tab stops

The Tabs section uses an unusual three-cells-per-stop layout, so per-stop manipulation through the raw cells is awkward:

* [Tab stops](text/tab-stops.md) &mdash; `TextHelper.SetTabStops` / `GetTabStops` and the typed `TabStop` record / `TabStopAlignment` enum.

For the *default* tab-stop interval (when no per-stop entries are defined), see `TextBlockCells.DefaultTabStop` on the [Text-block cells](text/block.md) page.

## Combine multiple records in one writer

A `SrcWriter` accepts as many `SetValues` calls as you want. Mix records in a single transaction so all the changes commit atomically:

```csharp
var writer = new VisioAutomation.ShapeSheet.Writers.SrcWriter();
writer.SetValues(ch);
writer.SetValues(pg);
writer.SetValues(tb);
writer.Commit(shape, VisioAutomation.Core.CellValueType.Formula);
```

## See also

* [Character cells](text/character.md), [Paragraph cells](text/paragraph.md), [Text-block cells](text/block.md), [Tab stops](text/tab-stops.md) &mdash; the per-record details.
* [Modify the ShapeSheet](shapesheet/modify-the-shapesheet.md) &mdash; the underlying `SrcWriter` API.
* [Custom properties](custom-properties.md), [Lock cells](lock-cells.md) &mdash; other per-shape ShapeSheet records.
