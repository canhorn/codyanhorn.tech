---
layout: post
author: Cody Merritt Anhorn
title: Blazor Dynamic JavaScript Runner
---

Even with Blazor you still need to use javascript in some form or fashion. And with this post we will go over how to create a simple JavaScript runner. The runner will just take a method identifier, the script as a string, and some arguments in the form of a simple object in C#.

## The Need

The main need for my personal work was to be able to focus an element on the page with very minimal code, not having to craft a whole interop between the C# and JavaScript code. The output was a simple JavaScript function that will be called by a simple abstraction in C#. With Blazor having a way to pass a reference to an element on the page thorough the JS Interop functionality this make is really easy to just focus the element using a short string script.

## Project Example

Below is an example Component and the the necessary interop files.

Here is project:
<a href="https://github.com/canhorn/EventHorizon.Blazor.Scripter" target="_blank">EventHorizon.Blazor.Scripter</a>

***Samples.razor***
~~~ 

<button @onclick="HandleAlertUser">Show Alert</button>
<br />
<button @onclick="HandlePromptUser">Prompt User</button>

@code {
    // Make sure to Register JavaScriptRunner with DI
    [Inject]
    public JavaScriptRunner JSRunner { get; set; }

    public async Task HandlePromptUser()
    {
        // Close the window with JavaScript.
        await JSRunner.Run(
            "closeWindow",
            "var name = prompt('What is your name?'); if(name) alert(name);",
            new { }
        );
    }

    public async Task HandleAlertUser()
    {
        // Say Hello to the UserName
        await JSRunner.Run(
            "alertUser",
            // To note that passed props will be converted to camel-case
            "alert('Hello, ' + $args.userName)",
            new
            {
                UserName = "Awesome",
            }
        );
    }
}
~~~

***JavaScriptRunner.cs***
~~~ csharp
public class JavaScriptInteropRunner : JavaScriptRunner
{
    readonly IJSRuntime _runtime;

    public JavaScriptInteropRunner(
        IJSRuntime runtime
    )
    {
        _runtime = runtime;
    }

    public ValueTask Run(
        string methodName,
        string script,
        object args
    )
    {
        return _runtime.InvokeVoidAsync(
            "scripter.runScript",
            new JavaScriptMethodRunner
            {
                MethodName = methodName,
                Script = script,
                Args = args
            }
        );
    }
}

public class JavaScriptMethodRunner
{
    public string MethodName { get; set; }
    public string Script { get; set; }
    public object Args { get; set; }
}
~~~

***scripter.js***
~~~ javascript
// This file is to show how a library package may provide JavaScript interop features
// wrapped in a .NET API

((_window) => {
    const methodCache = {};

    _window.scripter = {
        runScript: (methodRunner) => {
            methodCache[methodRunner.methodName] = new Function(
                "$args",
                methodRunner.script
            );
            methodCache[methodRunner.methodName](methodRunner.args);
        },
    };

})(window);
~~~