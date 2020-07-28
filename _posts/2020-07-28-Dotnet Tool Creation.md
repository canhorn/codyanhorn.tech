---
layout: post
author: Cody Merritt Anhorn
title: Dotnet Tool Creation
categories: [blog]
tags: [.NET Core, Tool]
---

As part of my development I wanted a way to easily use my <a href="https://github.com/canhorn/EventHorizon.Blazor.TypeScript.Interop.Generator" target="_blank">EventHorizon.Blazor.TypeScript.Interop.Generator</a> package, to do this I created a .NET Core Command Line Interface Tool. In this article I will go over how you too can create a simple tool of your own. 

Checkout the <a href="#how-to">How To</a> section for the code.

**Example Tool Project:** <a href="https://github.com/canhorn/EventHorizon.Blazor.TypeScript.Interop.Generator/tree/main/Tool/EventHorizon.Blazor.TypeScript.Interop.Tool" target="_blank">EventHorizon.Blazor.TypeScript.Interop.Tool</a>

## What is a Tool

A tool is a Command Line Interface keyword that runs some functionality. In my case I wanted a tool that would make it easy to create my Generated code for interfacing with the JavaScript layer from C# in the context of a Blazor WASM application. To do this I used a simple console application, but using the .NET Core tools functionality I was able to create an console package that would run when called by a command of my choosing.

## Usage

The usage for a Tool is quite limitless, what ever you can put into a .NET Core console application, you can turn into a tool. My example will be a very simple echo command, but your imagination is the limit here.

## How to

To start you will need two thinks, .NET Core SDK installed and to create a console application.

~~~ bash
# Create a dotnet console app
dotnet new console -o MyTool
~~~

It should generate a Program.cs similar to this:

~~~ csharp
using System;

namespace MyTool
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Hello World!");
        }
    }
}
~~~

Update the Program.cs to this:

~~~ csharp
using System;

namespace MyTool
{
    class Program
    {
        static void Main(string[] args)
        {
            // Validate input
            if (args.Length == 0)
            {
                // Show some help text
                Console.WriteLine("Pass in any text");
                return;
            }
            // Display first item
            Console.WriteLine(args[0]);
        }
    }
}
~~~

You can use the few commands below:

~~~ bash
# nothing passed in
> dotnet run
nothing to show

# hi passed in
> dotnet run -- hi
hi

# Two arguments passed in
> dotnet run -- hi hello
hi

# A single argument string passed in
> dotnet run -- "hi hello"
hi hello
~~~

## Enabled the Project as a Tool

To setup the project as a tool you only have to add a few attributes to the csproj file:

~~~ xml
<Project Sdk="Microsoft.NET.Sdk">

    <PropertyGroup>
        <OutputType>Exe</OutputType>
        <TargetFramework>net5.0</TargetFramework>
        <!-- Tell "dotnet pack" that this should show up in NuGet as a tool -->
        <PackAsTool>true</PackAsTool>
        <!-- Give your tool a custom short name for easier usage -->
        <ToolCommandName>mytool</ToolCommandName>
    </PropertyGroup>

</Project>
~~~

## References

Checkout Microsoft for a more encompassing tutorial: <a href="https://docs.microsoft.com/en-us/dotnet/core/tools/global-tools" target="_blank">How to manage .NET Core tools</a>