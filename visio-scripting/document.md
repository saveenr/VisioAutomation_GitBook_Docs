# client.Document

`client.Document` covers the full document lifecycle: open, create, save, close, find, and activate. Stencils are documents too, so `OpenStencilDocument` lives here as well. For the related stencil and master concepts, see [Stencils and masters](../stencils-and-masters.md).

## Methods

| Method                                                                  | Returns                  | Notes                                                                                |
| ----------------------------------------------------------------------- | ------------------------ | ------------------------------------------------------------------------------------ |
| `HasActiveDocument`                                                     | `bool` (property)        | True when a drawing or master window is active and ready for use.                    |
| `ActivateDocument(IVisio.Document)`                                     | `void`                   | Brings the window hosting `doc` to the front; throws if no such window exists.       |
| `NewDocument()`                                                         | `IVisio.Document`        | Creates a new blank document.                                                        |
| `NewDocumentFromTemplate(string template)`                              | `IVisio.Document`        | Creates a new document and docks the template's stencils.                            |
| `NewDocument(Size size)`                                                | `IVisio.Document`        | Creates a blank document and resizes page 1 to `size`.                               |
| `OpenDocument(string filename)`                                         | `IVisio.Document`        | Opens an existing `.vsd*` file; throws if the file does not exist.                   |
| `OpenStencilDocument(string name)`                                      | `IVisio.Document`        | Opens a stencil (read-only by default) and docks it.                                 |
| `SaveDocument(TargetDocument)`                                          | `void`                   | Saves the resolved document under its current path.                                  |
| `SaveDocumentAs(TargetDocument, string filename)`                       | `void`                   | Saves the resolved document to a new path.                                           |
| `CloseDocument(TargetDocuments)`                                        | `void`                   | Force-closes each resolved document, suppressing save / reset prompts.               |
| `CloseAllDocumentsWithoutSaving()`                                      | `void`                   | Convenience: force-closes all open drawing documents.                                |
| `FindDocuments(string namepattern)`                                     | `List<IVisio.Document>`  | Wildcard match on document name (`null` returns every open document).                |

`TargetDocument.Auto` resolves to the active document at call time, `TargetDocuments.Auto` to all open documents.

## Removal candidates (CY27)

These methods are currently `public` but have no known consumers as of 2026-05. They are candidates for removal in CY27 (see [issue #183](https://github.com/saveenr/VisioAutomation/issues/183)). New code should not depend on them.

| Method                                                       | Notes                                                                |
| ------------------------------------------------------------ | -------------------------------------------------------------------- |
| `ActivateDocumentWithName(string name)`                      | Looks up by name and delegates to `ActivateDocument`.                |
| `GetDocumentWithName(string name)`                           | Single-doc lookup; the wildcard `FindDocuments(name)` covers this.   |
| `NewDocumentFromTemplate(Size size, string template)`        | Two-arg overload; the parameterless and template-only forms suffice. |

## Example

```csharp
using IVisio = Microsoft.Office.Interop.Visio;

// Create a blank document and a stencil-backed one
var blank = client.Document.NewDocument();
var fromTemplate = client.Document.NewDocumentFromTemplate(@"C:\templates\flowchart.vstx");

// Open an existing drawing
var existing = client.Document.OpenDocument(@"C:\drawings\plan.vsdx");

// Save then close everything
client.Document.SaveDocument(VisioScripting.TargetDocument.Auto);
client.Document.CloseAllDocumentsWithoutSaving();
```

## See also

- [Stencils and masters](../stencils-and-masters.md): stencil documents and the masters they host.
- [Page](page.md): page-level operations on the document's pages.
- [Master](master.md): get and drop masters from a document's stencils.
- [VisioScripting.Client overview](../visio-scripting.md): how `client.Document` fits into the facade.
