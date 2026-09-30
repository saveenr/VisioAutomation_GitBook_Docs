# Compiling

## Requirements to compile & debug

* Windows 10 or above
* Visio 2010 or above to run automation and integration tests; compilation does not require Visio.
* Visual Studio 2026 (or matching Build Tools) with the .NET desktop build workload.
* The .NET 10 SDK, selected by the source repository's `global.json`. This is the build SDK, not the runtime target.

## Notes

The .NET Framework 4.5.2 and 4.7.2 reference assemblies are supplied by NuGet packages and restored automatically during build, as is the Visio interop assembly. No Developer Pack install is required. Projects are SDK-style and package versions are centralized in `Directory.Packages.props`.

C# 14 is selected explicitly by `LangVersion` in `Directory.Build.props`. Open `VisioAutomation_2010/VisioAutomation2010.slnx`; use Debug for development and Release for shipping packages. VisioAutomation and VisioAutomation.VDX share this toolchain while retaining their .NET Framework targets. For exact PowerShell build commands and all four test assemblies, see [BUILDING.md in the source repo](https://github.com/saveenr/VisioAutomation/blob/master/docs/BUILDING.md).
