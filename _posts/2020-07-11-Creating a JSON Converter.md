---
layout: post
author: Cody Merritt Anhorn
title: .NET Core - System.Text.Json - Create a JsonConverter
categories: [blog]
tags: [.NET, C#, JSON]
---

This article I will be going over how to create a Json Converter than can be used to create a Write/Read to a single property. 

## Why

While creating a proxy layer to BabylonJS from Blazor I needed a way to create a light weight Proxy abstraction around the BabylonJS API. When moving details between the two layers I did not want to serialize all of the details about a proxy. So I created a JsonConverter that I would use to take only on the cache identifier and serialize it. And adding this converter to any proxy extended classes will make sure they are serialized as well.

Doing this allowed for me to create a very focused serialized object, and the converter can be assigned to the proxy object directly by my interop generator. When assigned to the proxy object and used as an argument in a method, it would get serialized as small as possible. This makes it very efficient when passing proxies between the Blazor and JavaScript layers.

## Example of JsonConverter

Below is a very simple JsonConverter and a usage of the converter.

~~~ csharp
using System;
using System.Text.Json;
using System.Text.Json.Serialization;

public class ProxyConverter
    : JsonConverter<Proxy>
{
    // Check to see if we are Assignable from an inherited class.
    public override bool CanConvert(Type typeToConvert) =>
        typeof(Proxy).IsAssignableFrom(typeToConvert);

    // Read the proxy guid and put into a Created instanced.  
    public override Proxy Read(
        ref Utf8JsonReader reader,
        Type typeToConvert,
        JsonSerializerOptions options
    )
    {
        var proxy = (Proxy)Activator.CreateInstance(typeToConvert);

        // Keep reading till we are all done
        while (reader.Read())
        {
            // Or we reach the end of the object
            if (reader.TokenType == JsonTokenType.EndObject)
            {
                return proxy;
            }

            // If it is a Property type we want to parse the value
            if (reader.TokenType == JsonTokenType.PropertyName)
            {
                var propertyName = reader.GetString();
                reader.Read();
                switch (propertyName)
                {
                    // The property is they property we are looking for
                    case "___guid":
                        // From the reader put the value on the Proxy
                        proxy.___guid = reader.GetString();
                        break;
                }
            }
        }
        throw new JsonException("___guid was not found");
    }

    // Write out the guid from the Proxy
    public override void Write(
        Utf8JsonWriter writer, 
        Proxy value, 
        JsonSerializerOptions options
    )
    {
        writer.WriteStartObject();

        // Since we only care about the ___guid value 
        // we can just write it directly to the JsonWriter.
        writer.WriteString(
            "___guid", 
            value.___guid
        );

        writer.WriteEndObject();
    }
}
~~~

~~~ csharp
using System.Text.Json.Serialization;
using EventHorizon.Blazor.Interop;

// The simple class that holds the identifier on the client.
// Can be inherited by any other class, 
// and converted has to be attributed to it as well.
[JsonConverter(typeof(ProxyConverter))]
public class Proxy 
{
    public string ___guid { get; set; }
}

[JsonConverter(typeof(ProxyConverter))]
public class SuperComplicatedProxy : Proxy 
{
    // Lots of complicated proxy/interop implementations logic
}
~~~