# Compiling

## Requirements to compile & debug

* Windows 10 or above
* Visio 2010 or above to run automation and integration tests; compilation does not require Visio.
* Visual Studio 2022. Visual Studio 2026 is not yet supported: its MSBuild does not resolve the .NET Framework 4.5.2 reference assemblies that the shipping libraries target. Tracked in [issue #171](https://github.com/saveenr/VisioAutomation/issues/171). Last verified 2026-05-08.

## Notes

The .NET Framework 4.5.2 and 4.7.2 reference assemblies are supplied by NuGet packages and restored automatically during build, as is the Visio interop assembly. No Developer Pack install is required. Projects are SDK-style and package versions are centralized in `Directory.Packages.props`.

C# language selection is controlled by `LangVersion` in `Directory.Build.props`. Use Debug for development and Release for shipping packages. For exact PowerShell build commands and all four test assemblies, see [BUILDING.md in the source repo](https://github.com/saveenr/VisioAutomation/blob/master/docs/BUILDING.md).
