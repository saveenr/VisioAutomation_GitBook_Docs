# Compiling

## Requirements to compile & debug

* Windows 10 or above
* Visio 2010 or above
* Visual Studio 2022. Visual Studio 2026 is not yet supported: its MSBuild does not resolve the .NET Framework 4.5.2 reference assemblies that the shipping libraries target. Tracked in [issue #171](https://github.com/saveenr/VisioAutomation/issues/171). Last verified 2026-05-08.

## Notes

The .NET Framework 4.5.2 reference assemblies are supplied by the [`Microsoft.NETFramework.ReferenceAssemblies.net452`](https://www.nuget.org/packages/Microsoft.NETFramework.ReferenceAssemblies.net452) NuGet package and restored automatically during build. No Developer Pack install is required.

VisioAutomation uses C# 8.0 features. For full build / test / install instructions, see the [BUILDING.md in the source repo](https://github.com/saveenr/VisioAutomation/blob/master/docs/BUILDING.md).
