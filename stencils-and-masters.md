# Stencils and masters

These samples rely on extension methods defined by VisioAutomation. Add this `using` statement at the top of your source file to enable the extensions:

```csharp
using VisioAutomation.Extensions;
```

> **Stencil filenames vary by Visio version.** The examples below use `"basic_u.vss"`, the legacy filename for the "Basic Shapes" stencil. Visio 2013 and later may ship the same stencil under a different name (e.g. `"BASIC_U.VSSX"`). To find the correct path for the installed Visio version, see [`ApplicationHelper.GetContentLocation`](application.md#locate-the-visio-content-folder), which returns the per-locale content folder containing the built-in stencils.

## Drop one shape using a master

```csharp
var stencil = doc.Application.Documents.OpenStencil("basic_u.vss");
var rectmaster = stencil.Masters["Rectangle"];
var page = doc.Pages.Add();

var shape1 = page.Drop(rectmaster, 1.0, 2.0);

var p = new VisioAutomation.Core.Point(5.0, 4.0);
var shape2 = page.Drop(rectmaster, p);
```

## Drop multiple shapes

```csharp
var stencil = doc.Application.Documents.OpenStencil("basic_u.vss");
var rectmaster = stencil.Masters["Rectangle"];
var page = doc.Pages.Add();

var centerpoints = new[] {
                               new VisioAutomation.Core.Point(1, 2),
                               new VisioAutomation.Core.Point(5, 4)
                           };
var masters = new[] { rectmaster, rectmaster };
short[] shapeids = page.DropManyU(masters, centerpoints);
```
