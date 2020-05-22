---
layout: post
author: Cody Merritt Anhorn
title: Blazor - State Management - Cascading Parameter
categories: [blog, blazor]
tags: [.NET, C#, Blazor]
---

In this article I will be going over State Management using a Cascading Parameter in Blazor. I use this in my blazor applications when I only want to access some sort of state from my Components. 

## Usage

The act of using Cascading Parameters require two things, one is a component that implements the CascadingValue tag with ChildContent. And two is to use the CascadingParameter attribute to "Inject" the state of the data type. The Cascading Parameter also supports passing in a Name, that will look for a matching Name attribute on a CascadingValue.

Below is the State Provider I use in my Components, this provider is at the root of my pages, so anything that would need ZoneState could use a CascadingParameter attribute to get it. The ZoneStateProvider Component should be the only and at the root of the Page, this way a page can control the ZoneId through url path parameters. This Parameter does not allow for multiple Providers. Provider do not have this in common, a provider can be structured that allows for multiple on a page with different parameters, this was just a design decision for my application.

## Example

Here are some code examples of how this works using an example I currently use in my Game Editor application.

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

**ZoneEditorPage.razor**
~~~ html
@page "/zone/{zoneId}/editor"
@attribute [Authorize]

@code {
    [Parameter]
    public string ZoneId { get; set; }
}

<!-- Here we use the provider with the ZoneId from the Path -->
<ZoneStateProvider ZoneId="@ZoneId">
    <ZoneEntityTable />
</ZoneStateProvider>
~~~

**ZoneEntityTable.razor**
~~~ html
@inherits ZoneEntityTableModel

<h3>@Localize["ZoneEntityList"]</h3>

<table class="c-table">
    <thead>
        <tr>
            <th class="c-th">@Localize["Id"]</th>
            <th class="c-th">@Localize["Global Id"]</th>
            <th class="c-th">@Localize["Name"]</th>
            <th class="c-th">@Localize["Type"]</th>
        </tr>
    </thead>
    <tbody>
        @foreach (var entity in List)
        {
            <tr class="c-tr cursor-pointer" @onclick="@(() => HandleNavigationToEntity(entity.Id))">
                <td class="c-td">@entity.Id</td>
                <td class="c-td">@entity.GlobalId</td>
                <td class="c-td">@entity.Name</td>
                <td class="c-td">@entity.Type</td>
            </tr>
        }
    </tbody>
</table>
~~~

**ZoneEntityTable.razor.cs**
~~~ csharp
public class ZoneEntityTableModel : ComponentBase
{
    [CascadingParameter]
    public ZoneState State { get; set; }

    public IList<ObjectEntityDetails> List => State.ZoneInfo.EntityList;

    [Inject]
    public IStringLocalizer<SharedResource> Localize { get; set; }
    [Inject]
    public NavigationManager NavigationManager { get; set; }

    public void HandleNavigationToEntity(
        long id
    ) => NavigationManager.NavigateTo(
        $"/zone/{State.Zone.Id}/editor/entity/{id}"
    );
}
~~~