---
layout: post
author: Cody Merritt Anhorn
title: DotNet Core - Create a Script DLL
categories: [blog]
tags: [.NET Core, C#]
---

In this article we will go over creating a DLL for scripts, this will allow for a single file artifact that can be loaded up in just about any .NET Core application.

## Why 

Personally I wanted a way to to create CSX scripts and load to create dynamic logic for my game clients. First pass was to run the compiling of the scripts on the client from the string representation. But I hit some roadblocks while working in the cutting edge of .NET 5.0, so I look at a different route. The route I found was that I could easily have the server compile all the scripts into a single DLL and loading it into the client. I have included some source code examples and a project in Github that shows how easy it is to do this.

I did use CS Script Core to handle the have lifting of creating the DLL, but after it just turned into loading the DLL directly from .NET is all that is needed to get it into memory. Checkout out <a href="https://github.com/oleg-shilo/cs-script.core" target="_blank">CS Script Core</a> its an awesome library that hides alot of the complicated bits of working with dynamic C#.

## Example Project

Checkout <a href="https://github.com/canhorn/EventHorizon.CreateScriptDll" target="_blank">canhorn/EventHorizon.CreateScriptDll</a> for a working example of the code below.

**Startup.cs for project**
~~~ csharp
using System;
using System.IO;
using CSScriptLib;

namespace CreateScriptDll
{
    class Program
    {
        static void Main(string[] args)
        {
            // Create Save Directory and Get File Full Name
            var fileFullName = GetFileFullName();

            // Using CS-Script.Core create the DLL of our Script
            CSScript.RoslynEvaluator
                .ReferenceAssemblyOf<Microsoft.CSharp.RuntimeBinder.CSharpArgumentInfo>()
                .CompileAssemblyFromCode(
                    @"
                    using System;

                    public class Script
                    {
                        // Noticed the dynamic usage so we don't have to know exactly what is on the arguments.
                        // And we are able to access any of the argument properties, or get an exception if not found.
                        public ScriptResult Method(dynamic arg1)
                        {
                            return new ScriptResult { Name = $""({arg1.ArgName}) Script Result!"" };
                        }
                    }
                    public class ScriptResult
                    {
                        public string Name { get; set; }
                    }
                ",
                    fileFullName
                );

            // Use the File Full Name to load in the Assembly
            LoadScriptAssemblyAndRunMethod(
                fileFullName
            );
        }

        /// <summary>
        /// Use the File Full Name to load in the Assembly
        /// </summary>
        /// <param name="fileFullName">The fullName of the Assembly File</param>
        private static void LoadScriptAssemblyAndRunMethod(
            string fileFullName
        )
        {
            // Take the file and read the bytes into memory
            var bytes = File.ReadAllBytes(fileFullName);
            // Take the bytes from the file and use Reflection to Load the Assembly
            var assembly = System.Reflection.Assembly.Load(bytes);
            // Create an instance of our Script class located in the scripts
            dynamic t = assembly.CreateInstance("css_root+Script");
            // Create our Data that will be passed to our Script.Method call
            var data = new ScriptArg { ArgName = "Script Argument!" };
            // Place the result into a dynamic object for accessing
            dynamic result = t.Method(data);
            // Display the Name from the result to the console
            Console.WriteLine($"Script.Method(data): {result.Name}");
        }

        private static string GetFileFullName()
        {

            // Create the File Name and Path
            var path = Path.Combine(
                ".",
                "scripts"
            );
            var fileFullName = Path.Combine(
                path,
                "Scripts.dll"
            );
            if (!Directory.Exists(
                path
            ))
            {
                Directory.CreateDirectory(
                    path
                );
            }
            return fileFullName;
        }
    }

    /// <summary>
    /// This class holds our arguments passed into the Script.Method call
    /// </summary>
    public class ScriptArg
    {
        public string ArgName { get; set; }
    }
}
~~~