# Text-block cells

`VisioAutomation.Text.TextBlockCells` covers the box-level text settings stored in a shape's **Text Block** ShapeSheet section: margins, vertical alignment, default tab stop, background fill, and writing direction. Unlike [Character cells](character.md) and [Paragraph cells](paragraph.md), which can hold many rows per shape, the Text Block section has exactly one row.

It is one of the three text-formatting cell records; the others are [Character cells](character.md) and [Paragraph cells](paragraph.md). Tab stops within the block are handled separately by [`TextHelper`](tab-stops.md).

## The `TextBlockCells` record

| Field                    | ShapeSheet cell      | Purpose                                                                                  |
| ------------------------ | -------------------- | ---------------------------------------------------------------------------------------- |
| `LeftMargin`             | `LeftMargin`         | Left padding inside the text box.                                                        |
| `RightMargin`            | `RightMargin`        | Right padding.                                                                           |
| `TopMargin`              | `TopMargin`          | Top padding.                                                                             |
| `BottomMargin`           | `BottomMargin`       | Bottom padding.                                                                          |
| `VerticalAlign`          | `VerticalAlign`      | `0` = top, `1` = middle, `2` = bottom.                                                    |
| `DefaultTabStop`         | `DefaultTabStop`     | Default tab-stop width when no per-stop entries are defined (see [Tab stops](tab-stops.md)).|
| `Background`             | `TextBkgnd`          | Text-block background color (or `0` for none).                                            |
| `BackgroundTransparency` | `TextBkgndTrans`     | Background transparency (`0.0` = opaque, `1.0` = invisible).                              |
| `Direction`              | `TextDirection`      | Writing direction (`0` = horizontal, `1` = vertical).                                      |

All fields are `Core.CellValue`-typed and accept implicit conversions from strings, ints, doubles, and bools.

## Set text-block formatting

Construct the record, set the fields you want, hand it to a `SrcWriter`, commit. Cells you don't set are left untouched.

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

## Read existing values

The reader methods on this record are named **`GetTextBlockCells`** (not `GetCells` like the other text records). The single-shape overload returns one `TextBlockCells` instance; the page-level overload returns a `CellRecords<TextBlockCells>` for many shapes in one batched call.

```csharp
var tb = VisioAutomation.Text.TextBlockCells.GetTextBlockCells(
    shape,
    VisioAutomation.Core.CellValueType.Result);

System.Console.WriteLine($"VerticalAlign = {tb.VerticalAlign.Value}");
```

```csharp
// Many shapes on a page in one batched call:
var shapeids = new[] { shape1.ID, shape2.ID };
var rows = VisioAutomation.Text.TextBlockCells.GetTextBlockCells(
    page, shapeids, VisioAutomation.Core.CellValueType.Result);
```

Pass `CellValueType.Formula` to read the underlying formulas instead of evaluated results.

## See also

* [Text formatting overview](../text-formatting.md): the three text-formatting records at a glance.
* [Character cells](character.md), [Paragraph cells](paragraph.md), [Tab stops](tab-stops.md): the other text-formatting records.
* [Modify the ShapeSheet](../shapesheet/modify-the-shapesheet.md): the underlying `SrcWriter` API.
