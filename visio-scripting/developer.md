# client.Developer

`client.Developer` is a debug-aid group: it renders Visio diagrams of the library's own public types and of the Visio interop enumerations. The output is a regular Visio document that you can save, browse, or print. None of these methods are required for normal Visio automation.

## Methods

| Method                              | Returns           | Notes                                                                                |
| ----------------------------------- | ----------------- | ------------------------------------------------------------------------------------ |
| `DrawScriptingDocumentation()`      | `IVisio.Document` | Renders one page per command group, listing the public method signatures.            |
| `DrawInteropEnumDocumentation()`    | `IVisio.Document` | Renders one page per `Microsoft.Office.Interop.Visio` enum, listing values.          |
| `DrawNamespaces()`                  | `IVisio.Document` | Renders the public namespaces of `VisioAutomation` and `VisioScripting` as a tree.   |
| `DrawNamespacesAndClasses()`        | `IVisio.Document` | As above, with each namespace's public types listed inside its node.                 |

The methods are interactive in the sense that they create and activate a new document on the attached Visio. They require an attached `Application` (they call `GetCommandTarget(RequireApplication)` internally).

## Removal candidates (CY27)

These methods are currently `public` but have no known consumers as of 2026-05. They are candidates for removal in CY27 (see [issue #183](https://github.com/saveenr/VisioAutomation/issues/183)). New code should not depend on them.

| Method                                                | Notes                                                                       |
| ----------------------------------------------------- | --------------------------------------------------------------------------- |
| `DrawNamespaces(IList<System.Type>)`                  | Type-list overload; the no-arg form already covers the documented case.     |
| `DrawNamespacesAndClasses(IList<System.Type>)`        | Type-list overload; the no-arg form already covers the documented case.     |
| `GetInteropEnums()`                                   | Returns the same `EnumType` list `DrawInteropEnumDocumentation` consumes.   |
| `GetInteropEnum(string name)`                         | Single-name lookup helper for `GetInteropEnums`.                            |
| `GetEnum(System.Type type)`                           | Wraps a `System.Type` as a `Models.EnumType`.                               |

## Example

```csharp
using IVisio = Microsoft.Office.Interop.Visio;

// Render the full scripting surface as a browsable Visio document
var doc = client.Developer.DrawScriptingDocumentation();
System.Console.WriteLine("Wrote {0} pages of scripting docs", doc.Pages.Count);

// Render the namespace tree of the entire library
var nsdoc = client.Developer.DrawNamespaces();
System.Console.WriteLine("Wrote namespace tree to {0}", nsdoc.Name);
```

## See also

- [VisioScripting.Client overview](../visio-scripting.md): how `client.Developer` fits into the facade.
- [Namespaces](../namespaces.md): the static reference list of namespaces these methods render.
- [Classes](../classes.md): the static reference list of classes `DrawNamespacesAndClasses` renders.
- [Form pages](../models/forms.md): the underlying `FormDocument` model these methods build on.
