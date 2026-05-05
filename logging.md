# Visio error log

Visio writes a per-user **XML error log** that records errors and warnings the application has reported. `VisioAutomation.Application.Logging` provides typed access to that log.

## Locate the log file

The log's file name is stored in the registry; its folder is under the Internet Explorer Internet Cache. `LoggingHelper.GetXmlErrorLogFilename(app)` resolves both and returns the absolute path:

```csharp
string path = VisioAutomation.Application.Logging.LoggingHelper
    .GetXmlErrorLogFilename(app);
```

The typical resolved path looks like:

```
C:\Users\<alias>\AppData\Local\Microsoft\Windows\Temporary Internet Files\Content.MSO\VisioLogFiles\<filename>.xml
```

`GetXmlErrorLogFilename` returns `null` if the registry key exists but the `XMLErrorLogName` value is missing (i.e., Visio hasn't written a log yet).

## Parse the log

`XmlErrorLog` is a parser for the file format. Pass a path; iterate the `LogSessions` collection.

```csharp
var path = VisioAutomation.Application.Logging.LoggingHelper.GetXmlErrorLogFilename(app);
var log  = new VisioAutomation.Application.Logging.XmlErrorLog(path);

foreach (var session in log.LogSessions)
{
    System.Console.WriteLine($"Session at {session.StartTime} ({session.Source}):");
    foreach (var record in session.LogRecords)
    {
        System.Console.WriteLine($"  {record.Type}/{record.SubType}: {record.Description}");
    }
}
```

## Record shapes

| `LogSession` | `LogRecord` |
| --- | --- |
| `StartLine`, `EndLine`: the raw XML markers | `Type`, `SubType`: categorical fields written by Visio |
| `FileType`, `Source`: metadata about what Visio recorded | `Context`: what Visio was doing when it logged the record |
| `StartTimeRaw`, `StartTime`: raw and parsed timestamp | `Description`: human-readable detail |
| `LogRecords`: the records in this session | |

The constructor throws `ArgumentException` if the file doesn't exist.

## See also

* [Application](application.md): other application-level helpers.
