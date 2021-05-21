---
layout: post
author: Cody Merritt Anhorn
title: Blazor and SignalR
categories: [blog, blazor]
tags: [.NET, Blazor, C#, SignalR]
---

In this Article I go over how to connect out to a SignalR Server from Blazor Wasm, sneak peak its the same as the C# SignalR client.

## What is SignalR

> ASP.NET Core SignalR is an open-source library that simplifies adding real-time web functionality to apps. Real-time web functionality enables server-side code to push content to clients instantly. - <a href="https://docs.microsoft.com/en-us/aspnet/core/signalr/introduction?view=aspnetcore-5.0">SignalR Introduction</a>

This abstracts away all the complicated logic you would have to create your self to support real-time communication between all your connected users. SignalR does not just give you real-time communication, it also take care of choosing the best transport layer available to the client, has Authentication/Authorization built in, supports streaming data from the server, built in logging and diagnostics. SignalR comes is two parts, a server which hosts the connection points and logic, and a client which connects to the server to trigger logic and cross client communication. 

## What is ASP.NET Core Blazor

> Blazor is a framework for building interactive client-side web UI with .NET - <a href="https://docs.microsoft.com/en-us/aspnet/core/blazor/?view=aspnetcore-5.0">ASP.NET Core Blazor</a>

Blazor is built on top of Components, so having experience with other component based frameworks, like ReactJS or VueJS, should make the transition to Blazor easier. The components model also makes it easy to reuse rendering logic across your application, and with its use of the Razor markup reusing already acquired knowledge from prior ASP.NET building practices will make the transition to Blazor easy as well. In my personal opinion, Razor and Blazor feel more close to native HTML/JavaScript development than even the other Component based frameworks, since what you write in your HTML markup layer looks very similar to HTML you would write, with most of the Blazor functionality layered in with already existing attributes. The biggest gotcha in Blazor I would say is how to handle Forms, which can be different from what is normally done, we will touch on forms a little in this article.

## What are we doing

Here we will go over creating an application that takes care of CRUD operations on a user profile record, but using SignalR to keep all connected browser clients in sync and up-to-date on what the others are working on. Source snippets will be show in this article, and a fully functioning application on GitHub can be found on <a href="https://github.com/canhorn/EventHorizon.Blazor.UserManagement" title="A fully functioning example project using SignalR and Blazor to create a Real-Time Application.">canhorn/EventHorizon.Blazor.UserManagement</a> .

The <code>ManagementComponentBase</code> below is a nice abstraction that the pages from the project inherit from to help with the setup of the connection to the SignalR Hub <code>UserManagementHub</code>, which is also below.

**ManagementComponentBase**
~~~ csharp
/// <summary>
/// This is a Base component that allows for the easier setup and usage of the SignalR Connection.
/// Includes methods that can be used to call up to the SignalR Hub.
/// </summary>
public class ManagementComponentBase
    : ComponentBase,
    IAsyncDisposable
{
    [Inject]
    public IAccessTokenProvider TokenProvider { get; set; } = null!;
    [Inject]
    public AuthenticationStateProvider AuthenticationStateProvider { get; set; } = null!;
    [Inject]
    public NavigationManager NavigationManager { get; set; } = null!;

    protected string LoggedInUserId { get; private set; } = string.Empty;

    protected HubConnection? _hubConnection;

    protected async Task<bool> StartConnection()
    {
        // Here we are getting the Access Token for the current User
        // This is used by the Server Hub to validate that the user is Authorized.
        var accessTokenResult = await TokenProvider.RequestAccessToken();
        if (accessTokenResult.TryGetToken(
            out var accessToken
        ))
        {
            var auth = await AuthenticationStateProvider.GetAuthenticationStateAsync();
            var sub = auth.User.Claims.FirstOrDefault(a => a.Type == "sub");

            // Track the Logged In User Id.
            // This way management pages can know who is editing the Users.
            LoggedInUserId = sub?.Value ?? string.Empty;

            _hubConnection = new HubConnectionBuilder()
                .WithUrl(
                    // Here we assume that our Hub is also hosted from our current client domain
                    $"{NavigationManager.BaseUri}user-management",
                    options =>
                    {
                        // We use our AccessToken for Authentication
                        // This also has the added benefit, on the server, to validate reqeuests.
                        options.AccessTokenProvider = () => Task.FromResult(
                            accessToken.Value
                        );
                    }
                ).Build();

            // Start the connection!
            // So when we return the caller can setup any On Actions.
            await _hubConnection.StartAsync();

            return true;
        }

        return false;
    }

    public async ValueTask DisposeAsync()
    {
        // This is wired into the Blazor runtime management,
        // it will cleanup our open connection and remove any handlers.
        if (_hubConnection != null)
        {
            await _hubConnection.DisposeAsync();
        }
    }

    #region Implementation of Hub Methods
    protected async Task<UserProfileModel> ReadUser(
        string id
    )
    {
        return await _hubConnection.InvokeAsync<UserProfileModel>(
            "ReadUser",
            id
        );
    }

    protected async Task<EditUserModel> ReadEditUser(
        string id
    )
    {
        return await _hubConnection.InvokeAsync<EditUserModel>(
            "ReadEditUser",
            id
        );
    }

    protected async Task<bool> UpdateUser(
        EditUserModel updatedUser
    )
    {
        return await _hubConnection.InvokeAsync<bool>(
            "UpdateUser",
            updatedUser
        );
    }

    protected async Task<bool> DeleteUser(
        string userId
    )
    {
        return await _hubConnection.InvokeAsync<bool>(
            "DeleteUser",
            userId
        );
    }

    protected async Task<IEnumerable<UserProfileModel>> FetchAll()
    {
        return await _hubConnection.InvokeAsync<IEnumerable<UserProfileModel>>(
            "Fetch"
        );
    }

    protected async Task<bool> CreateNewUser(
        NewUserModel user
    )
    {
        return await _hubConnection.InvokeAsync<bool>(
            "CreateNewUser",
            user
        );
    }
    #endregion
}
~~~

**UserManagementHub**
~~~ csharp
[Authorize]
public class UserManagementHub
    : Hub
{
    private readonly UserManager<ApplicationUser> userManager;

    public UserManagementHub(
        UserManager<ApplicationUser> userManager
    )
    {
        this.userManager = userManager;
    }

    public IEnumerable<UserProfileModel> Fetch()
    {
        var userList = userManager.Users.ToList();
        return userList.Select(
            a => new UserProfileModel
            {
                UserId = a.Id,
                Email = a.Email,
                FirstName = a.FirstName,
                LastName = a.LastName,
            }
        );
    }

    public UserProfileModel ReadUser(
        string id
    )
    {
        return userManager.Users.Where(
            a => a.Id == id
        ).Select(
            a => new UserProfileModel
            {
                UserId = a.Id,
                Email = a.Email,
                FirstName = a.FirstName,
                LastName = a.LastName,
            }
        ).FirstOrDefault();
    }

    public EditUserModel ReadEditUser(
        string id
    )
    {
        return userManager.Users.Where(
            a => a.Id == id
        ).Select(
            a => new EditUserModel
            {
                UserId = a.Id,
                Email = a.Email,
                FirstName = a.FirstName,
                LastName = a.LastName,
            }
        ).FirstOrDefault();
    }

    public async Task<bool> UpdateUser(
        EditUserModel user
    )
    {
        var userToUpdate = userManager.Users.FirstOrDefault(
            a => a.Id == user.UserId
        );

        if (userToUpdate is null)
        {
            return false;
        }

        userToUpdate.FirstName = user.FirstName;
        userToUpdate.LastName = user.LastName;

        var result = await userManager.UpdateAsync(
            userToUpdate
        );

        if (result.Succeeded)
        {
            await Clients.All.SendAsync("Changed");
        }

        return result.Succeeded;
    }


    public async Task<bool> CreateNewUser(
        NewUserModel user
    )
    {
        var result = await userManager.CreateAsync(
            new ApplicationUser
            {
                Email = user.Email,
                UserName = user.Email,
                FirstName = user.FirstName,
                LastName = user.LastName,
            }
        );

        if (result.Succeeded)
        {
            await Clients.All.SendAsync("Changed");
        }

        return result.Succeeded;
    }

    public async Task<bool> DeleteUser(
        string userId
    )
    {
        var userToDelete = userManager.Users.FirstOrDefault(
            a => a.Id == userId
        );

        if (userToDelete is null)
        {
            return true;
        }

        var result = await userManager.DeleteAsync(
            userToDelete
        );

        if (result.Succeeded)
        {
            await Clients.All.SendAsync("Changed");
        }

        return result.Succeeded;
    }
}
~~~
