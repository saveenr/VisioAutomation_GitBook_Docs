# Text formatting

Visio stores text formatting in three ShapeSheet sections:

* **Character** &mdash; font, size, color, weight, italic, underline, strike-through, etc.
* **Paragraph** &mdash; alignment, indent, spacing, bullets.
* **Text Block** &mdash; the text-box-level settings (margins, vertical alignment, default tab stop, background).

VisioAutomation exposes these as cell records under `VisioAutomation.Text`, plus tab-stop helpers on `TextHelper`.

## Character formatting

`CharacterCells` covers the per-run character properties. Set the fields you want and commit through a `SrcWriter`:

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

A shape can have multiple character rows (one per text run); see [Multiple rows](#multiple-rows-character-and-paragraph) below.

## Paragraph formatting

`ParagraphCells` covers indent / spacing / alignment / bullet settings:

```csharp
var pg = new VisioAutomation.Text.ParagraphCells();
pg.HorizontalAlign = 1;       // center
pg.SpacingBefore   = "6 pt";
pg.SpacingAfter    = "6 pt";

var writer = new VisioAutomation.ShapeSheet.Writers.SrcWriter();
writer.SetValues(pg);
writer.Commit(shape, VisioAutomation.Core.CellValueType.Formula);
```

## Text-block formatting

`TextBlockCells` is the box-level settings &mdash; margins, vertical alignment, default tab stop, background:

```csharp
var tb = new VisioAutomation.Text.TextBlockCells();
tb.LeftMargin    = "4 pt";
tb.RightMargin   = "4 pt";
tb.TopMargin     = "2 pt";
tb.BottomMargin  = "2 pt";
tb.VerticalAlign = 1;          // middle

var writer = new VisioAutomation.ShapeSheet.Writers.SrcWriter();
writer.SetValues(tb);
writer.Commit(shape, VisioAutomation.Core.CellValueType.Formula);
```

## Multiple rows (character and paragraph)

Character and paragraph sections can hold multiple rows &mdash; one per text run / paragraph. Pass a row index to `SetValues` to target a specific row:

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

## Read existing values

Each record has `GetCells(shape, CellValueType)` and `GetCells(page, ShapeIDPairs, CellValueType)` overloads:

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

## Tab stops

`TextHelper` provides a typed `TabStop` record and read/write methods that hide the unusual three-cells-per-stop layout of the `Tabs.` section:

```csharp
var stops = new System.Collections.Generic.List<VisioAutomation.Text.TabStop>
{
    new VisioAutomation.Text.TabStop(1.0, VisioAutomation.Text.TabStopAlignment.Left),
    new VisioAutomation.Text.TabStop(2.5, VisioAutomation.Text.TabStopAlignment.Right),
};

VisioAutomation.Text.TextHelper.SetTabStops(shape, stops);

// Read them back
var existing = VisioAutomation.Text.TextHelper.GetTabStops(shape);
```

## See also

* [Modify the ShapeSheet](shapesheet/modify-the-shapesheet.md) &mdash; the underlying `SrcWriter` API.
* [Custom properties](custom-properties.md), [Lock cells](lock-cells.md) &mdash; other per-shape ShapeSheet records.
