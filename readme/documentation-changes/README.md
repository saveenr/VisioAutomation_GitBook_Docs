# Documentation updates

This page summarizes notable changes to the **VisioAutomation** documentation so returning readers can find what's new without re-reading every page.

[2026-05 doc updates](2026-05-doc-updates.md) - A major doc overhaul after years of inactivity

## 2026-05: Version compatibility reference

Added a new [Version compatibility](../../version-compatibility.md) page (under the **Reference** section) that maps each released `VisioAutomation2010` NuGet version to its target framework, C# language level, Visio PIA baseline, and bundled-PIA layout. The modern era (`2.6.0` and `3.0.0`) is sourced from the in-repo csproj and nuspec at each tag; the 2017-era `1.x` and `2.x` releases are summarized as a single best-effort row. Cross-links to the matching [PowerShell module compatibility table](https://saveenr.gitbook.io/visiopowershell/developer-info/version-compatibility). Closes [issue #161](https://github.com/saveenr/VisioAutomation/issues/161).

## 2026-05: VisioAutomation.Models documentation (Tier 3)

Closed the last remaining tier of the .NET-side doc audit by adding six pages under a new `models/` subsection of "Diagram models", covering everything in the [`VisioAutomation.Models`](https://github.com/saveenr/VisioAutomation/tree/master/VisioAutomation_2010/VisioAutomation.Models) project. The new pages are [Declarative DOM](../../models/dom.md), [Layouts: Tree, Grid, Box](../../models/layouts.md), [Directed graph](../../models/directed-graph.md) (paired with the existing XML-format page), [Layout styles](../../models/layout-styles.md), [Org charts](../../models/org-charts.md), and [Form pages](../../models/forms.md). Each page leads with a working code snippet adapted from the in-repo `VTest.Models/` test suite, then walks through the public API surface, with cross-links to the underlying shape and layout primitives where relevant. Closes [issue #132](https://github.com/saveenr/VisioAutomation/issues/132).

