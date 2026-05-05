# Compiling

## Requirements to compile & debug

* Windows 10 or above
* Visio 2010 or above
* Visual Studio 2022 (Visual Studio 2026 is not yet supported: its MSBuild does not resolve the .NET Framework 4.5.2 reference assemblies that the shipping libraries target)
* The .NET Framework **4.5.2 Developer Pack** (the reference assemblies are not bundled with modern Windows install media). Easiest install: `choco install netfx-4.5.2-devpack -y`.

## Notes

VisioAutomation uses C# 8.0 features. For full build / test / install instructions, see the [BUILDING.md in the source repo](https://github.com/saveenr/VisioAutomation/blob/master/docs/BUILDING.md).
