# Hyperlinks

A Visio shape can carry one or more **hyperlinks** — targets that the user can navigate to from the shape (URLs, file paths, or sub-addresses such as a specific page or bookmark). VisioAutomation exposes them via `VisioAutomation.Shapes.HyperlinkHelper` and the `HyperlinkCells` record.

## The `HyperlinkCells` record

`HyperlinkCells` mirrors the fields of one row in the `Hyperlink.` ShapeSheet section. All fields are `Core.CellValue`-typed and accept implicit conversions from strings:

| Field         | Purpose                                                         |
| ------------- | --------------------------------------------------------------- |
| `Address`     | The hyperlink target (URL, file path). Required when adding.    |
| `SubAddress`  | Sub-target inside `Address` (e.g. `"Page-2"` for a Visio page). |
| `Description` | Human-readable label shown to the user.                         |
| `ExtraInfo`   | Extra parameters appended to the URL.                           |
| `Frame`       | Target frame for browser navigation.                            |
| `SortKey`     | Sort key when a shape has several hyperlinks.                   |

## Add a hyperlink

`Add` creates a new hyperlink row and returns the row index Visio assigned it.

```csharp
var hl = new VisioAutomation.Shapes.HyperlinkCells();
hl.Address = "https://learn.microsoft.com";
hl.Description = "Microsoft Learn";

int row = VisioAutomation.Shapes.HyperlinkHelper.Add(shape, hl);
```

The `Address` field is required — `Add` throws if it's null.

## Update an existing hyperlink

`Set` writes a `HyperlinkCells` to a known row index (returned from `Add` or by walking the section). The row index is a `short`; cast when reusing the `int` returned by `Add`.

```csharp
var hl = new VisioAutomation.Shapes.HyperlinkCells();
hl.Address = "https://example.com";
hl.Description = "Updated description";

VisioAutomation.Shapes.HyperlinkHelper.Set(shape, (short)row, hl);
```

## Count hyperlinks

```csharp
int count = VisioAutomation.Shapes.HyperlinkHelper.GetCount(shape);
```

## Delete a hyperlink

```csharp
VisioAutomation.Shapes.HyperlinkHelper.Delete(shape, 0);   // first hyperlink
```

`Delete` removes the row at the given zero-based index.

## See also

* [`Custom properties`](custom-properties.md) — the parallel mechanism for arbitrary shape metadata.
