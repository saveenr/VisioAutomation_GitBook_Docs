# 2026-05 doc updates

## 2026-05: Custom properties typed setters and behavior matrix

[Issue #144](https://github.com/saveenr/VisioAutomation/issues/144) on the source repo landed: the API ergonomics around `CustomPropertyCells` and `UserDefinedCellCells` got two coordinated improvements that the docs now reflect.

* **Typed setters** are the new recommended way to populate the cells: `cp.SetString("v")` / `SetNumber(42)` / `SetBool(true)` / `SetDate(DateTime.Now)` / `SetFormula("=...")` for `CustomPropertyCells`; `cells.SetString(...)` / `SetFormula(...)` for `UserDefinedCellCells`. Each writes a correctly-encoded Visio formula and (for `CustomPropertyCells`) sets `Type` to match.
* **`Value` was renamed to `Formula`** to surface the fact that the field stores a Visio formula, not a literal value. The old `Value` name is kept as an `[Obsolete]` source-compat alias.
* **`CustomPropertyHelper.Set` and `UserDefinedCellHelper.Set` now wrap Visio's opaque `COMException: #NAME?`** in an `ArgumentException` with a self-explanatory message pointing at the typed setters. The trap is now diagnosable rather than cryptic.

[Custom properties](../../custom-properties.md) and [User-defined cells](../../user-defined-cells.md) rewritten to lead with the typed setters, document the full setter surface in a table, note the `Formula` rename, and (on the Custom properties page) add a "What Visio actually stores" matrix describing the per-Type surprises:

* numeric strings sneak through `Type=String`,
* `Type=Boolean` accepts `"1"` / `"0"` as numeric,
* `Type=Date` accepts arbitrary quoted strings as literals,
* empty / `null` / whitespace silently default the cell to `0`.

The full per-Type characterization (driving the test suite) is referenced as a link to [`docs/internal/custom-property-encoding.md`](https://github.com/saveenr/VisioAutomation/blob/master/docs/internal/custom-property-encoding.md) on the source repo, so the gitbook stays user-readable while the engineering-grade matrix has a stable home.

Closes the long-running thread that started with [issue #117](https://github.com/saveenr/VisioAutomation/issues/117).

## 2026-05: Custom-properties: explain that values are formulas, not literals

Surfaced by [issue #117](https://github.com/saveenr/VisioAutomation/issues/117) on the source repo: a user assigned `cp.Value = "testVal"` directly on a `CustomPropertyCells` and got a property whose value read back as `0` instead of `"testVal"`. Root cause is that each `Core.CellValue` field on `CustomPropertyCells` is a Visio *formula*, not a literal; the bare word `testVal` evaluates to a name reference and silently fails. The previous code example on [Custom properties](../../custom-properties.md) showed the same buggy pattern.

Patched the example to call `cp.EncodeValues()` before `Set`, and added a "String values are formulas, not literals" subsection that explains the formula-vs-literal distinction and shows both ways to store a string (`EncodeValues()` or pre-quoting). Cross-links the open [API ergonomics issue (#144)](https://github.com/saveenr/VisioAutomation/issues/144) so readers know the foot-gun is being looked at.

## 2026-05: New page documenting the directed-graph XML format

Added [Directed graph XML format](../../directed-graph-xml.md) under a new **Diagram models** section in the table of contents. The page documents the `<directedgraph>` schema consumed by `Import-VisioModel` and `DirectedGraphDocumentLoader.LoadFromXml`: the root element, `<page>`, `<renderoptions>` (including the `direction`, `connectortype`, and `layout` attributes added in the same pass on the source repo), `<shape>` and `<connector>` schemas, and what's not exposed in XML. Surfaced by [issue #105](https://github.com/saveenr/VisioAutomation/issues/105) on the source repo, where the user reported the format was undocumented.

The new "Diagram models" section is the entry point for the rest of the Tier 3 work (`OrgChart`, `DataTable`, `XmlModel`, `Forms`, the in-memory `DirectedGraphDocument` model, layout-style classes) tracked in [`docs/futures/docs.md`](https://github.com/saveenr/VisioAutomation/blob/master/docs/futures/docs.md) on the source repo.

## 2026-05: Doc-review feedback pass

Acted on feedback from a 2026-05-05 doc-review. Small fixes and a few content additions; bigger items captured as GitHub issues so the gitbook backlog stays focused.

* **[Introduction](../../README.md)**: replaced the bullet list of NuGet packages with a status-and-when-to-use table (clarifies the `VisioAutomation2007` package's deprecated state). Added a "Visio version compatibility" table covering 2010, 2013, 2016, 2019, 2021, and Microsoft 365.
* **[Getting started](../getting-started.md)**: substantial rewrite. Added a "Choosing an entry point" decision table at the top covering the four API tiers (PowerShell module, `VisioScripting.Client`, `VisioAutomation.Models` DOM, `VisioAutomation` helpers). Split the previously-concatenated `var doc = ...; var page = ...` line. Added `csharp` language tags to every code fence. Fixed the "day-to-day like easier so I do recommended it" typo. Added a `doc.SaveAs(...)` step so the tutorial actually finishes with a saved file. Added a "The full program" section that compiles cleanly in `VPlayground`. Added a "Next steps" section with cross-links to the topic pages.
* **[Stencils and masters](../../stencils-and-masters.md)**: removed the duplicated `## Stencils and masters` H2 (the page already had it as H1). Replaced it with a stencil-filenames callout box that cross-links to `ApplicationHelper.GetContentLocation` for finding the right path on the installed Visio version. Added `csharp` language tags to the code fences.
* **[Namespaces](../../namespaces.md)**, **[Classes](../../classes.md)**: added descriptive alt text for the diagram images so the page has fallback content if the image fails to load.
* **[Table of contents](../../SUMMARY.md)**: added section dividers (`## Core APIs`, `## Shape data`, `## Formatting and layout`, `## Diagnostics`, `## Reference`) so the now-50-entry sidebar is scannable.

Bigger items from the same review (out of scope for this pass) filed as GitHub issues on the source repo: [#131 VisioScripting docs gap](https://github.com/saveenr/VisioAutomation/issues/131), [#132 VisioAutomation.Models docs gap](https://github.com/saveenr/VisioAutomation/issues/132), [#133 troubleshooting page](https://github.com/saveenr/VisioAutomation/issues/133). Also recorded in the source repo's `docs/FUTURES.md`.

## 2026-05: Split text-formatting into per-record pages

Same treatment as the page-cells / shape-cells / extension-methods splits. The combined `text-formatting.md` page covered three cell records plus a tab-stops helper under one heading; split into one page per topic in a new `text/` subfolder, with the original page kept as the overview that links to all four (the `saveenr.gitbook.io/visioautomation/text-formatting` URL is preserved).

* [Character cells](../../text/character.md): `CharacterCells` (font, size, color, weight, decorations, localization). Field tables broken out by sub-section. The "Multiple rows" example moved here, since this is the canonical example of a section that holds many rows.
* [Paragraph cells](../../text/paragraph.md): `ParagraphCells` (alignment, indent, spacing, bullets). Cross-links to the multi-row pattern on the Character page rather than repeating it.
* [Text-block cells](../../text/block.md): `TextBlockCells` (margins, vertical alignment, default tab stop, background, direction). Documents the surprising-but-real method name `GetTextBlockCells` (the other text records expose `GetCells`); the previous overview's catch-all "every record has `GetCells`" claim was wrong for this one record.
* [Tab stops](../../text/tab-stops.md): `TextHelper.SetTabStops` / `GetTabStops` + the `TabStop` struct and `TabStopAlignment` enum (`Left`, `Center`, `Right`, `Decimal`, `Comma`).
* [Text formatting](../../text-formatting.md): rewritten as a one-page overview with a record table linking to the four sub-pages; the "combine multiple records in one writer" example stays here.

`SUMMARY.md` updated to nest the four sub-pages under the overview entry.

## 2026-05: Split page-cells into per-record pages

Same treatment as the extension-methods and shape-cells splits below. The combined `page-cells.md` page covered four cell records plus a helper class under one heading; split into one page per record in a new `pages/` subfolder, with the original page kept as the overview that links to all five (the `saveenr.gitbook.io/visioautomation/page-cells` URL is preserved).

* [Page format cells](../../pages/format.md): `PageFormatCells` (size, scale, drawing-units, page shadow). Field tables broken out by sub-section.
* [Page layout cells](../../pages/layout.md): `PageLayoutCells` (auto-layout: avenues, blocks, line-jump, placement, routing). Tables broken out by purpose.
* [Page print cells](../../pages/print.md): `PagePrintCells` (margins, paper, orientation, scale, tiling).
* [Page ruler and grid cells](../../pages/ruler-grid.md): `PageRulerAndGridCells` (per-axis ruler density / origin and grid density / origin / spacing).
* [Page helper](../../pages/helper.md): `PageHelper.SetSize` / `GetSize`, `ResizeToFitContents`, `Duplicate`.
* [Page cells](../../page-cells.md): rewritten as a one-page overview with a record table linking to the four cell-record pages and the helper page; the "combine multiple records in one writer" example stays here.

`SUMMARY.md` updated to nest the five sub-pages under the overview entry.

## 2026-05: Split extension-methods into per-category pages

The combined `extension-methods.md` page covered six independent groups of extensions (LINQ bridges, drawing primitives, master dropping, ShapeSheet I/O, coordinates, one-offs) under one heading. Split it into one page per category in a new `extensions/` subfolder, with the original page kept as the overview that links to all six (the `saveenr.gitbook.io/visioautomation/extension-methods` URL is preserved).

* [LINQ bridges](../../extensions/linq.md): `ToEnumerable()` / `ToList()` over `Pages`, `Shapes`, `Masters`, etc.
* [Drawing primitives](../../extensions/drawing.md): `DrawRectangle(rect)`, `DrawPolyline`, `DrawBezier`, `DrawQuarterArc`, `DrawNurbs`.
* [Master dropping](../../extensions/dropping.md): `Drop(master, point)`, `DropManyU(masters, points)`.
* [Typed ShapeSheet I/O](../../extensions/shapesheet.md): `GetFormulasU` / `GetResults<T>` / `SetFormulas` / `SetResults` keyed by `Src` / `SidSrc`.
* [Coordinates and bounding boxes](../../extensions/coordinates.md): `GetBoundingBox`, `XYFromPage`, `XYToPage`, `ResizeToFitContents`.
* [One-off extensions](../../extensions/misc.md): `Quit`, `Close`, `OpenStencil`, view / window rects, `GetShapesFromIDs`.
* [Extension methods](../../extension-methods.md): rewritten as a one-page overview with a category table linking to the six.

`SUMMARY.md` updated to nest the six sub-pages under the overview entry.

## 2026-05: Split shape-cells into per-record pages

The combined `shape-cells.md` page covered three separate cell records under one heading. Split it so each record has its own page, with the original page kept as a thin overview that links to the three:

* [Shape XForm cells](../../shape-xform-cells.md): `ShapeXFormCells` (size, position, rotation, local pin).
* [Shape format cells](../../shape-format-cells.md): `ShapeFormatCells` (fill, line, fill shadow). Field tables now broken out by sub-section.
* [Shape layout cells](../../shape-layout-cells.md): `ShapeLayoutCells` (auto-layout placement, routing, line-jump).
* [Shape cells](../../shape-cells.md): rewritten as a one-page overview pointing to the three above; the "combine multiple records in one writer" example stays here.

`SUMMARY.md` was updated to nest the three sub-pages under the overview entry.

## 2026-05: Compile-failure pass

A compile pass over every C# code block on the site, prompted by a stale snippet found in the source repo's `readme.md`. Fixes:

* **`shapesheet/modify-the-shapesheet.md`**: replaced `SrcConstants.PinX` / `SrcConstants.PinY` (which don't exist on `SrcConstants`) with `XFormPinX` / `XFormPinY`. Added a one-line note about the cell-record-prefixed naming so the right constant name is easier to predict.
* **`shapesheet/query-the-shapesheet.md`**: the `SectionQuery` examples were calling a 1-arg constructor that doesn't exist (`new SectionQuery(visSectionProp)`) and a `.Columns.Add` property that doesn't exist on `SectionQuery`. Rewrote both single-shape and multi-shape examples to use `new SectionQuery()` followed by `query.Add(sectionIndex)` (which returns a `DataColumns` you then add cell columns to). Also corrected the iteration: `DataRowGroup<T>` enumerates _sections_, not rows, so the `foreach` example needed a nested loop to reach a row.
* **`connection-points.md`**, **`hyperlinks.md`**, **`control-handles.md`**: the `Set` overload takes the row index as `short`, but `Add` returns `int`, so the as-written `Set(shape, row, ...)` calls didn't compile. Each example now casts to `(short)row` and a one-line note explains the type asymmetry.

## 2026-05: Refresh against current API

A pass aligning every page with the **VisioAutomation** library's current public API. The underlying library had drifted (class renames, namespace renames, struct renames) and several pages still described the older shape; the docs now match what's in the source.

### Page-level changes

* **`shapesheet/cells.md`**: rewritten for the modern naming. The struct is `Src` (not `SRC`); the helper class is `VisioAutomation.Core.SrcConstants` (not `VisioAutomation.ShapeSheet.SRCConstants`); constant names dropped underscores (`Char_Color` → `CharColor`, `FillForegnd` → `FillForeground`, etc.). Removed the "Converting between cell names and (s,r,c)" section: it documented a `ShapeSheetHelper.GetSRCFromName` helper that doesn't exist.
* **`shapesheet/query-the-shapesheet.md`**: rewritten with a return-type table that makes it clear which type each query overload returns (`DataRows<T>` / `DataRowGroup<T>` / `DataRowGroups<T>`). Fixed the indexer syntax (`[r][c]`, not `[r,c]`); fixed the multi-shape `SectionQuery` example to use `Core.ShapeIDPairs` instead of `IList<int>`.
* **`shapesheet/modify-the-shapesheet.md`**: rewritten for the current writer API: `SrcWriter` and `SidSrcWriter` (formerly `SRCUpdate` / `SIDSRCUpdate`); writer methods are `SetValue` / `SetFormula` / `Commit(target, CellValueType)` (formerly `SetFormula` / `SetResult` / `Execute`); `shape.CellsU["PinX"]` is now used in place of the case-sensitive `shape.Cells["Pinx"]`.
* **`user-defined-cells.md`**: rewritten using `UserDefinedCellHelper.GetDictionary(shape, CellValueType)`, `Set(shape, name, UserDefinedCellCells)`, and `ShapeIDPairs.FromShapes(...)` for the multi-shape pattern. The helper class was renamed `UserDefinedCellsHelper` → `UserDefinedCellHelper` (singular).
* **`custom-properties.md`**: rewritten with the full `CustomPropertyCells` field set (`Value`, `Prompt`, `Label`, `Format`, `Type`, `Calendar`, `Invisible`, `LangID`, `SortKey`, `Ask`); fixed namespace references (`Shapes.CustomProperties.X` was wrong). Multi-shape examples now use the `ShapeIDPairs` page-level overload.
* **`extension-methods.md`** + **`stencils-and-masters.md`**: replaced dead `VisioAutomation.Drawing.{Point, Size}` references with `VisioAutomation.Core.{Point, Size}`. Fixed broken `=>` lambda formatting and tightened prose.
* **`shapesheet/README.md`**: previously a stub. Now a real section landing introducing the ShapeSheet, the cell-addressing approach, and the query / modify split.
* **`compiling.md`**: updated to Visual Studio 2022 / C# 8.0; .NET Framework 4.5 references updated to **4.5.2** to reflect the recent TFM bump; pointed at the source repo's [BUILDING.md](https://github.com/saveenr/VisioAutomation/blob/master/docs/BUILDING.md) for the up-to-date dev-pack install instructions.

### Pages added

* `documentation-changes.md` (this page).
* [Hyperlinks](../../hyperlinks.md): documents `HyperlinkHelper` and `HyperlinkCells`.
* [Lock cells](../../lock-cells.md): documents the `LockCells` record and the `SrcWriter` pattern for applying / clearing locks.
* [Control handles](../../control-handles.md): documents `ControlHelper` and `ControlCells`.
* [Connection points](../../connection-points.md): documents `ConnectionPointHelper` and `ConnectionPointCells`.
* [Connectors](../../connectors.md): documents `ConnectorHelper`, including manual connection and AutoConnect.
* [Shape format / layout / xform cells](../../shape-cells.md): documents `ShapeFormatCells`, `ShapeLayoutCells`, `ShapeXFormCells` and how to combine them in a single `SrcWriter` transaction.
* [Page cells](../../page-cells.md): documents `PageHelper` plus the four PageSheet records (`PageFormatCells`, `PageLayoutCells`, `PagePrintCells`, `PageRulerAndGridCells`); covers `Duplicate`, `SetSize`/`GetSize`, and `ResizeToFitContents`.
* [Text formatting](../../text-formatting.md): documents `CharacterCells`, `ParagraphCells`, `TextBlockCells`, multi-row writes, plus the `TextHelper` tab-stop helpers (`GetTabStops`/`SetTabStops`).
* [Geometry](../../geometry.md): documents `GeometryHelper`, `GeometrySection`, `GeometryRow` for building custom paths.
* [Application](../../application.md): documents `ApplicationHelper`: version detection, alert-bypass quit, content-folder location, and the foreground-window helper.
* [Undo scope](../../undo-scope.md): documents `UndoScope`, the `IDisposable` wrapper around `BeginUndoScope`/`EndUndoScope`, including the `Commit = false` rollback pattern.
* [Analyzers](../../analyzers.md): documents `ConnectionAnalyzer.GetDirectedEdges`, `ConnectionAnalyzerOptions`, and the `EdgeDirectionSource` / `EdgeNoArrowsHandling` enums.
* [Visio error log](../../logging.md): documents `LoggingHelper.GetXmlErrorLogFilename`, `XmlErrorLog`, and the `LogSession` / `LogRecord` shape.
* [Exception types](../../exceptions.md): documents `AutomationException` and its subclasses `VisioOperationException` and `InternalAssertionException`.

### Pages rewritten

* `extension-methods.md`: replaced the original three-method intro with a comprehensive reference organized by purpose (LINQ bridges over the COM collection types, drawing primitives, master dropping, ShapeSheet I/O, geometry / coordinates, one-offs). Fixed an example that referenced a non-existent `Fonts.AsEnumerable()`: the actual extensions are named `ToEnumerable()` and `ToList()`.

### Pages removed

* `convert-values.md`: documented a `VisioAutomation.Convert` static class that doesn't exist in the library. Removed rather than rewritten.
* `resources/stencils-and-masters.md`: was a duplicate of the top-level `stencils-and-masters.md`. Removed.

### Strict-accuracy fixes

Smaller fixes to call out:

* **`README.md`**: replaced a stale link to a separate `VisioPowerShell` repo (the PowerShell module now lives in the main `VisioAutomation` repo).
* **`SUMMARY.md`**: programmatic verification that every link target exists; minor cleanup along the way.
* **`classes.md`**, **`namespaces.md`**, **`related-projects.md`**: added context paragraphs above the diagrams; expanded the related-projects list to include the Visio PowerShell module and PSVA.

For the in-repo source-of-truth detail (commit hashes per change), see the [project's commit history](https://github.com/saveenr/VisioAutomation/commits/master/).

