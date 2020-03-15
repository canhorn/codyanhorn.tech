---
layout: post
author: Cody Merritt Anhorn
title: Pickup a Proxies SSL Headers in an ASP.NET Core Hosted Container
categories: [blog, dotnet]
tags: [.NET, C#, Proxy]
---

One issue that is pretty tricky to setup is to correctly proxy SSL to a container. This issue usually does not affect much, just if the server should redirect to an https request, or in my case backend redirect URL generation.

## Background

The issue that I hit was that when making redirect requests to my Identity Server service, it would pass the redirect URI as http, even if the request was for an https request from the users point of view. Using the KnownNetworks and KnownProxies of the Forward Headers to configure I was able to get ASP.NET Core to correctly tell when a valid SSL request came in. 

## How To

This setup will require knowing the ProxyServer host IP address before hand. Below is the services configuration I use to setup the KnownNetworks and KnownProxies. It takes the ProxyServer from the Configuration, passed in by an Environment variable from the my docker compose.

~~~ csharp
public void ConfigureServices(IServiceCollection services)
{
    services.Configure<ForwardedHeadersOptions>(options =>
    {
        // Make sure to tell the the framework the ForwardHeaders to include.
        options.ForwardedHeaders = ForwardedHeaders.XForwardedFor | ForwardedHeaders.XForwardedProto;

        // This check for the existence of a ProxyServer Configuration property.
        if (!string.IsNullOrEmpty(Configuration["ProxyServer"]))
        {
            // This takes the proxy server and maps it into an IPv6 format.
            var proxyServerAddress = IPAddress.Parse(
                Configuration["ProxyServer"]
            ).MapToIPv6();
            options.KnownNetworks.Add(
                new IPNetwork(
                    proxyServerAddress,
                    104
                )
            );
            options.KnownProxies.Add(
                proxyServerAddress
            );
        }
    });
}

public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    // Make sure to use this so the next middleware chains are aware of the Forwarded headers.
    // Put this as high up your middleware chain as you can.
    app.UseForwardedHeaders();
}
~~~

## References

***--*** [docs.microsoft.com - Configure a Reverse Proxy Server](https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/linux-nginx?view=aspnetcore-3.1#configure-a-reverse-proxy-server)