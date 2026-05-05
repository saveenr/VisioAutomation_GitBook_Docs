# LINQ bridges over the COM collection types

Visio's COM collection types (`IVisio.Pages`, `IVisio.Shapes`, etc.) don't implement `IEnumerable<T>`; they predate generics and use the legacy `Count` + 1-based indexer pattern. The extensions add `ToEnumerable()` and `ToList()` overloads so you can use LINQ against them directly.

To use them, add a `using` directive at the top of your file:

```csharp
using VisioAutomation.Extensions;
```

## Available bridges

| Collection  | Extensions                                                                                                                            |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| `Pages`     | `ToEnumerable() : IEnumerable<Page>`, `ToList() : List<Page>`, `GetNamesU() : string[]`                                                |
| `Shapes`    | `ToEnumerable() : IEnumerable<Shape>`, `ToList() : List<Shape>`                                                                       |
| `Masters`   | `ToEnumerable() : IEnumerable<Master>`, `ToList() : List<Master>`                                                                     |
| `Documents` | `ToEnumerable() : IEnumerable<Document>`, `ToList() : List<Document>`                                                                 |
| `Fonts`     | `ToEnumerable() : IEnumerable<Font>`, `ToList() : List<Font>`                                                                         |
| `Layers`    | `ToEnumerable() : IEnumerable<Layer>`, `ToList() : List<Layer>`                                                                       |
| `Colors`    | `ToEnumerable() : IEnumerable<Color>`, `ToList() : List<Color>`                                                                       |
| `Connects`  | `ToEnumerable() : IEnumerable<Connect>`, `ToList() : List<Connect>`                                                                   |
| `Styles`    | `ToEnumerable() : IEnumerable<Style>`, `ToList() : List<Style>`, `GetNamesU() : string[]`                                              |
| `Selection` | `ToEnumerable() : IEnumerable<Shape>`, `ToList() : List<Shape>`, `GetIDs() : int[]`, `GetBoundingBox(args) : Rectangle`                |
| `Section`   | `ToEnumerable() : IEnumerable<Row>`, `ToList() : List<Row>`                                                                           |
| `Windows`   | `ToEnumerable() : IEnumerable<Window>`, `ToList() : List<Window>`                                                                     |

## Example

```csharp
// Build a font-name to font-ID dictionary
var fontname_to_id = doc.Fonts.ToEnumerable()
    .ToDictionary(f => f.Name, f => f.ID);

// Find every page whose name starts with "Backup"
var backup_pages = doc.Pages.ToEnumerable()
    .Where(p => p.NameU.StartsWith("Backup"))
    .ToList();
```

## See also

* [Extension methods overview](../extension-methods.md): the rest of the `VisioAutomation.Extensions` namespace.
