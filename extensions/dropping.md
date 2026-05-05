# Master dropping

Drop a master at a typed `Core.Point` instead of two loose doubles. The same set is available on `IVisio.Page`, `IVisio.Master`, and `IVisio.Shape` (the last lets you drop a master inside a group).

To use the extensions, add a `using` directive at the top of your file:

```csharp
using VisioAutomation.Extensions;
```

## Available methods

| Method                       | Returns                              |
| ---------------------------- | ------------------------------------ |
| `Drop(master, point)`        | `Shape`                              |
| `DropManyU(masters, points)` | `short[]` (the new shapes' IDs)      |

## Example

```csharp
var stencil    = doc.Application.Documents.OpenStencil("basic_u.vss");
var rectmaster = stencil.Masters["Rectangle"];

var s = page.Drop(rectmaster, new VisioAutomation.Core.Point(2.0, 1.0));
```

## Batch dropping

`DropManyU` is the same operation in batch &mdash; one `IVisio.DropManyU` COM call for many shapes, much faster than calling `Drop` in a loop. `DropManyU` returns shape IDs; turn them back into `Shape` objects with `Shapes.GetShapesFromIDs` (see the [one-offs](misc.md) page).

## See also

* [Stencils and masters](../stencils-and-masters.md) &mdash; opening stencils and finding masters.
* [Drawing primitives](drawing.md) &mdash; for drawn (non-master) shapes.
* [One-off extensions](misc.md) &mdash; `Shapes.GetShapesFromIDs` to convert the IDs returned by `DropManyU` back into objects.
* [Extension methods overview](../extension-methods.md).
