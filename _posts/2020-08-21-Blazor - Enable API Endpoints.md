---
layout: post
author: Cody Merritt Anhorn
title: Blazor - Enable API Endpoints
categories: [blog, blazor]
tags: [.NET Core, C#, Blazor]
---

Just a quick post showing how to enable API controllers in a Blazor server application. I noticed, for performance, the default templates I used does not automatically wire up API controllers that are in your application. To enable API controllers you just need to add to the endpoints mapping, <code>endpoints.MapControllers();</code>.

## Example Source Code

**Startup.cs for project**
~~~ csharp
public class Startup
{
    // ... Standard Startup Methods

    public void Configure(IApplicationBuilder app)
    {
        // ... Standard Startup.Configure Method

        app.UseEndpoints(endpoints =>
        {
            endpoints.MapBlazorHub();
            // Here is the mapping of Controllers, which includes our API Controller
            endpoints.MapControllers();
            endpoints.MapFallbackToPage("/_Host");
        });
    }
}
~~~

**ValuesApiController.cs**
~~~ csharp
using System.Collections.Generic;
using Microsoft.AspNetCore.Mvc;

[Route("api/values")]
[ApiController]
public class ValuesApiController : ControllerBase
{
    [HttpGet]
    [Route("")]
    public IActionResult Values()
    {
        return Ok(
            new ValuesResult
            {
                Values = new List<string>
                { 
                    "value1",
                    "value2",
                    "value3",
                    "value4",
                }
            }
        );
    }
}
public class ValuesResult
{
    public IList<string> Values { get; set; }
}
~~~