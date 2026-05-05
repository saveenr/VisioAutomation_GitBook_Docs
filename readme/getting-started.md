# Getting started

## Preparation

On any Windows machine, install Visual Studio 2022 or above.

This machine should also have Visio 2010 or later installed.

Create a new C# command line project in Visual Studio

## Add a reference to the Visio 2010 PIA

PIA = Primary Interop Assembly

Before you start using VisioAutomation you need to get access to the interop assembly that lets you communicate to Visio.

For Visio 2010: that is this NuGet package [https://www.nuget.org/packages/Visio2010.PrimaryInteropAssembly](https://www.nuget.org/packages/Visio2010.PrimaryInteropAssembly)

## Add a reference to the VisioAutomation2010 Nuget&#x20;

Link: [https://www.nuget.org/packages/VisioAutomation2010](https://www.nuget.org/packages/VisioAutomation2010)

## Launch the Visio app programmatically

This sample shows you how to launch Visio

```
var app = new Microsoft.Office.Interop.Visio.Application();
```

## Create a document

The document will come with a empty page

```
var app = new Microsoft.Office.Interop.Visio.Application();
var doc = app.Documents.Add("");
```

## Find the active page and draw a rectangle

```
var app = new Microsoft.Office.Interop.Visio.Application();
var doc = app.Documents.Add("");var page = app.ActivePage; // Creating a new doc should auto create a new empty page
var s1 = page.DrawRectangle( 0,0,5,3);
s1.Text = "Hello World";
```

## Checkpoint 1

At this point, you'd explored the very basics. These steps confirm that everything is working.

However, so far we haven't used VisioAutomation at all. Now we will start using it.

## Enabling VisioAutomation.Extensions

The Visio API has a broad surface area.

To make it more convenient in a number of places, a VisioAutomation offers a series of extension methods.

```
using VisioAutomation.Extensions; 
```

You don't have to use the extensions methods! But it will make your day-to-day like easier so I do recommended it.

If you are set against extensions methods, don't worry - these are very light methods that call static methods in various helper static classes. You can always call those helper classes directly.

## Dropping a shape

Previously we "manually" draw a rectangle. And sometimes that is the easiest thing to do.

But if you want to draw a lot of shapes and want to do that fast and want to do that in a more Visio-idiomatic way, then follow the pattern of "dropping" shapes.

So now you should learn how to drop a single shape.

This means loading a stencil document. The extension method `OpenStencil` makes this easy.

Then find the master of the shape you want to drop - in this case "Rectangle" - that is inside the stencil.

Then drop the shape.

```
var app = new Microsoft.Office.Interop.Visio.Application();
var doc = app.Documents.Add("");
var page = app.ActivePage; // Creating a new doc should auto create a new empty page
var stencil = app.Documents.OpenStencil("basic_u.vss");
var masters = stencil.Masters;
var rectangle_master = masters["Rectangle"];
page.Drop(rectangle_master, 3, 3);
```

A couple of notes here

* over the years the stencils have changed&#x20;
* they may have different filenames
* the masters they contain may have different names or visual looks

## Dropping multiple shapes

You could call Drop for each shape. That will be slow. Instead use `DropMany`.

```
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

