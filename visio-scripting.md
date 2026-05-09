# VisioScripting.Client

`VisioScripting.Client` is a high-level facade over the lower-level `VisioAutomation` namespaces. It groups operations into 25 verb-noun command groups, each hung off a property on the `Client` type. The same surface backs the [`Visio` PowerShell module](https://saveenr.gitbook.io/visiopowershell/), so any operation a cmdlet can do is also reachable directly from C#.

This is the C#-friendly entry point for users who want a more convenient API than [`VisioAutomation.Extensions`](extension-methods.md) without dropping into PowerShell. It hides the COM bookkeeping (look up the active window, find its document, find the active page, then act) behind named groups: `client.Page.GetActivePage()`, `client.Draw.DrawRectangle(...)`, `client.Document.NewDocument()`.

## Instantiation

```csharp
using IVisio = Microsoft.Office.Interop.Visio;

var app = new IVisio.Application();
var client = new VisioScripting.Client(app);
```

Once you have a `Client`, navigate to a command group via the property, then call methods on the group:

```csharp
client.Document.NewDocument();
client.Draw.DrawRectangle(VisioScripting.TargetPage.Auto,
                          new VisioAutomation.Core.Rectangle(0, 0, 4, 2));
client.Text.SetShapeText(VisioScripting.TargetShapes.Auto,
                          new[] { "Hello, Visio!" });
```

That snippet is also the C# half of the [project quick-start](https://github.com/saveenr/VisioAutomation/blob/master/readme.md#quick-example). It lives at the `VisioScripting.Client` layer because that layer reads as code that says what it does, while the same operations expressed in raw `Microsoft.Office.Interop.Visio` calls drift into bookkeeping.

## The Target* convention

Many `Client` methods take a `Target*` parameter that names *which* page, document, shape, or selection to act on. Each target type has an `Auto` factory that means "use the active one when this command runs":

```csharp
// Set text on whatever shapes are selected at call time
client.Text.SetShapeText(VisioScripting.TargetShapes.Auto,
                          new[] { "label" });

// Pass an explicit IVisio.Page when you have one
client.Page.SetActivePage(somePage);
```

Resolving an `Auto` target happens lazily at the moment the command runs, not at construction time. A `Target*.Auto` reference can be passed around safely even if the active page or selection changes between construction and the call.

The `Target*` types:

| Type                | Wraps                          | Common `Auto` meaning                |
|---------------------|--------------------------------|--------------------------------------|
| `TargetDocument`    | A single document              | The active document.                 |
| `TargetDocuments`   | A set of documents             | All currently open documents.        |
| `TargetPage`        | A single page                  | The active page.                     |
| `TargetPages`       | A set of pages                 | All pages in the active document.    |
| `TargetShapes`      | A set of shapes                | The currently selected shapes.       |
| `TargetSelection`   | An `IVisio.Selection`          | The active window's selection.       |
| `TargetWindow`      | A single window                | The active window.                   |

## Command groups

Each property on `Client` returns a single command-group object. The methods on that object do the work. Per-group documentation is linked from the table below.

| Property                   | Topic                                                                  | Page                                       |
|----------------------------|------------------------------------------------------------------------|--------------------------------------------|
| `client.Application`       | The Visio process: lifetime, version, window placement.                | (in progress)                              |
| `client.Arrange`           | Move, align, distribute, nudge selected shapes.                        | (in progress)                              |
| `client.Connection`        | Connect shapes; analyze connector edges on a page.                     | (in progress)                              |
| `client.ConnectionPoint`   | Add, get, delete connection points on shapes.                          | (in progress)                              |
| `client.Container`         | Drop a container shape around shapes.                                  | (in progress)                              |
| `client.Control`           | Add, get, delete control handles on shapes.                            | (in progress)                              |
| `client.CustomProperty`    | Read, write, delete custom properties (Shape Data) on shapes.          | (in progress)                              |
| `client.Developer`         | Render diagrams of internal types and namespaces (debug aid).          | (in progress)                              |
| `client.Document`          | Open, close, create, find, save documents and stencils.                | (in progress)                              |
| `client.Draw`              | Draw primitives: rectangles, ovals, lines, beziers, polylines.         | (in progress)                              |
| `client.Export`            | Save pages or selections to image or HTML files.                       | (in progress)                              |
| `client.Grouping`          | Group, ungroup shapes.                                                 | (in progress)                              |
| `client.Hyperlink`         | Add, get, delete hyperlinks on shapes.                                 | [client.Hyperlink](visio-scripting/hyperlink.md) |
| `client.Layer`             | Find layers by name on a page; get all layers on a page.               | (in progress)                              |
| `client.Lock`              | Get, set lock cells (`LockCalcWH`, `LockMoveX`, ...) on shapes.        | (in progress)                              |
| `client.Master`            | Get masters in stencils, drop masters by name onto a page.             | (in progress)                              |
| `client.Model`             | Render and load high-level models: org charts, directed graphs, grids. | (in progress)                              |
| `client.Output`            | Logging sink: verbose, debug, warning, error, user messages.           | (in progress)                              |
| `client.Page`              | Pages: create, delete, navigate, set orientation, get dimensions.      | [client.Page](visio-scripting/page.md)     |
| `client.Selection`         | Select, deselect shapes; act on the current selection.                 | (in progress)                              |
| `client.ShapeSheet`        | Read, write ShapeSheet cells in batch (via reader, writer).            | (in progress)                              |
| `client.Text`              | Get, set shape text.                                                   | (in progress)                              |
| `client.Undo`              | Open, commit, abort an undo scope; trigger Undo or Redo.               | (in progress)                              |
| `client.UserDefinedCell`   | Read, write, delete user-defined cells on shapes.                      | (in progress)                              |
| `client.View`              | Zoom: set the active window's zoom value or zoom-to-object.            | (in progress)                              |

## Output sinks: ClientContext and DefaultClientContext

`Client` has two constructors:

```csharp
public Client(IVisio.Application app);
public Client(IVisio.Application app, ClientContext clientContext);
```

The single-argument form uses `DefaultClientContext`, which writes verbose output to `Console.Out`. For embedding hosts (the `Visio` PowerShell module is the canonical example), pass a custom `ClientContext` subclass to route logging into the host's own machinery.

## See also

- [Getting started](readme/getting-started.md): the C# walkthrough that the `VisioScripting.Client` flow follows on from.
- [Visio PowerShell module](https://saveenr.gitbook.io/visiopowershell/): the cmdlet surface built on this facade.
- [Extension methods](extension-methods.md): the lower-level `VisioAutomation.Extensions` API.
- [Stencils and masters](stencils-and-masters.md): what `client.Master` and `client.Document.OpenStencilDocument` work with.
- [ShapeSheet](shapesheet/README.md): the underlying cell model that `client.ShapeSheet` exposes.
