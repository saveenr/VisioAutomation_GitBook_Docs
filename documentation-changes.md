# Documentation changes

This page summarizes notable changes to the **VisioAutomation** documentation so returning readers can find what's new without re-reading every page.

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

### Pages removed

* `convert-values.md` &mdash; documented a `VisioAutomation.Convert` static class that doesn't exist in the library. Removed rather than rewritten.
* `resources/stencils-and-masters.md` &mdash; was a duplicate of the top-level `stencils-and-masters.md`. Removed.

### Strict-accuracy fixes

Smaller fixes to call out:

* **`README.md`** &mdash; replaced a stale link to a separate `VisioPowerShell` repo (the PowerShell module now lives in the main `VisioAutomation` repo).
* **`SUMMARY.md`** &mdash; programmatic verification that every link target exists; minor cleanup along the way.
* **`classes.md`**, **`namespaces.md`**, **`related-projects.md`** &mdash; added context paragraphs above the diagrams; expanded the related-projects list to include the Visio PowerShell module and PSVA.

For the in-repo source-of-truth detail (commit hashes per change), see the [project's commit history](https://github.com/saveenr/VisioAutomation/commits/master/).
