---
layout: post
author: Cody Merritt Anhorn
title: Project - EventHorizon Blazor TypeScript Interop Generator
categories: [blog, blazor]
tags: [.NET Core, Blazor, WASM]
---

This project's name is a mouthful, but based on the name you should be able to get the gist of what the project does. The project generates a C# Blazor Interop abstraction from the Abstract Syntax Tree from a TypeScript definition file, giving the user a generated project that can make interfacing with JavaScript libraries easier from C#.

**Repository:** <a href="https://github.com/canhorn/EventHorizon.Blazor.TypeScript.Interop.Generator" target="_blank">canhorn/EventHorizon.Blazor.TypeScript.Interop.Generator</a>

**NuGet Package:** <a href="https://www.nuget.org/packages/EventHorizon.Blazor.TypeScript.Interop.Generator" target="_blank">EventHorizon.Blazor.TypeScript.Interop.Generator</a>

**Sample Site:** <a href="https://wonderful-pond-05f7b3b10.azurestaticapps.net/" target="_blank">BabylonJS Generated Example Site</a>

<a href="https://www.nuget.org/packages/EventHorizon.Blazor.TypeScript.Interop.Tool" target="_blank">.NET Core Tool Package</a> - Can be used to easily generate the project from the CLI.

## Why

For a personal project I am rewriting a game client using Blazor, specifically Blazor WASM, and needed a way to interface with BabylonJS from C#. To do this I found I was creating a the same patterns over and over for accessing properties and creating connection to the real object from the JavaScript. Out of these pattens I created the project <a href="https://github.com/canhorn/EventHorizon.Blazor.Interop" target="_blank">canhorn/EventHorizon.Blazor.Interop</a> that packages up these patterns into an easy to consume library.

## Project Details

Still while having packaged the patterns I still noticed that I was doing other patterns but using the interop. The one thing I noticed was that I kept going back to the BabylonJS TypeScript definition file to figure out how the API should be structured. I had the though, maybe I could use the type definition file to generate these typed abstractions for me, I then found a library that would parse a TypeScript definition file.

I took a weekend tinkering with the generated AST to see what details were represented. With that knowledge, I created a simple source generator, I say simple in that the generated project was very limited. But the code was very complicated by just trying to get the optimal type. The work that came after was to add Generics, callbacks to C# from the JavaScript and better type checking. This actually simplified the type checking and in turn matched the generated C# to the TypeScript representation.

## What is this project?

The generated project can be used with Blazor WASM to interface with JavaScript from C#, this gives most JavaScript libraries an easy to use interface from C#. It uses the JSRuntime to interop directly with the underlying JavaScript from C#, this is done with a custom interop abstraction.

The interop project is located in its own <a href="https://github.com/canhorn/EventHorizon.Blazor.Interop" target="_blank">package/repository</a>, it gives the generated code access to a common set of access patterns to interface with the JavaScript.

## API Details

Checkout the <a href="https://github.com/canhorn/EventHorizon.Blazor.TypeScript.Interop.Generator#supported-apis-generated" target="_blank">Supported API's Generated</a> on the project repository's homepage, it will have a more up-to-date version of the supported API's.

## Example of Generated Source

You can checkout the <a href="https://github.com/canhorn/EventHorizon.Blazor.TypeScript.Interop.Generator/tree/main/Sample/_generated/EventHorizon.Blazor.BabylonJS.WASM" target="_blank">source in GitHub</a> to see an example what it can generate, below is an example of the Observable class that was generated for interfacing with the BabylonJS version of the Observable API.

~~~ csharp
/// Generated - Do Not Edit
namespace BabylonJS
{
    using System;
    using System.Collections.Generic;
    using System.Text.Json.Serialization;
    using System.Threading.Tasks;
    using EventHorizon.Blazor.Interop;
    using Microsoft.JSInterop;
    
    [JsonConverter(typeof(CachedEntityConverter))]
    public class Observable<T> : CachedEntityObject where T : CachedEntity, new()
    {
        #region Static Accessors

        #endregion

        #region Static Properties

        #endregion

        #region Static Methods

        #endregion

        #region Accessors
        
        public Observer<T>[] observers
        {
            get
            {
            return EventHorizonBlazorInterop.GetArrayClass<Observer<T>>(
                    this.___guid,
                    "observers",
                    (entity) =>
                    {
                        return new Observer<T>() { ___guid = entity.___guid };
                    }
                );
            }
        }
        #endregion

        #region Properties

        #endregion
        
        #region Constructor
        public Observable() : base() { } 

        public Observable(
            ICachedEntity entity
        ) : base(entity)
        {
            ___guid = entity.___guid;
        }

        public Observable(
            CachedEntity onObserverAdded = null
        )
        {
            var entity = EventHorizonBlazorInterop.New(
                new string[] { "BABYLON", "Observable" },
                onObserverAdded
            );
            ___guid = entity.___guid;
        }
        #endregion

        #region Methods
        #region add TODO: Get Comments as metadata identification
        private bool _isAddEnabled = false;
        private readonly IDictionary<string, Func<T, EventState, Task>> _addActionMap = new Dictionary<string, Func<T, EventState, Task>>();

        public string add(
            Func<T, EventState, Task> callback
        )
        {
            SetupAddLoop();

            var handle = Guid.NewGuid().ToString();
            _addActionMap.Add(
                handle,
                callback
            );

            return handle;
        }

