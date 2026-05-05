# Modify the ShapeSheet

## Introduction

A ShapeSheet update allows you to efficiently modify the contents (formulas or results) of a shape's, master's, or page's ShapeSheet in a single batched operation.

## Getting started

Imagine we want to set the PinX of a shape. The slow way uses Visio's interop API directly:

```csharp
var pinx_cell = shape.CellsU["PinX"];
pinx_cell.FormulaU = "5.0";
```

That works but performs one COM round-trip per cell. The faster way uses a writer that batches many cell updates into one COM call.

## Updating one or more cells on a single shape

For a single shape, use `SrcWriter` (cells addressed by `Src`):

```csharp
var writer = new VA.ShapeSheet.Writers.SrcWriter();
writer.SetValue(VA.Core.SrcConstants.XFormPinX, "5.0");
writer.Commit(shape1, VA.Core.CellValueType.Formula);
```

`SrcConstants` exposes the well-known cells using their cell-record-prefixed names: `XFormPinX` / `XFormPinY` for the position cells, `XFormWidth` / `XFormHeight` for the size cells, etc. The `SrcConstants.cs` source file is the authoritative list.

`SetValue` accepts strings, ints, doubles, and bools directly thanks to implicit conversion to `Core.CellValue`, so the example below also works:

```csharp
var writer = new VA.ShapeSheet.Writers.SrcWriter();
writer.SetValue(VA.Core.SrcConstants.XFormPinX, 5.0);
writer.Commit(shape1, VA.Core.CellValueType.Formula);
```

To set multiple cells, just call `SetValue` repeatedly before committing:

```csharp
var writer = new VA.ShapeSheet.Writers.SrcWriter();
writer.SetValue(VA.Core.SrcConstants.XFormPinX, 5.0);
writer.SetValue(VA.Core.SrcConstants.XFormPinY, 3.5);
writer.Commit(shape1, VA.Core.CellValueType.Formula);
```

The same writer also supports committing to a master or a page:

```csharp
writer.Commit(master1, VA.Core.CellValueType.Formula);
writer.Commit(page1, VA.Core.CellValueType.Formula);
```

## Setting results instead of formulas

The choice between writing formulas (the textual expression) and writing results (the evaluated value) is made at `Commit` time, by passing the appropriate `CellValueType`:

```csharp
var writer = new VA.ShapeSheet.Writers.SrcWriter();
writer.SetValue(VA.Core.SrcConstants.XFormPinX, 5.0);
writer.Commit(shape1, VA.Core.CellValueType.Result);
```

A single writer commits either all-formulas or all-results; to do both, build two writers and commit each separately.

## Updating cells across multiple shapes

For multiple shapes on the same page, use `SidSrcWriter` (cells addressed by shape-ID + `Src`, packaged together as `SidSrc`):

```csharp
var writer = new VA.ShapeSheet.Writers.SidSrcWriter();
writer.SetValue(shape1.ID16, src_pinx, 0.5);
writer.SetValue(shape1.ID16, src_piny, 0.5);
writer.SetValue(shape2.ID16, src_pinx, 1.5);
writer.SetValue(shape2.ID16, src_piny, 1.5);
writer.SetValue(shape3.ID16, src_pinx, 2.5);
writer.SetValue(shape3.ID16, src_piny, 2.5);
writer.Commit(page1, VA.Core.CellValueType.Formula);
```
