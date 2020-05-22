---
layout: post
author: Cody Merritt Anhorn
title: Blazor - State Management - Shared Inject
categories: [blog, blazor]
tags: [.NET, C#, Blazor]
---

In this article I will be going over State Management using an Inject Attribute in Blazor. I use this in my Blazor applications when I only want to access some sort of state from anywhere not in the context of a CascadingParameter. This is usually for components or code that is triggered by observers and could only be triggered in the context of a valid Zone. 

## Usage

For more details about the Cascading Provider and more details about the ZoneStateProvider checkout the prior article about <a href="/blog/blazor/2020/05/21/Blazor-State-Management-Cascading-Parameter.html">Cascading Parameters</a>. This article focuses on the SelectedEntityActions Component, this component is not in the context of a Cascading Provider, so it needs to use an Injection of the ZoneState. It then uses the ZoneState to create requests for updates of a Select ClientEntity for a specific Zone, using the ServerAddress in the request.

## Example

Here are some code examples of how this works using an example I currently use in my Game Editor application.

**SelectedEntityActions.razor**
~~~ html
@inherits SelectedEntityActionsModel

@if (IsClientEntitySelected)
{
    <button class="c-button"
            @onclick="HandleCloneClientEntity">
        @Localizer["Clone"]
    </button>
    <button class="c-button"
            @onclick="HandleDeleteClientEntity">
        @Localizer["Delete"]
    </button>
    <button class="c-button"
            @onclick="HandleClearClientEntity">
        @Localizer["Clear"]
    </button>
}
~~~

**SelectedEntityActions.razor.cs**
~~~ csharp
// This component is used to trigger ClientEntity State changes.
// The ClientEntity is a static object in the Game world.
// The editor can edit them and this class will trigger updates to 
//  the Game server to persist those changes.
public class SelectedEntityActionsModel : EditorComponentBase,
    IDisposable,
    ZoneStateChangedObserver,
    ClientEntitySelectedObserver,
    ClientEntityUnselectedObserver,
    ClientEntityChangedEventObserver
{
    private ClientEntityDetails _clientEntity = default;

    // Here we don't expect the Parameter to be cascaded down,
    // so we Inject the ZoneState in from the Current Scope.
    [Inject]
    public ZoneState State { get; set; }

    public bool IsClientEntitySelected => _clientEntity.IsValid();

    protected override async Task OnInitializedAsync()
    {
        await base.OnInitializedAsync();

        await Mediator.Send(
            new RegisterObserverCommand(
                this
            )
        );
    }

    protected async Task HandleCloneClientEntity()
    {
        if (!_clientEntity.IsValid())
        {
            return;
        }
        await Mediator.Send(
            new CloneClientEntityToZoneServerAddress(
                State.Zone.ServerAddress,
                _clientEntity
            )
        );
    }

    protected async Task HandleDeleteClientEntity()
    {
        if (!_clientEntity.IsValid())
        {
            return;
        }
        await Mediator.Send(
            new DeleteClientEntityFromZoneServerAddress(
                State.Zone.ServerAddress,
                _clientEntity
            )
        );
    }

    protected async Task HandleClearClientEntity()
    {
        await Mediator.Send(
            new GameClientUnselectEntity(
                _clientEntity.GlobalId
            )
        );
    }

    public void Dispose()
    {
        Mediator.Send(
            new UnregisterObserverCommand(
                this
            )
        ).GetAwaiter().GetResult();
    }

    public async Task Handle(
        ZoneStateChangedObserverArgs _
    )
    {
        if (_clientEntity.IsValid())
        {
            _clientEntity = this.State.ZoneInfo.ClientEntityList.FirstOrDefault(
                entity => entity.GlobalId == _clientEntity.GlobalId
            );
            await InvokeAsync(StateHasChanged);
        }
    }

    public async Task Handle(
        ClientEntitySelectedObserverArgs args
    )
    {
        args.NullCheck();
        _clientEntity = State.ZoneInfo.ClientEntityList.FirstOrDefault(
            a => a.GlobalId == args.GlobalId
        );
        await InvokeAsync(StateHasChanged);
    }

    public async Task Handle(
        ClientEntityUnselectedObserverArgs args
    )
    {
        args.NullCheck();
        if (args.GlobalId == _clientEntity.GlobalId)
        {
            _clientEntity = default;
            await InvokeAsync(StateHasChanged);
        }
    }

    public async Task Handle(
        ClientEntityChangedEventObserverArgs args
    )
    {
        args.NullCheck();
        if (_clientEntity.ClientEntityId == args.Data.NewEntity.ClientEntityId)
        {
            _clientEntity = args.Data.NewEntity;
        }
        else
        {
            _clientEntity = default;
        }
        await InvokeAsync(StateHasChanged);
    }
}
~~~

**ZoneStateProvider.razor**
~~~ html
@inherits ZoneStateProviderModel

@if (State.IsLoading)
{
    <div>@Localizer["Loading..."]</div>
}
else if (!State.Zone.IsValid() || State.ZoneInfo == null)
{
    <div>@Localizer["Zone Invalid."]</div>
    <NavLink href="/zone-admin">@Localizer["Back to Zone Admin"]</NavLink>
}
else
{
    <!-- If the State is not Loading and Is Valid we create a CascadingValue with our State -->
    <CascadingValue Name="ZoneState" Value="@this.State">
        @ChildContent
    </CascadingValue>
}
~~~

**ZoneStateProvider.razor.cs**
~~~ csharp
[Authorize]
public class ZoneStateProviderModel : ComponentBase,
    IDisposable,
    // This observer that helps to trigger state changes outside of the Blazor lifecycle
    ZoneStateChangedObserver 
{
    [Inject]
    public ZoneStateUpdater StateUpdater { get; set; }
    [Inject]
    public ZoneState State { get; set; }
    [Inject]
    public IStringLocalizer<SharedResource> Localizer { get; set; }
    [Inject]
    public IMediator Mediator { get; set; }

    [Parameter]
    public string ZoneId { get; set; }
    [Parameter]
    public RenderFragment ChildContent { get; set; }

    public void Dispose()
    {
        // Observer Un-Registration
        Mediator.Send(
            new UnregisterObserverCommand(this)
        ).GetAwaiter().GetResult();
    }

    public Task Handle(
        ZoneStateChangedObserverArgs args
    )
    {
        // We handle the State Change observer event by telling Blazor it had a State Change Event.
        return InvokeAsync(StateHasChanged);
    }

    protected override async Task OnInitializedAsync()
    {
        // When this components is initialized we updated the State to this ZoneId.
        await StateUpdater.SwitchZone(ZoneId);
        // Observer Registration
        await Mediator.Send(
            new RegisterObserverCommand(
                this
            )
        );
    }
}
~~~
