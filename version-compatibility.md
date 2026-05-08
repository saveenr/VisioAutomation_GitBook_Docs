# Version compatibility

Per-release reference table for the [`VisioAutomation2010`](https://www.nuget.org/packages/VisioAutomation2010/) NuGet package, mapping each version to the .NET target framework, Visio baseline, and supporting runtime details. Use it to pick the right package version for a given environment, or to confirm that an older package will install into a project on a constrained runtime.

For the matching PowerShell module's per-version matrix see the [VisioPowerShell version compatibility](https://saveenr.gitbook.io/visiopowershell/developer-info/version-compatibility) page.

## Modern era (changelog-tracked)

These versions are documented in [`NuGet/CHANGELOG.md`](https://github.com/saveenr/VisioAutomation/blob/master/NuGet/CHANGELOG.md). Every row's data comes from the corresponding git tag's `csproj` and `nuspec`.

| Version | Released | Library TFM | C# language | Visio PIA baseline | PIA bundled in `.nupkg` | Release notes |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `3.0.0` | 2026-05-06 | `net452` | C# (latest, SDK-style csproj) | Visio 2010 PIA (v14) | Yes, under `lib/net452/` | [3.0.0 changelog](https://github.com/saveenr/VisioAutomation/blob/master/NuGet/CHANGELOG.md#300---2026-05-07) |
| `2.6.0` (tag only) | 2026-05-04 | `net452` | C# 8.0 | Visio 2010 PIA (v14) | Yes, under `lib/net452/` | Tag in repo, never published end-to-end to nuget.org. Superseded by 3.0.0 two days later. |

`3.0.0` was the first release published to nuget.org via the `publish-nuget.yml` CI flow under the `SevenPens` publishing identity. It is the recommended starting point for new consumers.

## Pre-changelog era (2017)

These versions predate the [`NuGet/CHANGELOG.md`](https://github.com/saveenr/VisioAutomation/blob/master/NuGet/CHANGELOG.md) and the SDK-style csproj migration. The data is reconstructed from the git tags' `csproj` files and `.nuspec`. Some of these tags may not have been published to nuget.org as standalone packages, but the binaries they would have produced share the runtime profile shown.

| Version range | Released | Library TFM (built) | TFM (claimed in nuspec) | C# language | Visio PIA baseline |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `1.19.0` to `2.3.0` | 2017-02-19 to 2017-04-24 | `net40` | `lib/net40` | C# default for VS 2017 era | Visio 2010 PIA (v14) |

For the exact set of tags in this range and their dates see the repo's [release tags](https://github.com/saveenr/VisioAutomation/releases) and [git history](https://github.com/saveenr/VisioAutomation/commits/master/).

## Constants across all versions

Some properties have not changed across the package's lifetime; they apply to every row above.

* **Visio version baseline** is **Visio 2010** (PIA major version 14). The library's typed surface targets the 2010 PIA so it loads cleanly on any Visio install from 2010 onward; APIs added in later Visio versions are reachable through the underlying `IVisio` COM objects but are not part of the typed wrapper.
* **Bitness:** the package is AnyCPU. The consuming process must match Visio's bitness at runtime (the in-process COM bridge cannot cross a 32-bit / 64-bit boundary).
* **License:** MIT, both in source and in package metadata. The metadata representation switched from the deprecated `<licenseUrl>` to the SPDX `<license type="expression">MIT</license>` form in `3.0.0`; the license itself is unchanged.

## Future TFM moves

The shipping libraries' minimum target framework will rise from `net452` to `net472` once Windows 10 LTSB 2016 leaves Extended Support on **2026-10-13**. That move is tracked in [`docs/futures/build-and-code.md`](https://github.com/saveenr/VisioAutomation/blob/master/docs/futures/build-and-code.md) and will land as part of the Phase 3 modernization (currently scoped against Visual Studio 2026's MSBuild, which has a `net462` floor).

A subsequent move to multi-targeting (`net472` plus a modern `net10.0-windows`) is on the roadmap under Milestone C but has no fixed date.

## See also

* [VisioPowerShell version compatibility](https://saveenr.gitbook.io/visiopowershell/developer-info/version-compatibility) (the matching matrix for the PowerShell module)
* [`NuGet/CHANGELOG.md`](https://github.com/saveenr/VisioAutomation/blob/master/NuGet/CHANGELOG.md) (full per-version release notes)
* [`VisioAutomation2010` on nuget.org](https://www.nuget.org/packages/VisioAutomation2010/) (current published versions)
