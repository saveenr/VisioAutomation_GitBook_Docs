# client.ConnectionPoint

`client.ConnectionPoint` adds, reads, and deletes connection points on shapes. Connection points are the gluable spots that connectors snap to. The lower-level cells API for these is on the [Connection points](../connection-points.md) page.

## Methods

| Method                                                                                  | Returns                                                              | Notes                                                                                       |
| --------------------------------------------------------------------------------------- | -------------------------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| `AddConnectionPoint(TargetShapes, string fx, string fy, ConnectionPointType type)`      | `List<int>`                                                          | Appends one connection point per shape at formula `(fx, fy)`; returns the new row index per shape. |
| `DeleteConnectionPointAtIndex(TargetShapes, int index)`                                 | `void`                                                               | Skips shapes that have fewer than `index + 1` connection points.                            |
| `GetConnectionPoints(TargetShapes)`                                                     | `IDictionary<Shape, CellRecords<ConnectionPointCells>>`              | One record-set per shape; values returned as formulas.                                      |

`TargetShapes.Auto` resolves to the currently selected shapes at call time. Both write methods open an undo scope so the whole batch reverses together.

## Example

```csharp
// Two new points on every selected shape: midpoint of left edge, midpoint of right edge
client.ConnectionPoint.AddConnectionPoint(VisioScripting.TargetShapes.Auto,
                                            "Width*0",
                                            "Height*0.5",
                                            VisioScripting.Models.ConnectionPointType.Inward);

client.ConnectionPoint.AddConnectionPoint(VisioScripting.TargetShapes.Auto,
                                            "Width*1",
                                            "Height*0.5",
                                            VisioScripting.Models.ConnectionPointType.Inward);

// Inspect them
var dict = client.ConnectionPoint.GetConnectionPoints(VisioScripting.TargetShapes.Auto);
foreach (var kv in dict)
{
    foreach (var record in kv.Value)
    {
        System.Console.WriteLine("{0} cp[{1}]: ({2}, {3})",
            kv.Key.Name, record.Row, record.Cells.X, record.Cells.Y);
    }
}
```

## See also

- [Connection points](../connection-points.md): the lower-level `ConnectionPointCells` and `ConnectionPointHelper` APIs.
- [Connectors](../connectors.md): connectors are what glue to connection points (`client.Connection`).
- [VisioScripting.Client overview](../visio-scripting.md): how `client.ConnectionPoint` fits into the facade.
- [Control](control.md): another per-shape row-indexed cell section with a similar shape.
