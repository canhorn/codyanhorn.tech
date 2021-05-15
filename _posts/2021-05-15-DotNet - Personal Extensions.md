---
layout: post
author: Cody Merritt Anhorn
title: .NET - Personal Extensions
categories: [uses,personal]
tags: [.NET, C#, Personal Code]
---

This article I will show off some of my personal C# extension methods. Below I list out a bunch of code extensions I use extensively, ranging from Null checking to Task.FromResult wapping. These are personal to me so are built to synergies with my programming style and my quick code snippets.

<style>
    .table-of-contents__list {
        padding-left: 2rem;
    }
</style>

<ul class="table-of-contents__list">
    <li>
        <a href="#taskfromresult" title="Jump to the Task.FromResult">
            Task.FromResult
        </a>
    </li>
    <li>
        <a href="#to" title="Jump to the To">
            To
        </a>
    </li>
    <li>
        <a href="#null-check" title="Jump to the Null Check">
            Null Check
        </a>
    </li>
    <li>
        <a href="#isnull" title="Jump to the IsNull">
            IsNull
        </a>
    </li>
    <li>
        <a href="#isnotnull" title="Jump to the IsNotNull">
            IsNotNull
        </a>
    </li>
    <li>
        <a href="#istrue" title="Jump to the IsTrue">
            IsTrue
        </a>
    </li>
    <li>
        <a href="#isnottrue" title="Jump to the IsNotTrue">
            IsNotTrue
        </a>
    </li>
    <li>
        <a href="#tosha256hash" title="Jump to the ToSha256Hash">
            ToSha256Hash
        </a>
    </li>
    <li>
        <a href="#dictionarymerge" title="Jump to the Dictionary.Merge">
            Dictionary.Merge
        </a>
    </li>
    <li>
        <a href="#dictionaryaddrangifmissing" title="Jump to the Dictionary.AddRangeIfMissing">
            Dictionary.AddRangeIfMissing
        </a>
    </li>
    <li>
        <a href="#enumerableextensions" title="Jump to the EnumerableExtensions">
            EnumerableExtensions
        </a>
    </li>
    <li>
        <a href="#jsonextensions" title="Jump to the JsonExtensions">
            JsonExtensions
        </a>
    </li>
    <li>
        <a href="#optionextensions" title="Jump to the OptionExtensions">
            OptionExtensions
        </a>
    </li>
    <li>
        <a href="#stringbase64encode" title="Jump to the String.Base64Encode">
            String.Base64Encode
        </a>
    </li>
    <li>
        <a href="#stringbase64decode" title="Jump to the String.Base64Decode">
            String.Base64Decode
        </a>
    </li>
</ul>

## Task.FromResult

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

## To

I like this method because it allows me to easily pattern match cast an object. Also if it happens to be a raw JSON primitive it will cast it to a concrete object.

~~~ csharp
public static class ObjectExtensions 
{
    /// <summary>
    /// Converts the object into the Type Parameters, uses a combination of pattern matching and JSON de-serialization.
    /// Supported JSON Types: Newtonsoft.Json.Linq.JObject and System.Text.Json.JsonElement
    /// </summary>
    /// <typeparam name="T">The type this should transform to.</typeparam>
    /// <param name="objectToCast">The object to be converted to the Type Parameter.</param>
    /// <returns>The object casted to the type, can return a new object if is a raw Json Element.</returns>
    public static T To<T>(
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

## Null Check

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

## IsNull

My personal preference to make my code more readable.

~~~ csharp
    /// <summary>
    /// A nice readable way to write code that will check for a object of its null state, includes Code Analysis attributes to help with code flow validation.
    /// </summary>
    /// <param name="objectToCheck">The object that will be checked for null.</param>
    /// <returns>Is null check result of objectToCheck</returns>
    public static bool IsNull(
        [NotNullWhen(false)] this object? objectToCheck
    )
    {
        return objectToCheck == null;
    }
~~~

## IsNotNull

Same as <a href="isnull">IsNull</a>, but with logic inverted.

~~~ csharp
    /// <summary>
    /// A nice readable way to write code that will check an object of it null state, includes Code Analysis attribute to help with code flow validate.
    /// </summary>
    /// <param name="objectToCheck">The object that will bec checked for not null.</param>
    /// <returns>Is not null check result of objectToCheck</returns>
    public static bool IsNotNull(
        [NotNullWhen(true)] this object? objectToCheck
    )
    {
        return objectToCheck != null;
    }
~~~

## IsTrue

~~~ csharp
public static class BooleanExtensions
{
    /// <summary>
    /// This method is useful with Properties that are bool/Boolean values and you want to make them more readable.
    /// </summary>
    /// <param name="value">The value to check the truthy state of.</param>
    /// <returns>If the bool is true.</returns>
    public static bool IsTrue(
        this bool value
    )
    {
        return value;
    }
}
~~~

## IsNotTrue

~~~ csharp
public static class BooleanExtensions
{
    /// <summary>
    /// This method is useful with Properties that are bool/Boolean values and you want to make them more readable.
    /// This helps with writing code in that you do not have to remember to jump to the front to add '!' to invert the check for an if statement.
    /// </summary>
    /// <param name="value">The value to check the truthy state of.</param>
    /// <returns>If the state of bool is not true.</returns>
    public static bool IsNotTrue(
        this bool value
    )
    {
        return !value;
    }
}
~~~

## ToSha256Hash

~~~ csharp
public static class Sha256Extensions
{
    /// <summary>
    /// Turn the rawData into a SHA256 hashed string.
    /// </summary>
    /// <param name="rawData">The string to hash.</param>
    /// <returns>A SHA256 hashed string.</returns>
    public static string ToSha256Hash(
        this string rawData
    )
    {
        using var sha256Hash = SHA256.Create();
        var bytes = sha256Hash.ComputeHash(
            Encoding.UTF8.GetBytes(
                rawData
            )
        );

        var builder = new StringBuilder();
        for (int i = 0; i < bytes.Length; i++)
        {
            builder.Append(
                bytes[i].ToString(
                    "x2"
                )
            );
        }
        return builder.ToString();
    }
}
~~~

## Dictionary.Merge 

This is great in that it allows me to merge two separate dictionaries, and getting a new one out of it. This builds on top of the <a href="enumerableextensions">EnumerableExtensions</a> methods.

~~~ csharp
namespace System.Collections.Generic
{
    using System.Linq;

    public static class DictionaryExtensions
    {
        /// <summary>
        /// Take in 1..n number of matching dictionaries and merge them into the root.
        /// The order of the values matters, as matching values will be replaced with keys further into the list.
        /// </summary>
        /// <typeparam name="TKey">The type of the keys in the dictionary.</typeparam>
        /// <typeparam name="TValue">The type of values in the dictionary.</typeparam>
        /// <param name="root">Our root Dictionary that the values will start from.</param>
        /// <param name="values">An array of Dictionary values to merge together.</param>
        /// <returns>Returns a new Dictionary of the merged key/values.</returns>
        public static IDictionary<TKey, TValue> Merge<TKey, TValue>(
            this IDictionary<TKey, TValue> root,
            params IDictionary<TKey, TValue>[] values
        ) => values.InsertItem(
            0,
            root
        ).SelectMany(
            x => x
        ).GroupBy(
            d => d.Key
        ).ToDictionary(
            x => x.Key,
            y => y.Last().Value
        );
    }
}
~~~

## Dictionary.AddRangeIfMissing 
    
I use this one to add default values to a map that might of been passed in from outside my own code.

~~~ csharp
namespace System.Collections.Generic
{
    using System;

    public static class IDictionaryExtensions
    {
        /// <summary>
        /// Take in 1..n Key/ValueFactory parameters to add values to a Dictionary if the key does not exist.
        /// The factory makes it so the values will not be processed unless they are triggered.
        /// </summary>
        /// <param name="map">The Map to do an in-line update to.</param>
        /// <param name="valueBuilders">The array of key to valueFactories to check against passed in map</param>
        /// <returns>The map with the added range parameters.</returns>
        public static IDictionary<string, string> AddRangeIfMissing(
            this IDictionary<string, string> map,
            params (string Key, Func<string> ValueFactory)[] valueBuilders
        )
        {
            foreach (var (Key, ValueFactory) in valueBuilders)
            {
                if (!map.ContainsKey(
                    Key
                ))
                {
                    map[Key] = ValueFactory();
                }
            }
            return map;
        }
    }
}
~~~

## EnumerableExtensions

Below is a collection of helper function for IEnumerable collections management, it will unwrap the IEnumerable into a List and return readonly versions of the updated changed IEnumerable.

~~~ csharp
namespace System.Collections.Generic
{
    using System;
    using System.Linq;
    using System.Text;

    public static class EnumerableExtensions
    {
        /// <summary>
        /// Add an item to an Enumerable list and returns a new readonly version.
        /// </summary>
        /// <typeparam name="T">The type for the items in the enumerable.</typeparam>
        /// <param name="enumerable">The enumerable that the item will be added to.</param>
        /// <param name="item">The item to add.</param>
        /// <returns>A new ReadOnly Enumerable.</returns>
        public static IEnumerable<T> AddItem<T>(
            this IEnumerable<T> enumerable,
            T item
        )
        {
            var newList = enumerable.ToList();
            newList.Add(item);
            return newList.AsReadOnly();
        }

        /// <summary>
        /// Insert an item into a specific index of an Enumerable list and returns a new readonly version.
        /// </summary>
        /// <typeparam name="T">The type for the items in the enumerable.</typeparam>
        /// <param name="enumerable">The enumerable that the item will be inserted into.</param>
        /// <param name="index">The index location to insert at.</param>
        /// <param name="item">The item to insert.</param>
        /// <returns>A new ReadOnly Enumerable.</returns>
        public static IEnumerable<T> InsertItem<T>(
            this IEnumerable<T> enumerable,
            int index,
            T item
        )
        {
            var newList = enumerable.ToList();
            newList.Insert(index, item);
            return newList.AsReadOnly();
        }

        /// <summary>
        /// This will remove the item from the Enumerable.
        /// </summary>
        /// <typeparam name="T">The type for the items in the enumerable.</typeparam>
        /// <param name="enumerable">The enumerable that the item will be removed from.</param>
        /// <param name="item">The item to remove</param>
        /// <returns>A new ReadOnly Enumerable.</returns>
        public static IEnumerable<T> RemoveItem<T>(
            this IEnumerable<T> enumerable,
            T item
        )
        {
            var newList = enumerable.ToList();
            newList.Remove(item);
            return newList.AsReadOnly();
        }

        /// <summary>
        /// A predicate based lookup for an item in a IEnumerable.
        /// </summary>
        /// <typeparam name="TSource">The type for the items in the enumerable.</typeparam>
        /// <param name="enumerable">The enumerable that the item will be search in.</param>
        /// <param name="predicate">The function to test each element for a condition.</param>
        /// <param name="item">When this method returns, contains the value that satisfies the predicate; otherwise, the default for the type of the value parameter. This parameter is passed uninitialized.</param>
        /// <returns>true if the IEnumerable contains an element that satisfies the predicate; otherwise, false.</returns>
        public static bool TryGetItem<TSource>(
            this IEnumerable<TSource> enumerable,
            Func<TSource, bool> predicate,
            out TSource item
        ) => (item = enumerable.FirstOrDefault(
            predicate
        )).IsNotNull();
    }
}
~~~

## JsonExtensions

~~~ csharp
namespace System.Text.Json
{
    using System;
    using System.Buffers;
    using System.Diagnostics;

    public static partial class JsonExtensions
    {
        private readonly static JsonSerializerOptions DEFAULT_OPTIONS = new JsonSerializerOptions
        {
            PropertyNameCaseInsensitive = true,
        };

        /// <summary>
        /// Deserialize the passed in JsonElement into the typeparam.
        /// </summary>
        /// <typeparam name="T">The Type of object to marshal into.</typeparam>
        /// <param name="element">The JsonElement we wish to marshal.</param>
        /// <param name="options">Any options we want to override from the default.</param>
        /// <returns>A marshalled representation of the element.</returns>
        public static T ToObject<T>(
            this JsonElement element,
            JsonSerializerOptions? options = null
        )
        {
            var bufferWriter = new ArrayBufferWriter<byte>();
            using (var writer = new Utf8JsonWriter(
                bufferWriter
            ))
            {
                element.WriteTo(writer);
            }
            return JsonSerializer.Deserialize<T>(
                bufferWriter.WrittenSpan,
                options ?? DEFAULT_OPTIONS
            );
        }

        /// <summary>
        /// Deserialize the passed in JsonDocument into the typeparam.
        /// </summary>
        /// <typeparam name="T">The Type of object to marshal into.</typeparam>
        /// <param name="document">The JsonDocument we wish to marshal.</param>
        /// <param name="options">Any options we want to override from the default.</param>
        /// <returns>A marshalled representation of the document.</returns>
        public static T ToObject<T>(
            this JsonDocument document,
            JsonSerializerOptions? options = null
        )
        {
            if (document == null)
            {
                throw new ArgumentNullException(
                    nameof(document)
                );
            }
            return document.RootElement.ToObject<T>(
                options
            );
        }
    }
}
~~~

## OptionExtensions

This is a simple nullability abstraction I like to use to help with external services. This wraps any typed values, giving me a common API that I can use to check for Null values. The magic comes in with the <code>implicit operator</code> methods below, these allow for easier checking for null and wrapping of values.

~~~ csharp
using System;
using System.Collections.Generic;
using System.Diagnostics.CodeAnalysis;

[Serializable]
public struct Option<T>
    : IEquatable<Option<T>>
{
    private readonly bool _hasValue;
    private readonly T _value;

    public Option(T? value)
    {
        _hasValue = value != null;
        _value = value!;
    }

    public bool HasValue
    {
        get
        {
            return _hasValue;
        }
    }

    [NotNull]
    public T Value
    {
        get
        {
            if (!_hasValue)
            {
                throw new InvalidOperationException("Value is not present.");
            }
            return _value!;
        }
    }

    public static implicit operator Option<T>(
        T? result
    ) => new(
        result
    );

    public static implicit operator bool(
        Option<T> result
    ) => result.HasValue;

    #region Generated
    public override bool Equals(object obj)
    {
        return obj is Option<T> option && Equals(option);
    }

    public bool Equals(Option<T> other)
    {
        return _hasValue == other._hasValue &&
               EqualityComparer<T>.Default.Equals(_value, other._value);
    }

    public override int GetHashCode()
    {
        return HashCode.Combine(_hasValue, _value);
    }

    public static bool operator ==(Option<T> left, Option<T> right)
    {
        return left.Equals(right);
    }

    public static bool operator !=(Option<T> left, Option<T> right)
    {
        return !(left == right);
    }

    public Option<T> ToOption() => new(
        Value
    );

    public bool ToBoolean()
        => HasValue;
    #endregion
}
~~~

## String.Base64Encode

~~~ csharp
using System;
using System.Text;

public static class StringExtensions
{
    /// <summary>
    /// Take in a plain text string and Base64 encode the string.
    /// </summary>
    /// <param name="plainText">The text to encode.</param>
    /// <returns>A Base64 encoded representation of the text.</returns>
    public static string Base64Encode(
        this string plainText
    ) => Convert.ToBase64String(
        Encoding.UTF8.GetBytes(
            plainText
        )
    );
}
~~~

## String.Base64Decode

~~~ csharp
using System;
using System.Text;

public static class StringExtensions
{
    /// <summary>
    /// Take in a Base64 encoded string and return its decoded string.
    /// </summary>
    /// <param name="base64EncodedData">The text to decode.</param>
    /// <returns>A decoded Base64 representation of the data.</returns>
    public static string Base64Decode(
        this string base64EncodedData
    ) => Encoding.UTF8.GetString(
        Convert.FromBase64String(
            base64EncodedData
        )
    );
}
~~~
