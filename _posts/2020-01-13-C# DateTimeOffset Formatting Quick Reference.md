---
layout: post
author: Cody Merritt Anhorn
title: C# DateTimeOffset Formatting Quick Reference
---

This post is a quick reference for the DateTime/DateTimeOffset formatting. Using the .ToString you can pass in a string, using the below "Format" text, you can get example text below. You can mix and match the formats and create your own custom formatted dates.

Using <a href="https://dotnet.microsoft.com/platform/try-dotnet" target="_blank">Try .Net</a> take the below <a href="#code-for-reference">Code For Reference</a> to see and play with your self the text.

## Example Date

| Year | Month | Day | Hour | Minute | Second | Millisecond | Offset |
|---|---|---|---|---|---|---|---|
| 2011 | 6 | 10 | 15 | 24 | 16 | 213 | +6 |

## Example Formatted Date

| Format | | Formatted |
|---|---|---|
| ***%d*** | | 10 |
| ***dd*** | | 10 |
| ***ddd*** | | Fri |
| ***dddd*** | | Friday |
| ***%h*** | | 3 |
| ***hh*** | | 03 |
| ***%H*** | | 15 |
| ***HH*** | | 15 |
| ***%m*** | | 24 |
| ***mm*** | | 24 |
| ***%M*** | | 24 |
| ***MM*** | | 06 |
| ***MMM*** | | Jun |
| ***MMMM*** | | June |
| ***%s*** | | 16 |
| ***ss*** | | 16 |
| ***%t*** | | P |
| ***tt*** | | PM |
| ***%y*** | | 11 |
| ***yy*** | | 11 |
| ***yyy*** | | 2011 |
| ***yyyy*** | | 2011 |
| ***%K*** | | +06:00 |
| ***%z*** | | +6 |
| ***zz*** | | +06 |
| ***zzz*** | | +06:00 |
| ***%f*** | | 2 |
| ***ff*** | | 21 |
| ***fff*** | | 213 |
| ***ffff*** | | 2130 |
| ***fffff*** | | 21300 |
| ***ffffff*** | | 213000 |
| ***fffffff*** | | 2130000 |

## References

<a href="https://docs.microsoft.com/en-us/dotnet/standard/base-types/custom-date-and-time-format-strings">https://docs.microsoft.com/en-us/dotnet/standard/base-types/custom-date-and-time-format-strings</a>

## Code For Reference
~~~ csharp
DateTimeOffset exampleDate = new DateTimeOffset(
    2011,
    6,
    10,
    15,
    24,
    16,
    213,
    TimeSpan.FromHours(6)
); 

var formatString = "MM.dd-yyyy h:m:ss zzz";
var formatted = exampleDate.ToString(
    formatString
);
Console.WriteLine($"{formatString} >> {formatted}");
~~~

## Code Used to Generate Table

~~~ csharp
DateTimeOffset exampleDate = new DateTimeOffset(
    2011,
    6,
    10,
    15,
    24,
    16,
    213,
    TimeSpan.FromHours(6)
);

var formatters = new string[] {
    "%d",
    "dd",
    "ddd",
    "dddd",
    "%h",
    "hh",
    "%H",
    "HH",
    "%m",
    "mm",
    "%M",
    "MM",
    "MMM",
    "MMMM",
    "%s",
    "ss",
    "%t",
    "tt",
    "%y",
    "yy",
    "yyy",
    "yyyy",
    "%K",
    "%z",
    "zz",
    "zzz",
    "%f",
    "ff",
    "fff",
    "ffff",
    "fffff",
    "ffffff",
    "fffffff"
};

Console.WriteLine("| Year | Month | Day | Hour | Minute | Second | Millisecond | Offset |");
Console.WriteLine("|---|---|---|---|---|---|---|---|");
Console.WriteLine("| 2011 | 6 | 10 | 15 | 24 | 16 | 213 | +6 |");

Console.WriteLine();

Console.WriteLine("| Format | | Formatted |");
Console.WriteLine("|---|---|---|");

foreach (var formatString in formatters)
{
    // Here is the quick way to format a Date
    var formatted = exampleDate.ToString(
        formatString
    );
    Console.WriteLine($"| ***{formatString}*** | | {formatted} |");
}
~~~