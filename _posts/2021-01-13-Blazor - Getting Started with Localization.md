---
layout: post
author: Cody Merritt Anhorn
title: Blazor - Getting Started with Localization
categories: [blog]
tags: [.NET, C#, Database, Dapper]
---

This will be a quick article just stating the details needed to get a fresh ASP.NET Core Blazor application up and running with localization, and pointing out one main pitfall that is easy to overlook when setting up Localization. 

## Why

When working on setting up Localization for Blazor project I was working on and trying to get the Localization working I ran into an issue. The issue I was running into was not a bug, but in the way the configuration works and it was just not clicking with what I was doing wrong. The issue was with the way that the localization .resx files are loaded, and I was not correlating the namespace to the location that the files will be loaded from.

### Example

Below is the standard Startup setup I landed on that makes it easy to get most Blazor applications setup with localization. The issue I had was with not putting my .resx into the namespace the correlated to the location of the generic injection.

~~~ csharp
// Here is an example of the DI
IStringLocalizer<EventHorizon.Platform.Docs.Localization.SharedResource> Localizer;
~~~

In the case of the example, you would not just put the <code>SharedResource.resx</code> into the <code>~/Resources</code> root folder, you need to put it into the <code>~/Resources/Localization</code>, this way matching the <code>~/Localization/SharedResource.cs</code> location/namespace.

The Startup.cs, this is just the standard localization setup, the spot to point out is the <code>options.ResourcesPath = "Resources"</code>. This the root folder of where we you want to put the .resx files that will be used for holding your value translations.
~~~ csharp
namespace EventHorizon.Platform.Docs
{
    using Microsoft.AspNetCore.Builder;
    using Microsoft.AspNetCore.Hosting;
    using Microsoft.AspNetCore.Localization;
    using Microsoft.Extensions.DependencyInjection;
    using Microsoft.Extensions.Hosting;
    using System.Collections.Generic;
    using System.Globalization;

    public class Startup
    {
        // omitted for brevity

        public void ConfigureServices(
            IServiceCollection services
        )
        {
            // omitted for brevity

            // I18n Services
            services
                .AddLocalization(options => options.ResourcesPath = "Resources")
                .Configure<RequestLocalizationOptions>(
                    opts =>
                    {
                        var supportedCultures = new List<CultureInfo>
                        {
                            // Set Supported Locales
                            new CultureInfo("en-US"),
                        };

                        opts.DefaultRequestCulture = new RequestCulture("en-US");
                        // Formatting numbers, dates, etc.
                        opts.SupportedCultures = supportedCultures;
                        // UI strings that we have localized.
                        opts.SupportedUICultures = supportedCultures;
                    }
                );
        }

        // omitted for brevity

    }
}
~~~
