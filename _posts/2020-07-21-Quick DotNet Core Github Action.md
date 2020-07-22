---
layout: post
author: Cody Merritt Anhorn
title: Quick .NET Core Blazor Build/Test/Code Coverage/Publish/Deploy Github Action
categories: [blog]
tags: [.NET Core, Github, Blazor]
---

This article is just a quick snippet of code showing off a github action that will build a .NET Core application, test the application with code coverage, publish that code coverage to Codecov, publish a ASP.NET Core Blazor WASM application, and deploy the published Blazor application as an Azure Static Web App.

## Example YML

~~~ yml
name: .NET Core

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  buildTestUpload:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2
    - name: Setup .NET Core
      uses: actions/setup-dotnet@v1
      with:
        dotnet-version: 3.1.301
    - name: Install dependencies
      run: dotnet restore
    - name: Build
      run: dotnet build --configuration Release --no-restore
    - name: Test
      run: dotnet test --no-restore --verbosity normal /p:CollectCoverage=true /p:CoverletOutputFormat=opencover
    - name: Upload Coverage to Codecov
      uses: codecov/codecov-action@v1
      with:
        token: ${{ secrets.CODECOV_TOKEN }}
        file: ./Tests/Blazor.Tests/coverage.opencover.xml
    - name: .NET Publish
      run: dotnet publish --configuration Release -o published ./Source/Blazor.Website
    - name: Publish Static Site
      id: builddeploy
      uses: Azure/static-web-apps-deploy@v0.0.1-preview
      with:
        azure_static_web_apps_api_token: ${{ secrets.AZURE_STATIC_WEB_APPS_API_TOKEN }}
        action: "upload"
        app_location: "published/wwwroot" # App source code path
        app_artifact_location: "published/wwwroot" # Built app content directory - optional
~~~