        private void SetupAddLoop()
        {
            if (_isAddEnabled)
            {
                return;
            }
            EventHorizonBlazorInterop.FuncCallback(
                this,
                "add",
                "CallAddActions",
                _invokableReference
            );
            _isAddEnabled = true;
        }

        [JSInvokable]
        public async Task CallAddActions(T eventData, EventState eventState)
        {
            foreach (var action in _addActionMap.Values)
            {
                await action(eventData, eventState);
            }
        }
        #endregion

        #region addOnce TODO: Get Comments as metadata identification
        private bool _isAddOnceEnabled = false;
        private readonly IDictionary<string, Func<T, EventState, Task>> _addOnceActionMap = new Dictionary<string, Func<T, EventState, Task>>();

        public string addOnce(
            Func<T, EventState, Task> callback
        )
        {
            SetupAddOnceLoop();

            var handle = Guid.NewGuid().ToString();
            _addOnceActionMap.Add(
                handle,
                callback
            );

            return handle;
        }

        private void SetupAddOnceLoop()
        {
            if (_isAddOnceEnabled)
            {
                return;
            }
            EventHorizonBlazorInterop.FuncCallback(
                this,
                "addOnce",
                "CallAddOnceActions",
                _invokableReference
            );
            _isAddOnceEnabled = true;
        }

        [JSInvokable]
        public async Task CallAddOnceActions(T eventData, EventState eventState)
        {
            foreach (var action in _addOnceActionMap.Values)
            {
                await action(eventData, eventState);
            }
        }
        #endregion

        public bool remove(Observer<T> observer)
        {
            return EventHorizonBlazorInterop.Func<bool>(
                new object[] 
                {
                    new string[] { this.___guid, "remove" }, observer
                }
            );
        }

        #region removeCallback TODO: Get Comments as metadata identification
        private bool _isRemoveCallbackEnabled = false;
        private readonly IDictionary<string, Func<T, EventState, Task>> _removeCallbackActionMap = new Dictionary<string, Func<T, EventState, Task>>();

        public string removeCallback(
            Func<T, EventState, Task> callback
        )
        {
            SetupRemoveCallbackLoop();

            var handle = Guid.NewGuid().ToString();
            _removeCallbackActionMap.Add(
                handle,
                callback
            );

            return handle;
        }

        private void SetupRemoveCallbackLoop()
        {
            if (_isRemoveCallbackEnabled)
            {
                return;
            }
            EventHorizonBlazorInterop.FuncCallback(
                this,
                "removeCallback",
                "CallRemoveCallbackActions",
                _invokableReference
            );
            _isRemoveCallbackEnabled = true;
        }

        [JSInvokable]
        public async Task CallRemoveCallbackActions(T eventData, EventState eventState)
        {
            foreach (var action in _removeCallbackActionMap.Values)
            {
                await action(eventData, eventState);
            }
        }
        #endregion

        public void makeObserverTopPriority(Observer<T> observer)
        {
            EventHorizonBlazorInterop.Func<CachedEntity>(
                new object[] 
                {
                    new string[] { this.___guid, "makeObserverTopPriority" }, observer
                }
            );
        }

        public void makeObserverBottomPriority(Observer<T> observer)
        {
            EventHorizonBlazorInterop.Func<CachedEntity>(
                new object[] 
                {
                    new string[] { this.___guid, "makeObserverBottomPriority" }, observer
                }
            );
        }

        public bool notifyObservers(T eventData, System.Nullable<decimal> mask = null, object target = null, object currentTarget = null)
        {
            return EventHorizonBlazorInterop.Func<bool>(
                new object[] 
                {
                    new string[] { this.___guid, "notifyObservers" }, eventData, mask, target, currentTarget
                }
            );
        }

        public void notifyObserversWithPromise(T eventData, System.Nullable<decimal> mask = null, object target = null, object currentTarget = null)
        {
            EventHorizonBlazorInterop.Func<CachedEntity>(
                new object[] 
                {
                    new string[] { this.___guid, "notifyObserversWithPromise" }, eventData, mask, target, currentTarget
                }
            );
        }

        public void notifyObserver(Observer<T> observer, T eventData, System.Nullable<decimal> mask = null)
        {
            EventHorizonBlazorInterop.Func<CachedEntity>(
                new object[] 
                {
                    new string[] { this.___guid, "notifyObserver" }, observer, eventData, mask
                }
            );
        }

        public bool hasObservers()
        {
            return EventHorizonBlazorInterop.Func<bool>(
                new object[] 
                {
                    new string[] { this.___guid, "hasObservers" }
                }
            );
        }

        public void clear()
        {
            EventHorizonBlazorInterop.Func<CachedEntity>(
                new object[] 
                {
                    new string[] { this.___guid, "clear" }
                }
            );
        }

        public Observable<T> clone()
        {
            return EventHorizonBlazorInterop.FuncClass<Observable<T>>(
                entity => new Observable<T>() { ___guid = entity.___guid },
                new object[] 
                {
                    new string[] { this.___guid, "clone" }
                }
            );
        }

        public bool hasSpecificMask(System.Nullable<decimal> mask = null)
        {
            return EventHorizonBlazorInterop.Func<bool>(
                new object[] 
                {
                    new string[] { this.___guid, "hasSpecificMask" }, mask
                }
            );
        }
        #endregion
    }
}
~~~