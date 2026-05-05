# Typed ShapeSheet I/O

Typed accessors that take `Core.Src` / `Core.SidSrc` instead of raw section/row/cell shorts. The same set is available on `Shape`, `Page`, and `Master`.

To use the extensions, add a `using` directive at the top of your file:

```csharp
using VisioAutomation.Extensions;
```

## Available methods

| Method                                            | Returns      | Purpose                                            |
| ------------------------------------------------- | ------------ | -------------------------------------------------- |
| `GetFormulasU(streamarray)`                       | `string[]`   | Read formulas.                                     |
| `GetResults<TResult>(streamarray, unitcodes)`     | `TResult[]`  | Read evaluated results in the requested type.       |
| `SetFormulas(streamarray, formulas, flags)`       | `int`        | Bulk-write formulas.                               |
| `SetResults(streamarray, unitcodes, results, flags)` | `int`     | Bulk-write evaluated values.                       |

## Prefer the higher-level wrappers

In practice you'll usually go through the higher-level `CellQuery` / `SectionQuery` ([Query the ShapeSheet](../shapesheet/query-the-shapesheet.md)) and `SrcWriter` / `SidSrcWriter` ([Modify the ShapeSheet](../shapesheet/modify-the-shapesheet.md)), which wrap these extensions. Drop down to the typed I/O methods directly only when you need to assemble a stream array yourself.

## See also

* [Query the ShapeSheet](../shapesheet/query-the-shapesheet.md), [Modify the ShapeSheet](../shapesheet/modify-the-shapesheet.md) &mdash; the recommended high-level wrappers.
* [SRC structs](../shapesheet/cells.md) &mdash; how cells are addressed.
* [Extension methods overview](../extension-methods.md).
