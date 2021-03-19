---
layout: post
author: Cody Merritt Anhorn
title: Display a Docker Build Version
categories: [blog]
tags: [Docker, .NET, Blazor, C#]
---

In this Article I will show one way to show the Build or Application version, passing it down through a Docker build.

## Details

While developing my applications for the Game Development Platform I wanted an easy to access the build version. Since all of my service applications use Docker as their main artifact deployment pattern, passing in a build-arg and setting an ENV var for the image. This allows for a variety of ways to access the Application Version, and make all of my Dockerfile's have the same structure for each application build.

I have included a sample application, using Blazor as the main hosting framework, that when built and packaged up will show the version that was passed in during the build.

## Example Project

Below are a few snippets of code from the project you can find at <a href="https://github.com/canhorn/EventHorizon.DeployDockerBuildVersion" target="_blank">canhorn/EventHorizon.DeployDockerBuildVersion</a>.


**Index.razor**<br />
<span style="font-size: 0.8rem">
    Since we are using an environment variable to hold the Application Version, we can read that from the standard IConfiguration setup provided by Blazor Server. Below we just inject the configuration, and ask for the <code>@Configuration["APPLICATION_VERSION"]</code>.
</span>
~~~ html
@page "/"
@using Microsoft.Extensions.Configuration

<h1>Hello, Version!</h1>

<h2>
    Application Version: @Configuration["APPLICATION_VERSION"]
</h2>

@code {
    [Inject]
    public IConfiguration Configuration { get; set; } = null!;
}
~~~

<br />

**Docker Build**<br />
<span style="font-size: 0.8rem">
    This command will use docker, pass in the <code>BUILD_VERSION</code> argument and tag with the same version to build a docker image.
</span>
~~~ bash
docker build --build-arg BUILD_VERSION=1.2.3-alpha -t docker-build-version:1.2.3-alpha .
~~~

<br />

**Dockerfile**<br />
<span style="font-size: 0.8rem">
    This is the Dockerfile used to set the environment variable **(APPLICATION_VERSION)** that will get picked up by the application.
</span>
~~~ dockerfile
# Stage 1 - Publish .NET Project
FROM mcr.microsoft.com/dotnet/sdk:5.0 AS dotnet-build
WORKDIR /source

COPY EventHorizon.DeployDockerBuildVersion.csproj ./

RUN dotnet restore ./EventHorizon.DeployDockerBuildVersion.csproj

COPY . .

RUN dotnet build -c Release --no-restore ./EventHorizon.DeployDockerBuildVersion.csproj

RUN dotnet publish --output /app/ --configuration Release --no-restore --no-build ./EventHorizon.DeployDockerBuildVersion.csproj

# Stage 2 - Runtime
FROM mcr.microsoft.com/dotnet/aspnet:5.0 AS runtime
ARG BUILD_VERSION=0.0.0-build
ENV APPLICATION_VERSION=$BUILD_VERSION

WORKDIR /app
COPY --from=dotnet-build /app .
ENTRYPOINT ["dotnet", "EventHorizon.DeployDockerBuildVersion.dll"]
~~~


