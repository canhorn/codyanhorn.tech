---
layout: post
author: Cody Merritt Anhorn
title: Project - EventHorizon Blazor Interop
categories: [blog]
tags: [.NET Core, Blazor, WASM]
---

I have been working on a project that will generate a C# Blazor WASM abstraction from a TypeScript definition file, as part of that project I needed a way to run the equivalent functionality in JavaScript. This article will go over that WASM Interop project I created, I also package it up in an easy to use NuGet package if you want to use it as well. 

**Repository:** <a href="https://github.com/canhorn/EventHorizon.Blazor.Interop" target="_blank">canhorn/EventHorizon.Blazor.Interop</a>

**NuGet Package:** <a href="https://www.nuget.org/packages/EventHorizon.Blazor.Interop" target="_blank">EventHorizon.Blazor.Interop</a>

**Sample Site:** <a href="https://polite-plant-0f8750a10.azurestaticapps.net/" target="_blank">EventHorizon Blazor Interop</a>

> To note, this project uses very low level/undocumented WASM API's provided by .NET Core, so changes to the framework could break this library. I will do my best to always release a version that will support the currently released version.

## What is it?

The EventHorizon Blazor Interop project exposes a C# JSRuntime/Interop abstraction over a common set of JavaScript functionality. 

The functionality includes:

**-** Calling a function on a Cached Entity

**-** Calling any function attached to the JS window

**-** Setting any value attached to the JS window

**-** Getting any value attached to the JS window

**-** Creating a new object attached to the JS window

**-** Running string based scripts

**-** Function Callbacks, will call a C# action from JavaScript. 

The project includes a caching layer, so you can pass in other Cached Entities to the function of other Cached Entities. The JavaScript bridge includes logic that can introspect the passed in identifier and arguments and tie them to the Cached Entities. The project also includes a JsonConverter that will create a slim communication layer, this allows for the communication to send the bear minimum between the layers.

## Functionality Details

You can use the API's provided by <code>EventHorizonBlazorInterop</code> to call into JavaScript, the API is very generic, just taking in <code>object[] args</code> in most cases. This was done to make it as generic as possible to cover my use cases. The heavy lifting is done in the JavaScript layer, to parse the identifier and any arguments that should be used. 

The API calls handle responses broken up into Primitive, Class and Array types, this allowed me to easily type the responses. The Class responses will take in a <code>classBuilder</code> action that passes the CachedEntity from the API to the builder, the allows for a Class to be tied to the entity on the JavaScript client.

## API Details

**-** Calling a function with arguments, function can be on the window or a <a href="https://github.com/canhorn/EventHorizon.Blazor.Interop/blob/main/EventHorizon.Blazor.Interop/ICachedEntity.cs" target="_blank">ICachedEntity</a>. Helps returning primitive or a Class type results, can also return arrays.

**-** Getting or setting values, can be from/set on the window or a <a href="https://github.com/canhorn/EventHorizon.Blazor.Interop/blob/main/EventHorizon.Blazor.Interop/ICachedEntity.cs" target="_blank">ICachedEntity</a>. Helps getting primitive or a Class type results, can also return arrays.

**-** Has the ability to create a "new" object based on a constructor function in JavaScript, will return a cached entity for easy usage against the other APIs.

**-** Can run JavaScript script text with arguments passed from C#.

**-** Has the ability to wire up DotNetObjectReference's for callback base methods from JavaScript libraries. Also provides an Assembly callback API for C# static method calls.

You can checkout the GitHub source for <a href="https://github.com/canhorn/EventHorizon.Blazor.Interop/blob/main/EventHorizon.Blazor.Interop/EventHorizonBlazorInterop.cs" target="_blank">EventHorizonBlazorInterop.cs</a> for more details about the different APIs.

## Sample Usage

I have a Blazor WASM website deployed that uses the API to run a variety of performance tests against the APIs: <a href="https://polite-plant-0f8750a10.azurestaticapps.net/" target="_blank">https://polite-plant-0f8750a10.azurestaticapps.net/</a>

## Implementation Example

The project <a href="https://github.com/canhorn/EventHorizon.Blazor.TypeScript.Interop.Generator" target="_blank">canhorn/EventHorizon.Blazor.TypeScript.Interop.Generator</a> uses the functionality provided by this project to generate an abstraction project from a TypeScript definition file. This project can then be used in C# to interface with the JavaScript portions of the library with very little JavaScript.

## Usage Source Example

Below is an example of the BabylonJS Observable class looks like after generated and using this project. This might not be exactly the same as of writing this, so checkout the <a href="https://github.com/canhorn/EventHorizon.Blazor.TypeScript.Interop.Generator" target="_blank">repository</a> for the source.

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