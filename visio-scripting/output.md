# client.Output

`client.Output` is the host's logging sink. Every formatted message routes through the `ClientContext` injected at `Client` construction. The default context writes verbose output to `Console.Out`; embedding hosts (the `Visio` PowerShell module is the canonical example) substitute a context that forwards into the host's own logging system.

## Methods

| Method                                              | Returns | Notes                                                                                |
| --------------------------------------------------- | ------- | ------------------------------------------------------------------------------------ |
| `WriteVerbose(string fmt, params object[] items)`   | `void`  | The only log level used by the rest of the library; honors `string.Format` syntax.   |

## Removal candidates (CY27)

These methods are currently `public` but have no known consumers as of 2026-05. They are candidates for removal in CY27 (see [issue #183](https://github.com/saveenr/VisioAutomation/issues/183)). New code should not depend on them.

| Method                                              | Notes                                                                          |
| --------------------------------------------------- | ------------------------------------------------------------------------------ |
| `WriteUser(string fmt, params object[] items)`      | Routes to `ClientContext.WriteUser`; not exercised inside the library.         |
| `WriteDebug(string fmt, params object[] items)`     | Routes to `ClientContext.WriteDebug`; not exercised inside the library.        |
| `WriteWarning(string fmt, params object[] items)`   | Routes to `ClientContext.WriteWarning`; not exercised inside the library.      |
| `WriteError(string fmt, params object[] items)`     | Routes to `ClientContext.WriteError`; not exercised inside the library.        |

## Example

The library's own commands use `WriteVerbose` to trace what they are doing. Application code following the same pattern looks like:

```csharp
client.Output.WriteVerbose("About to draw {0} shapes", count);

for (int i = 0; i < count; i++)
{
    var rect = new VisioAutomation.Core.Rectangle(i, 0, i + 1, 1);
    client.Draw.DrawRectangle(VisioScripting.TargetPage.Auto, rect);
}

client.Output.WriteVerbose("Done");
```

When the host is the `Visio` PowerShell module, those `WriteVerbose` calls turn into PowerShell `Write-Verbose` records that are visible only with `-Verbose` on the cmdlet call.

## See also

- [VisioScripting.Client overview](../visio-scripting.md): how `client.Output` fits into the facade and the `ClientContext` it routes through.
- [Visio PowerShell module](https://saveenr.gitbook.io/visiopowershell/): the canonical embedding host that overrides `ClientContext`.
