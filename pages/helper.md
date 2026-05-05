# Page helper methods

`VisioAutomation.Pages.PageHelper` is a small static class of one-line shortcuts for common page-level operations &mdash; getting / setting the page size, resizing to fit contents (with optional padding), and duplicating an entire page including its PageSheet cells and shapes.

These are convenience wrappers; for full control over the underlying ShapeSheet cells use the four cell records ([format](format.md), [layout](layout.md), [print](print.md), [ruler-grid](ruler-grid.md)) directly.

## Page size

`SetSize` sets `PageWidth` and `PageHeight` in one call:

```csharp
VisioAutomation.Pages.PageHelper.SetSize(page, new VisioAutomation.Core.Size(11, 8.5));
```

`GetSize` reads them back:

```csharp
var size = VisioAutomation.Pages.PageHelper.GetSize(page);
System.Console.WriteLine($"{size.Width} x {size.Height}");
```

For finer-grained control of size, scale, drawing-units, etc., use [Page format cells](format.md) directly.

## Resize a page to fit its contents

`ResizeToFitContents` runs Visio's native resize-to-fit and then optionally adds padding around the result.

```csharp
var padding = new VisioAutomation.Core.Size(0.5, 0.5);
VisioAutomation.Pages.PageHelper.ResizeToFitContents(page, padding);
```

When the padding is non-zero the helper resizes the page to (resize-to-fit + 2 &times; padding) on each axis and re-centers the contents. Pass `Size.Zero` (or `new Size(0, 0)`) for plain resize-to-fit with no padding.

The page-level extension method `page.ResizeToFitContents(padding)` (in [`VisioAutomation.Extensions`](../extensions/coordinates.md)) calls this same helper.

## Duplicate a page

`Duplicate(src, dest)` copies all PageSheet cells and all shapes from `src_page` to `dest_page`. The source page must be the active page; the destination must be a different page.

```csharp
var src  = doc.Pages[1];
var dest = doc.Pages.Add();
VisioAutomation.Pages.PageHelper.Duplicate(src, dest);
```

`Duplicate` copies a curated list of PageSheet cells (size, scale, print margins, ruler / grid, layout, page shadow). Cells outside that list (custom user-defined cells on the PageSheet, for example) are not copied.

## See also

* [Page cells overview](../page-cells.md) &mdash; the four PageSheet records at a glance.
* [Coordinates and bounding boxes](../extensions/coordinates.md) &mdash; the matching extension method that wraps `ResizeToFitContents`.
