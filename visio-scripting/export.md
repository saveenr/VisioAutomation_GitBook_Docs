# client.Export

`client.Export` saves a page or a selection to an image file (PNG, JPEG, GIF, EMF, SVG, depending on the extension), or to an XHTML document with the SVG embedded inline. The image-export side of these calls thinly wraps Visio's own `Page.Export` and `Selection.Export`; the HTML form layers an XHTML wrapper around an SVG export.

## Methods

| Method                                                                | Returns | Notes                                                                                |
| --------------------------------------------------------------------- | ------- | ------------------------------------------------------------------------------------ |
| `ExportPageToImage(TargetPage, string filename)`                      | `void`  | File format is selected by the extension (`.png`, `.svg`, `.emf`, etc.).             |
| `ExportSelectionToImage(TargetSelection, string filename)`            | `void`  | Exports only the resolved selection's bounding region.                               |
| `ExportSelectionToHtml(TargetSelection, string filename)`             | `void`  | Writes XHTML with the selection's SVG embedded inline; uses a temp file internally.  |

`TargetPage.Auto` resolves to the active page; `TargetSelection.Auto` resolves to the active window's selection. Exports to existing files overwrite without prompting.

## Example

```csharp
// Active page to PNG
client.Export.ExportPageToImage(VisioScripting.TargetPage.Auto,
                                  @"C:\out\page.png");

// Active page to SVG
client.Export.ExportPageToImage(VisioScripting.TargetPage.Auto,
                                  @"C:\out\page.svg");

// Just the current selection to embedded-SVG XHTML
client.Export.ExportSelectionToHtml(VisioScripting.TargetSelection.Auto,
                                      @"C:\out\selection.html");
```

## See also

- [VisioScripting.Client overview](../visio-scripting.md): how `client.Export` fits into the facade.
- [Selection](selection.md): change the selection that `ExportSelection*` consumes.
- [Page](page.md): the page being exported.
- [Output](output.md): `ExportSelectionToHtml` writes progress messages via `client.Output`.
