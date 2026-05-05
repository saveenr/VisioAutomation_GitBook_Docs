# Connectors

VisioAutomation exposes connector-drawing helpers on `VisioAutomation.Shapes.ConnectorHelper`. There are two patterns: **manual connection** (you provide a connector shape and the helper glues its endpoints to the source / destination shapes) and **AutoConnect** (Visio's built-in connection routing).

## Connect two shapes with a manual connector

Drop a connector master onto the page first, then glue its endpoints with `ConnectShapes`:

```csharp
var stencil = doc.Application.Documents.OpenStencil("basic_u.vss");
var dyncon  = stencil.Masters["Dynamic Connector"];

var shape1 = page.DrawRectangle(0, 0, 1, 1);
var shape2 = page.DrawRectangle(2, 2, 3, 3);
var con    = page.Drop(dyncon, 0, 0);

VisioAutomation.Shapes.ConnectorHelper.ConnectShapes(shape1, shape2, con);
```

The `connector_shape` cannot be the same shape as `from_shape` or `to_shape`.

## Use AutoConnect

When you don't supply a connector explicitly, Visio's AutoConnect chooses one for you:

```csharp
VisioAutomation.Shapes.ConnectorHelper.ConnectShapes(shape1, shape2, null, false);
```

The `manual_connection: false` argument tells the helper to call `IVisio.Shape.AutoConnect` instead of gluing endpoints itself.

## Connect many shapes at once

The page-level overload connects an array of `from` shapes to an array of `to` shapes (zipped) and returns the new connector shapes. The two arrays must be the same length. Pass a connector master if you want a specific connector style; pass `null` to use AutoConnect.

```csharp
var stencil = doc.Application.Documents.OpenStencil("basic_u.vss");
var dyncon  = stencil.Masters["Dynamic Connector"];

var connectors = VisioAutomation.Shapes.ConnectorHelper.ConnectShapes(
    page,
    new[] { shape_a, shape_b, shape_c },
    new[] { shape_b, shape_c, shape_d },
    dyncon);
```

`connectors` is the list of `IVisio.Shape` objects representing the new connector instances on the page.

## See also

* [Connection points](connection-points.md) — where on a shape a connector can attach.
* [Stencils and masters](stencils-and-masters.md) — loading connector masters from a stencil.
