# client.Application

`client.Application` covers the lifetime and state of the Visio process: attaching to or starting an instance, validating that the attached instance is still healthy, querying its version, and moving its main window. The lower-level helpers behind these methods live on the [Application](../application.md) page.

## Methods

| Method                                          | Returns                       | Notes                                                                                |
| ----------------------------------------------- | ----------------------------- | ------------------------------------------------------------------------------------ |
| `HasApplication`                                | `bool` (property)             | True when a Visio `Application` is attached to this client.                          |
| `GetApplication()`                              | `IVisio.Application`          | Returns the attached `Application`, or `null`.                                       |
| `NewApplication()`                              | `IVisio.Application`          | Creates a new Visio process and attaches it to this client.                          |
| `CloseApplication()`                            | `void`                        | Force-closes every open document, then quits the application; clears the reference.  |
| `ValidateApplication()`                         | `bool`                        | Probes a read-only property to detect a stale or destroyed COM reference.            |
| `ApplicationVersion`                            | `System.Version` (property)   | Cached major.minor version of the attached Visio.                                    |
| `MoveWindowToFront()`                           | `void`                        | Brings the Visio main window to the foreground via Win32.                            |
| `GetWindowRectangle()`                          | `System.Drawing.Rectangle`    | Pixel rect of the main Visio window.                                                 |
| `SetWindowRectangle(System.Drawing.Rectangle)`  | `void`                        | Repositions and resizes the main Visio window.                                       |
| `DeleteShapes(TargetShapes)`                    | `void`                        | Deletes the resolved shapes; falls back to the active selection when unresolved.     |

`ValidateApplication` is the safe way to check a long-lived `Client`: it catches the `COMException` that the user closing Visio out from under you would otherwise produce on the next call.

## Removal candidates (CY27)

These methods are currently `public` but have no known consumers as of 2026-05. They are candidates for removal in CY27 (see [issue #183](https://github.com/saveenr/VisioAutomation/issues/183)). New code should not depend on them.

| Method                  | Notes                                                                       |
| ----------------------- | --------------------------------------------------------------------------- |
| `AssertHasApplication()`| Internal precondition guard exposed publicly; throws on missing application. |

## Example

```csharp
using IVisio = Microsoft.Office.Interop.Visio;

var client = new VisioScripting.Client(null);

// Attach a fresh Visio if we have none
if (!client.Application.HasApplication)
{
    client.Application.NewApplication();
}

// Make sure the attached Visio is still alive (the user may have closed it)
if (!client.Application.ValidateApplication())
{
    client.Application.NewApplication();
}

System.Console.WriteLine("Running Visio {0}", client.Application.ApplicationVersion);

client.Application.MoveWindowToFront();
client.Application.SetWindowRectangle(new System.Drawing.Rectangle(100, 100, 1200, 800));
```

## See also

- [Application](../application.md): the lower-level `VisioAutomation.Application.ApplicationHelper` and `AlertResponseScope` APIs.
- [VisioScripting.Client overview](../visio-scripting.md): how `client.Application` fits into the facade.
- [Document](document.md): once an application is attached, document-level operations live here.
- [Undo scope](../undo-scope.md): undo wrapping is handled by `client.Undo`.
