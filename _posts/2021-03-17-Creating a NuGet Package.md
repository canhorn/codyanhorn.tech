---
layout: post
author: Cody Merritt Anhorn
title: Creating a NuGet Package
categories: [blog]
tags: [.NET, Blazor, C#, GitHub]
---


There are many ways to create a NuGet package, and in this article I will be using Github Actions to create the package, then publish it to NuGet.org.

## Background

While developing a Game Editor, built using Blazor Wasm, I wanted a way to easily create the interop layer from the C# to the BabylonJS WebGL render. Since BabylonJS is a massive game development framework in its own right I wanted a tool that I could use to easily create this abstraction. And with that I created the <a href="https://github.com/canhorn/EventHorizon.Blazor.TypeScript.Interop.Generator" target="_blank">canhorn/EventHorizon.Blazor.TypeScript.Interop.Generator</a> project, that bundles up NuGet packages and publishes a dotnet tool that I can use from the command line to generate my Interop abstraction.

As part of this I needed a way to easily create artifacts for NuGet, using GitHub Actions made this easy, and with the tools provided by the Marketplace I was also able to mostly automate the versioning and tagging of the project. In the <a href="#github-action-example">GitHub Action Example</a> section you can see the workflow file, but you can also see in the repository <a href="https://github.com/canhorn/EventHorizon.Blazor.TypeScript.Interop.Generator" target="_blank">canhorn/EventHorizon.Blazor.TypeScript.Interop.Generator</a> a working example with all the moving parts of Building, Publishing, Versioning, and Tagging of .NET artifacts.

## GitHub Action Example

Here we have a GitHub Action for generating a version based on the git tags, using GitVersion. We tell GitVersion to update the AssemblyInfo files of the projects, this embeds the into the generate .NET artifacts, GitVersion also sets some steps variables that we can then use to pack our .NET Solution. Then using a little bash we query for all the .nupkg files and pipe them into the dotnet push, which using secrets on the repository we publish the packages to NuGet.org.

~~~ yml
name: EventHorizon NuGet Packages

on:
  push:
    branches: [ main ]
  pull_request:
    types: [opened, synchronize]
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
      with:
        fetch-depth: '0'
    - name: Install GitVersion
      uses: gittools/actions/gitversion/setup@v0.9.6
      with:
        versionSpec: '5.x'
    - name: Use GitVersion
      id: gitversion
      uses: gittools/actions/gitversion/execute@v0.9.6
      with:
        additionalArguments: '/updateAssemblyInfo'
    - run: |
        echo "NuGetVersionV2: ${{ steps.gitversion.outputs.NuGetVersionV2 }}"
    - name: Setup .NET Core
      uses: actions/setup-dotnet@v1
      with:
        dotnet-version: '5.0.100-rc.2.20479.15'
    - name: Build with dotnet
      run: dotnet build --configuration Release
    - name: Pack with dotnet
      run: dotnet pack Blazor.Solution.sln --output nuget-packages --configuration Release -p:PackageVersion=${{ steps.gitversion.outputs.NuGetVersionV2 }}
    - name: Push with dotnet
      run: find nuget-packages -name '*.nupkg' | xargs -i dotnet nuget push --api-key ${{ secrets.NUGET_API_KEY }} --source https://api.nuget.org/v3/index.json {}
~~~