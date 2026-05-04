# Control handles

A shape's **control handles** are the yellow draggable diamonds Visio draws on a shape so the user can adjust geometry interactively (e.g., drag the corner radius of a rounded rectangle). Each handle is a row in the `Controls.` ShapeSheet section. VisioAutomation exposes them via `VisioAutomation.Shapes.ControlHelper` and the `ControlCells` record.

## The `ControlCells` record

| Field | Purpose |
| --- | --- |
| `X`, `Y` | Position formula(s). Typically expressions like `Width*0.5`, `Height*0.5`. |
| `XBehavior`, `YBehavior` | How the handle's position adapts when the shape resizes. |
| `XDynamics`, `YDynamics` | Whether the handle's coordinate is "free" or driven by another formula. Auto-defaulted when added. |
| `CanGlue` | Whether the handle accepts glue connections. |
| `Tip` | Tooltip shown on hover. |

All fields are `Core.CellValue`-typed.

## Add a control handle

`Add` appends a new control row to the shape and returns its row index. Without a `ControlCells`, you get a default-positioned handle; pass one to specify position / tooltip / behavior.

```csharp
// Default-positioned handle
int row1 = VisioAutomation.Shapes.ControlHelper.Add(shape);

// Centered handle with a tooltip
var ctrl = new VisioAutomation.Shapes.ControlCells();
ctrl.X = "Width*0.5";
ctrl.Y = "Height*0.5";
ctrl.Tip = "Adjust";
int row2 = VisioAutomation.Shapes.ControlHelper.Add(shape, ctrl);
```

The `Add` overload that takes `ControlCells` auto-defaults `XDynamics` / `YDynamics` to `Controls.Row_N` / `Controls.Row_N.Y` if you leave them null &mdash; that's the standard self-referential dynamics for a free handle.

## Update an existing handle

```csharp
var ctrl = new VisioAutomation.Shapes.ControlCells();
ctrl.Tip = "Resize me";

VisioAutomation.Shapes.ControlHelper.Set(shape, row, ctrl);
```

## Read existing handles

`ControlCells.GetCells` reads the cells off a single shape; the multi-shape overload reads from a page in one batched call.

```csharp
var cells = VisioAutomation.Shapes.ControlCells.GetCells(
    shape,
    VisioAutomation.Core.CellValueType.Formula);
```

## Count and delete

```csharp
int count = VisioAutomation.Shapes.ControlHelper.GetCount(shape);

VisioAutomation.Shapes.ControlHelper.Delete(shape, 0);   // first handle
```
