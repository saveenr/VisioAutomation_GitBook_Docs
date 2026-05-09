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
| `client.Application`       | The Visio process: lifetime, version, window placement.                | [client.Application](visio-scripting/application.md) |
| `client.Arrange`           | Move, align, distribute, nudge selected shapes.                        | [client.Arrange](visio-scripting/arrange.md) |
| `client.Connection`        | Connect shapes; analyze connector edges on a page.                     | [client.Connection](visio-scripting/connection.md) |
| `client.ConnectionPoint`   | Add, get, delete connection points on shapes.                          | [client.ConnectionPoint](visio-scripting/connection-point.md) |
| `client.Container`         | Drop a container shape around shapes.                                  | [client.Container](visio-scripting/container.md) |
| `client.Control`           | Add, get, delete control handles on shapes.                            | [client.Control](visio-scripting/control.md) |
| `client.CustomProperty`    | Read, write, delete custom properties (Shape Data) on shapes.          | [client.CustomProperty](visio-scripting/custom-property.md) |
| `client.Developer`         | Render diagrams of internal types and namespaces (debug aid).          | [client.Developer](visio-scripting/developer.md) |
| `client.Document`          | Open, close, create, find, save documents and stencils.                | [client.Document](visio-scripting/document.md) |
| `client.Draw`              | Draw primitives: rectangles, ovals, lines, beziers, polylines.         | [client.Draw](visio-scripting/draw.md) |
| `client.Export`            | Save pages or selections to image or HTML files.                       | [client.Export](visio-scripting/export.md) |
| `client.Grouping`          | Group, ungroup shapes.                                                 | [client.Grouping](visio-scripting/grouping.md) |
| `client.Hyperlink`         | Add, get, delete hyperlinks on shapes.                                 | [client.Hyperlink](visio-scripting/hyperlink.md) |
| `client.Layer`             | Find layers by name on a page; get all layers on a page.               | [client.Layer](visio-scripting/layer.md) |
| `client.Lock`              | Get, set lock cells (`LockCalcWH`, `LockMoveX`, ...) on shapes.        | [client.Lock](visio-scripting/lock.md) |
| `client.Master`            | Get masters in stencils, drop masters by name onto a page.             | [client.Master](visio-scripting/master.md) |
| `client.Model`             | Render and load high-level models: org charts, directed graphs, grids. | [client.Model](visio-scripting/model.md) |
| `client.Output`            | Logging sink: verbose, debug, warning, error, user messages.           | [client.Output](visio-scripting/output.md) |
| `client.Page`              | Pages: create, delete, navigate, set orientation, get dimensions.      | [client.Page](visio-scripting/page.md)     |
| `client.Selection`         | Select, deselect shapes; act on the current selection.                 | [client.Selection](visio-scripting/selection.md) |
| `client.ShapeSheet`        | Read, write ShapeSheet cells in batch (via reader, writer).            | [client.ShapeSheet](visio-scripting/shape-sheet.md) |
| `client.Text`              | Get, set shape text.                                                   | [client.Text](visio-scripting/text.md) |
| `client.Undo`              | Open, commit, abort an undo scope; trigger Undo or Redo.               | [client.Undo](visio-scripting/undo.md) |
| `client.UserDefinedCell`   | Read, write, delete user-defined cells on shapes.                      | [client.UserDefinedCell](visio-scripting/user-defined-cell.md) |
| `client.View`              | Zoom: set the active window's zoom value or zoom-to-object.            | [client.View](visio-scripting/view.md) |

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
