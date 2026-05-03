# Convert values

The static class VisioAutomation.Convert contains many useful methods for converting values

## Useful conversions for ShapeSheets

### Convert.BoolToFormula

true is converted to "1" false is converted to "0"

```
string formula_true = VA.Convert.BoolToFormula( true );
string formula_false = VA.Convert.BoolToFormula( false );
```

### Convert.BoolToShort

true is converted to 1 false is converted to 0

```
short t = VA.Convert.BoolToShort( true );
short  f = VA.Convert.BoolToShort( false );
```

### Convert.ColorToFormulaHSL

```
short formula = VA.Convert.ColorToFormulaHSL( 100, 130,47 );
```

formula will have a value like "HSL(100,130,47)"

### Convert.ColorToFormulaRGB

```
short formula = VA.Convert.ColorToFormulaRGB( 100, 130,47 );
```

formula will have value "RGB(100,130,47)"

### Convert.ColorToFormulaRGB

```
short formula = VA.Convert.ColorToFormulaRGB( 100, 130,47 );
```

formula will have value "RGB(100,130,47)"

### Convert.DegreesToRadians and Convert.RadiansToDegrees

```
double d = Convert.RadiansToDegrees( System.Path.Pi );
double r = Convert.DegreesToRadians( 25.0 );
```

### Convert.DoubleToBool and Convert.ShortToBool

```
bool a = Convert.DoubleToBool( 1.0 );
bool b = Convert.DoubleToBool( 0.0 );
```

zero -> false non-zero -> true

```
bool a = Convert.ShortToBool( (short) 1 );
bool b = Convert.ShortToBool( (short) 0 );
```

zero -> false non-zero -> true

## StringToFormulaString

Converts strings that contain quotes and spaces into a form that can be used in a formula

### FormulaStringToString

Decodes a formula into a normal string

### InchesToPoints & PointsToInches

```
double p = Convert.InchesToPoints ( 2.50 );
double p = Convert.PointsToInches( 25.0 );
```
