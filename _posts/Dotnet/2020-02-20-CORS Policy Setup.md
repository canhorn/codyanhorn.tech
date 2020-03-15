---
layout: post
author: Cody Merritt Anhorn
title: ASP.NET Core CORS Policy Setup
categories: [blog, dotnet]
tags: [.NET, C#, CORS]
---

With ASP.NET Core it has become quite easy to setup CORS for a server. Using the built in libraries we are able with a few lines of code setup CORS and accept requests from a whitelist of Origins.

# How To

The way to setup CORS requires a few lines of code in the ConfigurationServices and Configure of your Startup file. Your are even able to setup multiple policies, which are configuration that dictate Method, Header, Origins, etc. Most of the time I just setup a single policy and use that at the global level for each request. But .NET allows for you to enable CORS at the Controller or Action layer using the EnableCors policy.

~~~ csharp
using Microsoft.AspNetCore.Builder;
using Microsoft.Extensions.DependencyInjection;

public void ConfigureServices(IServiceCollection services)
{
    // This will register a policy with your specified configuration
    services.AddCors(
        options => options.AddPolicy(
            "CorsPolicy",
            builder =>
            {
                builder.AllowAnyMethod()
                    .AllowAnyHeader()
                    .WithOrigins(
                        // I store my whitelist of origins in my appsettings.
                        // This allows for me to easily change this based on where the application is deployed.
                        GetOriginsFromConfiguration()
                    ).AllowCredentials();
            }
        )
    );
}

public void Configure(IApplicationBuilder app)
{
    // Make sure to use this so the next middleware chains are aware of the Forwarded headers.
    // Put this as high up your middleware chain as you can.
    // This will allow it to resolve the requests before assets are served.
    app.UseCors("CorsPolicy");
}
~~~

~~~ csharp
using Microsoft.AspNetCore.Cors;

// Using the EnableCors attribute you are able to setup a policy for specific Api or Action endpoint.
[EnableCors("PublicCorsPolicy")]
public class MyPublicApi
{
    
}
~~~