---
layout: post
author: Cody Merritt Anhorn
title: Blazor - EventHorizon UX Component Update
categories: [blog, blazor]
tags: [.NET 5.0, HTML, CSS]
---

With this article I will go over the recent updates to the <a href="https://github.com/canhorn/EventHorizon.Blazor.UX" target="_blank">EventHorizon Blazor UX Controls</a>. Checkout the deployed static website of the sample <a href="https://lively-mud-0597d4e10.azurestaticapps.net/" target="_blank">Blazor Wasm EventHorizon.Blazor.UX</a> using the updated controls.

## Update to EventHorizon UX Controls

### New Table Control

A new Table control was added to the components library, this will allow for creating a consistent look and feel for tables. And with the new Theming updates it will also make it easier to theme for both the developer and the user, more details in the <a href="#theming-support" title="Go checkout the Theming update for more details on what was updated">Theming Support</a> section for more details on what was updated and future plans for it.

**Table Example**<br />
<span style="font-size: 0.8rem">
    Here is a example of the new Table Control being used to display forecast information.
</span>
~~~ html
<Table>
    <Head>
        <TableColumn>Date</TableColumn>
        <TableColumn>Temp. (C)</TableColumn>
        <TableColumn>Temp. (F)</TableColumn>
        <TableColumn>Summary</TableColumn>
    </Head>
    <Body>
        @foreach (var forecast in forecasts)
        {
            <TableRow>
                <TableCell>@forecast.Date.ToShortDateString()</TableCell>
                <TableCell>@forecast.TemperatureC</TableCell>
                <TableCell>@forecast.TemperatureF</TableCell>
                <TableCell>@forecast.Summary</TableCell>
            </TableRow>
        }
    </Body>
</Table>
~~~

### Theming Support

The application now had the theming centralized to make it easier to globally change the look and feel of the controls. This was only done for the color's currently, more elements will be added to the base theme in the future as they are deemed necessary. The theme allows for two main areas of override, the base theme can be overridden by the <a href="#developer-theme-overriding">developer in the html setup</a> or by the developer using a Theme Provider in code. 

The theming support is built entirely on top of <a href="https://www.w3schools.com/css/css3_variables.asp" target="_blank" title="Checkout W3 Schools for more details on CSS Variables">CSS Variables</a> making it easier to manage and update themes.

#### Theme Provider Overriding

Since the theming support is built entirely on top of CSS Variables it makes it a little bit easier to create a Theme Provider that can be dynamically updated with some code. Using a Dictionary/Map of key/value pairs you are able to pass in overrides, that will be changed by the platform through JavaScript. Checkout the <a href="#current-list-of-theme-css-variables" title="Checkout the CSS Variables available for Override">current list of Theme CSS Variables</a> for a list of available override.

##### Theme Provider Overriding Example

Below is an example of using the ThemeProvider component provided by the library to pass in an override of Theme values.

~~~ html
<ThemeProvider Theme="DictionaryOfThemeOverrides">
    <Router AppAssembly="@typeof(Program).Assembly">
        <LayoutView Layout="@typeof(MainLayout)">
            ...
        </LayoutView>
    </Router>
</ThemeProvider>
~~~

#### CSS Theme Overriding Example

To get the default theme, without having to provided all the values, the control library provides the a theme css file that will populate the CSS Variables. Since the controls use scoped styles for the components the including the <code>_framework/scoped.styles.css</code> will roll up the styles from the controls into the deployment artifact done by the ASP.NET Blazor 5.0 publish. Checkout the <a href="#current-list-of-theme-css-variables" title="Checkout the CSS Variables available for Override">current list of Theme CSS Variables</a> for a list of available override and below is an example of the structure for the head tag to cascade the CSS theming.

~~~ html
<html>
<head>
    ...
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/animate.css/4.0.0/animate.min.css" />
    <link href="_content/EventHorizon.Blazor.UX.Controls/ehz-blazor-ux-theme.css" rel="stylesheet" />
    <link href="css/theme.css" rel="stylesheet" />
    <link href="css/app.css" rel="stylesheet" />
    <link href="_framework/scoped.styles.css" rel="stylesheet" />
    ...
</head>
<body>...</body>
</html>
~~~

#### Current list of Theme CSS Variables

<span style="font-size: 0.8rem">
    This is the current list of available CSS Variables available for overriding for a CSS theme override file or key/value's for the Theme Provider.
</span>

~~~ css
:root {
    --font-family: "Electrolize", "sans-serif";
    /* Button Styles */
    --button-color: #26dafd;
    --button-background-color: rgba(6, 43, 45, 0.65);
    --button-blink-color: white;
    --button-border-color: rgba(6, 43, 45, 0.65);
    --button-corner-background-color: #26dafd;
    /* Link Styles */
    --link-color: #acf9fb;
    --link-shadow-color: rgba(172,249,251,0.65);
    /* Table Styles */
    --table-head-color: #a1ecfb;
    --table-row-color: rgba(2,157,187,0.325);
}
~~~

### Blazor Server Sample

As part of the theming process I was able to break out the common parts of the sample applications into a shared library, this library includes pages, components and base theming that both the Blazor Server and Wasm application uses. So the repository has an example of Blazor Server/Wasm and how to share pages and components between them, checkout the <a href="https://github.com/canhorn/EventHorizon.Blazor.UX" target="_blank">canhorn/EventHorizon.Blazor.UX Repository</a> for the finer details on this can be achieved.

### Bug Fix for Flashing Scrollbar on Navigation

This is just a mention that an annoying bug was fixed, the bug it self was the scrollbar would flash on screen after an page transition from a page that had scroll bars to one that did not. The fix was to fix the timing of the transitions, so they don't finish then for a split second show the transition away from content.

## Future Plans

My current plans for this library is to have a centralized control library I can personally use for my Game Development Platform, a future article will be posted here when I release it. 

<ul class="future-plans__list">
    <li>
        Release as a NuGet Package
    </li>
    <li>
        Add Form Controls
    </li>
    <li>
        Added Theme override Control - So the user can edit a theme for the application.
    </li>
</ul>

<style>
    .future-plans__list {
        padding-left: 2rem;
    }
</style>