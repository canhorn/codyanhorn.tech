---
layout: post
author: Cody Merritt Anhorn
title: Blazor - How to Create a Code-Behind Model
categories: [blog, blazor]
tags: [.NET, C#, blazor]
---

To create a model for your Blazor Component just ***@inherits*** from the model. 😊 Not much else than that is needed, checkout the example below for exact details. 

***Pros:***

***--*** Helps to encapsulate complicated logic into an easier to read area/file. 

***--*** This helps to eliminate logic from the view/razor layer of the component.

***--*** Makes it easier to to see where the model is used on the page. 

***--*** You can make methods private encapsulating the logic. 

***--*** Testing just the Model is easier as well.

***Gotcha:***

***--*** Make sure to extend from ***ComponentBase***, gives you the life-cycle methods.

## Some Example Code 

***Terminal.razor***
~~~ html
@using Microsoft.AspNetCore.Components.Forms
<!-- HERE IS HOW YOU SET THE MODEL -->
@inherits TerminalModel

<h1>Terminal Components</h1>

<EditForm class="terminal__form" Model="@State" OnSubmit="@HandleSubmit">
    <div class="terminal">
        <div class="terminal_output">
            @foreach (var text in State.OutputTextList)
            {
                <div class="terminal__output-text">@text</div>
            }
        </div>
        <input class="terminal__input"
               @bind-value="@State.InputText"
               @bind-value:event="oninput"
               @onkeypress="@HandleKeyPress" />
    </div>
</EditForm>
~~~


***Terminal.razor.cs***
~~~ csharp
using Microsoft.AspNetCore.Components;
using Microsoft.AspNetCore.Components.Web;
using System.Collections.Generic;
using System.Threading.Tasks;

namespace Terminal.Component
{
    public class TerminalModel : ComponentBase
    {
        [Parameter]
        public EventCallback<string> OnSubmit { get; set; }

        protected TerminalState State { get; set; } = new TerminalState();

        protected async Task HandleKeyPress(
            KeyboardEventArgs arg
        )
        {
            State.InputText += arg.Key;
        }

        protected async Task HandleSubmit()
        {
            await Print(
                $"{State.PrefixedInputText}",
                $"Command run: {State.InputText}"
            );
            var input = State.InputText;
            State.InputText = "";
            await OnSubmit.InvokeAsync(
                input
            );
            await InvokeAsync(StateHasChanged);
        }

        private async Task Print(
            params string[] messages
        )
        {
            foreach (var message in messages)
            {
                State.OutputTextList.Add(
                    message
                );
            }
        }
    }

    public class TerminalState
    {
        public string InputText { get; set; }
        protected IList<string> OutputTextList { get; set; } = new List<string>();
    }
}
~~~