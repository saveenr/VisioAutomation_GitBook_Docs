# Documentation changes

This page summarizes notable changes to the **VisioAutomation** documentation so returning readers can find what's new without re-reading every page.

## 2026-05 &mdash; Compile-failure pass

A compile pass over every C# code block on the site, prompted by a stale snippet found in the source repo's `readme.md`. Fixes:

* **`shapesheet/modify-the-shapesheet.md`** &mdash; replaced `SrcConstants.PinX` / `SrcConstants.PinY` (which don't exist on `SrcConstants`) with `XFormPinX` / `XFormPinY`. Added a one-line note about the cell-record-prefixed naming so the right constant name is easier to predict.
* **`shapesheet/query-the-shapesheet.md`** &mdash; the `SectionQuery` examples were calling a 1-arg constructor that doesn't exist (`new SectionQuery(visSectionProp)`) and a `.Columns.Add` property that doesn't exist on `SectionQuery`. Rewrote both single-shape and multi-shape examples to use `new SectionQuery()` followed by `query.Add(sectionIndex)` (which returns a `DataColumns` you then add cell columns to). Also corrected the iteration: `DataRowGroup<T>` enumerates *sections*, not rows, so the `foreach` example needed a nested loop to reach a row.
* **`connection-points.md`**, **`hyperlinks.md`**, **`control-handles.md`** &mdash; the `Set` overload takes the row index as `short`, but `Add` returns `int`, so the as-written `Set(shape, row, ...)` calls didn't compile. Each example now casts to `(short)row` and a one-line note explains the type asymmetry.

## 2026-05 &mdash; Refresh against current API

A pass aligning every page with the **VisioAutomation** library's current public API. The underlying library had drifted (class renames, namespace renames, struct renames) and several pages still described the older shape; the docs now match what's in the source.

### Page-level changes

* **`shapesheet/cells.md`** &mdash; rewritten for the modern naming. The struct is `Src` (not `SRC`); the helper class is `VisioAutomation.Core.SrcConstants` (not `VisioAutomation.ShapeSheet.SRCConstants`); constant names dropped underscores (`Char_Color` &rarr; `CharColor`, `FillForegnd` &rarr; `FillForeground`, etc.). Removed the "Converting between cell names and (s,r,c)" section &mdash; it documented a `ShapeSheetHelper.GetSRCFromName` helper that doesn't exist.
* **`shapesheet/query-the-shapesheet.md`** &mdash; rewritten with a return-type table that makes it clear which type each query overload returns (`DataRows<T>` / `DataRowGroup<T>` / `DataRowGroups<T>`). Fixed the indexer syntax (`[r][c]`, not `[r,c]`); fixed the multi-shape `SectionQuery` example to use `Core.ShapeIDPairs` instead of `IList<int>`.
* **`shapesheet/modify-the-shapesheet.md`** &mdash; rewritten for the current writer API: `SrcWriter` and `SidSrcWriter` (formerly `SRCUpdate` / `SIDSRCUpdate`); writer methods are `SetValue` / `SetFormula` / `Commit(target, CellValueType)` (formerly `SetFormula` / `SetResult` / `Execute`); `shape.CellsU["PinX"]` is now used in place of the case-sensitive `shape.Cells["Pinx"]`.
* **`user-defined-cells.md`** &mdash; rewritten using `UserDefinedCellHelper.GetDictionary(shape, CellValueType)`, `Set(shape, name, UserDefinedCellCells)`, and `ShapeIDPairs.FromShapes(...)` for the multi-shape pattern. The helper class was renamed `UserDefinedCellsHelper` &rarr; `UserDefinedCellHelper` (singular).
* **`custom-properties.md`** &mdash; rewritten with the full `CustomPropertyCells` field set (`Value`, `Prompt`, `Label`, `Format`, `Type`, `Calendar`, `Invisible`, `LangID`, `SortKey`, `Ask`); fixed namespace references (`Shapes.CustomProperties.X` was wrong). Multi-shape examples now use the `ShapeIDPairs` page-level overload.
* **`extension-methods.md`** + **`stencils-and-masters.md`** &mdash; replaced dead `VisioAutomation.Drawing.{Point, Size}` references with `VisioAutomation.Core.{Point, Size}`. Fixed broken `=>` lambda formatting and tightened prose.
* **`shapesheet/README.md`** &mdash; previously a stub. Now a real section landing introducing the ShapeSheet, the cell-addressing approach, and the query / modify split.
* **`compiling.md`** &mdash; updated to Visual Studio 2022 / C# 8.0; .NET Framework 4.5 references updated to **4.5.2** to reflect the recent TFM bump; pointed at the source repo's [BUILDING.md](https://github.com/saveenr/VisioAutomation/blob/master/docs/BUILDING.md) for the up-to-date dev-pack install instructions.

### Pages added

* `documentation-changes.md` (this page).
* [Hyperlinks](hyperlinks.md) &mdash; documents `HyperlinkHelper` and `HyperlinkCells`.
* [Lock cells](lock-cells.md) &mdash; documents the `LockCells` record and the `SrcWriter` pattern for applying / clearing locks.
* [Control handles](control-handles.md) &mdash; documents `ControlHelper` and `ControlCells`.
* [Connection points](connection-points.md) &mdash; documents `ConnectionPointHelper` and `ConnectionPointCells`.
* [Connectors](connectors.md) &mdash; documents `ConnectorHelper`, including manual connection and AutoConnect.
* [Shape format / layout / xform cells](shape-cells.md) &mdash; documents `ShapeFormatCells`, `ShapeLayoutCells`, `ShapeXFormCells` and how to combine them in a single `SrcWriter` transaction.
* [Page cells](page-cells.md) &mdash; documents `PageHelper` plus the four PageSheet records (`PageFormatCells`, `PageLayoutCells`, `PagePrintCells`, `PageRulerAndGridCells`); covers `Duplicate`, `SetSize`/`GetSize`, and `ResizeToFitContents`.
* [Text formatting](text-formatting.md) &mdash; documents `CharacterCells`, `ParagraphCells`, `TextBlockCells`, multi-row writes, plus the `TextHelper` tab-stop helpers (`GetTabStops`/`SetTabStops`).
* [Geometry](geometry.md) &mdash; documents `GeometryHelper`, `GeometrySection`, `GeometryRow` for building custom paths.
* [Application](application.md) &mdash; documents `ApplicationHelper`: version detection, alert-bypass quit, content-folder location, and the foreground-window helper.
* [Undo scope](undo-scope.md) &mdash; documents `UndoScope`, the `IDisposable` wrapper around `BeginUndoScope`/`EndUndoScope`, including the `Commit = false` rollback pattern.
* [Analyzers](analyzers.md) &mdash; documents `ConnectionAnalyzer.GetDirectedEdges`, `ConnectionAnalyzerOptions`, and the `EdgeDirectionSource` / `EdgeNoArrowsHandling` enums.
* [Visio error log](logging.md) &mdash; documents `LoggingHelper.GetXmlErrorLogFilename`, `XmlErrorLog`, and the `LogSession` / `LogRecord` shape.
* [Exception types](exceptions.md) &mdash; documents `AutomationException` and its subclasses `VisioOperationException` and `InternalAssertionException`.

### Pages rewritten

* `extension-methods.md` &mdash; replaced the original three-method intro with a comprehensive reference organized by purpose (LINQ bridges over the COM collection types, drawing primitives, master dropping, ShapeSheet I/O, geometry / coordinates, one-offs). Fixed an example that referenced a non-existent `Fonts.AsEnumerable()` &mdash; the actual extensions are named `ToEnumerable()` and `ToList()`.

### Pages removed

* `convert-values.md` &mdash; documented a `VisioAutomation.Convert` static class that doesn't exist in the library. Removed rather than rewritten.
* `resources/stencils-and-masters.md` &mdash; was a duplicate of the top-level `stencils-and-masters.md`. Removed.

### Strict-accuracy fixes

Smaller fixes to call out:

* **`README.md`** &mdash; replaced a stale link to a separate `VisioPowerShell` repo (the PowerShell module now lives in the main `VisioAutomation` repo).
* **`SUMMARY.md`** &mdash; programmatic verification that every link target exists; minor cleanup along the way.
* **`classes.md`**, **`namespaces.md`**, **`related-projects.md`** &mdash; added context paragraphs above the diagrams; expanded the related-projects list to include the Visio PowerShell module and PSVA.

For the in-repo source-of-truth detail (commit hashes per change), see the [project's commit history](https://github.com/saveenr/VisioAutomation/commits/master/).
