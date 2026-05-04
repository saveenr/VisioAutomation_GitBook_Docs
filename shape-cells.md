# Shape format / layout / xform cells

A shape's ShapeSheet has many sections. The ones a typical script touches most often are split across three records under `VisioAutomation.Shapes`:

* **`ShapeXFormCells`** &mdash; size and position (`PinX`, `PinY`, `Width`, `Height`, `Angle`, `LocPinX`, `LocPinY`).
* **`ShapeFormatCells`** &mdash; visual appearance: fill colors / patterns / shadows, line color / weight / pattern, line arrow ends.
* **`ShapeLayoutCells`** &mdash; layout / routing behavior (used by Visio's auto-layout): `ConnectorFixedCode`, `ShapePlaceStyle`, `ShapePermeableX/Y`, `LineJump*`, etc.

Each follows the same cell-record pattern as [`Custom properties`](custom-properties.md), [`User-defined cells`](user-defined-cells.md), and [`Lock cells`](lock-cells.md): set the fields you care about, hand the record to a `SrcWriter`, commit. Cells you don't set are left untouched.

## Set position and size

```csharp
var xform = new VisioAutomation.Shapes.ShapeXFormCells();
xform.PinX   = 4.0;
xform.PinY   = 2.0;
xform.Width  = 1.5;
xform.Height = 1.0;

var writer = new VisioAutomation.ShapeSheet.Writers.SrcWriter();
writer.SetValues(xform);
writer.Commit(shape, VisioAutomation.Core.CellValueType.Formula);
```

`PinX`/`PinY` are the on-page coordinates of the shape's pin (its rotation/anchor point). `Width`/`Height` are the shape's size. `Angle` is rotation.

## Set fill and line formatting

```csharp
var fmt = new VisioAutomation.Shapes.ShapeFormatCells();
fmt.FillForeground = "rgb(255,128,0)";
fmt.FillPattern    = 1;
fmt.LineColor      = "rgb(0,0,0)";
fmt.LineWeight     = "0.5 pt";
fmt.LinePattern    = 1;

var writer = new VisioAutomation.ShapeSheet.Writers.SrcWriter();
writer.SetValues(fmt);
writer.Commit(shape, VisioAutomation.Core.CellValueType.Formula);
```

`ShapeFormatCells` covers the whole `Fill Format` and `Line Format` sections plus the shadow cells. See the `SrcConstants.Fill*` / `SrcConstants.Line*` constants for the underlying cell list.

## Set layout / routing behavior

`ShapeLayoutCells` is mostly relevant when working with Visio's auto-layout engine (see [`Format-VisioPage -LayoutStyle`](https://saveenr.gitbook.io/visiopowershell/cmdlets/pages/format-visiopage) on the PowerShell side).

```csharp
var layout = new VisioAutomation.Shapes.ShapeLayoutCells();
layout.ShapePlaceStyle = 0;          // PlaceCenter
layout.ShapePermeableX = 1;
layout.ShapePermeableY = 1;

var writer = new VisioAutomation.ShapeSheet.Writers.SrcWriter();
writer.SetValues(layout);
writer.Commit(shape, VisioAutomation.Core.CellValueType.Formula);
```

## Read existing values

Each record exposes a static `GetCells` method:

```csharp
var xform = VisioAutomation.Shapes.ShapeXFormCells.GetCells(
    shape,
    VisioAutomation.Core.CellValueType.Result);

System.Console.WriteLine($"Width = {xform.Width.Value}");
```

Pass `CellValueType.Formula` to read the underlying formulas instead of evaluated results.

## Combine multiple records in one writer

A `SrcWriter` accepts as many `SetValues` calls as you want. Mix records in a single transaction so all the changes commit atomically:

```csharp
var writer = new VisioAutomation.ShapeSheet.Writers.SrcWriter();
writer.SetValues(xform);
writer.SetValues(fmt);
writer.SetValues(layout);
writer.Commit(shape, VisioAutomation.Core.CellValueType.Formula);
```

## See also

* [Modify the ShapeSheet](shapesheet/modify-the-shapesheet.md) &mdash; the underlying `SrcWriter` / `SidSrcWriter` API.
* [Lock cells](lock-cells.md), [Custom properties](custom-properties.md), [User-defined cells](user-defined-cells.md) &mdash; other per-shape ShapeSheet records.
