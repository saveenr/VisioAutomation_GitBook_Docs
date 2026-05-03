# Compiling

## Requirements to compile & debug

* Windows 10 or above
* Visio 2010 or above
* Visual Studio 2022 (Visual Studio 2026 is not yet supported — its MSBuild does not resolve the .NET Framework 4.5 reference assemblies that the shipping libraries target)

## Notes

VisioAutomation uses C# 8.0 features. For full build / test / install instructions including the `Microsoft.NETFramework.ReferenceAssemblies.net45` NuGet workaround for the v4.5 reference assemblies, see the [BUILDING.md in the source repo](https://github.com/saveenr/VisioAutomation/blob/master/docs/BUILDING.md).
