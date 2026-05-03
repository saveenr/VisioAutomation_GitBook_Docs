# Extension methods

## Namespace

```
VisioAutomation.Extensions
```

## Summary

The VisioAutomation.Extensions contains a series of useful extension methods. To use simple add a “using” statement using VisioAutomation.Extensions;

## Examples

```
Shapes.GetShapesFromIDs
```

Methods like DropManyU don't return Shape objects but rather their IDs. If you want to quickly get the shape objects from a list of shape IDs then .Shapes.GetShapesFromIDs can be used.

```
var shapes = page.Shapes.GetShapesFromIDs(shapeids)

Page.ResizeToFitContents(w,h)
```

Visio aready has a Page.ResizeToFitContents() method but this extension method makes it easy to ensure a width and height for the margins

```
var margin_size = new VA.Drawing.Size(0.5,0.5);
page.ResizeToFitContents(margin_size);
```

## Fonts.AsEnumerable()

Sometimes one needs to keep looking up the ID of a font. By creating a dictionary, you can perform the lookup once and cache the results.

```
var fontname_to_id = doc.Fonts.AsEnumerable().ToDictionary(f =
>
 f.Name, f =
>
 f.ID);
```
