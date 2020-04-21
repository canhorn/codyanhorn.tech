---
layout: post
author: Cody Merritt Anhorn
title: Blazor - Mobile Blazor Bindings WebView Wrapper
categories: [blog, blazor]
tags: [.NET, C#, Blazor, Xamarin, Mobile]
---

While getting into Mobile development using Xamarin, I found that I could use Blazor to create a Xamarin application. I find this more conformable, than working in the XAML of Xamarin, and I get to use my new favorite framework Blazor! But while creating the my application I found that the current supported bindings provided by the [MobileBlazorBindings](https://github.com/xamarin/MobileBlazorBindings) project did not have support for the WebView, so I had to create my own control wrapper around it, below is the example I created using the samples from the project.

## Details About the Binding Control

When creating a wrapper around any type of control using the Mobile Blazor Binding library it requires two main parts; the wrapper around the control and the handler. The wrapper is where you map the Blazor parameters to their equivalent control properties, in the case of the WebView we pass down the Blazor Source property down to the Xamarin Forms WebView Source property. As for the Handler we use this to encapsulate all the EventHandler/EventCallback logic and wiring up.

One thing that was not covered in the samples were the accessing of Getter properties and the triggering of methods. Using the @ref in Blazor I am able to get direct access to the WebView wrapper, with this I am able to trigger JavaScript in the WebView. This personally allows me to create a wrapper around my game, pointing it to the website the game will be hosted at. Then using JavaScript I can trigger, from a Xamarin button, logic on the webpage.

## Example 

Below is the example of a WebView wrapper that I developed in an afternoon's amount of time, checkout the comments for more details about different areas in the code.

***WebView.cs***
~~~ csharp
using Microsoft.AspNetCore.Components;
using Microsoft.MobileBlazorBindings.Core;
using Microsoft.MobileBlazorBindings.Elements;
using System.Threading.Tasks;
using XF = Xamarin.Forms;

public class WebView : View
{
    static WebView()
    {
        ElementHandlerRegistry
            .RegisterElementHandler<WebView>(renderer => new WebViewHandler(renderer, new XF.WebView()));
    }

    // These are the parameters that can be passed into this Control
    //  and then delegated to the Underlying WebView Control.
    [Parameter] public string Source { get; set; }

    // These are the Events that can be bound to from the Blazor level.
    [Parameter] public EventCallback GoBackRequested { get; set; }
    [Parameter] public EventCallback<XF.WebNavigatingEventArgs> Navigating { get; set; }
    [Parameter] public EventCallback<XF.WebNavigatedEventArgs> Navigated { get; set; }
    [Parameter] public EventCallback ReloadRequested { get; set; }
    [Parameter] public EventCallback GoForwardRequested { get; set; }

    // This helps cast the NativeControl to the expected Type for property access below.
    public XF.WebView InternalControl => (NativeControl as XF.WebView);

    // These are readonly properties, so we only have to give the user access to these from the Blazor binding.
    public bool CanGoForward => InternalControl.CanGoForward;
    public bool CanGoBack => InternalControl.CanGoBack;

    // These are abstraction over the Xamarin Forms WebView, so we can call the method from our Blazor binding.
    public void Eval(string script) => InternalControl.Eval(script);
    public Task<string> EvaluateJavaScriptAsync(string script) => InternalControl.EvaluateJavaScriptAsync(script);
    public void GoBack() => InternalControl.GoBack();
    public void GoForward() => InternalControl.GoForward();
    public void Reload() => InternalControl.Reload();

    protected override void RenderAttributes(AttributesBuilder builder)
    {
        base.RenderAttributes(builder);

        if (Source != null)
        {
            builder.AddAttribute(nameof(Source), Source);
        }

        RenderAdditionalAttributes(builder);
    }

    // Here is where we bind to our Handler events allowing for use
    //  to get notified of these events from the Xamarin Forms WebView.
    void RenderAdditionalAttributes(AttributesBuilder builder)
    {
        builder.AddAttribute("ongobackrequested", GoBackRequested);
        builder.AddAttribute("onnavigating", Navigating);
        builder.AddAttribute("onnavigated", Navigated);
        builder.AddAttribute("onreloadrequested", ReloadRequested);
        builder.AddAttribute("ongoforwardrequested", GoForwardRequested);
    }
}
~~~

***WebViewHandler.cs***
~~~ csharp
using Microsoft.MobileBlazorBindings.Core;
using Microsoft.MobileBlazorBindings.Elements.Handlers;
using System;
using XF = Xamarin.Forms;

public class WebViewHandler : ViewHandler
{
    public WebViewHandler(NativeComponentRenderer renderer, XF.WebView webViewControl)
        : base(renderer, webViewControl)
    {
        WebViewControl = webViewControl ?? throw new ArgumentNullException(nameof(webViewControl));

        Initialize(renderer);
    }

    public XF.WebView WebViewControl { get; }

    public override void ApplyAttribute(ulong attributeEventHandlerId, string attributeName, object attributeValue, string attributeEventUpdatesAttributeName)
    {
        switch (attributeName)
        {
            case nameof(XF.WebView.Source):
                WebViewControl.Source = attributeValue.ToString();
                break;
            default:
                base.ApplyAttribute(attributeEventHandlerId, attributeName, attributeValue, attributeEventUpdatesAttributeName);
                break;
        }
    }

    void Initialize(NativeComponentRenderer renderer)
    {

        RegisterEvent(
            eventName: "ongobackrequested",
            setId: id => GoBackRequestedEventHandlerId = id,
            clearId: id => { if (GoBackRequestedEventHandlerId == id) GoBackRequestedEventHandlerId = 0; });
        WebViewControl.GoBackRequested += (s, e) =>
        {
            if (GoBackRequestedEventHandlerId != default)
            {
                renderer.Dispatcher.InvokeAsync(() => renderer.DispatchEventAsync(GoBackRequestedEventHandlerId, null, e));
            }
        };

        RegisterEvent(
            eventName: "onnavigating",
            setId: id => NavigatingEventHandlerId = id,
            clearId: id => { if (NavigatingEventHandlerId == id) NavigatingEventHandlerId = 0; });
        WebViewControl.Navigating += (s, e) =>
        {
            if (NavigatingEventHandlerId != default)
            {
                renderer.Dispatcher.InvokeAsync(() => renderer.DispatchEventAsync(NavigatingEventHandlerId, null, e));
            }
        };

        RegisterEvent(
            eventName: "onnavigated",
            setId: id => NavigatedEventHandlerId = id,
            clearId: id => { if (NavigatedEventHandlerId == id) NavigatedEventHandlerId = 0; });
        WebViewControl.Navigated += (s, e) =>
        {
            if (NavigatedEventHandlerId != default)
            {
                renderer.Dispatcher.InvokeAsync(() => renderer.DispatchEventAsync(NavigatedEventHandlerId, null, e));
            }
        };

        RegisterEvent(
            eventName: "onreloadrequested",
            setId: id => ReloadRequestedEventHandlerId = id,
            clearId: id => { if (ReloadRequestedEventHandlerId == id) ReloadRequestedEventHandlerId = 0; });
        WebViewControl.ReloadRequested += (s, e) =>
        {
            if (ReloadRequestedEventHandlerId != default)
            {
                renderer.Dispatcher.InvokeAsync(() => renderer.DispatchEventAsync(ReloadRequestedEventHandlerId, null, e));
            }
        };

        RegisterEvent(
            eventName: "ongoforwardrequested",
            setId: id => GoForwardRequestedEventHandlerId = id,
            clearId: id => { if (GoForwardRequestedEventHandlerId == id) GoForwardRequestedEventHandlerId = 0; });
        WebViewControl.GoForwardRequested += (s, e) =>
        {
            if (GoForwardRequestedEventHandlerId != default)
            {
                renderer.Dispatcher.InvokeAsync(() => renderer.DispatchEventAsync(GoForwardRequestedEventHandlerId, null, e));
            }
        };
    }

    public ulong GoBackRequestedEventHandlerId { get; set; }
    public ulong NavigatingEventHandlerId { get; set; }
    public ulong NavigatedEventHandlerId { get; set; }
    public ulong ReloadRequestedEventHandlerId { get; set; }
    public ulong GoForwardRequestedEventHandlerId { get; set; }

}
~~~