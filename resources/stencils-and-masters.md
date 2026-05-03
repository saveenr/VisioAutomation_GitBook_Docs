# Stencils and Masters

## Loading a Stencil

```
short stencilflags = (short)IVisio.VisOpenSaveArgs.visOpenRO | (short)IVisio.VisOpenSaveArgs.visOpenDocked; 
var stencildoc = visapp.Documents.OpenEx(“basic_u.vss”, stencilflags);
```

## Dropping a Single Shape

```
var rectangle = stencildoc.Masters["Rectangle" ]; 
var shape = page.Drop(rectangle, 1.0, 2.0 );
```

## Dropping Multiple Shapes

```
var rectangle = stencildoc.Masters[ "Rectangle"]; 
var masters = new object[] { rectangle }; 
var xys = new double[] { 1.0, 2.0}; 
System.Array out_ids_sa; 
page.DropManyU(masters, xys, out out_ids_sa); 
short[] out_ids = (short[]) out_ids_sa;
```

## Getting a Cell by Name

```
// C# Visual Studio 2010
var cell = shape.CellsU[ "Char.Style" ]
```
