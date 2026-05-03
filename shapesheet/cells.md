# SRC structs

## Identifying cells

Visio has two ways of identifying a cell:

* By name - for example "FillForegnd"
* By a triple of indices - (a **section index**, a **row index**, and a **cell index**). We'll refer to this (s,r,c) pattern often

These (s,r,c) values are used a lot when working with the Visio ShapeSheet. VisioAutomation defines a special struct called SRC to store these values. The full name of the struct is **VisioAutomation.ShapeSheet.SRC**.

### Create a SRC struct from literal numbers

```
using VA=VisioAutomation;

var src_0 = new VA.Core.SRC(0,0,0);
var src_1 = new VA.Core.SRC((short) 0,(short) 0,(short) 0);
```

### Create a SRC struct from well known Indices from the Visio PIA

The SRC below to points the fill foreground color

```
using SEC=Microsoft.Office.Interop.Visio.VisSectionIndices;
using ROW= Microsoft.Office.Interop.Visio.VisRowIndices;
using CEL= Microsoft.Office.Interop.Visio.VisCellIndices;
using VA=VisioAutomation;

var fg_src = VA.Core.SRC(
  SEC.visSectionObject, 
  ROW.visRowFill, 
  CEL.visFillForegnd);  
```

### Common Cell SRCs in VisioAuatomation



VisioAutomation comes with a class that has predefined values for many of these (s,r,c) values for cells.

The static class called VisioAutomation.ShapeSheet.SRCConstant contains hundreds of cells.&#x20;

So to get the (s,r,c) value for FillPattern we just need to write this

```
using VA=VisioAutomation;

var filpat_src = VA.ShapeSheet.SRCConstants.FillPattern;
```

Here are some commonly used ones:

```
VA.Core.SRCConstants.Char_Color 
VA.Core.SRCConstants.Char_Font 
VA.Core.SRCConstants.Char_Size 
VA.Core.SRCConstants.FillForegnd 
VA.Core.SRCConstants.FillForegndTrans 
VA.Core.SRCConstants.FillPattern 
VA.Core.SRCConstants.FillBkgnd 
VA.Core.SRCConstants.FillBkgndTrans 
VA.Core.SRCConstants.LinePattern 
VA.Core.SRCConstants.LineWeight 
VA.Core.SRCConstants.LineColor 
VA.Core.SRCConstants.Height 
VA.Core.SRCConstants.Width 
VA.Core.SRCConstants.PinX 
VA.Core.SRCConstants.PinY
```



## Immutability

SRC values are immutable structs. Once created their value cannot be changed

## Dealing with the same cell in Multiple rows&#x20;

Often you may have a (s,r,c) value , and you need almost the same cell, but for a different row. use the **SRC.ForRow** method to construct a new (s,r,c) value from an existing one&#x20;

```
var cc_src_row0 = VA.Core.SRCConstants.Char_Color;
var cc_src_row1 = cc_src_row0.ForRow(1);
```

## Converting between cell names and (s,r,c) values&#x20;

Suppose you have a cell name stored as a string - "PinX" for example - and now you need to get the (s,r,c) value. There are a static methods that will provide the conversion for you for many common cell that are referred to by names.&#x20;

The first is ShapeSheetHelper.GetSRCFromName

```
 var pinx_src = VA.ShapeSheet.ShapeSheetHelper.GetSRCFromName("PinX");
```

If the cell name is not one the method handles, then an exception will be thrown.&#x20;

So, as an alternative you can try the **ShapeSheetHelperTryGetSRCFromName** which returns a nullbable SRC value;

```
var pinx_src = VA.ShapeSheet.ShapeSheetHelper.TryGetSRCFromName("PinX");
if (pinx_src.HasValue)
{
    // do nothing
}
```

## Converting between (s,r,c) values and cell names

There's no helper method in the library that will do this. It will be considered for a future version.
