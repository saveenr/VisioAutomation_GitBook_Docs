# client.View

`client.View` controls the active window's view: zoom, scale, and fit-to-object. Most methods take a `TargetWindow` and operate on the resolved window's `Zoom` or `ViewFit` setting.

## Methods

| Method                                                            | Returns | Notes                                                                                |
| ----------------------------------------------------------------- | ------- | ------------------------------------------------------------------------------------ |
| `SetZoomValue(TargetWindow, double amount)`                       | `void`  | Sets the absolute zoom factor (e.g. `1.0` is 100 percent). Throws on non-positive.   |
| `SetZoomValueRelative(TargetWindow, double scale)`                | `void`  | Multiplies the current zoom by `scale`. Throws on non-positive.                      |
| `SetZoomToObject(TargetWindow, ZoomToObject)`                     | `void`  | Fits the view to `Page`, `PageWidth`, or `Selection` (with a 10 percent pad).        |

`TargetWindow.Auto` resolves to the active window at call time.

## Removal candidates (CY27)

These methods are currently `public` but have no known consumers as of 2026-05. They are candidates for removal in CY27 (see [issue #183](https://github.com/saveenr/VisioAutomation/issues/183)). New code should not depend on them.

| Method                                              | Notes                                                                          |
| --------------------------------------------------- | ------------------------------------------------------------------------------ |
| `GetActiveWindow()`                                 | Wraps `Application.ActiveWindow`; the same `IVisio.Window` is reachable directly. |
| `GetZoom(TargetWindow)`                             | Wraps `Window.Zoom`.                                                           |

## Example

```csharp
// Fit the active page in the active window
client.View.SetZoomToObject(VisioScripting.TargetWindow.Auto,
                              VisioScripting.Models.ZoomToObject.Page);

// Then zoom in by 50 percent
client.View.SetZoomValueRelative(VisioScripting.TargetWindow.Auto, 1.5);

// And later, set absolute zoom to 100 percent
client.View.SetZoomValue(VisioScripting.TargetWindow.Auto, 1.0);
```

## See also

- [VisioScripting.Client overview](../visio-scripting.md): how `client.View` fits into the facade.
- [Selection](selection.md): `SetZoomToObject(Selection)` zooms to whatever the selection is.
- [Page](page.md): page-level operations on the page that the view shows.
- [Application](application.md): window placement (move, resize) for the host window.
