---
layout: post
author: Cody Merritt Anhorn
title: Adding Statically Typed Configuration to ASP.NET Core DI
---

This is a short and sweet post, it should give you a general idea on how to add typed Configuration/Settings to the .NET Core/ASP.NET Core Dependency Injection container.

## Setup 

The setup is broken up into 4 files, with minimal necessary logic. 

First is the configuration file appsettings.json, the only thing about this is that the object you want to make statically typed needs to be parsed from json and marshalled into a class.

Second is the actual typed representation of the configuration, the TwitchSettings.cs property names correlates to the appsettings.json properties of the Twitch object.

Third is to "Configure" the service layer of the ASP.NET Core application, this happens during startup in Startup.cs. The ***Configuration.GetSection*** parameter should point to the "Twitch" section in the appsettings.json file. 

And finally to get access to the configuration you can use constructor injection, like so ***IOptions<TwitchSettings> twitchSettings***. This give you access to the configuration, an example can be seen below in ***BotController.cs***.

## Example Code

### ***appsettings.json***
~~~ json
{
  "Twitch": {
    "Channel": "canhorn",
    "ClientId": "12345",
    "ClientSecret": "<secret>"
  }
}
~~~

### ***TwitchSettings.cs***
~~~ csharp
namespace EventHorizon.Platform.Data
{
    public class TwitchSettings
    {
        public string Channel { get; set; } = string.Empty;
        public string ClientId { get; set; } = string.Empty;
        public string ClientSecret { get; set; } = string.Empty;
    }
}
~~~

### ***Startup.cs***
~~~ csharp
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using EventHorizon.Platform.Data;

namespace EventHorizon.Platform
{
    public class Startup
    {
        public IConfiguration Configuration { get; }
        
        public Startup(IConfiguration configuration)
        {
            Configuration = configuration;
        }

        public void ConfigureServices(IServiceCollection services)
        {
            // ...
            // Here is the format to get it into the DI container.
            services.Configure<TwitchSettings>(Configuration.GetSection("Twitch"));
        }

        public void Configure(IApplicationBuilder app)
        {
            // ...
        }
    }
}
~~~

### ***BotController.cs***
~~~ csharp
using EventHorizon.Platform.Data;

namespace EventHorizon.Plaform.Bot 
{
    public class BotController
    {
        private readonly TwitchSettings _twitchSettings;

        // Here is how to get the value from DI
        public BotController(IOptions<TwitchSettings> twitchSettings)
        {
            _twitchSettings = twitchSettings.Value;
        }
        // ...
    }
}
~~~
