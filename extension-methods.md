# Extension methods

## Namespace

```csharp
VisioAutomation.Extensions
```

The `VisioAutomation.Extensions` namespace adds a small set of extension methods to the Visio COM types. To use them, add a `using` directive at the top of your file:

```csharp
using VisioAutomation.Extensions;
```

## `Shapes.GetShapesFromIDs`

Methods like `Page.DropManyU` return shape **IDs**, not `Shape` objects. To turn the IDs back into shapes, use `GetShapesFromIDs`:

```csharp
var shapes = page.Shapes.GetShapesFromIDs(shapeids);
```

## `Page.ResizeToFitContents(Size)`

Visio already has a `Page.ResizeToFitContents()` method, but this extension method makes it easy to add a fixed margin around the resized page.

```csharp
var margin_size = new VisioAutomation.Core.Size(0.5, 0.5);
page.ResizeToFitContents(margin_size);
```

## `Fonts.AsEnumerable()`

Visio's `Fonts` collection is COM-typed and not friendly to LINQ. The `AsEnumerable()` extension yields each `Font` so you can build a name-to-ID lookup once and reuse it:

```csharp
var fontname_to_id = doc.Fonts.AsEnumerable()
    .ToDictionary(f => f.Name, f => f.ID);
```
