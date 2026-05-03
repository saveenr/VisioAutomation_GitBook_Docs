# Modify the ShapeSheet

## Introduction

A ShapeSheet update allows you to efficiently modify the the contents (formulas, results, or both) of a shapesheet.

## Getting started

Imagine we want to set the pinx of a shape. To start we'll use the slow way

```
var pinx_cell = shape.Cells["Pinx"];
pinx_cell.FormulaU = "5.0";
```

Updating a single cell on a single shape Now let's map this to an update structure. Because this update occurs to a single specific shape we use the SRCUpdate class

```
var update = new VA.ShapeSheet.Update.SRCUpdate();
update.SetFormula(VA.ShapeSheet.SRCConstants.PinX, "5.0");
update.Execute(shape1);
```

The SetFormula method will automatically convert integers and doubles to the correct string form, so the example below is also valid

```
var update = new VA.ShapeSheet.Update.SRCUpdate();
update.SetFormula(VA.ShapeSheet.SRCConstants.PinX, 5.0);
update.Execute(shape1);
```

And of course to set multiple cells is simple...

The SRCUpdate.SetFormula method will automatically convert integers and doubles to the correct string form, so the example below is also valid

```
var update = new VA.ShapeSheet.Update.SRCUpdate();
update.SetFormula(VA.ShapeSheet.SRCConstants.PinX, 5.0);
update.SetFormula(VA.ShapeSheet.SRCConstants.PinY, 3.5);
update.Execute(shape1);
```

You can set the Results of a cell also

```
var update = new VA.ShapeSheet.Update.SRCUpdate();
update.SetResult(VA.ShapeSheet.SRCConstants.PinX, 5.0, IVisio.VisUnitCodes.visNoCast);
update.Execute(shape1);
```

If you mix both formulas and results in an update, the results will be set first and then the formulas will be set

Setting multiple cells for multiple shapes This is simple. First we need to identify the shapes by their ID. And then we use the SIDSRCUpdate object and execute the update against a page that contains the object.

```
var update = new VA.ShapeSheet.Update.SIDSRCUpdate();
update.SetFormula(shape1.ID16, src_pinx, 0.5);
update.SetFormula(shape1.ID16, src_piny, 0.5);
update.SetFormula(shape2.ID16, src_pinx, 1.5);
update.SetFormula(shape2.ID16, src_piny, 1.5);
update.SetFormula(shape3.ID16, src_pinx, 2.5);
update.SetFormula(shape3.ID16, src_piny, 2.5);
update.Execute(page1);
```
