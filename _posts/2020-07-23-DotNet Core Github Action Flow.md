---
layout: post
author: Cody Merritt Anhorn
title: DotNet Core Github Action Flow
categories: [blog]
tags: [.NET Core, Github, Azure, NuGet, Blazor, GitVersion]
---

To expand on my last article creating a quick GitHub action to build/test/deploy a .NET Core application. In this article I will go over the flow and action I created for my projects. You can jump right to the source project here: <a href="https://github.com/canhorn/EHZ_PlayGround" target="_blank">canhorn/EHZ_PlayGround</a>

## Details

While setting up my first NuGet published project, I wanted to setup an easy to use workflow that would be relatively automated using GitHub Actions. The trial and error I went through landed on a set of tools in the form of GitHub actions that will manage the versioning and publishing of my package. The flow I will be going over will include Build, Test, Deploy, and Publish, but it will also go over tagging the main branch in GitHub. By combining these these in a GitHub workflow I was able to create a flow that will take care of Publishing my package and correctly tagging a point in time on the repository.

The versioning is supplied by **GitVersion**, this tool will inspect your codebase and based on your configuration give you a wide range of properties you can use. Using GitVersion in a GitHub Action you get an automatic setup of environment variables you can use in the future steps. And since the triggers for these workflows are using the GitVersion action generation it will generate consistent version numbers. So the tagging of main can be a single workflow that only triggers on main pushes and then have another package deployment job done for both the main and pull request branches.

The static website for the repository will be using the Azure Static Web Apps (Preview as of writing this) to automatically publish a Blazor WASM static site. The workflow currently create a main branch website, this is like the "production" site for the repository. The workflow also takes into account pull requests and will publish your site into a staging environment specific for the pull request.

> One thing to note is since Azure Static Web Apps are still in preview, you can only have a single production and staging environment deployed. 

> So opening more than a single PR will have errors after the second is open.

## Necessary GitHub Setup

GitHub will need two secret keys for these workflows to function, the AZURE_STATIC_WEB_APPS_API_TOKEN_(...) and the NUGET_API_KEY, a third will need to be setup for private repositories CODECOV_TOKEN. The NuGet and Codecov API keys will need to be done at their respective sites, I will include a reference link to the GitHub actions that can be use to get more details about them. 

As for the Azure Static Web App API Token, that will be better to make sure you have at least an Azure subscription. Then from the generated website token you can replace <code>secrets.AZURE_STATIC_WEB_APPS_API_TOKEN</code> with our specific repositories secret key name.

> Make sure to delete the Microsoft generated workflow yml after you grab the generated secret key name.

## References

