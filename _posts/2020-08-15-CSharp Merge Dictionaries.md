---
layout: post
author: Cody Merritt Anhorn
title: C# Merge Dictionaries
categories: [blog]
tags: [.NET Core, C#]
---

While working on my Game client conversion, which is why I have not posted many articles for a while, I found a need to merge two dictionaries into a single dictionary. Its not a real complicated process, but I all the examples I found would alway use the item from the first dictionary or would throw exceptions on a duplicate. To get around these two scenarios I created the **MergeLayoutControlOptions** method below, it takes in a list of Dictionary typed objects and merges them with the last grouped value taking priority in the new Dictionary.

To give a real world scenario I am using a class that extends the framework Dictionary. I like this pattern since it allows me to create explicit wrappers around a Dictionary, but still get the functionality of the framework provided Dictionary. Using this pattern also allows me to add functionality to the way a Dictionary works without having to create extensions methods, that make it hard to test. Another added benefit is that I also get a typed object for my options instead of a generic dictionary that I have to know I have extension methods to work with it.

## Example of Generated Source

You can checkout this fiddle <a href="https://dotnetfiddle.net/Jfrjtq" target="_blank">Merge Dictionaries</a> to play around with the code that is provided below.

Example output:
~~~
Key: key1 | Value: value1
Key: key2 | Value: value2-from2
Key: key3 | Value: value3
Key: key4 | Value: value4-from2
Key: key5 | Value: value5
~~~

~~~ csharp
using System;
using System.Linq;
using System.Collections.Generic;
					
public class Program
{
	public static void Main()
	{
		var options1 = new GuiControlOptionsModel(
			new Dictionary<string, object>
			{
				{"key1", "value1"},
				{"key2", "value2"},
				{"key3", "value3"},
				{"key4", "value4"},
				{"key5", "value5"},
			}
		);
		var options2 = new GuiControlOptionsModel(
			new Dictionary<string, object>
			{
				{"key2", "value2-from2"},
				{"key4", "value4-from2"},
			}
		);

		var mergedOptions = MergeLayoutControlOptions(
			options1,
			options2
		);

		foreach (var option in mergedOptions)
		{
			System.Console.WriteLine(
				$"Key: {option.Key} | Value: {option.Value}"
			);
		}
	}

    
    /// <summary>
    /// Will merge the provided options into a single options.
    /// Will use the last option value as priority in new Option Model.
    /// </summary>
    /// <param name="options">Options to merge values from.</param>
    /// <returns>A consolidated Options Model of all passed in Option Key/Value</returns>
	private static GuiControlOptionsModel MergeLayoutControlOptions(
		params IGuiControlOptions[] options
	)
	{
		return new GuiControlOptionsModel(
			options.SelectMany(x => x)
				.GroupBy(d => d.Key)
				.ToDictionary(x => x.Key, y => y.Last().Value)
		);
	}

	public interface IGuiControlOptions
		: IDictionary<string, object>
	{

	}

	public class GuiControlOptionsModel
		: Dictionary<string, object>,
		IGuiControlOptions
	{
		public GuiControlOptionsModel(
			IDictionary<string, object> dictionary
		) : base(dictionary ?? new Dictionary<string, object>())
		{
		}
	}
}
~~~