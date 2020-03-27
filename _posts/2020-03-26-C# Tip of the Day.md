---
layout: post
author: Cody Merritt Anhorn
title: C# Tip of the Day - C# const - 2020.03.26
categories: [blog, dotnet]
tags: [.NET, C#, const, Tip of the Day]
---

In C# their is the 'const' keyword, this little guy does just that makes a field constant to the application. But one thing I learned today is that these fields can be baked into your application source.

This is a performance optimization the compiler takes care of for you, and something to watch out for when you use dynamically loaded or drop in libraries you are using.

Great reference to Shiv Kumar talk on YouTube going over this in more detail: <a href="https://youtu.be/OrpPfOu4PQ0" target="_blank">So You Think You Know C#? Constants & Default Parameter Values</a>.

## Steps to Reproduce

***-*** Compile your Console Application

***-*** Make a change to your 'const' in the Class Library project.

***-*** Compile only the Class Library project

***-*** Move the Compiled Class Library files into the run directory of the Console Application, replacing the already existing.

***-*** What this can show is that your Application, when run, will still use the old value.

Below is an example of the IL generated for the Console Application, showing that the compiler will bake in the string from the const right into the call site of the code. So if your using constants you need to make sure that you have the ability to compile anew where it is used. I have not ran into this personally, yet, but this does help to get a better understanding of what power the complier has over what you code.

![This image shows that the IL generated will bake a const into a call site of code.](/image\Posts\CSharp\2020-03-26\Generated_IL.png)

## Project Example

Below is an example project I created that can be used to simulate this, checkout the README in the project for details on how to run this.

Here is project:
<a href="https://github.com/canhorn/EventHorizon.CSharp.Constants" target="_blank">EventHorizon.CSharp.Constants</a>

### Console Project

***Program.cs***
~~~ csharp
public class Program
{
	public static void Main()
	{
		Console.WriteLine(LibraryExposed.ConstantValue);
	}
}
~~~

### Class Library Project

***LibraryExposed.cs***
~~~ csharp
public class LibraryExposed
{
    public const string ConstantValue = "Hello World.";
}
~~~