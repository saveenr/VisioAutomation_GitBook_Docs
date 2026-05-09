# client.Page

`client.Page` covers page-level operations: create, delete, navigate, query dimensions, set orientation, set background, lay out shapes. Each method takes a `Target*` parameter (`TargetPage`, `TargetPages`, or `TargetDocument`) that resolves to the relevant page-or-pages at call time.

## Methods

| Method                                                       | Returns                       | Notes                                                                                            |
| ------------------------------------------------------------ | ----------------------------- | ------------------------------------------------------------------------------------------------ |
| `GetActivePage()`                                            | `IVisio.Page`                 | Returns the active page on the active window.                                                    |
| `SetActivePage(IVisio.Page page)`                            | `void`                        | Makes the given page the active page on the active window.                                       |
| `NewPage(TargetDocument, Size? size, bool isbackgroundpage)` | `IVisio.Page`                 | Creates a new page; `size` optional; `isbackgroundpage` flags the new page as a background page. |
| `DeletePages(TargetPages, bool renumber)`                    | `void`                        | Deletes the resolved pages; `renumber` rebuilds page numbering after deletion.                   |
| `DuplicatePage(TargetPage)`                                  | `IVisio.Page`                 | Duplicates the resolved page in the same document; activates the duplicate.                      |
| `DuplicatePageToDocument(TargetPage, IVisio.Document dest)`  | `IVisio.Page`                 | Duplicates into a different document; throws if `dest` equals the source page's document.        |
| `SetPageBackground(TargetPages, string bg_page_name)`        | `void`                        | Sets the named background page as the back page of each resolved page.                           |
| `GetPageSize(TargetPages)`                                   | `List<Core.Size>`             | One size per resolved page.                                                                      |
| `GetPageDimensions(TargetPages)`                             | `List<Models.PageDimensions>` | One full dimension record per page (size + print margins).                                       |
| `GetPageOrientation(TargetPage)`                             | `Models.PageOrientation`      | `Portrait` or `Landscape`.                                                                       |
| `SetPageOrientation(TargetPages, PageOrientation)`           | `void`                        | Swaps Width and Height to match the new orientation; no-op when already matching.                |
| `ResizePageToFitContents(TargetPages, Size bordersize)`      | `void`                        | Resizes each page tight around its content with `bordersize` of padding.                         |
| `SetFormatCells(TargetPages, PageFormatCells cells)`         | `void`                        | Writes the page-format cell-record on each resolved page (with `BlastGuards`).                   |
| `LayoutPage(TargetPages, LayoutStyleBase layout)`            | `void`                        | Applies a [layout style](../models/layout-styles.md) to the resolved pages.                      |
| `FindPagesInDocument(TargetDocument, string name)`           | `List<IVisio.Page>`           | Wildcard match on page name (`null` or `"*"` means all pages).                                   |
| `GetShapesOnPage(TargetPage)`                                | `List<IVisio.Shape>`          | All top-level shapes on the resolved page.                                                       |
| `GetShapesOnPageByID(TargetPage, int[] shapeids)`            | `List<IVisio.Shape>`          | Lookup by Visio ID; preserves the requested order.                                               |
| `GetShapesOnPageByName(TargetPage, string[] names)`          | `List<IVisio.Shape>`          | Wildcard match on shape names; passing `"*"` short-circuits to all shapes.                       |

`Target*.Auto` resolves to the active page or pages at call time; pass an `IVisio.Page` (or array) directly when you have a specific page in mind.

## Removal candidates (CY27)

These methods are currently `public` but have no known consumers as of 2026-05. They are candidates for removal in CY27 (see [issue #183](https://github.com/saveenr/VisioAutomation/issues/183)). New code should not depend on them.

| Method                                                       | Notes                                                                                |
| ------------------------------------------------------------ | ------------------------------------------------------------------------------------ |
| `SetActivePage(TargetDocument, PageRelativePosition)`        | Navigate by relative position (`Next`, `Previous`, `First`, `Last`).                 |
| `SetPageSize(TargetPages, Core.Size new_size)`               | Use `NewPage(targetdoc, size, …)` for new pages; mutate via `SetFormatCells` if needed. |
| `SetPageSize(TargetPage, double? width, double? height)`     | As above; the optional-component variant lets you pass only width or only height.    |

## Example

```csharp
// Create a Letter-sized page in the active document
var newpage = client.Page.NewPage(VisioScripting.TargetDocument.Auto,
                                  new VisioAutomation.Core.Size(8.5, 11),
                                  isbackgroundpage: false);

// Switch to landscape orientation across every page in the active document
client.Page.SetPageOrientation(VisioScripting.TargetPages.Auto,
                                VisioScripting.Models.PageOrientation.Landscape);

// Read each page's full dimension record
var dims = client.Page.GetPageDimensions(VisioScripting.TargetPages.Auto);
foreach (var d in dims)
{
    System.Console.WriteLine("Page {0}: {1} x {2} (margins L={3} R={4})",
        d.PageID, d.PageWidth, d.PageHeight,
        d.PrintLeftMargin, d.PrintRightMargin);
}

// Tighten the active page around its content with a 0.25" border
client.Page.ResizePageToFitContents(VisioScripting.TargetPages.Auto,
                                    new VisioAutomation.Core.Size(0.25, 0.25));
```

## See also

- [Page cells](../page-cells.md): the lower-level page-cells API (format, layout, print, ruler / grid).
- [VisioScripting.Client overview](../visio-scripting.md): how `client.Page` fits into the facade.
- [Layout styles](../models/layout-styles.md): the layouts that `LayoutPage` consumes.
- [Application](../application.md): `client.Application` operates on the parent Visio process; `client.Document` operates on documents (which contain pages).
