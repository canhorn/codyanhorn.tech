---
layout: post
author: Cody Merritt Anhorn
title: Blazor Server as Desktop Application
categories: [blog, blazor]
tags: [.NET, C#, Blazor]
---

In this post we will go through my personal code used to create an Desktop Application. Using the WebWindow and an ASP.NET Core Blazor Server I can create a Desktop application that looks and behaves like a native application. Using this setup I am able to use my knowledge of website creation to expedite my creation of a desktop application, since I find it easier to think of my desktop applications in the manner.

The main reason for developing my application in this way is to keep my technology stack simplified and segregated. I am able to encapsulate my different stacks of my application into discrete areas, like Desktop Client (the renderer), Client Server (the client page/composer), and Asset Server (static resources). Breaking up the Desktop Application into a single project, it allows me to only have to worry about it as a renderer for a client. 

## Desktop Client

The Desktop client is composed in a way that allows for it to host its own client server or connect up to a publicly hosted Client Server. Below is a pretty close example of how I currently structure my Desktop Client program file. The Program file is broken up into three distinct parts; generating configuration from a desktopsettings.json file, starting up the Client server if configuration does not contain a ServerUrl, and the actual starting of the WebWindow and pointing it to the ServerUrl.

Using the package referenced Client Server, this application is able to startup a locally hosted server, but have it confined to a desktop window instead of accessing it from the local browser. 

## Technology Used

Here is the list of tech used, is mostly built on the .NET Core framework.

***-*** ASP.NET Core

***-*** Blazor

***-*** WebWindow

## Example Project

Below is an example Program.cs file that can be used as a reference, if you want a more formalized working example checkout out this repository: 
<a href="https://github.com/canhorn/EventHorizon.Desktop.Blazor" target="_blank">EventHorizon.Desktop.Blazor</a>

***Desktop Client Program.cs***
~~~ csharp
using EventHorizon.Client.Desktop.Configuration;
using EventHorizon.Client.Server;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.Hosting;
using System;
using System.Diagnostics;
using System.IO;
using System.Net;
using System.Net.Sockets;
using System.Threading.Tasks;
using WebWindows;

namespace EventHorizon.Client.Desktop
{
    class Program
    {
        static IHost host;
        static void Main(string[] args)
        {
            Console.WriteLine("Main Window Startup");
            // This builds the configuration 
            var configuration = ApplicationConfiguration.Build();

            var serverUrl = configuration["ServerUrl"];
            // Check ServerUrl in Configuration
            if (string.IsNullOrEmpty(serverUrl))
            {
                // No Configured ServerUrl, create one.
                serverUrl = "http://localhost:8080";
                StartServer(
                    args,
                    serverUrl,
                    configuration
                );
            }

            // Build WebWindow with Configuration Title
            var window = new WebWindow(
                configuration["Title"]
            );
            // Point the WebWindow at the ServerUrl
            window.NavigateToUrl(
                serverUrl
            );
            // Stop the current thread, waiting on Window to be closed/exit.
            window.WaitForExit();
            host?.Dispose();

            Console.WriteLine("Main Window Closed.");
        }

        private static void StartServer(
            string[] args,
            string serverUrl,
            IConfigurationRoot configuration
        )
        {
            // Start Server in background thread.
            // This way it does not stop the rest of the application from starting/showing window.
            Task.Run(() =>
            {
                var pathToExe = Process.GetCurrentProcess().MainModule.FileName;
                var pathToContentRoot = Path.GetDirectoryName(pathToExe);
                Console.WriteLine(pathToExe);
                Console.WriteLine(pathToContentRoot);

                Array.Resize(ref args, args.Length + 1);
                // This takes the configuration Environment, can be used for debugging.
                args[args.GetUpperBound(0)] = $"--environment={configuration["Environment"]}";
                Array.Resize(ref args, args.Length + 1);
                // Start the backend server with this ServerUrl 
                args[args.GetUpperBound(0)] = $"URLS={serverUrl}";

                host = CreateHostBuilder(args, pathToContentRoot).Build();
                host.Start();
                host.WaitForShutdown();
            });
        }

        public static IHostBuilder CreateHostBuilder(string[] args, string contentRoot) =>
            Host.CreateDefaultBuilder(args)
                .UseContentRoot(contentRoot)
                .ConfigureWebHostDefaults(webBuilder =>
                {
                    webBuilder.UseStartup<Startup>()
                        // Make sure to set the content root to this startup location.
                        // Might not be needed when no static resources are provided by the server.
                        .UseWebRoot(Path.Combine(contentRoot, "wwwroot"))
                        .UseStaticWebAssets()
                    ;
                });
    }
}
~~~