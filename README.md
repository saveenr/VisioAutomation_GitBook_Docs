# Introduction

## Overview

**VisioAutomation** is a .NET library that simplifies automating **Microsoft Visio**.

**VisioAutomation** helps you:

* Write Visio Add-ins
* Control the Visio application from other applications

## NuGet packages

| Package | Status | Use this if... |
|---|---|---|
| [**VisioAutomation2010**](http://www.nuget.org/packages/VisioAutomation2010/) | Maintained | You target Visio 2010 or any later version (Visio 2013, 2016, 2019, 2021, Microsoft 365). This is the right choice for almost everyone. |
| [**VisioAutomation2007**](http://www.nuget.org/packages/VisioAutomation2007/) | Unmaintained (legacy) | You are stuck on Visio 2007. The package no longer receives updates; existing users on Visio 2010+ should migrate to **VisioAutomation2010**. |

## Visio version compatibility

The shipping `VisioAutomation2010` package binds to the **Visio 2010 PIA** (Primary Interop Assembly, internal version 14). Newer Visio versions remain backward-compatible with the v14 type library, so a single binary works against every supported Visio:

| Visio version           | Internal version | Supported | Notes                                                        |
|-------------------------|------------------|-----------|--------------------------------------------------------------|
| Visio 2010              | 14               | Yes       | Minimum supported version. Sets the API floor.               |
| Visio 2013              | 15               | Yes       |                                                              |
| Visio 2016              | 16               | Yes       |                                                              |
| Visio 2019              | 16               | Yes       |                                                              |
| Visio 2021              | 16               | Yes       |                                                              |
| Visio for Microsoft 365 | 16+              | Yes       |                                                              |

A handful of helpers gate behavior on the running Visio's reported version (e.g. [`ApplicationHelper.GetContentLocation`](application.md) handles 2010 and 2013+ slightly differently). Where that matters, the relevant page calls it out.

## Source code

[https://github.com/saveenr/VisioAutomation/](https://github.com/saveenr/VisioAutomation/)
