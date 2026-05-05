# Proposed GitHub Issues for saveenr/VisioAutomation

All issues below are to be filed at <https://github.com/saveenr/VisioAutomation/issues/new>.

---

## Issue 1 — `docs: add VisioScripting.Client documentation`

**Labels:** `documentation`, `priority: critical`

### Problem

The GitBook user guide (<https://saveenr.gitbook.io/visioautomation/>) has no coverage of
`VisioScripting.Client` at all, even though the source `readme.md` leads with a
`VisioScripting.Client` snippet as its "Quick example".

`Client.cs` exposes **27 command groups**: `Application`, `Arrange`, `Model`,
`Connection`, `ConnectionPoint`, `Container`, `Control`, `CustomProperty`, `Developer`,
`Document`, `Draw`, `Export`, `Grouping`, `Hyperlink`, `Layer`, `Lock`, `Master`,
`Output`, `Page`, `Selection`, `ShapeSheet`, `Text`, `Undo`, `UserDefinedCell`, `View`.
A `VisioScripting/README.md` exists in the source repo but is not linked from the GitBook.

### Impact

A new developer following the source `readme.md` will reach `VisioScripting.Client` first
and have no docs to consult. The scripting façade is the recommended first-use API for C#,
yet it is entirely undocumented in the user-facing site.

### Suggested work

- Add a top-level `visio-scripting.md` page (and sub-pages per command group) in
  `saveenr/VisioAutomation_GitBook_Docs`.
- At minimum, add a section to the Introduction / Getting Started explaining the three API
  tiers: raw COM interop → `VisioAutomation` helpers → `VisioScripting.Client`.
- Adapt the content of `VisioAutomation_2010/VisioScripting/README.md` for the GitBook.
- Update `SUMMARY.md` with new entries.

### References

- `VisioAutomation_2010/VisioScripting/Client.cs`
- `VisioAutomation_2010/VisioScripting/Commands/` (27 command-set files)
- GitBook docs repo: `saveenr/VisioAutomation_GitBook_Docs`

---

## Issue 2 — `docs: add VisioAutomation.Models documentation (DOM + layout algorithms)`

**Labels:** `documentation`, `priority: critical`

### Problem

`VisioAutomation.Models` is entirely undocumented in the GitBook user guide.
It contains two major sub-systems:

**DOM** (`VisioAutomation_2010/VisioAutomation.Models/DOM/`): a declarative
document-object model — `Document`, `Page`, `PageList`, `Shape`, `ShapeList`,
`Connector`, `BezierCurve`, `Line`, `Oval`, `PolyLine`, `Rectangle`, `MasterRef`,
`Hyperlink`, `ShapeCells`, `PageCells`, etc. — that lets callers build entire Visio
diagrams declaratively and render them in a single batch.

**Layouts** (`VisioAutomation_2010/VisioAutomation.Models/Layouts/`): automated layout
algorithms — `DirectedGraph`, `Tree`, `Grid`, `Box`, `Container` — for data-driven diagram
generation.

### Impact

These features distinguish the library from raw COM interop and are likely the primary
reason many users adopt it. A user wanting to render a tree diagram or directed graph has
no documentation at all.

### Suggested work

- Add a `models/` section in `saveenr/VisioAutomation_GitBook_Docs` documenting the DOM
  API with a getting-started example (build a small diagram declaratively, render it).
- Add individual layout pages: DirectedGraph, Tree, Grid, Box/Container.
- Cross-link from the existing `connectors.md` and `analyzers.md` pages to the
  DirectedGraph layout page (complementary: `ConnectionAnalyzer` reads a graph;
  `DirectedGraph` renders one).
- Update `SUMMARY.md`.

### References

- `VisioAutomation_2010/VisioAutomation.Models/DOM/`
- `VisioAutomation_2010/VisioAutomation.Models/Layouts/`
- GitBook docs repo: `saveenr/VisioAutomation_GitBook_Docs`

---

## Issue 3 — `docs: add API-selection guide for new users`

**Labels:** `documentation`, `priority: critical`

### Problem

`readme/getting-started.md` in the GitBook docs opens with raw Visio COM interop and
pivots to `VisioAutomation.Extensions`, but never explains:

- When to use raw `VisioAutomation` helpers vs. `VisioScripting.Client` vs. the
  `VisioAutomation.Models` DOM.
- What `VisioScripting.Client` is.
- That a PowerShell module exists.

### Suggested work

Add a short decision table near the top of `getting-started.md` or as a standalone
`readme/choosing-an-api.md` page:

| Use case | Entry point |
|---|---|
| Scripting from PowerShell | `Visio` module (`Import-Module Visio`) |
| Scripting from C# at a high level | `VisioScripting.Client` |
| Generating diagrams from data | `VisioAutomation.Models` DOM + Layouts |
| Low-level ShapeSheet control | `VisioAutomation` helpers directly |

### References

- `saveenr/VisioAutomation_GitBook_Docs` — `readme/getting-started.md`

---

## Issue 4 — `docs: fix single-line code example and add language tags in getting-started.md`

**Labels:** `documentation`, `priority: high`

### Problem

In `readme/getting-started.md`, lines 43–44 of the "Find the active page and draw a
rectangle" example concatenate two statements on one line:

```text
var doc = app.Documents.Add("");var page = app.ActivePage; // ...
```

This is the most prominent code sample on the primary onboarding page. The code fence
also lacks a `csharp` language tag, while all other pages in the 2026 overhaul use
` ```csharp ` consistently.

### Suggested fix

Split into two lines and add the `csharp` language tag:

```csharp
var doc  = app.Documents.Add("");
var page = app.ActivePage; // Creating a new doc auto-creates a new empty page
```

### References

- `saveenr/VisioAutomation_GitBook_Docs` — `readme/getting-started.md` lines 43–44

---

## Issue 5 — `docs: complete the getting-started tutorial end-to-end and add "Next steps"`

**Labels:** `documentation`, `priority: high`

### Problem

`readme/getting-started.md` ends at "Dropping multiple shapes" with no "what you built"
summary, no saved-file step, and no next-steps links. There is a "Checkpoint 1" section
that says "you haven't used VisioAutomation at all yet" — but the page ends before
completing the journey.

### Suggested work

1. Complete the tutorial so the reader ends with a saved `.vsdx` file (add a
   `doc.SaveAs(path)` step).
2. Add a "Next steps" section at the end linking to: ShapeSheet, Custom properties,
   Connectors, and the VisioScripting façade.
3. Fix the typo on line 65: "will make your day-to-day **like** easier" → "**life**
   easier".

### References

- `saveenr/VisioAutomation_GitBook_Docs` — `readme/getting-started.md`

---

## Issue 6 — `docs: cross-link stencil examples to ApplicationHelper.GetContentLocation`

**Labels:** `documentation`, `priority: high`

### Problem

`readme/getting-started.md` (lines 95–97) and `stencils-and-masters.md` both reference
the stencil filename `"basic_u.vss"`, then acknowledge it may not exist in modern Visio
(2016+):

> "over the years the stencils have changed; they may have different filenames"

But neither page directs the reader to a solution.

`application.md` already documents `ApplicationHelper.GetContentLocation(app)`, which
resolves the Visio Content folder (where stencils live).

### Suggested fix

Add a note in `getting-started.md` and `stencils-and-masters.md` explaining how to find
the correct stencil path:

```csharp
string contentDir = VisioAutomation.Application.ApplicationHelper.GetContentLocation(app);
// then enumerate or construct the stencil path relative to contentDir
```

Cross-link to `application.md`.

### References

- `saveenr/VisioAutomation_GitBook_Docs` — `readme/getting-started.md` lines 88, 95–97
- `saveenr/VisioAutomation_GitBook_Docs` — `stencils-and-masters.md` lines 14, 27
- `saveenr/VisioAutomation_GitBook_Docs` — `application.md` lines 30–37

---

## Issue 7 — `docs: update resources/README.md with current links`

**Labels:** `documentation`, `priority: high`

### Problem

`resources/README.md` in the GitBook docs currently lists only:

- visguy.com forum (low activity)
- StackOverflow `[visio]` tag
- *Visio 2003 Developer's Survival Pack* (ISBN 1412011124, a 23-year-old book)

It does **not** include:

- The Microsoft Visio developer documentation on learn.microsoft.com
  (<https://learn.microsoft.com/en-us/office/client-developer/visio/visio-home>)
- The official Visio ShapeSheet function reference
- The source repo's `docs/GLOSSARY.md` and `docs/ARCHITECTURE.md`, both of which are
  listed in the source `readme.md` as recommended reading but are not linked from the
  GitBook `resources` page.

### Suggested fix

Rewrite `resources/README.md` to:

1. Add the Microsoft Learn Visio developer docs link.
2. Link to `docs/GLOSSARY.md` and `docs/ARCHITECTURE.md` in the source repo.
3. Move the 2003 book to a footnote or remove it.

### References

- `saveenr/VisioAutomation_GitBook_Docs` — `resources/README.md`
- `saveenr/VisioAutomation` — `docs/ARCHITECTURE.md`, `docs/GLOSSARY.md`

---

## Issue 8 — `docs: add troubleshooting page`

**Labels:** `documentation`, `priority: high`

### Problem

There is no troubleshooting or FAQ section in the GitBook docs. Common failure modes have
no guidance:

- Visio not installed / COM registration failures
- NuGet package version mismatches (PIA version vs. `VisioAutomation2010` package version)
- Stencil filename differences between Visio versions (already noted in
  `getting-started.md` but not resolved)
- Running under 64-bit vs. 32-bit PowerShell with the `Visio` module
- "Failed to log in to github.com" errors when using the PowerShell module

### Suggested work

Add a `troubleshooting.md` page linked from both `getting-started.md` and `compiling.md`.
Include at minimum:

- How to verify Visio is registered for COM automation
- Correct PIA/VisioAutomation2010 version pairing
- Finding the correct stencil path for the installed Visio version
- 32-bit vs. 64-bit process note for the PowerShell module

### References

- `saveenr/VisioAutomation_GitBook_Docs` — `readme/getting-started.md`, `compiling.md`

---

## Issue 9 — `docs: add version compatibility table (Visio 2010–M365)`

**Labels:** `documentation`, `priority: medium`

### Problem

The docs mention `VisioAutomation2010` as "maintained" but give no table of which Visio
versions (2010, 2013, 2016, 2019, 2021, M365) are supported and how behavior might differ.
`application.md` hints at version-gating (throws for Visio < 14, handles 14 and 15+
differently), but there is no centralized compatibility reference.

### Suggested work

Add a version-compatibility table to `README.md` (or `getting-started.md`) such as:

| Visio version | Internal version | Supported? | Notes |
|---|---|---|---|
| Visio 2010 | 14 | ✅ | Minimum supported |
| Visio 2013 | 15 | ✅ | |
| Visio 2016/2019/2021 | 16 | ✅ | |
| Visio for M365 | 16+ | ✅ | |

Cross-reference `application.md` where version-dependent behavior is documented.

### References

- `saveenr/VisioAutomation_GitBook_Docs` — `README.md`, `application.md` lines 31–37

---

## Issue 10 — `docs: add alt text to namespace and class diagram images`

**Labels:** `documentation`, `priority: medium`

### Problem

Two pages render architecture diagrams with no alt text:

- `namespaces.md` line 5: `![](.gitbook/assets/image.png)` — blank alt
- `classes.md` line 5: `![](.gitbook/assets/NamespacesAndClasses.svg)` — blank alt

If the images fail to load (or for screen-reader users), there is zero fallback content.
The pages already note that the diagrams "may lag behind the latest code," which makes a
textual description even more important.

### Suggested fix

1. Add descriptive alt text, e.g.:
   `![Namespace hierarchy: Core, ShapeSheet, Shapes, Pages, Documents, Extensions]`
2. Optionally add a short textual namespace table below each image as a fallback.

### References

- `saveenr/VisioAutomation_GitBook_Docs` — `namespaces.md` line 5
- `saveenr/VisioAutomation_GitBook_Docs` — `classes.md` line 5

---

## Issue 11 — `docs: add structural section groupings to SUMMARY.md`

**Labels:** `documentation`, `priority: medium`

### Problem

`SUMMARY.md` moves directly from "Getting started" into dozens of reference topics
(Stencils, ShapeSheet, custom properties, text formatting, …) without any grouping
headers. New users and experienced users cannot quickly orient themselves.

### Suggested work

Add GitBook-style section dividers to `SUMMARY.md`. Suggested groupings:

- **Getting started** — Introduction, Related projects, Getting started, Choosing an API
- **Core APIs** — Stencils & masters, Extension methods, ShapeSheet, Application, Undo scope
- **Shape data** — Custom properties, User-defined cells, Hyperlinks, Lock cells, Control handles, Connection points, Connectors
- **Formatting** — Shape cells, Text formatting, Geometry
- **Pages** — Page cells
- **Scripting façade** — VisioScripting (new)
- **Models & layouts** — VisioAutomation.Models (new)
- **Reference** — Namespaces, Classes, Compiling, Resources, Analyzers, Logging, Exceptions

### References

- `saveenr/VisioAutomation_GitBook_Docs` — `SUMMARY.md`

---

## Issue 12 — `docs: remove duplicate H2 heading in stencils-and-masters.md`

**Labels:** `documentation`, `priority: medium`

### Problem

`stencils-and-masters.md` has the page title as H1 on line 1 and then immediately repeats
it as H2 on line 3:

```markdown
# Stencils and masters

## Stencils and masters
```

This creates a redundant heading that looks odd in the rendered GitBook nav and breaks the
heading hierarchy.

### Suggested fix

Remove the H2 repetition (lines 3–4) and replace it with a brief introductory sentence
before the first code example.

### References

- `saveenr/VisioAutomation_GitBook_Docs` — `stencils-and-masters.md` lines 1–5

---

## Issue 13 — `docs: fix typo "day-to-day like easier" in getting-started.md`

**Labels:** `documentation`, `priority: medium`

### Problem

`readme/getting-started.md` line 65 reads:

> "will make your day-to-day **like** easier so I do recommended it"

Two errors on the same line: "like" should be "life", and "I do recommended" should be
"I do recommend".

### Suggested fix

> "will make your day-to-day **life** easier, so I do recommend it."

### References

- `saveenr/VisioAutomation_GitBook_Docs` — `readme/getting-started.md` line 65

---

## Issue 14 — `docs: update or remove the Visual Studio 2026 compatibility note in compiling.md`

**Labels:** `documentation`, `priority: low`

### Problem

`compiling.md` line 7 states:

> "Visual Studio 2026 is not yet supported — its MSBuild does not resolve the
> .NET Framework 4.5.2 reference assemblies that the shipping libraries target"

This note was added in May 2026. It should either:

1. Link to a tracking issue so readers can follow resolution progress, or
2. Be removed once VS2026 support is confirmed.

### Suggested work

- If a tracking issue exists for the VS2026 / TFM 4.5.2 problem, link to it.
- Add a "last verified" date so maintainers know when to re-evaluate.

### References

- `saveenr/VisioAutomation_GitBook_Docs` — `compiling.md` line 7

---

## Issue 15 — `docs: add deprecation version info for VisioAutomation2007 NuGet reference`

**Labels:** `documentation`, `priority: low`

### Problem

The GitBook `README.md` (lines 13–14) lists:

> **Visio Automation 2007** NuGet package (no longer maintained)

But it gives no information about:

- When it was deprecated (which library version?)
- Whether existing users should migrate, and if so, to what
- Whether the package still receives security fixes

### Suggested fix

Add a note such as:

> **VisioAutomation2007** is no longer maintained as of version X.Y. Users of Visio 2010
> or later should use **VisioAutomation2010** instead.

If migration is required, link to or create a brief migration guide.

### References

- `saveenr/VisioAutomation_GitBook_Docs` — `README.md` lines 13–14
