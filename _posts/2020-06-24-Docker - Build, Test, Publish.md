---
layout: post
author: Cody Merritt Anhorn
title: Docker - Build, Test, Publish
categories: [blog]
tags: [.NET, C#, Docker, NodeJS]
---

To simplify my deployment process I use Docker, not many of my project dont use docker in some way or another. I like the flexibility I get from Docker to create my deployment artifacts, and by using Docker I don't have to remember how my artifacts are created. This article is for documenting my standard .NET Core Dockerfile that I can easily reference in the future without remembering what project I have used this kind of Dockerfile in before.

## Dockerfile

This article will mostly be an example Dockerfile with comments in line, look for the "##" the code section below for an explanation of what the line is doing. The one thing to notice is that this Dockerfile is broken up into three stages, the building of the NodeJS/Client side libraries, the building, testing, and publishing of the .NET Core application, and lastly combining it all into a single image. The runtime image is a very small, only generated artifacts, without contain the source or any build generated artifacts.

~~~dockerfile
# Stage 1 - Build NodeJS Artifacts
FROM node AS node-build

# caches restore result by copying package.json file separately
WORKDIR /source/src/Servers/EventHorizon.Game.Client.Assets.Server/wwwroot

# Copy over the package and lock file for yarn
## This allows for caching of stages, for faster future builds.
COPY ./src/Servers/EventHorizon.Game.Client.Assets.Server/wwwroot/package.json .
COPY ./src/Servers/EventHorizon.Game.Client.Assets.Server/wwwroot/yarn.lock .
# Cache the node package installs stage for faster future builds.
RUN yarn install

# Copy rest of source into wwwroot folder for build
COPY ./src/Servers/EventHorizon.Game.Client.Assets.Server/wwwroot .

# Build the Client artifacts, based on any build script in the package.json
RUN yarn build


# Stage 2 - Build .NET Project
FROM mcr.microsoft.com/dotnet/core/sdk:3.1 AS dotnet-build
WORKDIR /source

COPY ./*.sln ./

# Copy the main source project files
## This takes the sources directories and pushes them into the WORKDIR folder.
## Since Docker does not keep directory structure the RUN line will move the files to the correct folder.
COPY src/Clients/*/*.csproj ./
## This command takes the .csproj file and puts it into the src/Clients/** sub directory with the csproj of the same name.
RUN for file in $(ls *.csproj); do mkdir -p src/Clients/${file%.*}/ && mv $file src/Clients/${file%.*}/; done
COPY src/Servers/*/*.csproj ./
RUN for file in $(ls *.csproj); do mkdir -p src/Servers/${file%.*}/ && mv $file src/Servers/${file%.*}/; done
COPY src/Shared/*/*.csproj ./
RUN for file in $(ls *.csproj); do mkdir -p src/Shared/${file%.*}/ && mv $file src/Shared/${file%.*}/; done

# Copy the test project files
COPY test/*/*.csproj ./
## This command takes the .csproj file and puts it into the test/** sub directory with the csproj of the same name.
RUN for file in $(ls *.csproj); do mkdir -p test/${file%.*}/ && mv $file test/${file%.*}/; done 

# Restore dotnet dependencies, 
## This is done here with files that readily change to get a cached stage for faster future builds.
RUN dotnet restore

# Copy over rest of source files
COPY ./test ./test
COPY ./src ./src

# Run build of root project
RUN dotnet build -c Release --no-restore ./src/Servers/EventHorizon.Game.Client.Assets.Server/EventHorizon.Game.Client.Assets.Server.csproj

# Test - if you want to test a specific project
RUN dotnet test --no-restore ./test/EventHorizon.Game.Client.Assets.Server.Tests/EventHorizon.Game.Client.Assets.Server.Tests.csproj

# Test - If you want to test the whole solution
RUN dotnet test

# Copy Generated JavaScript from node-build stage
COPY --from=node-build /source/src/Servers/EventHorizon.Game.Client.Assets.Server/wwwroot/js /source/src/Servers/EventHorizon.Game.Client.Assets.Server/wwwroot/js

# Generate Published application
## Puts the source in /app/ << will be needed when building the slimmed runtime image
RUN dotnet publish --output /app/ --configuration Release --no-restore ./src/Servers/EventHorizon.Game.Client.Assets.Server/EventHorizon.Game.Client.Assets.Server.csproj


# Stage 3 - Create Slim .NET Core Runtime Image
FROM mcr.microsoft.com/dotnet/core/aspnet:3.1 AS runtime
WORKDIR /app
## This step takes the /app folder from the dotnet-build stage and injects it into here.
## This helps to decrease the size of the image, but not including the source that was used by the build stage's.
COPY --from=dotnet-build /app .

# This will run the dotnet command against the generated dll.
ENTRYPOINT ["dotnet", "EventHorizon.Game.Client.Assets.Server.dll"]
~~~