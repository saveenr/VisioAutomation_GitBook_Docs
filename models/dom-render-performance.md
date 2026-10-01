# DOM render performance

While a [DOM](dom.md) `Page` renders, it temporarily changes four Visio application settings to make rendering faster, then restores your original values afterward. This page describes those settings and what each one does.

**You almost certainly do not need to change them.** The defaults are what every layout and model in the library renders with, and they were chosen deliberately: for example, `ScreenUpdating` stays on because turning it off breaks page resizing. The library itself never changes them, and its tests only run with the defaults, so other combinations are untested. Experimenting is more likely to give you wrong results than a faster render. Read this page to understand what the DOM is doing, not as a list of knobs to turn.

## How it works

Each `Page` has a read-only `RenderPerformanceSettings` property. `Page.Render` applies the settings before it draws anything and restores the values Visio had before, even if the render throws. Each setting is nullable, and `null` means leave that setting alone.

Only `Page.Render` applies the settings. `ShapeList.Render` does not, although it still drops shapes and writes cell values in bulk. See [Where the output goes](dom.md#where-the-output-goes) for which render calls go through `Page.Render`.

## The settings

A new `Page` starts with these values:

* **`DeferRecalc`**: Visio's `Application.DeferRecalc`, which decides whether Visio recalculates cell formulas during a series of actions.
  * Type: `short?`
  * Default: `0`
  * `0`: Visio recalculates formulas as needed.
  * Any nonzero value: Visio defers recalculating formulas until the render is finished.
* **`ScreenUpdating`**: Visio's `Application.ScreenUpdating`, which decides whether the window is redrawn during a series of actions.
  * Type: `short?`
  * Default: `1`
  * `1` (any nonzero value): Visio redraws the window as normal. This is the default because turning screen updating off can break page resizing.
  * `0`: Visio does not redraw the window while the page renders.
* **`EnableAutoConnect`**: Visio's `Application.Settings.EnableAutoConnect`, which turns Visio's AutoConnect feature on or off. Visio itself has it on by default.
  * Type: `bool?`
  * Default: `false`
  * `false`: AutoConnect is off while the page renders.
  * `true`: AutoConnect is on.
* **`LiveDynamics`**: Visio's `Application.LiveDynamics`, which decides how often Visio recalculates shape properties during drag operations.
  * Type: `bool?`
  * Default: `false`
  * `false`: Visio recalculates only after the mouse button is released.
  * `true`: Visio recalculates on every mouse move, which raises more events such as `CellChanged`. Add-ins that respond to those events can run faster with `false`.

Each of these is documented in Microsoft's Visio reference: [`DeferRecalc`](https://learn.microsoft.com/en-us/office/vba/api/Visio.Application.DeferRecalc), [`ScreenUpdating`](https://learn.microsoft.com/en-us/office/vba/api/Visio.Application.ScreenUpdating), [`EnableAutoConnect`](https://learn.microsoft.com/en-us/office/vba/api/Visio.ApplicationSettings.EnableAutoConnect) and [`LiveDynamics`](https://learn.microsoft.com/en-us/office/vba/api/visio.application.livedynamics).

## Changing a setting

If you do want to experiment, set the value before you call `Render`:

```csharp
var page_node = new VADOM.Page();
page_node.RenderPerformanceSettings.DeferRecalc = 1;   // adjust before Render
```

Check the result carefully, especially the page size and the positions of connected shapes, and compare it with a render that uses the defaults.

## See also

* [DOM](dom.md) (the model these settings belong to)
