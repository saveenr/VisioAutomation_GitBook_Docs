# Getting started

This walkthrough takes a brand-new C# project from "no Visio code at all" to a saved `.vsdx` file produced by VisioAutomation. The full final program is at the end.

## Choosing an entry point

VisioAutomation ships several layered APIs. Pick the one that matches your scenario:

| Use case                                      | Entry point                                                                                                  |
|-----------------------------------------------|--------------------------------------------------------------------------------------------------------------|
| Scripting from PowerShell                     | The [`Visio` PowerShell module](https://saveenr.gitbook.io/visiopowershell/) (`Import-Module Visio`)         |
| Scripting from C# at a high level             | `VisioScripting.Client` (a topic-grouped facade over the rest)                                               |
| Generating diagrams from data declaratively   | `VisioAutomation.Models` (DOM + automated layouts: tree, directed graph, org chart, grid)                    |
| Low-level helpers over the Visio COM API      | The `VisioAutomation` namespaces (this guide)                                                                |
| Raw COM interop                               | `Microsoft.Office.Interop.Visio` directly (this guide bootstraps from raw interop, then layers the helpers) |

This page works at the **VisioAutomation helpers** layer because it makes the underlying COM model visible. Once you're comfortable here, the higher-level APIs are easy follow-ons.

## Preparation

* On any Windows machine, install Visual Studio 2022 or above.
* The same machine should also have Visio 2010 or later installed.
* Create a new C# command-line project in Visual Studio.

## Add a reference to the Visio 2010 PIA

PIA = Primary Interop Assembly.

Before you start using VisioAutomation you need to get access to the interop assembly that lets you communicate with Visio.

For Visio 2010: [`Visio2010.PrimaryInteropAssembly`](https://www.nuget.org/packages/Visio2010.PrimaryInteropAssembly) on NuGet. The same assembly works against newer Visio versions too (see the [version-compatibility table](../README.md#visio-version-compatibility)).

## Add a reference to the VisioAutomation2010 NuGet

[`VisioAutomation2010`](https://www.nuget.org/packages/VisioAutomation2010) on NuGet.

## Launch the Visio app programmatically

```csharp
var app = new Microsoft.Office.Interop.Visio.Application();
```

## Create a document

The new document automatically gets one empty page.

```csharp
var app = new Microsoft.Office.Interop.Visio.Application();
var doc = app.Documents.Add("");
```

## Find the active page and draw a rectangle

```csharp
var app = new Microsoft.Office.Interop.Visio.Application();
var doc = app.Documents.Add("");
var page = app.ActivePage; // Creating a new doc auto-creates an empty active page.
var s1 = page.DrawRectangle(0, 0, 5, 3);
s1.Text = "Hello World";
```

## Checkpoint 1

At this point you have a working interop loop. Visio launches, a document opens, a rectangle appears, the rectangle is labelled. You haven't used VisioAutomation itself yet, only the raw COM interop, and that's fine, the next steps add the helpers.

## Enable VisioAutomation.Extensions

The Visio API has a broad surface area. To make it more convenient in a number of places, VisioAutomation offers a series of extension methods, grouped under [`VisioAutomation.Extensions`](../extension-methods.md).

```csharp
using VisioAutomation.Extensions;
```

You don't strictly need the extension methods, they are thin wrappers over static methods on various helper classes, and you can call those helpers directly. Adding the `using` is much more pleasant in day-to-day work, so this guide assumes it from here on.

## Drop a shape from a stencil

Drawing a rectangle by hand is fine when you only need a rectangle. To get the predefined Visio shapes (with all the smart geometry, custom properties, snap behaviour, and themes that go with them), the idiomatic move is to **drop** a master from a stencil.

This means: load a stencil document, find the master inside it for the shape you want, then drop a new instance of that master onto the page.

```csharp
var app = new Microsoft.Office.Interop.Visio.Application();
var doc = app.Documents.Add("");
var page = app.ActivePage;
var stencil = app.Documents.OpenStencil("basic_u.vss");
var masters = stencil.Masters;
var rectangle_master = masters["Rectangle"];
page.Drop(rectangle_master, 3, 3);
```

A few notes on stencils:

* Stencils have evolved across Visio versions. The filename, master names, and visual appearance of "the same" stencil may differ between Visio 2010 and Visio for Microsoft 365.
* The hard-coded `"basic_u.vss"` above is the legacy filename for the "Basic Shapes" stencil. Newer Visio versions may ship it under a different name (e.g. `BASIC_U.VSSX`).
* To find the correct stencil path for the installed Visio version, use [`ApplicationHelper.GetContentLocation(app)`](../application.md#locate-the-visio-content-folder), which returns the per-locale Visio Content folder where the built-in stencils live.

## Drop multiple shapes at once

Calling `Drop` in a loop works but is slow, every drop is its own COM round-trip. `DropManyU` is the batch version: one COM call drops many shapes at once.

```csharp
var stencil = doc.Application.Documents.OpenStencil("basic_u.vss");
var rectmaster = stencil.Masters["Rectangle"];
var page2 = doc.Pages.Add();

var centerpoints = new[] {
    new VisioAutomation.Core.Point(1, 2),
    new VisioAutomation.Core.Point(5, 4),
};
var masters = new[] { rectmaster, rectmaster };
short[] shapeids = page2.DropManyU(masters, centerpoints);
```

## Save the document

Pass an absolute path to `SaveAs`. Visio infers the file format from the extension (`.vsdx` for the modern XML format).

```csharp
doc.SaveAs(@"C:\Temp\hello.vsdx");
```

You now have a real Visio file on disk.

## The full program

Putting all of the above together:

```csharp
using VisioAutomation.Extensions;
using IVisio = Microsoft.Office.Interop.Visio;

var app = new IVisio.Application();
var doc = app.Documents.Add("");
var page = app.ActivePage;

// Manual draw
var s1 = page.DrawRectangle(0, 0, 5, 3);
s1.Text = "Hello World";

// Drop a single shape from a stencil
var stencil = app.Documents.OpenStencil("basic_u.vss");
var rectangle_master = stencil.Masters["Rectangle"];
page.Drop(rectangle_master, 7, 3);

// Drop many shapes in a single batched call
var page2 = doc.Pages.Add();
var centerpoints = new[] {
    new VisioAutomation.Core.Point(1, 2),
    new VisioAutomation.Core.Point(5, 4),
};
var masters = new[] { rectangle_master, rectangle_master };
short[] shapeids = page2.DropManyU(masters, centerpoints);

doc.SaveAs(@"C:\Temp\hello.vsdx");
```

## Next steps

You've now seen the basic shape: get an `Application`, add a `Document`, work with its `Pages`, drop or draw `Shapes`, save. From here, the rest of the user guide covers each topic in depth:

* **[Extension methods](../extension-methods.md)**: the full set of LINQ bridges, drawing primitives, master-dropping helpers, and coordinate utilities under `VisioAutomation.Extensions`.
* **[Stencils and masters](../stencils-and-masters.md)**: more on opening stencils and dropping masters, with batched and per-shape examples.
* **[ShapeSheet](../shapesheet/README.md)**: the spreadsheet-like grid of cells that backs every shape; high-performance read (`CellQuery` / `SectionQuery`) and write (`SrcWriter` / `SidSrcWriter`) helpers.
* **[Custom properties](../custom-properties.md)**: read and write user-defined named fields ("Shape Data" in the Visio UI).
* **[Connectors](../connectors.md)**: wire shapes together with manual connectors or AutoConnect.
* **[Application](../application.md)**: version detection, alert-bypass quit, finding the Visio Content folder.

For the full list, see the table of contents in the sidebar.
