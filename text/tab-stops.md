# Tab stops

Tab stops live in a shape's **Tabs** ShapeSheet section. Each stop is stored as **three consecutive cells** in one row (position, alignment, and an "other" cell), so per-stop manipulation through the raw cells is awkward. `VisioAutomation.Text.TextHelper` hides that layout behind a typed `TabStop` record and read / write methods.

This is one of the four text-formatting topics; the others are [Character cells](character.md), [Paragraph cells](paragraph.md), and [Text-block cells](block.md).

## The `TabStop` record

```csharp
public struct TabStop
{
    public double Position { get; }
    public TabStopAlignment Alignment { get; }

    public TabStop(double pos, TabStopAlignment align);
}
```

`TabStopAlignment` values:

| Value     | Meaning                                |
| --------- | -------------------------------------- |
| `Left`    | Text is left-aligned at the stop.      |
| `Center`  | Text is centered around the stop.      |
| `Right`   | Text is right-aligned at the stop.     |
| `Decimal` | Decimal points line up at the stop.    |
| `Comma`   | Commas line up at the stop (locale-dependent decimal). |

For the *default* tab-stop interval (when no per-stop entries are defined), see `TextBlockCells.DefaultTabStop` on the [Text-block cells](block.md) page.

## Set tab stops

`SetTabStops` replaces all stops on the shape with the supplied list. Pass an empty list to clear them.

```csharp
var stops = new System.Collections.Generic.List<VisioAutomation.Text.TabStop>
{
    new VisioAutomation.Text.TabStop(1.0, VisioAutomation.Text.TabStopAlignment.Left),
    new VisioAutomation.Text.TabStop(2.5, VisioAutomation.Text.TabStopAlignment.Right),
};

VisioAutomation.Text.TextHelper.SetTabStops(shape, stops);
```

Internally, `SetTabStops` clears the existing stops, sets the count cell, picks the right `visTagTab*` row tag for the new count (2, 10, or 60), and writes the per-stop cells in one batched `SrcWriter` commit.

## Read tab stops

`GetTabStops` returns the current list of stops on the shape, or an empty list if none are defined.

```csharp
var existing = VisioAutomation.Text.TextHelper.GetTabStops(shape);

foreach (var ts in existing)
{
    System.Console.WriteLine($"{ts.Position} ({ts.Alignment})");
}
```

## See also

* [Text formatting overview](../text-formatting.md) &mdash; the three text-formatting cell records and this helper at a glance.
* [Text-block cells](block.md) &mdash; `DefaultTabStop` on the box governs the *default* spacing when no per-stop entries are defined.
* [Modify the ShapeSheet](../shapesheet/modify-the-shapesheet.md) &mdash; the underlying `SrcWriter` API used by `SetTabStops`.
