---
layout: post
author: Cody Merritt Anhorn
title: C# Tip of the Day 2020.01.15
categories: [blog, dotnet]
tags: [.NET, C#, Enumerable, Tip of the Day]
---

Want a quick and dirty way to randomly order a list in C#, use the OrderBy like below.

~~~ csharp
using System;
using System.Linq;
					
public class Program
{
	public static void Main()
	{
		var neatList = Enumerable.Range(1, 100).Select(
			o => new ComplexObject
			{
				Name = $"I am complex object {o}. ;)"
			}
		);
		var dirtyList = neatList.OrderBy(_ => Guid.NewGuid());
		Console.WriteLine("First in 'neatList': {0}", neatList.First().Name);
		Console.WriteLine("First in 'dirtyList': {0}", dirtyList.First().Name);
	}
	public class ComplexObject
	{
		public string Name { get; set; }
	}
}
~~~
