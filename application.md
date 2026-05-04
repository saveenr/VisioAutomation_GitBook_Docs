# Application

`VisioAutomation.Application.ApplicationHelper` provides a small set of helpers around the `IVisio.Application` COM object &mdash; version detection, alert-bypass shutdown, content-folder location, and bringing the Visio window to the foreground.

## Get the Visio version

`Application.Version` returns a string with locale-dependent separators (e.g. `"16,0"` on some locales). `ApplicationHelper.GetVersion` normalizes the separator and parses the result into a `System.Version`.

```csharp
var version = VisioAutomation.Application.ApplicationHelper.GetVersion(app);

if (version.Major >= 15)
{
    // Visio 2013 or later
}
```

## Quit Visio without prompting

The default `app.Quit()` may prompt the user about unsaved changes. `Quit(app, force_close: true)` sets `AlertResponse` to "No" first so the prompt is auto-dismissed:

```csharp
VisioAutomation.Application.ApplicationHelper.Quit(app, force_close: true);
```

Pass `force_close: false` for the normal prompting behavior.

## Locate the Visio Content folder

Visio installs its built-in stencils and templates under a `Visio Content/<lang-id>` folder beneath the application install path. `GetContentLocation` returns that folder, with the language ID resolved against the current `app.Language`:

```csharp
string content_dir = VisioAutomation.Application.ApplicationHelper.GetContentLocation(app);
// e.g. C:\Program Files\Microsoft Office\root\Office16\Visio Content\1033
```

Throws `ArgumentException` for Visio versions below 14 (Visio 2010); supported on 2010 (`14`) and 2013+ (`15+`).

## Bring the Visio window to the foreground

When automating Visio from a script that also has its own console, the Visio window can end up behind other windows. `BringWindowToTop` is a thin wrapper over the Win32 `BringWindowToTop` API that targets the current `app.WindowHandle32`.

```csharp
VisioAutomation.Application.ApplicationHelper.BringWindowToTop(app);
```

## See also

* [Stencils and masters](stencils-and-masters.md) &mdash; opening stencils relative to the Visio Content folder.
