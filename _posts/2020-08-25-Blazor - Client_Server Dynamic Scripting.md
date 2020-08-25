---
layout: post
author: Cody Merritt Anhorn
title: Blazor - Client/Server Dynamic Scripting
categories: [blog, blazor]
tags: [.NET Core, C#]
---

For my new Blazor Game Client I wanted a way to create scripts that could be written by the user of the platform, and automatically load them from Client, without having to pull down a new client or having to reload the client. This article will go over the project I created to show how this might be done.

## The Project

I create a PoC project, <a href="https://github.com/canhorn/EventHorizon.Blazor.Dynamic.Scripting" target="_blank">canhorn/EventHorizon.Blazor.Dynamic.Scripting</a>, you can checkout that repository for a working example of this PoC. The project is broken up into two main components, a Blazor Server where the .NET Core DLL is assembled and hosted. And the Blazor WASM Client, where the .NET Core DLL is consumed. The server relies on <a href="https://github.com/oleg-shilo/cs-script.core" target="_blank">CS-Script.Core</a> to create the DLL, below is an example how easy it is to use CS-Script Core to create a DLL, calling <code>CompileAssemblyFromCode</code> will create a DLL and save it to the files system to the file path provided.

The server will then take the generated .NET Core Assembly DLL file and encode it for easy transfer between the server to the Client. By encoding it the client just has to decode the string and pass it into the <code>AssemblyLoadContext.Default.LoadFromStream</code>. By using <code>AssemblyLoadContext</code> we can get all the code wired up into the current running context of the code, this is very important as it makes sure the Assembly is not loaded into its own isolated context, blocking it from interacting with the rest of the application code. You can read up on the .NET Core AssemblyLoadContext here, <a href="https://docs.microsoft.com/en-us/dotnet/core/dependency-loading/understanding-assemblyloadcontext" target="_blank">Understanding System.Runtime.Loader.AssemblyLoadContext</a>.

The PoC also shows how code can be shared between the Client and Server, and making it usable in the script. The script takes the arguments passed in and uses them, giving them the ability to interact outside of the script, even though they were not compiled in Client project. 

## Example Source Code

Working example of the code below: <a href="https://github.com/canhorn/EventHorizon.Blazor.Dynamic.Scripting" target="_blank">canhorn/EventHorizon.Blazor.Dynamic.Scripting</a>

**Example Script created using CS-Script.Core**
~~~ csharp
CSScript.RoslynEvaluator
    .CompileAssemblyFromCode(
        @"
            using System;
            using System.Collections.Generic;
            using System.Threading.Tasks;
            using System.Threading;
            using MediatR;
            using EventHorizon.Blazor.Dynamic.Scripting.Model;
            using EventHorizon.Blazor.Dynamic.Scripting.Events;
            using EventHorizon.Blazor.Dynamic.Scripting.Services;
            
            public class Scripts_Assets_Tree_Create
            {
                public async Task<GenericModel> Run(
                    IMediator mediator,
                    IClientInterop interopService, 
                    IDictionary<string, object> data
                )
                {
                    interopService.Alert();
                    await mediator.Send(new GenericCommand { Echo = ""Script Command Echo Arg"" });
                    var @event = new GenericEvent { Echo = ""Script Event Echo Arg"" };
                    await mediator.Publish(@event, CancellationToken.None);
                    
                    var queryResults = await mediator.Send(new GenericQuery());
                    foreach(var queryResult in queryResults)
                    {
                        Console.WriteLine(""From Script: "" + queryResult.Value);
                    }
                    var fromData = data[""arg1""];
                    return new GenericModel { Value = $""From Script: {fromData}"" };
                }
            }
        ",
        path
    );
~~~

**Loading the DLL and Calling a Method**
~~~ csharp
var assembly = AssemblyLoadContext.Default.LoadFromStream(
    new MemoryStream(
        bytes
    )
);
dynamic script = assembly.CreateObject(
    "css_root+Scripts_Assets_Tree_Create"
);
var data = new Dictionary<string, object>
{
    { "arg1", "Argument from Client" },
};

dynamic result = await script.Run(
    Mediator,
    ClientInterop,
    data
);
Console.WriteLine(
    $"Script.Run(): {result.Value}"
);
~~~