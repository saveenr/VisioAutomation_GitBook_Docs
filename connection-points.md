# Connection points

A **connection point** is a small `x` mark on a shape that connectors can snap to. The `Connection Points.` section of a shape's ShapeSheet defines them. VisioAutomation exposes them via `VisioAutomation.Shapes.ConnectionPointHelper` and the `ConnectionPointCells` record.

## The `ConnectionPointCells` record

| Field          | Purpose                                              |
| -------------- | ---------------------------------------------------- |
| `X`, `Y`       | Position formulas. Required when adding a row.       |
| `DirX`, `DirY` | Direction vector for the connection point.           |
| `Type`         | `0` = inward, `1` = outward, `2` = inward & outward. |

All fields are `Core.CellValue`-typed.

## Add a connection point

`Add` requires both `X` and `Y` formulas; it throws otherwise. Returns the new row index.

```csharp
var cp = new VisioAutomation.Shapes.ConnectionPointCells();
cp.X = "Width*0.5";
cp.Y = "Height";

int row = VisioAutomation.Shapes.ConnectionPointHelper.Add(shape, cp);
```

## Update an existing connection point

`Set` takes the row index as a `short` (whereas `Add` returns `int`), so cast when round-tripping a row index from `Add`:

```csharp
var cp = new VisioAutomation.Shapes.ConnectionPointCells();
cp.X = "Width";
cp.Y = "Height*0.5";

VisioAutomation.Shapes.ConnectionPointHelper.Set(shape, (short)row, cp);
```

## Read existing connection points

`GetCells` is on the `ConnectionPointCells` class:

```csharp
var cp = VisioAutomation.Shapes.ConnectionPointCells.GetCells(
    shape,
    VisioAutomation.Core.CellValueType.Formula);
```

## Count connection points

```csharp
int count = VisioAutomation.Shapes.ConnectionPointHelper.GetCount(shape);
```

## Delete connection points

```
VisioAutomation.Shapes.ConnectionPointHelper.Delete(shape, 0);   // delete row 0

// Delete all connection points on the shape
int removed = VisioAutomation.Shapes.ConnectionPointHelper.Delete(shape);
```

## See also

* [Connectors](connectors.md) — how shapes get wired together. Connection points define where a connector can attach.
