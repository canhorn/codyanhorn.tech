---
layout: post
author: Cody Merritt Anhorn
title: Blazor - Library Interop Generator 1.0 Release 
categories: [blog, blazor]
tags: [.NET 5, C#, Blazor, WASM, BabylonJS]
---

I am finally comfortable enough to put the 1.0 Version on this project! The project it self is quite simple to grasp, you give it a TypeScript definition file and it will create a C# interop library for easier usage with ASP.NET Core Blazor. 

## Project Details

With the release of .NET 5, and adding Task/Promise support to the API generation, I am happy with the MVP and wanted to release the project as 1.0! The main reason for the wait was for the .NET 5 release, since it was still in preview and the release allows for such better performance than what I was getting from the 3.1 releases.

You can checkout the prior article for more details on this project and reason behind it. But for this article I will go over tool usage and the new feature of awaiting Tasks for a promise based API from the Library API.

## Project Details

First here is a list of the REpository, some packages in NuGet if you want to see what the tools builds on top of. An example site showing how it can be used to generate the Library API for BabylonJS, and the location of the Tool package.

**Repository:** <a href="https://github.com/canhorn/EventHorizon.Blazor.TypeScript.Interop.Generator" target="_blank">canhorn/EventHorizon.Blazor.TypeScript.Interop.Generator</a>

**NuGet Package:** <a href="https://www.nuget.org/packages/EventHorizon.Blazor.TypeScript.Interop.Generator" target="_blank">EventHorizon.Blazor.TypeScript.Interop.Generator</a>

**Sample Site:** <a href="https://wonderful-pond-05f7b3b10.azurestaticapps.net/" target="_blank">BabylonJS Generated Example Site</a>

**The Tool:** <a href="https://www.nuget.org/packages/EventHorizon.Blazor.TypeScript.Interop.Tool" target="_blank">.NET Core Tool Package</a> - Can be used to easily generate the project from the CLI.

## Tool Usage

Below I have a few commands, to install the tool and to generate a project based on files or ULR based TypeScript Definition files.

<br />
Installing the Tool, it is a .NET tool, to make it easy to run from the command line.
~~~ bash
## Install the Tool
dotnet tool install --global EventHorizon.Blazor.TypeScript.Interop.Tool --version 1.0.1
~~~

<br />
After the above command you will be able to use the CLI command, <code>ehz-generate</code>, to run the generation from any location. Below is the currently supported table of the identifiers that are used in the generation of the library. 

One thing to point out is that the tool will only generate a small part of the library, you can give it a list of roo Classes to generate from. The tool is smart enough to find explicit usage to other classes and it will generate those as well, so you don't have to find out the cross referenced classes you self.

Identifier | Details | Required/Default
--- | --- | ---
-s, --source &lt;source&gt; | TypeScript Definition to generate from, can be a File or URL, accepts multiple or as Array |  REQUIRED
-c, --class-to-generate &lt;class-to-generate&gt; | A Class to generate, accepts multiple or as Array |  REQUIRED
-a, --project-assembly &lt;project-assembly&gt; | The project name of the Assembly that will be generated | Default: "Generated.WASM"
-l, --project-generation-location &lt;project-generation-location&gt; | The directory where the Generated Project assembly will be saved | Default: "_generated"
-f, --force | This will force generation, by deleting --project-generation-location | Default: (False)

### Generating Projects

The below command shows, using the BabylonJS TypeScript Definition file, off how to create a proxy to the Vector3 class provided by BabylonJS. 

~~~ bash
ehz-generate --project-assembly Blazor.BabylonJS --class-to-generate Vector3 --source https://raw.githubusercontent.com/BabylonJS/Babylon.js/master/dist/babylon.d.ts
~~~

<br />
For the next section we will first create a custom TypeScript Definition file and then run that through the CLI command.

**The example.d.ts TypeScript Definition File:**
~~~ typescript
export module Blazor.Example {
    export class CustomApi {
        static CallMyAip(): Promise<string>;
    }
}
~~~

<br />
**Running this command you will generate your Blazor.Example project.**
~~~ bash 
ehz-generate --project-assembly Blazor.Example --class-to-generate CustomApi --source example.d.ts
~~~

<br />
**Example output after the command is ran:**
~~~ bash
> ehz-generate --project-assembly Blazor.Example --class-to-generate CustomApi --source example.d.ts
Info : projectAssembly: Blazor.Example
Info : projectGenerationLocation: _generated
Info : classToGenerate.Length: 1
Info : classToGenerateItem: CustomApi
Info : sourceFile.Length: 1
Info : sourceFile: example.d.ts
Info : === Consolidate Source Files
Info : === Consolidated Source Files | ElapsedTime: 2ms
Info : === Generated AST
Info : === Generated AST | ElapsedTime: 44ms
Info : === Generate Cached Entity Object
Info : === Generated Cached Entity Object | ElapsedTime: 1ms
Info : === Generate Class Statements
Info : Took 14ms to generate CustomApi
Info : === Generated Class Statements | ElapsedTime: 21ms
Info : === Generate Statements
Info : Generating Method: public static () =>  Task< string> CallMyAip();
Info : Generating Base Constructor: constructor(){  }
Info : === Generated Statements | ElapsedTime: 17ms
Info : === Generating Shimmed Classes
Info : === Generated Shimmed Classes | ElapsedTime: 0ms
Info : === Writing Generated Statements
Info : === Finished Writing Generated Statements | ElapsedTime: 19ms
Success : === Finished Run | ElapsedTime: 109ms
Success : Took 150ms to Generate Source Project.
~~~

<br />

**Here is the example folder tree of the generated project files:**

~~~ bash
│   example.d.ts
├───_generated
│   └───Blazor.Example
│       │   Blazor.Example.csproj
│       ├───Example
│       │       CustomApi.cs <- Our Generated File
│       └───Provided
│               CachedEntityObject.cs
└───_sourceFiles
        example.d.ts
~~~

<br />
Below you can see the class generated for our JavaScript library, you can see it does generate a lot of boilerplate, and I also omit a bit for brevity. But what you get is that it will allow for you to drop in already existing code from TypeScript/JavaScript into C#. 

**Generated CustomApi class file:**
~~~ csharp
/// Generated - Do Not Edit
namespace Blazor.Example
{
    // ... Removed Content for Brevity
    [JsonConverter(typeof(CachedEntityConverter<CustomApi>))]
    public class CustomApi : CachedEntityObject
    {
        public static ValueTask<string> CallMyAip()
        {
            return EventHorizonBlazorInterop.Task<string>(
                new object[]
                {
                    new string[] { "Blazor", "Example", "CustomApi", "CallMyAip" }
                }
            );
        }
        // ... Removed Content for Brevity
    }
}
~~~

The best example I have for drop in, with minimal changes, are the Pirate Fort example from the BabylonJS Playground. Checkout the <a href="https://github.com/canhorn/EventHorizon.Blazor.TypeScript.Interop.Generator/blob/main/Sample/EventHorizon.Blazor.BabylonJS/Pages/PirateFort.razor.cs" target="_blank">PirateFort.razor.cs</a> file that uses the generated BabylonJS Library to interface with BabylonJS from C# using Blazor Wasm. This file takes the source from the <a href="https://www.babylonjs-playground.com/#L80HVL" target="_blank">Pirate Fort Playground</a>, which is TypeScript/JavaScript, and makes a few small changes to make it function in C#.

You can see a working site at <a href="https://wonderful-pond-05f7b3b10.azurestaticapps.net/pirate-fort" target="_blank">Pirate Fort</a>, it includes click action callbacks, animations, pixel system running, and sounds, all triggered from C#!
