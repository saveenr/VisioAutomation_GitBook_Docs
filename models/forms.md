# Form pages

`VisioAutomation.Models.Documents.Forms` is a small generator for **printable, document-style Visio pages**: think one-pagers with a title, a body, and a margin. The output is a Visio document where each page has a single fixed-size text block with a title and a paragraph, suitable for printing or PDF export.

This is a niche but useful escape hatch for using Visio as a layout-and-print engine when full Word or LaTeX would be overkill, and you already have the data in a Visio-adjacent pipeline.

## Hello-world

A one-page document with a title and body:

```csharp
using VAFORMS = VisioAutomation.Models.Documents.Forms;
using VA = VisioAutomation;

var form = new VAFORMS.FormDocument();
form.Title = "Quarterly summary";
form.Subject = "Q1 2026 internal";
form.Creator = "Alex";
form.Company = "ExampleCorp";

var page = new VAFORMS.FormPage();
page.Name = "Summary";
page.Title = "Q1 2026 Summary";
page.Body  = "This page summarises the Q1 results...\n\n" +
             "Section 2. Highlights of the quarter were..." ;

form.Pages.Add(page);

form.Render(visioApp);
```

`Render(IVisio.Application)` creates a new Visio document, sets the document-level metadata (`Subject`, `Title`, `Creator`, `Company`), then walks `Pages` and draws each one.

## Per-page properties

`FormPage` has a small but specific set of knobs:

| Property | Default | Effect |
| :--- | :--- | :--- |
| `Name` | (none) | Visio page name. |
| `Size` | `8.5 x 11` (US Letter) | Page dimensions in inches. |
| `PageMargin` | `0.5` on each side | Margin (inches) used by the renderer to position the title and body shapes. |
| `Title` | (none) | Title string, drawn as a top-of-page text block. |
| `Body` | (none) | Body string, drawn as a fixed-width text block below the title. Newlines become paragraph breaks. |
| `TitleTextSize` | `15.0` | Title font size in points. |
| `BodyTextSize` | `8.0` | Body font size in points. |
| `BodyParaSpacingAfter` | `0.0` | Trailing spacing (points) between paragraphs in the body. |
| `DefaultFont` | `Segoe UI` | Font for both title and body. Resolved through Visio's font table at render. |

After render, `FormPage.VisioPage` holds the underlying COM object so you can perform follow-up edits (e.g. add a header, drop a corporate logo).

## Multiple pages

`FormDocument.Pages` is a list; add as many `FormPage` objects as needed and they render in order:

```csharp
var p1 = new VAFORMS.FormPage { Name = "Cover", Title = "Title page", Body = "" };
var p2 = new VAFORMS.FormPage { Name = "Summary", Title = "Summary", Body = "..." };
var p3 = new VAFORMS.FormPage { Name = "Detail", Title = "Detail", Body = "..." };
form.Pages.Add(p1);
form.Pages.Add(p2);
form.Pages.Add(p3);

form.Render(app);
```

The renderer also deletes the implicit blank first page that `Documents.Add("")` creates, so the output document contains exactly the pages you added.

## InteractiveRenderer and TextBlock

Internally `FormPage.Draw()` constructs an `InteractiveRenderer` that emits `TextBlock` shapes one after another, each on its own line. Both types are public; you can use them directly if you want to lay out a page with multiple text blocks of different sizes:

```csharp
var renderer = new VAFORMS.InteractiveRenderer(visioDoc);
var page = renderer.CreatePage(formPage);

var heading = new VAFORMS.TextBlock(new VA.Core.Size(7.5, 0.5), "Section heading");
heading.CharacterCells.Size = "12pt";
renderer.AddShape(heading);
renderer.Linefeed();

var paragraph = new VAFORMS.TextBlock(new VA.Core.Size(7.5, 2.0), "Lorem ipsum...");
renderer.AddShape(paragraph);
renderer.Linefeed();

renderer.Finish();
```

This is the right level of abstraction when you need a structured form (heading, then content, then a sub-heading, then a list) without going all the way down to the [DOM](dom.md).

## When to use Forms vs. the DOM

* **Forms** if your output is "documents pretending to be Visio diagrams": print-targeted layouts with predictable text-block geometry. Forms are simpler and more direct for that use case.
* **The [DOM](dom.md)** if your output is a real Visio diagram with shapes, connectors, masters, and shapesheet styling. Forms doesn't support any of that.

For mixed cases, render with Forms first then drop into `FormPage.VisioPage` to add diagram shapes via the imperative API or the DOM.

## See also

* [Declarative DOM](dom.md) (general-purpose declarative shape model)
* [Text formatting](../text-formatting.md) (the cell vocabulary `TextBlock` writes to)
* [Page cells](../page-cells.md) (page-level cells that affect text rendering)
