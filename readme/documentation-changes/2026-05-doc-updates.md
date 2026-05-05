# 2026-05 doc updates

## 2026-05 — Split text-formatting into per-record pages

Same treatment as the page-cells / shape-cells / extension-methods splits. The combined `text-formatting.md` page covered three cell records plus a tab-stops helper under one heading; split into one page per topic in a new `text/` subfolder, with the original page kept as the overview that links to all four (the `saveenr.gitbook.io/visioautomation/text-formatting` URL is preserved).

* [Character cells](../../text/character.md) — `CharacterCells` (font, size, color, weight, decorations, localization). Field tables broken out by sub-section. The "Multiple rows" example moved here, since this is the canonical example of a section that holds many rows.
* [Paragraph cells](../../text/paragraph.md) — `ParagraphCells` (alignment, indent, spacing, bullets). Cross-links to the multi-row pattern on the Character page rather than repeating it.
* [Text-block cells](../../text/block.md) — `TextBlockCells` (margins, vertical alignment, default tab stop, background, direction). Documents the surprising-but-real method name `GetTextBlockCells` (the other text records expose `GetCells`); the previous overview's catch-all "every record has `GetCells`" claim was wrong for this one record.
* [Tab stops](../../text/tab-stops.md) — `TextHelper.SetTabStops` / `GetTabStops` + the `TabStop` struct and `TabStopAlignment` enum (`Left`, `Center`, `Right`, `Decimal`, `Comma`).
* [Text formatting](../../text-formatting.md) — rewritten as a one-page overview with a record table linking to the four sub-pages; the "combine multiple records in one writer" example stays here.

`SUMMARY.md` updated to nest the four sub-pages under the overview entry.

## 2026-05 — Split page-cells into per-record pages

Same treatment as the extension-methods and shape-cells splits below. The combined `page-cells.md` page covered four cell records plus a helper class under one heading; split into one page per record in a new `pages/` subfolder, with the original page kept as the overview that links to all five (the `saveenr.gitbook.io/visioautomation/page-cells` URL is preserved).

* [Page format cells](../../pages/format.md) — `PageFormatCells` (size, scale, drawing-units, page shadow). Field tables broken out by sub-section.
* [Page layout cells](../../pages/layout.md) — `PageLayoutCells` (auto-layout: avenues, blocks, line-jump, placement, routing). Tables broken out by purpose.
* [Page print cells](../../pages/print.md) — `PagePrintCells` (margins, paper, orientation, scale, tiling).
* [Page ruler and grid cells](../../pages/ruler-grid.md) — `PageRulerAndGridCells` (per-axis ruler density / origin and grid density / origin / spacing).
* [Page helper](../../pages/helper.md) — `PageHelper.SetSize` / `GetSize`, `ResizeToFitContents`, `Duplicate`.
* [Page cells](../../page-cells.md) — rewritten as a one-page overview with a record table linking to the four cell-record pages and the helper page; the "combine multiple records in one writer" example stays here.

`SUMMARY.md` updated to nest the five sub-pages under the overview entry.

## 2026-05 — Split extension-methods into per-category pages

The combined `extension-methods.md` page covered six independent groups of extensions (LINQ bridges, drawing primitives, master dropping, ShapeSheet I/O, coordinates, one-offs) under one heading. Split it into one page per category in a new `extensions/` subfolder, with the original page kept as the overview that links to all six (the `saveenr.gitbook.io/visioautomation/extension-methods` URL is preserved).

* [LINQ bridges](../../extensions/linq.md) — `ToEnumerable()` / `ToList()` over `Pages`, `Shapes`, `Masters`, etc.
* [Drawing primitives](../../extensions/drawing.md) — `DrawRectangle(rect)`, `DrawPolyline`, `DrawBezier`, `DrawQuarterArc`, `DrawNurbs`.
* [Master dropping](../../extensions/dropping.md) — `Drop(master, point)`, `DropManyU(masters, points)`.
* [Typed ShapeSheet I/O](../../extensions/shapesheet.md) — `GetFormulasU` / `GetResults<T>` / `SetFormulas` / `SetResults` keyed by `Src` / `SidSrc`.
* [Coordinates and bounding boxes](../../extensions/coordinates.md) — `GetBoundingBox`, `XYFromPage`, `XYToPage`, `ResizeToFitContents`.
* [One-off extensions](../../extensions/misc.md) — `Quit`, `Close`, `OpenStencil`, view / window rects, `GetShapesFromIDs`.
* [Extension methods](../../extension-methods.md) — rewritten as a one-page overview with a category table linking to the six.

