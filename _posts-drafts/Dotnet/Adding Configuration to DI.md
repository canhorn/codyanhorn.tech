---
layout: post
author: Cody Merritt Anhorn
title: Adding Configuration to ASP.NET Core DI
---

***appsettings.json***
~~~ json
{
  "Twitch": {
    "Channel": "canhorn",
    "ClientId": "12345",
    "ClientSecret": "<secret>"
  }
}
~~~

***TwitchSettings.cs***
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

***Startup.cs***
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
