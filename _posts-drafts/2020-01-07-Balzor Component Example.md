---
layout: post
author: Cody Merritt Anhorn
title: Blazor - Component Example
---

In this post I will go over a relatively complicated Razor Component that can be used in a Blazor application.

It will include the Razor, Model, and SCSS file details. We will also go over a example usage, simple and advanced.

The component will be a Terminal Component, just a simple command input/display component. What gives it power to the implementer is the exposed methods to run different types of prompts, inject their own text into the component, and having hooks into the javascript of the component.

This has a single service injected in, this service is a simple abstraction around the JSRuntime that allows for simple scripts to be ran easily against the page.


***Terminal.razor***
~~~ html
@using Microsoft.AspNetCore.Components.Forms
@inherits TerminalModel

<h1>Terminal Components</h1>

<EditForm class="terminal__form" Model="@State" OnSubmit="@HandleSubmit">
    <div class="terminal">
        <div @ref="@TerminalWindow"
             class="terminal__window"
             tabindex="0"
             @onkeypress="@HandleKeyPressOnWindow"
             @onpaste="@HandlePastToWindow">
            <div class="terminal__output">
                @foreach (var text in OutputTextList)
                {
                    <div class="terminal__output-text">@text</div>
                }
            </div>
            @if(!IsDisabled)
            {
                <span class="terminal__input-line">
                    <span class="terminal__input">@State.PrefixedInputText</span>
                    <span class="terminal__cursor">
                        C
                    </span>
                </span>
            }
        </div>
        <input class="terminal__hidden-input"
               autocomplete="off"
               @ref="@HiddenInput"
               disabled="@IsDisabled"
               @bind-value="@State.InputText"
               @bind-value:event="oninput"
               @onkeypress="@HandleKeyPressOnHiddenInput"
               @onchange="@HandleChangeOnHiddenInput" />
    </div>
</EditForm>
~~~


***Terminal.razor.cs***
~~~ csharp
using EventHorizon.Blazor.Scripter;
using Microsoft.AspNetCore.Components;
using Microsoft.AspNetCore.Components.Forms;
using Microsoft.AspNetCore.Components.Web;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace EventHorizon.Terminal.Component
{
    public enum TerminalPromptType
    {
        DEFAULT = 0,
        INPUT = 1,
        PASSWORD = 2,
        CONFIRM = 3,
    }
    public class TerminalModel : ComponentBase
    {
        [Inject]
        public JavaScriptRunner ScriptRunner { get; set; }

        [Parameter]
        public EventCallback<string> OnSubmit { get; set; }

        private bool _isDisabled = false;
        public bool IsDisabled
        {
            get => _isDisabled;
            set
            {
                _isDisabled = value;
                if (!_isDisabled)
                {
                    ResetFocus();
                }
            }
        }

        protected ElementReference HiddenInput { get; set; }
        protected ElementReference TerminalWindow { get; set; }

        protected TerminalState State { get; set; } = new TerminalState
        {
            Prefix = "> "
        };

        protected IList<string> OutputTextList { get; set; } = new List<string>();

        protected override async Task OnAfterRenderAsync(
            bool firstRender
        )
        {
            base.OnAfterRender(firstRender);
            if (!_isDisabled)
            {
                await ScrollToBottomOfWindow();
            }
        }

        public async Task Input(
            string message
        )
        {
            if (IsDisabled)
            {
                return;
            }
            await PromptInput(
                message,
                TerminalPromptType.INPUT
            );
        }

        public async Task PromptInput(
            string message,
            TerminalPromptType promptType
        )
        {
            if (IsDisabled)
            {
                return;
            }
            await Print(
                promptType == TerminalPromptType.CONFIRM
                    ? message + " (y/n)"
                    : message
            );
        }

        public Task Write(
            string input
        )
        {
            if (IsDisabled)
            {
                return Task.CompletedTask;
            }
            State.InputText += input;
            InvokeAsync(StateHasChanged);
            return Task.CompletedTask;
        }

        public Task Print(
            params string[] messages
        )
        {
            foreach (var message in messages)
            {
                OutputTextList.Add(
                    message
                );
            }
            InvokeAsync(StateHasChanged);
            return Task.CompletedTask;
        }

        public void ResetFocus() => InvokeAsync(async () =>
            {
                StateHasChanged();
                await FocusInput();
                await ScrollToBottomOfWindow();
            });

        protected async Task HandleSubmit()
        {
            if (IsDisabled)
            {
                return;
            }
            await Print(
                $"{State.PrefixedInputText}",
                $"Command run: {State.InputText}"
            );
            var input = State.InputText;
            State.InputText = "";
            await ScrollToBottomOfWindow();
            await OnSubmit.InvokeAsync(
                input
            );
        }

        protected async Task HandleKeyPressOnWindow(
            KeyboardEventArgs arg
        )
        {
            State.InputText += arg.Key;
            await FocusInput();
            await ScrollToBottomOfWindow();
        }

        protected async Task HandlePastToWindow()
        {
            await FocusInput();
            await ScrollToBottomOfWindow();
            await Print(
                "$$ Paste failed, please try again."
            );
        }

        protected async Task HandleKeyPressOnHiddenInput()
        {
            await ScrollToBottomOfWindow();
        }

        protected async Task HandleChangeOnHiddenInput()
        {
            await ScrollToBottomOfWindow();
        }

        private async Task FocusInput()
        {
            await ScriptRunner.Run(
                "terminal.focusElement",
                "$args.element.focus();",
                new
                {
                    element = HiddenInput
                }
            );
        }

        private async Task ScrollToBottomOfWindow()
        {
            await ScriptRunner.Run(
                "terminal.scrollToBottom",
                "$args.element.scrollTop = $args.element.scrollHeight;",
                new
                {
                    element = TerminalWindow
                }
            );
        }
    }

    public class TerminalState
    {
        public string Prefix { get; set; }
        public string InputText { get; set; }
        public string PrefixedInputText => $"{Prefix}{InputText}";
    }
}
~~~
