# SRC structs

## Identifying cells

Visio has two ways of identifying a cell:

* By name - for example "FillForeground"
* By a triple of indices - (a **section index**, a **row index**, and a **cell index**). We'll refer to this (s,r,c) pattern often

These (s,r,c) values are used a lot when working with the Visio ShapeSheet. VisioAutomation defines a special struct called SRC to store these values. The full name of the struct is **VisioAutomation.Core.Src**.

### Create a SRC struct from literal numbers

```
using VA=VisioAutomation;

var src_0 = new VA.Core.Src(0,0,0);
var src_1 = new VA.Core.Src((short) 0,(short) 0,(short) 0);
```

### Create a SRC struct from well known Indices from the Visio PIA

The SRC below points to the fill foreground color

```
using SEC=Microsoft.Office.Interop.Visio.VisSectionIndices;
using ROW= Microsoft.Office.Interop.Visio.VisRowIndices;
using CEL= Microsoft.Office.Interop.Visio.VisCellIndices;
using VA=VisioAutomation;

var fg_src = new VA.Core.Src(
  SEC.visSectionObject, 
  ROW.visRowFill, 
  CEL.visFillForeground);  
```

### Common Cell SRCs in VisioAutomation



VisioAutomation comes with a class that has predefined values for many of these (s,r,c) values for cells.

The static class called VisioAutomation.Core.SrcConstants contains hundreds of cells.&#x20;

So to get the (s,r,c) value for FillPattern we just need to write this

```
using VA=VisioAutomation;

var filpat_src = VA.Core.SrcConstants.FillPattern;
```

Here are some commonly used ones:

```
VA.Core.SrcConstants.CharColor 
VA.Core.SrcConstants.CharFont 
VA.Core.SrcConstants.CharSize 
VA.Core.SrcConstants.FillForeground 
VA.Core.SrcConstants.FillForegroundTransparency 
VA.Core.SrcConstants.FillPattern 
VA.Core.SrcConstants.FillBackground 
VA.Core.SrcConstants.FillBackgroundTransparency 
VA.Core.SrcConstants.LinePattern 
VA.Core.SrcConstants.LineWeight 
VA.Core.SrcConstants.LineColor 
VA.Core.SrcConstants.Height 
VA.Core.SrcConstants.Width 
VA.Core.SrcConstants.PinX 
VA.Core.SrcConstants.PinY
```



## Immutability

SRC values are immutable structs. Once created their value cannot be changed

## Dealing with the same cell in Multiple rows&#x20;

Often you may have a (s,r,c) value , and you need almost the same cell, but for a different row. use the **SRC.ForRow** method to construct a new (s,r,c) value from an existing one&#x20;

```
var cc_src_row0 = VA.Core.SrcConstants.CharColor;
var cc_src_row1 = cc_src_row0.ForRow(1);
```

## Converting between cell names and (s,r,c) values

There is no built-in helper method in the library to convert between a cell name string (like `"PinX"`) and an `Src` value, or vice versa. The current recommended pattern is to use the `SrcConstants` named constants directly — e.g., `VA.Core.SrcConstants.PinX` — rather than starting from a string.
