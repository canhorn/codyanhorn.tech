---
layout: post
author: Cody Merritt Anhorn
title: Blazor Server - Get Access Token for User 
categories: [blog, blazor]
tags: [Blazor, .NET, C#, Wasm]
---

In this article I show, using Blazor Server, a few snippets to get the AccessToken for a logged in User. It is not as straight forward to get the AccessCode for a user, the snippets will be using an Authentication type of OpenID. I have included comments in the code below with helpful details that might be useful.

Checkout the <a href="/blog/blazor/2020/09/05/Blazor-Get-Access-Token-for-User.html">Blazor Wasm - Get Access Token for User</a> article with details on how this can be done for a Blazor Server Application!

## Blazor - Getting Access Token from AuthenticationStateProvider

**Startup.cs**
~~~ csharp
using System.IdentityModel.Tokens.Jwt;
using System.Security.Claims;
using System.Threading.Tasks;

namespace EventHorizon.Game.Client.Server
{
    public class Startup
    {
        public Startup(
            IConfiguration configuration
        )
        {
            Configuration = configuration;
        }

        public IConfiguration Configuration { get; }

        public void ConfigureServices(IServiceCollection services)
        {
            // ... Omitted for Brevity 

            // Authentication Services
            services.AddScoped<Authentication.AuthenticationService, IdentityAuthenticationService>()
                .AddAuthentication(options =>
                {
                    options.DefaultScheme = CookieAuthenticationDefaults.AuthenticationScheme;
                    options.DefaultChallengeScheme = OpenIdConnectDefaults.AuthenticationScheme;
                })
                .AddCookie(CookieAuthenticationDefaults.AuthenticationScheme, options =>
                {
                    options.ExpireTimeSpan = TimeSpan.FromMinutes(15);
                    options.SlidingExpiration = true;
                })
                .AddOpenIdConnect(OpenIdConnectDefaults.AuthenticationScheme, options =>
                {
                    // We pull the necessary configuration from the framework configuration.
                    // You will also noticed that we dont have to  use the SecretKey with this type of authentication.
                    options.Authority = Configuration["Auth:Authority"];
                    options.ResponseType = Configuration["Auth:ResponseType"];
                    var scopes = Configuration["Auth:Scope"].Split(" ");
                    options.Scope.Clear();
                    foreach (var scope in scopes)
                    {
                        options.Scope.Add(scope);
                    }

                    options.ClientId = Configuration["Auth:ClientId"];
                    options.SaveTokens = true;
                    options.GetClaimsFromUserInfoEndpoint = true;
                    // We are telling the authentication that the roles by this name will be used
                    // to display the name, and for checking roles.
                    options.TokenValidationParameters.NameClaimType = "name";
                    options.TokenValidationParameters.RoleClaimType = "role";

                    options.Events.OnUserInformationReceived += eventArgs =>
                    {
                        // We get the AccessToken from the ProtocolMessage.
                        // WARNING: This might change based on what type of Authentication Provider you are using
                        var accessToken = eventArgs.ProtocolMessage.AccessToken;
                        eventArgs.Principal.AddIdentity(new ClaimsIdentity(
                            new Claim[]
                            {
                                // Make note of the claim with the name "access_token"
                                // We will use it in an Authentication Service for look up.
                                new Claim("access_token", accessToken)
                            }
                        ));

                        // Here we take the accessToken and put all the claims into another
                        // Identity on the users Principal, giving us access to them when needed.
                        var jwtToken = new JwtSecurityToken(accessToken);
                        eventArgs.Principal.AddIdentity(new ClaimsIdentity(
                            jwtToken.Claims,
                            "jwt",
                            eventArgs.Options.TokenValidationParameters.NameClaimType,
                            eventArgs.Options.TokenValidationParameters.RoleClaimType
                        ));
                        return Task.CompletedTask;
                    };
                })
            ;
        }

        public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
        {
            // ... Omitted for Brevity 
            app.UseAuthentication();
            // ... Omitted for Brevity 
        }
    }
}
~~~

**IdentityAuthenticationService.cs**

The IdentityAuthenticationService is a nice abstraction I personally put on top of my User, to make it easier to consume and a central point in code that checks authentication for the User.
~~~ csharp

using Microsoft.AspNetCore.Components.Authorization;
using System.Linq;
using System.Threading.Tasks;

public class AuthUser
{
    public string AccessToken { get; set; }
    public AuthUserProfile Profile { get; set; }
}
public interface AuthenticationService
{
    bool IsAuthenticated { get; }
    AuthUser User { get; }
    string AccessToken { get; }

    Task Setup();
    string LoginUrl(
        string redirectUri = null
    );
    string LogoutUrl();
}
public class IdentityAuthenticationService : AuthenticationService
{
    private readonly AuthenticationStateProvider _authenticationStateProvider;

    public IdentityAuthenticationService(
        AuthenticationStateProvider authenticationStateProvider
    )
    {
        _authenticationStateProvider = authenticationStateProvider;
        // This brings me great *shame*, but it works.
        Setup().GetAwaiter().GetResult();
    }

    public bool IsAuthenticated { get; private set; } = false;
    public AuthUser User { get; private set; }
    public string AccessToken => User?.AccessToken ?? "";

    public async Task Setup()
    {
        var authState = await _authenticationStateProvider.GetAuthenticationStateAsync();
        IsAuthenticated = false;
        if (authState.User.Identity.IsAuthenticated)
        {
            // Here we grab some details we need from the AuthState User
            var inRole = authState.User.IsInRole("Admin");
            // Here is where we cache the AccessToke for later use.
            var accessToken = authState.User.Claims.Where(a => a.Type == "access_token").FirstOrDefault()?.Value;
            var name = authState.User.Claims.Where(a => a.Type == "name").FirstOrDefault()?.Value;
            var preferredUsername = authState.User.Claims.Where(a => a.Type == "preferred_username").FirstOrDefault()?.Value;
            
            User = new AuthUser
            {
                AccessToken = accessToken,
                Profile = new AuthUserProfile
                {
                    Name = name ?? preferredUsername,
                    PreferredUsername = preferredUsername
                }
            };
            IsAuthenticated = true;
        }
    }

    public string LoginUrl(
        string redirectUri
    ) => $"/Account/Login?redirectUri={redirectUri}";

    public string LogoutUrl() => "/Account/Logout";
}
~~~

**Index.razor**
~~~ html
@page "/"

<div>
    @AccessToken
</div>
<div>
    @(IsAuthenticated ? "User Authenticated" : "User NOT Authenticated")
</div>

~~~

**Index.razor.cs**
~~~ csharp

using System.Threading.Tasks;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Components;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;

[Authorize]
public partial class Index : ComponentBase
{
    [Inject]
    public AuthenticationService AuthenticationService { get; set; }

    public bool IsAuthenticated { get; set; }
    public string AccessToken { get; set; }

    protected override async Task OnInitializedAsync()
    {
        IsAuthenticated = AuthenticationService.IsAuthenticated;
        AccessToken = AuthenticationService.AccessToken;
    }
}
~~~