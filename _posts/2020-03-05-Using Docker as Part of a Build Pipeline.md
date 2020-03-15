---
layout: post
author: Cody Merritt Anhorn
title: Using Docker as Part of a Build Pipeline
categories: [blog, architecture]
tags: [docker, scss, nodejs]
---

Docker is a great technology, allowing for quick and easy development of software. Using Docker you can create a consistent deployment artifact, and in the post I will go over how I use Docker. To help simplify my build pipeline by running NodeJS build commands and running more complicated .NET script's.

## Simple SCSS Script

When working with my Blazor project's I still use CSS, to help simplify my development of this CSS I use SCSS. And if you know SCSS it needs to be complied down to CSS, to do that I use the <a href="https://www.npmjs.com/package/sass" title="NPM SASS package">sass</a> NPM package. To simplify the running the script to generate the CSS I used the NodeJS package runner, npx.

## Complex .NET Script

I will not go over the finer details of how the script works, but the scripts I have in place are used to take C# files and generate TypeScript representations of them. These scripts are designed to be run by the <a href="https://github.com/filipw/dotnet-script" title="GitHub repository page for dotnet-script">dotnet-script</a> tool ran with the .NET Core framework/environment.

## Using Docker

Using docker is not about creating a single image/container to do it all, it's about finding the best image for the job. In the case of the SCSS compiling, it was just using a node image. And for the dotnet scripts it is all about using a dotnet core sdk version that supports the necessary global tool, in this case 3.0+ was enough to handle running dotnet-script tool. 

To make this easier it to not use docker run directly, but to wrap the logic into a docker compose configuration. Below is a simple script but can very powerful if constructed correctly, reading the comments to get a better understanding of what can be done. 

~~~ yml
version: '3'
services:
    generate_css:
        # Uses the latest NodeJS version
        image: node:latest
        # This is an npx command using sass to take in a root scss file 
        #  then generating a site.css
        command: npx -q sass _site.scss wwwroot/css/site.css
        # This is were in the source code folder that the _site.scss is located
        working_dir: /app/server
        # Because we just mount all of the source code into the container we can run
        #  any script we would, this make it so any output will be included locally as well.
        volumes:
            - ./:/app:cached
            
    generate_files:
        image: mcr.microsoft.com/dotnet/core/sdk:3.0.100
        # This runs a script located in the working directory.
        # Doing it this way allows for things that don't fit on a single line, or complex setup that
        #  to be easily ran. 
        command: sh -c './run.sh'
        # Since the all the source code is mounted, running scripts that work against the whole code
        #  base can read and output changes as needed by the scripts.
        working_dir: /app/dotnet-scripts
        # Because we just mount all of the source code into the container we can run
        #  any script we would, this make it so any output will be included locally as well.
        volumes:
            - ./:/app:cached
~~~


