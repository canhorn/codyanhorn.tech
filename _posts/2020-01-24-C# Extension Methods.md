---
layout: post
author: Cody Merritt Anhorn
title: C# Extension Methods
---

Use Extension methods to add functionality to already compiled/existing objects, doing so allows for simplified abstractions to be created for complex objects. This also allows for the ability to extend primitive types or framework provided objects.

Below is an extension method that does a null check, including a bonus attribute to help with fixing the false negatives in Visual Studio for CA1062.

~~~csharp
using System;

public static class ObjectExtensions
{
    public static void NullCheck(
        [ValidatedNotNull] this object objectToCheck,
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
[AttributeUsage(AttributeTargets.Parameter)]
internal sealed class ValidatedNotNullAttribute : Attribute
{
}
~~~
