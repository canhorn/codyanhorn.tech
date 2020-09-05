---
layout: post
author: Cody Merritt Anhorn
title: Blazor - Page Animated Transitions
categories: [blog, blazor]
tags: [.NET Core, C#]
---

With this article I will go over how to use <a href="https://www.nuget.org/packages/BlazorTransitionableRoute/" target="_blank">BlazorTransitionableRoute</a> to create an animated transition between Blazor page transitions.

When playing around with my <a href="https://github.com/canhorn/EventHorizon.Blazor.UX" target="_blank">canhorn/EventHorizon.Blazor.UX</a> project I wanted to add a bit of movement to every action the user does, and the most up front way to give this impression is during page navigation. I use the BlazorTransitionableRoute package with my Blazor application to help control the timing of page transitions, below I include a hight over view of the code I used, it's not a working example here, but you can checkout <a href="https://github.com/canhorn/EventHorizon.Blazor.UX" target="_blank">canhorn/EventHorizon.Blazor.UX</a> for the source code, and the <a href="https://lively-mud-0597d4e10.azurestaticapps.net/" target="_blank">EventHorizon.Blazor.UX</a> sample site for a live demo.

## Example

Checkout this awesome Blazor package that I used to take care of the plumbing of hooking into the Route events: <a href="https://www.nuget.org/packages/BlazorTransitionableRoute/" target="_blank">BlazorTransitionableRoute</a>

To handle the heavy lifting of the transition animations this solution uses <a href="https://animate.style/" target="_blank">Animate.css</a>.

Checkout a live Demo here <a href="https://lively-mud-0597d4e10.azurestaticapps.net/" target="_blank">EventHorizon.Blazor.UX</a>.

You can also checkout the source of the whole site in the <a href="https://github.com/canhorn/EventHorizon.Blazor.UX" target="_blank">canhorn/EventHorizon.Blazor.UX</a> Github repository. 

**App.razor**<br />
<span style="font-size: 0.8rem">
    Here is the LayoutView, with two new routes. The routes are the current and next route.
</span>
~~~ html
<Router AppAssembly="@typeof(Program).Assembly">
    <Found Context="routeData">
        <LayoutView Layout="@typeof(MainLayout)">
            <TransitionableRoutePrimary RouteData="@routeData">
                <TransitionableRouteView DefaultLayout="@typeof(MyViewLayout)" />
            </TransitionableRoutePrimary>
            <TransitionableRouteSecondary RouteData="@routeData">
                <TransitionableRouteView DefaultLayout="@typeof(MyViewLayout)" />
            </TransitionableRouteSecondary>
        </LayoutView>
    </Found>
    <NotFound>
        <LayoutView Layout="@typeof(MainLayout)">
            <p>Sorry, there's nothing at this address.</p>
        </LayoutView>
    </NotFound>
</Router>
~~~

**MyViewLayout.razor**<br />
<span style="font-size: 0.8rem">
    This custom View Layout, when used with the App.razor above, helps with transition and timing of all the pages content.
</span>
~~~ html
@inherits TransitionableLayoutComponent

<div class="view-content transition @transitioningClass">
    @Body
</div>

@code {
    // Triggers the transition in/out of this view
    private string transitioningClass => TransitioningIn ? "transition-in transitioning" : "transition-out transitioning";
}
~~~

**MyViewLayout.razor.css**<br />
<span style="font-size: 0.8rem">
    Helps to style the page content for a smoother transition.
</span>
~~~ css
.view-content {
    padding-left: 2rem !important;
    padding-right: 1.5rem !important;
}

.transition-in {
    opacity: 0;
}

.transitioning {
    position: absolute;
    left: 0;
    right: 0;
}

.transition-display-none {
    display: none;
}
~~~

**JavaScript**<br />
<span style="font-size: 0.8rem">
    Slightly complicated but handles the in/out animation of the transition marked elements on the page and other edge cases.
</span>
~~~ javascript
window.ehzBlazorUx = {
    transitionFunction: function (back) {

        let transitionIn = document.getElementsByClassName('transition-in')[0];
        let transitionOut = document.getElementsByClassName('transition-out')[0];

        if (transitionIn && transitionOut) {
            const handle_transitionOut_onanimationend = () => {
                console.log('Transition ended');
                transitionOut.removeEventListener("animationend", handle_transitionOut_onanimationend);
                transitionOut.classList.remove("transitioning");
                if (transitionOut.classList.contains(
                    "transition__out"
                )) {
                    transitionOut.classList.add('transition-display-none');
                }
                if (transitionOut.classList.contains(
                    "transition__in"
                )) {
                    transitionOut.classList.remove('transition-display-none');
                }
            };
            const handle_transitionOut_onanimationcancel = () => {
                console.log('Animation Cancel');
                transitionOut.removeEventListener("animationend", handle_transitionOut_onanimationend);
                transitionOut.removeEventListener("animationcancel", handle_transitionOut_onanimationcancel);
            };
            transitionOut.addEventListener('animationend', handle_transitionOut_onanimationend);
            transitionOut.addEventListener('animationcancel', handle_transitionOut_onanimationcancel);

            transitionOut.classList.remove('transition-out');
            transitionOut.classList.remove('transition-display-none');
            transitionOut.classList.add(
                "transition__out",
                // These are Animate.css provided css
                "animate__zoomOut",
                "animate__slow",
                "animate__animated"
            );

            const handle_transitionIn_onanimationend = () => {
                console.log('Animation ended');
                transitionIn.removeEventListener("animationend", handle_transitionIn_onanimationend);
                if (transitionIn.classList.contains(
                    "transition__in"
                )) {
                    transitionIn.classList.remove('transition-display-none');
                }
                if (transitionIn.classList.contains(
                    "transition__out"
                )) {
                    transitionIn.classList.add('transition-display-none');
                }
            };
            const handle_transitionIn_onanimationcancel = () => {
                console.log('Animation Cancel');
                transitionIn.removeEventListener("animationend", handle_transitionIn_onanimationend);
                transitionIn.removeEventListener("animationcancel", handle_transitionIn_onanimationcancel);
            };
            transitionIn.addEventListener('animationend', handle_transitionIn_onanimationend);
            transitionIn.addEventListener('animationcancel', handle_transitionIn_onanimationcancel);

            transitionIn.classList.remove('transition-in');
            transitionIn.classList.add(
                "transition__in",
                // These are Animate.css provided css
                "animate__zoomIn",
                "animate__animated"
            );
        }
    }
}
~~~