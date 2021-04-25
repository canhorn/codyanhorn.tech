---
layout: post
author: Cody Merritt Anhorn
title: Blazor - IntersectionObserver WebApi
categories: [blog, blazor]
tags: [.NET Core, C#, Blazor, WebApi]
---

This article will go over the IntersectionObserver WebApi and how to use it within the context of a Blazor application. Checkout the <a href="https://proud-coast-03b6fa410.azurestaticapps.net/" target="_blank">Blazor Wasm Example Website</a> and the <a href="https://github.com/canhorn/EventHorizon.Blazor.WebApi.IntersectionObserver" target="_blank">GitHub Repository</a>.

## Details

<a href="/image/Posts/2021-04-24/WebApi-IntersectionObserver.png" target="_blank">
    <img src="/image/Posts/2021-04-24/WebApi-IntersectionObserver.png" title="Example of what the deployed application looks like, showing how the intersection observer sections are triggering an update with the visible ratio" />
</a>

The main reason for checking out the IntersectionObserver was to see how the <a href="https://github.com/canhorn/EventHorizon.Blazor.TypeScript.Interop.Generator" target="_blank">canhorn/EventHorizon.Blazor.TypeScript.Interop.Generator</a> can be used with an Api provided by the Browser. I created a TypeScript definition file that contains the API that can be used by the generator to create an C# interop abstraction. This interop abstraction can then be used by any Blazor Wasm/Server application to gain access to the IntersectionObserver API of the browser.

## Example

Checkout a the <a href="https://proud-coast-03b6fa410.azurestaticapps.net/" target="_blank">Static Blazor Wasm Website</a>, also included below you can see the TypeScript definition file and the Source Code generated for the IntersectionObserver. You can checkout the source code in this <a href="https://github.com/canhorn/EventHorizon.Blazor.WebApi.IntersectionObserver" target="_blank">canhorn/EventHorizon.Blazor.WebApi.IntersectionObserver</a> Github repository. It includes the TypeScript definition file and has the generated project to be viewed as well.

**The TypeScript definition file used to generate the .NET C# project.**
~~~ typescript
class IntersectionObserver {
    readonly root: Element | Document | null;
    readonly rootMargin: string;
    readonly thresholds: number;
    disconnect(): void;
    observe(target: Element): void;
    takeRecords(): IntersectionObserverEntry[];
    unobserve(target: Element): void;
    constructor(callback: (entries: IntersectionObserverEntry[], observer: IntersectionObserver) => void, options?: IntersectionObserverInit);
}

class IntersectionObserverEntry {
    readonly boundingClientRect: DOMRectReadOnly;
    readonly intersectionRatio: number;
    readonly intersectionRect: DOMRectReadOnly;
    readonly isIntersecting: boolean;
    readonly rootBounds: DOMRectReadOnly | null;
    readonly target: Element;
    readonly time: number;
    constructor(intersectionObserverEntryInit: IntersectionObserverEntryInit);
}

interface IntersectionObserverEntryInit {
    boundingClientRect: DOMRectInit;
    intersectionRatio: number;
    intersectionRect: DOMRectInit;
    isIntersecting: boolean;
    rootBounds: DOMRectInit | null;
    target: Element;
    time: number;
}

interface IntersectionObserverInit {
    root?: Element | Document | null;
    rootMargin?: string;
    threshold?: number | number[];
}

interface DOMRectInit {
    height?: number;
    width?: number;
    x?: number;
    y?: number;
}

interface DOMRectReadOnly {
    readonly bottom: number;
    readonly height: number;
    readonly left: number;
    readonly right: number;
    readonly top: number;
    readonly width: number;
    readonly x: number;
    readonly y: number;
    toJSON(): any;
}

/**
 * This is the DOM Element, includes only what is needed by the application to function.
 */
class Element {
    id: string;
    innerHTML: string;
    appendChild(child: Element): void;
}
~~~

**Example of a Source File Generated**<br />
<span style="font-size: 0.8rem">
    The IntersectionObserver generated C# source code. 
</span>
~~~ csharp
/// Generated - Do Not Edit
using System;
using System.Collections.Generic;
using System.Text.Json.Serialization;
using System.Threading.Tasks;
using EventHorizon.Blazor.Interop;
using EventHorizon.Blazor.Interop.Callbacks;
using Microsoft.JSInterop;



[JsonConverter(typeof(CachedEntityConverter<IntersectionObserver>))]
public class IntersectionObserver : CachedEntityObject
{
    #region Static Accessors

    #endregion

    #region Static Properties

    #endregion

    #region Static Methods

    #endregion

    #region Accessors

    #endregion

    #region Properties
        private Element __root;
        public Element root
        {
            get
            {
            if(__root == null)
            {
                __root = EventHorizonBlazorInterop.GetClass<Element>(
                    this.___guid,
                    "root",
                    (entity) =>
                    {
                        return new Element() { ___guid = entity.___guid };
                    }
                );
            }
            return __root;
            }
        }

        
        public string rootMargin
        {
            get
            {
            return EventHorizonBlazorInterop.Get<string>(
                    this.___guid,
                    "rootMargin"
                );
            }
        }

        
        public decimal thresholds
        {
            get
            {
            return EventHorizonBlazorInterop.Get<decimal>(
                    this.___guid,
                    "thresholds"
                );
            }
        }
    #endregion
    
    #region Constructor
        public IntersectionObserver() : base() { }

        public IntersectionObserver(
            ICachedEntity entity
        ) : base(entity)
        {
            ___guid = entity.___guid;
        }

        public IntersectionObserver(
            ActionCallback<IntersectionObserverEntry[], IntersectionObserver> callback, IntersectionObserverInit options = null
        )
        {
            var entity = EventHorizonBlazorInterop.New(
                new string[] { "IntersectionObserver" },
                callback, options
            );
            ___guid = entity.___guid;
        }
    #endregion

    #region Methods
        public void disconnect()
        {
            EventHorizonBlazorInterop.Func<CachedEntity>(
                new object[]
                {
                    new string[] { this.___guid, "disconnect" }
                }
            );
        }

        public void observe(Element target)
        {
            EventHorizonBlazorInterop.Func<CachedEntity>(
                new object[]
                {
                    new string[] { this.___guid, "observe" }, target
                }
            );
        }

        public IntersectionObserverEntry[] takeRecords()
        {
            return EventHorizonBlazorInterop.FuncArrayClass<IntersectionObserverEntry>(
                entity => new IntersectionObserverEntry() { ___guid = entity.___guid },
                new object[]
                {
                    new string[] { this.___guid, "takeRecords" }
                }
            );
        }

        public void unobserve(Element target)
        {
            EventHorizonBlazorInterop.Func<CachedEntity>(
                new object[]
                {
                    new string[] { this.___guid, "unobserve" }, target
                }
            );
        }
    #endregion
}
~~~