[codecov/codecov-action](https://github.com/codecov/codecov-action)

[Quickstart: Building your first static web app](https://docs.microsoft.com/en-us/azure/static-web-apps/getting-started)

[GitVersion Docs](https://gitversion.net/docs/)

[GitTools includes GitVersion](https://github.com/GitTools/actions)

## Features of the Workflows

The example I will be going over will have the following features:

**-** Build

**-** Test

**-** Collect Code Coverage

**-** Publish Code Coverage to Codecov

**-** Publish a Blazor WASM site to Azure Static Web Apps

**-** Website creation for main Branch and 1 Pull Request

**-** Cleanup Pull Request Static site when closed

**-** Configure and Version the commit using GitVersion

**-** Package a library for NuGet 

**-** Push the library to NuGet

**-** Create a Tab based on the GitVersion

## Necessary Files Explanation

### <a href="#dotnet-build-test-deployyml">dotnet-build-test-deploy.yml File</a>

This will Build, Test, Collect Coverage, Push Coverage Details to Codecov, Publish our Azure Static Web App, comment on our PRs with the Static site, cleanup closed Pull Requests. 

### <a href="#dotnet-packageyml">dotnet-package.yml File</a>

This will generate our version, package our .NET NuGet package, then publish that package to NuGet.

### <a href="#main-tag-bumpyml">main-tag-bump.yml File</a>

This is the GitVersion workflow that is responsible for tagging the main repository with your committed version. So as you merge in PR's or make commits against the main branch it will tag your repository with what that version would be based on the version that GitVersion gives.

### <a href="#gitversionyml">GitVersion.yml File</a>

This file contains the GitVersion configuration that will get picked up by the actions in the flow. You can edit this with anything you want, but I have included a default that formats my versions to my liking.

Version example of main Branch: ***0.1.0***

Version example of Pull Request Branch: ***0.1.1-alpha0001***

### <a href="#projectcsproj">Project.csproj File</a>

Shows tags that should be in your NuGet csproj file to get all the metadata that NuGet will used when displaying details about your package on the site.

### <a href="#routesjson">routes.json File</a>

This file just shows how the routes.json should be setup to correctly route the requests from paths in a Blazor WASM site deployed in Azure Static Web Apps. You can put this file right in your /WebSiteProject/wwwroot, so it will be published with the rest of your website.


## Example Project

I have a public GitHub project you see the files:
<a href="https://github.com/canhorn/EHZ_PlayGround" target="_blank">canhorn/EHZ_PlayGround</a>


## Example Files

### dotnet-build-test-deploy.yml
~~~ yml
name: EventHorizon .NET Core Build/Test/Publish

on:
  push:
    branches: [ main ]
  pull_request:
    types: [opened, synchronize, reopened, closed]
    branches: [ main ]

jobs:
  buildTestUpload:
    # This makes sure this job only runs on a push or if your PR is opened, synchronize, and reopened, NOT closed.
    if: github.event_name == 'push' || (github.event_name == 'pull_request' && github.event.action != 'closed')
    runs-on: ubuntu-latest
    steps:
    # Standard Check of the current git repository
    - uses: actions/checkout@v2
    # Install .NET Core version of 5.0.100-preview.7.20366.6
    - name: Setup .NET Core
      uses: actions/setup-dotnet@v1
      with:
        dotnet-version: '5.0.100-preview.7.20366.6'
    # Run dotnet restore to speed up build steps that would auto run it
    - name: Install dependencies
      run: dotnet restore
    # We want to build the Release version, to make sure it compiles.
    - name: Build
      run: dotnet build --configuration Release --no-restore
    # We want to test, 
    # Run with normal logging, so we get some nice debugging details on failure
    # Collect Code Coverage and in the form of opencover
    - name: Test
      run: dotnet test --no-restore --verbosity normal /p:CollectCoverage=true /p:CoverletOutputFormat=opencover
    # Using Codecov action published our Test Code Coverage, 
    # You don't need to supply a key if it is public.
    # If your not a public repository you will need to setup a secret key for Codecov
    - name: Upload Coverage to Codecov;
      uses: codecov/codecov-action@v1
      with:
        file: ./Projects/EventHorizon.Package.Test.Tests/coverage.opencover.xml
    # We .NET Core publish our Website project
    # This specific site will publish a Blazor WASM static website
    # We put the output into /published, will be used by the Publish Static Site setp
    - name: .NET Publish
      run: dotnet publish --configuration Release -o published ./Projects/EventHorizon.Test.Website 
    # Take our /published/wwwroot site content and publish into Azure
    - name: Publish Static Site
      id: builddeploy
      uses: Azure/static-web-apps-deploy@v0.0.1-preview
      with:
        # secrets.AZURE_STATIC_WEB_APPS_API_TOKEN will need to come from your setup in Azure
        azure_static_web_apps_api_token: ${{ secrets.AZURE_STATIC_WEB_APPS_API_TOKEN }}
        repo_token: ${{ secrets.GITHUB_TOKEN }} # Used for Github integrations (i.e. PR comments)
        action: "upload"
        ###### Repository/Build Configurations - These values can be configured to match you app requirements. ######
        # For more information regarding Static Web App workflow configurations, please visit: https://aka.ms/swaworkflowconfig
        app_location: "published/wwwroot" # App source code path
        app_artifact_location: "published/wwwroot" # Built app content directory - optional
        ###### End of Repository/Build Configurations ######

  close_pull_request_job:
    # This makes sure this job only runs when our PR's are closed.
    if: github.event_name == 'pull_request' && github.event.action == 'closed'
    runs-on: ubuntu-latest
    name: Close Pull Request Job
    steps:
    # This step makes sure our Azure Static Web App is removed for the PR.
    - name: Close Pull Request
      id: closepullrequest
      uses: Azure/static-web-apps-deploy@v0.0.1-preview
      with:
        azure_static_web_apps_api_token: ${{ secrets.AZURE_STATIC_WEB_APPS_API_TOKEN }}
        action: 'close'
~~~

### dotnet-package.yml
~~~ yml
name: EventHorizon Nuget Packages

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
    # Prune and unshallow Checkout of the current git repository
    - name: Fetch all history for all tags and branches
      run: git fetch --prune --unshallow
    # Install and Setup GitVersion version of 5.3.x
    - name: Install GitVersion
      uses: gittools/actions/gitversion/setup@v0.9.4
      with:
        versionSpec: '5.3.x'
    # Execute GitVersion against the current repository
    - name: Use GitVersion
      # Step id is used as reference for the output values
      id: gitversion 
      uses: gittools/actions/gitversion/execute@v0.9.4
    # Echo out the version, helps with quick debugging of the build without having to expand the prior step.
    - run: |
        echo "NuGetVersionV2: ${{ steps.gitversion.outputs.NuGetVersionV2 }}"
    # Install .NET Core version of 5.0.100-preview.7.20366.6
    - name: Setup .NET Core
      uses: actions/setup-dotnet@v1
      with:
        dotnet-version: '5.0.100-preview.7.20366.6'
    # Run dotnet build, with restore not disabled.
    - name: Build with dotnet
      run: dotnet build --configuration Release ./Projects/EventHorizon.Package.Test/EventHorizon.Package.Test.csproj
    # Run dotnet test against the Tests Project.
    # This helps to catch any failures that might be in PR's or main,
    # so we don't accidentally publish an untested package.
    - name: Test with dotnet
      run: dotnet test --no-restore --verbosity normal ./Projects/EventHorizon.Package.Test.Tests/EventHorizon.Package.Test.Tests.csproj
    # Pack up using dotnet, this will package up our NuGet package for upload.
    # Using the gitversion output we can set our PackageVersion.
    # We put the generated package into /nuget-packages, will be used in the push step.
    - name: Pack with dotnet
      run: dotnet pack Projects/EventHorizon.Package.Test/EventHorizon.Package.Test.csproj --output nuget-packages --configuration Release -p:PackageVersion=${{ steps.gitversion.outputs.NuGetVersionV2 }}
    # Take our /nuget-pacakges/* and push them to NuGet
    # Using the NUGET_API_KEY we push our package to
    - name: Push with dotnet
      run: dotnet nuget push nuget-packages/*.nupkg --api-key ${{ secrets.NUGET_API_KEY }} --source https://api.nuget.org/v3/index.json
~~~

### main-tag-bump.yml
~~~ yml
name: Bump main Tag version

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    # Standard Check of the current git repository
    - uses: actions/checkout@v2
      with:
        fetch-depth: '0'
    # Install and Setup GitVersion version of 5.3.x
    - name: Install GitVersion
      uses: gittools/actions/gitversion/setup@v0.9.4
      with:
        versionSpec: '5.3.x'
    # Execute GitVersion against the current repository
    - name: Use GitVersion
      # Step id is used as reference for the output values
      id: gitversion 
      uses: gittools/actions/gitversion/execute@v0.9.4
    # Echo out the version, helps with quick debugging of the build without having to expand the prior step.
    - name: Bump version and push tag
      uses: anothrNick/github-tag-action@1.17.2
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        # We want to tag in our repo the same version we publish to NuGet
        CUSTOM_TAG: ${{ steps.gitversion.outputs.NuGetVersionV2 }}
        RELEASE_BRANCHES: main
~~~

### GitVersion.yml
~~~ yml
mode: ContinuousDelivery
branches:
  master:
    regex: ^main
    tag: ''
    increment: Patch
    prevent-increment-of-merged-branch-version: true
    track-merge-target: false
    tracks-release-branches: false
    is-release-branch: true
  release:
    regex: ^releases?[/-]
    tag: beta
    increment: Patch
    prevent-increment-of-merged-branch-version: true
    track-merge-target: false
    tracks-release-branches: false
    is-release-branch: true
    pre-release-weight: 1000
  feature:
    regex: ^features?[/-]
    tag: 'alpha'
    increment: Inherit
    prevent-increment-of-merged-branch-version: false
    track-merge-target: false
    tracks-release-branches: false
    is-release-branch: false
  pull-request:
    regex: ^(pull|pull\-requests|pr)[/-]
    tag: alpha
    increment: Inherit
    prevent-increment-of-merged-branch-version: false
    tag-number-pattern: '[/-](?<number>\d+)[-/]'
    track-merge-target: false
    tracks-release-branches: false
    is-release-branch: false
  hotfix:
    regex: ^hotfix(es)?[/-]
    tag: beta
    increment: Patch
    prevent-increment-of-merged-branch-version: false
    track-merge-target: false
    tracks-release-branches: false
    is-release-branch: false
  support:
    regex: ^support[/-]
    tag: ''
    increment: Patch
    prevent-increment-of-merged-branch-version: true
    track-merge-target: false
    tracks-release-branches: false
    is-release-branch: false
  develop:
    regex: ^dev(elop)?(ment)?$
    tag: beta
    increment: Minor
    prevent-increment-of-merged-branch-version: false
    track-merge-target: true
    tracks-release-branches: true
    is-release-branch: false
ignore:
  sha: []
merge-message-formats: {}
~~~

### Project.csproj
~~~ xml
<Project Sdk="Microsoft.NET.Sdk">
    <PropertyGroup>
        <TargetFramework>net5.0</TargetFramework>

        <AssemblyTitle>EventHorizon Package Test</AssemblyTitle>
        <Description>Update description, this is just a test!</Description>
        <AssemblyName>EventHorizon.Package.Test</AssemblyName>
        <Authors>Cody Merritt Anhorn</Authors>
        <PackageId>EventHorizon.Package.Test</PackageId>
        <PackageTags>testing</PackageTags>
        <PackageProjectUrl>https://github.com/canhorn/EHZ_PlayGround</PackageProjectUrl>
        <RepositoryUrl>https://github.com/canhorn/EHZ_PlayGround</RepositoryUrl>
    </PropertyGroup>

</Project>
~~~

### routes.json
~~~ json
{
    "routes": [
        {
            "route": "/*",
            "serve": "/index.html",
            "statusCode": 200
        }
    ]
}
~~~