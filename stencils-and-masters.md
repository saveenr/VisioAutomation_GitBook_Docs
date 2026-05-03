# Stencils and masters

## Stencils and masters

These samples rely on extension methods defined by VisioAutomation. Add this using statement at the top to enable the extensions in your source code.

```
using VisioAutomation.Extensions;
```

## Drop one shape using a master

```
var stencil = doc.Application.Documents.OpenStencil("basic_u.vss");
var rectmaster = stencil.Masters["Rectangle"];
var page = doc.Pages.Add();

var shape1 = page.Drop(rectmaster, 1.0, 2.0);

var p = new VisioAutomation.Drawing.Point(5.0, 4.0);
var shape2 = page.Drop(rectmaster, p);
```

## Drop multiple shapes

```
var stencil = doc.Application.Documents.OpenStencil("basic_u.vss");
var rectmaster = stencil.Masters["Rectangle"];
var page = doc.Pages.Add();

var centerpoints = new[] {
                               new VisioAutomation.Drawing.Point(1, 2),
                               new VisioAutomation.Drawing.Point(5, 4)
                           };
var masters = new[] { rectmaster, rectmaster };
short[] shapeids = page.DropManyU(masters, centerpoints);
```
