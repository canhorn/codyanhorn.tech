---
layout: post
author: Cody Merritt Anhorn
title: Blazor Interop Generation Tool Release
categories: [blog]
tags: [.NET Core, Blazor, WASM]
---

This article is the Release blog of my new Blazor Interop Generation Tool! Here I will do a quick overview of the tool and how to use it.

## Overview

This tool allows for the easy creation of Blazor TypeScript Interop Generated projects, I packaged up my own library into an convince tool with ease of use features built in. By giving the CLI command a few flags you can generate a project with just a single command, no external files necessary.

## Install

Below you can install the, it was built in .NET Core 5.0, so having the SDK installed is required.

~~~ bash
dotnet tool install -g EventHorizon.Blazor.TypeScript.Interop.Tool
~~~

## Command Line Options

I pulled the below table from the <a href="https://github.com/canhorn/EventHorizon.Blazor.TypeScript.Interop.Generator/tree/main/Tool/EventHorizon.Blazor.TypeScript.Interop.Tool">Tool's Readme in GitHub</a>, check out the readme to see current supported options.

Identifier | Details | Required/Default
--- | --- | ---
-s, --source &lt;source&gt; | TypeScript Definition to generate from, can be a File or URL, accepts multiple or as Array |  REQUIRED
-c, --class-to-generate &lt;class-to-generate&gt; | A Class to generate, accepts multiple or as Array |  REQUIRED
-a, --project-assembly &lt;project-assembly&gt; | The project name of the Assembly that will be generated | Default: "Generated.WASM"
-l, --project-generation-location &lt;project-generation-location&gt; | The directory where the Generated Project assembly will be saved | Default: "_generated"
-f, --force | This will force generation, by deleting --project-generation-location | Default: (False)

## Usage 

The example below shows off a command that will generate a BabylonJS project, it takes in multiple sources from the internet and multiple classes to generate.

~~~ bash
# ehz-generate -f -a Blazor.BabylonJS.WASM.Generated
# -s https://raw.githubusercontent.com/BabylonJS/Babylon.js/master/dist/babylon.d.ts
# -s https://raw.githubusercontent.com/BabylonJS/Babylon.js/master/dist/gui/babylon.gui.d.ts
# -c Button -c MeshBuilder -c PointLight
# -c StandardMaterial -c HemisphericLight
# -c UniversalCamera -c Grid -c StackPanel
ehz-generate -f -a Blazor.BabylonJS.WASM.Generated -s https://raw.githubusercontent.com/BabylonJS/Babylon.js/master/dist/babylon.d.ts -s https://raw.githubusercontent.com/BabylonJS/Babylon.js/master/dist/gui/babylon.gui.d.ts -c Button -c MeshBuilder -c PointLight -c StandardMaterial -c HemisphericLight -c UniversalCamera -c Grid -c StackPanel
~~~