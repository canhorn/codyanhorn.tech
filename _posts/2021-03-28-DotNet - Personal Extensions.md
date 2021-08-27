---
layout: post
author: Cody Merritt Anhorn
title: .NET - Personal Extensions
categories: [blog, personal]
tags: [.NET, C#, Personal Code]
---

This article I will show off some of my personal C# extension methods. Below I list out a bunch of code extensions I use extensively, ranging from Null checking to Task.FromResult wapping. These are personal to me so are built to synergies with my programming style and my quick code snippets.

## C# Code Snippets

### Task.FromResult

~~~ csharp
namespace System.Threading.Tasks
{
    public static class TaskExtensions 
    {
        /// <summary>
        /// This will turn the object into a Task using the Task.FromResult.
        /// </summary>
        /// <param name="obj">The Object you want to wrap.</param>
        /// <typeparam name="T">The Type of the obj.</typeparam>
        /// <returns>The Task.FromResult of obj.</returns>
        public static Task<T> FromResult<T>(
            this T obj
        )
        {
            return Task.FromResult<T>(
                obj
            );
        }
    }
}
~~~

### Cast

I like this method because it allows me to easily pattern match cast an object. Also if it happens to be a raw JSON primitive it will cast it to a concrete object.

~~~ csharp
public static class ObjectExtensions 
{
    /// <summary>
    /// Converts the object into the Type Parameters, uses a combination of pattern matching and JSON deserialization.
    /// Supported JSON Types: Newtonsoft.Json.Linq.JObject and System.Text.Json.JsonElement
    /// </summary>
    /// <typeparam name="T">The type this should transform to.</typeparam>
    /// <param name="objectToCast">The object to be converted to the Type Parameter.</param>
    /// <returns>The object casted to the type, can return a new object if is a raw Json Element.</returns>
    public static T Cast<T>(
        this object objectToCast
    )
    {
        if (objectToCast is T typedObject)
        {
            return typedObject;
        }
        else if (!typeof(T).IsInterface
            && objectToCast != null
            && objectToCast is Newtonsoft.Json.Linq.JObject jObject)
        {
            return jObject.ToObject<T>() ?? default;
        }
        else if (!typeof(T).IsInterface
            && objectToCast != null
            && objectToCast is System.Text.Json.JsonElement jsonElement)
        {
            return jsonElement.ToObject<T>() ?? default;
        }
        return (T)objectToCast;
    }
}
~~~

### Null Check

~~~ csharp
public static class ObjectExtensions 
{
    /// <summary>
    /// Throws an ArgumentNullException if null.
    /// </summary>
    /// <param name="objectToCheck">The object to check for null.</param>
    /// <param name="paramName">Optional parameter to pass into Exception if null.</param>
    public static void NullCheck(
        [System.Diagnostics.CodeAnalysis.NotNull] this object objectToCheck,
        string paramName = ""
    )
    {
        if (objectToCheck == null)
        {
            throw new System.ArgumentNullException(
                paramName
            );
        }
    }
}
~~~


- IsNull
  - Cuts down on spaces and = signs
- IsNotNull
- IsTrue
- ToSha256Hash
- Dictionary.Merge - This is great in that it allows me to merge two separate dictionaries, and getting a new one out of it.
- Dictionary.AddRangeIfMissing - I use this one to add default values to a map that might of been passed in from outside my own code.
- EnumerableExtensions
- JsonExtensions
- OptionExtensions
- String.Base64Encode
- String.Base64Decode