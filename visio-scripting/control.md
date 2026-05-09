# client.Control

`client.Control` adds, reads, and deletes control handles on shapes. A control handle is a draggable yellow diamond that maps to a row in the shape's `Controls` section. The lower-level cell-level API for these is on the [Control handles](../control-handles.md) page.

## Methods

| Method                                                                        | Returns                                                | Notes                                                                              |
| ----------------------------------------------------------------------------- | ------------------------------------------------------ | ---------------------------------------------------------------------------------- |
| `AddControlToShapes(TargetShapes, ControlCells)`                              | `List<int>`                                            | Appends one control row per shape; returns the new row index per shape.            |
| `DeleteControlWithIndex(TargetShapes, int index)`                             | `void`                                                 | Skips shapes that have fewer than `index + 1` controls.                            |
| `GetControls(TargetShapes, CellValueType cvt)`                                | `Dictionary<Shape, CellRecords<ControlCells>>`         | One record-set per shape; `cvt` selects formula or resolved value.                 |

`TargetShapes.Auto` resolves to the currently selected shapes at call time. Both write methods open an undo scope so the whole batch reverses together.

## Example

```csharp
var ctrl = new VisioAutomation.Shapes.ControlCells();
ctrl.X = "Width*0.5";
ctrl.Y = "Height*0.5";
ctrl.XBehavior = "0";
ctrl.YBehavior = "0";

// Add a centered control handle to every selected shape
var indices = client.Control.AddControlToShapes(VisioScripting.TargetShapes.Auto, ctrl);

var dict = client.Control.GetControls(VisioScripting.TargetShapes.Auto,
                                        VisioAutomation.Core.CellValueType.Formula);
foreach (var kv in dict)
{
    foreach (var record in kv.Value)
    {
        System.Console.WriteLine("{0} ctl[{1}]: ({2}, {3})",
            kv.Key.Name, record.Row, record.Cells.X, record.Cells.Y);
    }
}
```

## See also

- [Control handles](../control-handles.md): the lower-level `ControlCells` and `ControlHelper` APIs.
- [Connection point](connection-point.md): another per-shape row-indexed cell section with a similar shape.
- [VisioScripting.Client overview](../visio-scripting.md): how `client.Control` fits into the facade.
- [Custom property](custom-property.md): related per-shape data with the same dictionary-by-shape return shape.