`SUMMARY.md` updated to nest the six sub-pages under the overview entry.

## 2026-05 — Split shape-cells into per-record pages

The combined `shape-cells.md` page covered three separate cell records under one heading. Split it so each record has its own page, with the original page kept as a thin overview that links to the three:

* [Shape XForm cells](../../shape-xform-cells.md) — `ShapeXFormCells` (size, position, rotation, local pin).
* [Shape format cells](../../shape-format-cells.md) — `ShapeFormatCells` (fill, line, fill shadow). Field tables now broken out by sub-section.
* [Shape layout cells](../../shape-layout-cells.md) — `ShapeLayoutCells` (auto-layout placement, routing, line-jump).
* [Shape cells](../../shape-cells.md) — rewritten as a one-page overview pointing to the three above; the "combine multiple records in one writer" example stays here.

`SUMMARY.md` was updated to nest the three sub-pages under the overview entry.

## 2026-05 — Compile-failure pass

A compile pass over every C# code block on the site, prompted by a stale snippet found in the source repo's `readme.md`. Fixes:

* **`shapesheet/modify-the-shapesheet.md`** — replaced `SrcConstants.PinX` / `SrcConstants.PinY` (which don't exist on `SrcConstants`) with `XFormPinX` / `XFormPinY`. Added a one-line note about the cell-record-prefixed naming so the right constant name is easier to predict.
* **`shapesheet/query-the-shapesheet.md`** — the `SectionQuery` examples were calling a 1-arg constructor that doesn't exist (`new SectionQuery(visSectionProp)`) and a `.Columns.Add` property that doesn't exist on `SectionQuery`. Rewrote both single-shape and multi-shape examples to use `new SectionQuery()` followed by `query.Add(sectionIndex)` (which returns a `DataColumns` you then add cell columns to). Also corrected the iteration: `DataRowGroup<T>` enumerates _sections_, not rows, so the `foreach` example needed a nested loop to reach a row.
* **`connection-points.md`**, **`hyperlinks.md`**, **`control-handles.md`** — the `Set` overload takes the row index as `short`, but `Add` returns `int`, so the as-written `Set(shape, row, ...)` calls didn't compile. Each example now casts to `(short)row` and a one-line note explains the type asymmetry.

## 2026-05 — Refresh against current API

A pass aligning every page with the **VisioAutomation** library's current public API. The underlying library had drifted (class renames, namespace renames, struct renames) and several pages still described the older shape; the docs now match what's in the source.

### Page-level changes

* **`shapesheet/cells.md`** — rewritten for the modern naming. The struct is `Src` (not `SRC`); the helper class is `VisioAutomation.Core.SrcConstants` (not `VisioAutomation.ShapeSheet.SRCConstants`); constant names dropped underscores (`Char_Color` → `CharColor`, `FillForegnd` → `FillForeground`, etc.). Removed the "Converting between cell names and (s,r,c)" section — it documented a `ShapeSheetHelper.GetSRCFromName` helper that doesn't exist.
* **`shapesheet/query-the-shapesheet.md`** — rewritten with a return-type table that makes it clear which type each query overload returns (`DataRows<T>` / `DataRowGroup<T>` / `DataRowGroups<T>`). Fixed the indexer syntax (`[r][c]`, not `[r,c]`); fixed the multi-shape `SectionQuery` example to use `Core.ShapeIDPairs` instead of `IList<int>`.
* **`shapesheet/modify-the-shapesheet.md`** — rewritten for the current writer API: `SrcWriter` and `SidSrcWriter` (formerly `SRCUpdate` / `SIDSRCUpdate`); writer methods are `SetValue` / `SetFormula` / `Commit(target, CellValueType)` (formerly `SetFormula` / `SetResult` / `Execute`); `shape.CellsU["PinX"]` is now used in place of the case-sensitive `shape.Cells["Pinx"]`.
* **`user-defined-cells.md`** — rewritten using `UserDefinedCellHelper.GetDictionary(shape, CellValueType)`, `Set(shape, name, UserDefinedCellCells)`, and `ShapeIDPairs.FromShapes(...)` for the multi-shape pattern. The helper class was renamed `UserDefinedCellsHelper` → `UserDefinedCellHelper` (singular).
* **`custom-properties.md`** — rewritten with the full `CustomPropertyCells` field set (`Value`, `Prompt`, `Label`, `Format`, `Type`, `Calendar`, `Invisible`, `LangID`, `SortKey`, `Ask`); fixed namespace references (`Shapes.CustomProperties.X` was wrong). Multi-shape examples now use the `ShapeIDPairs` page-level overload.
* **`extension-methods.md`** + **`stencils-and-masters.md`** — replaced dead `VisioAutomation.Drawing.{Point, Size}` references with `VisioAutomation.Core.{Point, Size}`. Fixed broken `=>` lambda formatting and tightened prose.
* **`shapesheet/README.md`** — previously a stub. Now a real section landing introducing the ShapeSheet, the cell-addressing approach, and the query / modify split.
* **`compiling.md`** — updated to Visual Studio 2022 / C# 8.0; .NET Framework 4.5 references updated to **4.5.2** to reflect the recent TFM bump; pointed at the source repo's [BUILDING.md](https://github.com/saveenr/VisioAutomation/blob/master/docs/BUILDING.md) for the up-to-date dev-pack install instructions.

### Pages added

* `documentation-changes.md` (this page).
* [Hyperlinks](../../hyperlinks.md) — documents `HyperlinkHelper` and `HyperlinkCells`.
* [Lock cells](../../lock-cells.md) — documents the `LockCells` record and the `SrcWriter` pattern for applying / clearing locks.
* [Control handles](../../control-handles.md) — documents `ControlHelper` and `ControlCells`.
* [Connection points](../../connection-points.md) — documents `ConnectionPointHelper` and `ConnectionPointCells`.
* [Connectors](../../connectors.md) — documents `ConnectorHelper`, including manual connection and AutoConnect.
* [Shape format / layout / xform cells](../../shape-cells.md) — documents `ShapeFormatCells`, `ShapeLayoutCells`, `ShapeXFormCells` and how to combine them in a single `SrcWriter` transaction.
* [Page cells](../../page-cells.md) — documents `PageHelper` plus the four PageSheet records (`PageFormatCells`, `PageLayoutCells`, `PagePrintCells`, `PageRulerAndGridCells`); covers `Duplicate`, `SetSize`/`GetSize`, and `ResizeToFitContents`.
* [Text formatting](../../text-formatting.md) — documents `CharacterCells`, `ParagraphCells`, `TextBlockCells`, multi-row writes, plus the `TextHelper` tab-stop helpers (`GetTabStops`/`SetTabStops`).
* [Geometry](../../geometry.md) — documents `GeometryHelper`, `GeometrySection`, `GeometryRow` for building custom paths.
* [Application](../../application.md) — documents `ApplicationHelper`: version detection, alert-bypass quit, content-folder location, and the foreground-window helper.
* [Undo scope](../../undo-scope.md) — documents `UndoScope`, the `IDisposable` wrapper around `BeginUndoScope`/`EndUndoScope`, including the `Commit = false` rollback pattern.
* [Analyzers](../../analyzers.md) — documents `ConnectionAnalyzer.GetDirectedEdges`, `ConnectionAnalyzerOptions`, and the `EdgeDirectionSource` / `EdgeNoArrowsHandling` enums.
* [Visio error log](../../logging.md) — documents `LoggingHelper.GetXmlErrorLogFilename`, `XmlErrorLog`, and the `LogSession` / `LogRecord` shape.
* [Exception types](../../exceptions.md) — documents `AutomationException` and its subclasses `VisioOperationException` and `InternalAssertionException`.

### Pages rewritten

* `extension-methods.md` — replaced the original three-method intro with a comprehensive reference organized by purpose (LINQ bridges over the COM collection types, drawing primitives, master dropping, ShapeSheet I/O, geometry / coordinates, one-offs). Fixed an example that referenced a non-existent `Fonts.AsEnumerable()` — the actual extensions are named `ToEnumerable()` and `ToList()`.

### Pages removed

* `convert-values.md` — documented a `VisioAutomation.Convert` static class that doesn't exist in the library. Removed rather than rewritten.
* `resources/stencils-and-masters.md` — was a duplicate of the top-level `stencils-and-masters.md`. Removed.

### Strict-accuracy fixes

Smaller fixes to call out:

* **`README.md`** — replaced a stale link to a separate `VisioPowerShell` repo (the PowerShell module now lives in the main `VisioAutomation` repo).
* **`SUMMARY.md`** — programmatic verification that every link target exists; minor cleanup along the way.
* **`classes.md`**, **`namespaces.md`**, **`related-projects.md`** — added context paragraphs above the diagrams; expanded the related-projects list to include the Visio PowerShell module and PSVA.

For the in-repo source-of-truth detail (commit hashes per change), see the [project's commit history](https://github.com/saveenr/VisioAutomation/commits/master/).

