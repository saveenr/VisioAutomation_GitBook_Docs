# Page format cells

`VisioAutomation.Pages.PageFormatCells` covers the cells in a page's **Page Properties** section &mdash; size, drawing scale, drawing-size type, page-shadow cells, and the ResizeType / UIVisibility flags. It is one of the four PageSheet records; the others are [Page layout cells](layout.md), [Page print cells](print.md), and [Page ruler and grid cells](ruler-grid.md).

## The `PageFormatCells` record

### Size and scale

| Field               | ShapeSheet cell    | Purpose                                                                        |
| ------------------- | ------------------ | ------------------------------------------------------------------------------ |
| `Width`             | `PageWidth`        | Page width.                                                                    |
| `Height`            | `PageHeight`       | Page height.                                                                   |
| `Scale`             | `PageScale`        | Drawing scale (one ruler unit = `Scale` real-world units).                     |
| `DrawingScale`      | `DrawingScale`     | Real-world distance one drawing unit represents.                               |
| `DrawingScaleType`  | `DrawingScaleType` | The kind of scale (no scale / mechanical / architectural / metric / custom).   |
| `DrawingSizeType`   | `DrawingSizeType`  | Page-size mode (same as printer / fit to drawing / standard / custom / etc.).  |
| `DrawingResizeType` | `DrawingResizeType`| Resize-page-on-drop behavior. *(Visio 2010+.)*                                 |
| `InhibitSnap`       | `InhibitSnap`      | If `1`, snapping is suppressed for the page.                                    |
| `UIVisibility`      | `UIVisibility`     | Whether the page is shown in the page-tab UI (e.g. background pages can hide). |

### Page-shadow cells

| Field                 | ShapeSheet cell      | Purpose                                       |
| --------------------- | -------------------- | --------------------------------------------- |
| `ShadowOffsetX`       | `PageShdwOffsetX`    | Horizontal offset of page-level shadows.      |
| `ShadowOffsetY`       | `PageShdwOffsetY`    | Vertical offset of page-level shadows.        |
| `ShadowObliqueAngle`  | `PageShdwObliqueAngle` | Oblique (skew) angle for page-level shadows. |
| `ShadowScaleFactor`   | `PageShdwScaleFactor`  | Scale factor for page-level shadows.          |
| `ShadowType`          | `PageShdwType`       | Page-shadow type (drop / oblique / inner / etc.). |

All fields are `Core.CellValue`-typed and accept implicit conversions from strings, ints, doubles, and bools.

## Set page size

Construct the record, set the fields you want, hand it to a `SrcWriter`, commit. Cells you don't set are left untouched. PageSheet writes use the same `SrcWriter` as Shape writes &mdash; just commit against `page.PageSheet` instead of a shape.

```csharp
var fmt = new VisioAutomation.Pages.PageFormatCells();
fmt.Width  = 11;
fmt.Height = 8.5;

var writer = new VisioAutomation.ShapeSheet.Writers.SrcWriter();
writer.SetValues(fmt);
writer.Commit(page.PageSheet, VisioAutomation.Core.CellValueType.Formula);
```

For a one-liner that just sets size, use [`PageHelper.SetSize` / `GetSize`](helper.md) instead.

## Read existing values

`GetCells` returns a single `PageFormatCells` instance for one page (pass the page's `PageSheet` shape).

```csharp
var fmt = VisioAutomation.Pages.PageFormatCells.GetCells(
    page.PageSheet,
    VisioAutomation.Core.CellValueType.Result);

System.Console.WriteLine($"Width = {fmt.Width.Value}");
```

Pass `CellValueType.Formula` to read the underlying formulas instead of evaluated results.

## See also

* [Page cells overview](../page-cells.md) &mdash; the four PageSheet records at a glance.
* [Page helper](helper.md) &mdash; `SetSize` / `GetSize` shortcut and other page-level operations.
* [Page layout cells](layout.md), [Page print cells](print.md), [Page ruler and grid cells](ruler-grid.md) &mdash; the other PageSheet records.
* [Modify the ShapeSheet](../shapesheet/modify-the-shapesheet.md) &mdash; the underlying `SrcWriter` API.
