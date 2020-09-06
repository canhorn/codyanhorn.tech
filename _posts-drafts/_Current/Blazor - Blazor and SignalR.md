

~~~
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
    <CascadingValue Value="@this.State">
        @ChildContent
    </CascadingValue>
}
~~~



# About

The EventHorizon.Blazor.Interop is a slim WASM project I created to help with my integration of my Game Client. 

# Sample

The EventHorizon.Blazor.Interop.Sample Project contains a suite of performance tests I created to help me verify the different scenarios I will be using.

# Use Libraries

[BlazorJsFastDataExchanger](https://github.com/Lupusa87/BlazorJsFastDataExchanger)

[MediatR](https://github.com/jbogard/MediatR)

[Mono.WebAssembly.Interop](https://www.nuget.org/packages/Mono.WebAssembly.Interop)