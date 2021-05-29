---
layout: post
author: Cody Merritt Anhorn
title: Blazor and Contentful CMS Blog Template
categories: [blog, blazor, contentful]
tags: [Blazor, Contentful, ASP.NET, SEO, WebHook, Azure]
---

In this article I go over a Blog Starter Template built on Blazor using the Contentful Headless Content Management System. 

## What

The <a href="https://github.com/canhorn/Blazor.Contentful.Blog.Starter" target="_blank">Blazor.Contentful.Blog.Starter</a> Blog Template repository was created to make it easier to get up and running with <a href="https://www.contentful.com/" target="_blank">Contentful</a> built on top of ASP.NET Core Blazor. This template has a bunch of features that will be outlined below, you can see an <a href="https://blazor-contentful-blog-starter.azurewebsites.net/" target="_blank">Live Demo</a> or follow the <a href="https://github.com/canhorn/Blazor.Contentful.Blog.Starter#deploy-this-site-to-azure" target="_blank">Deploy this site to Azure</a> instructions to deploy your own example.

## Motivation

The main motivation for creating this template was so I could have an easy way to create new sites built using a CMS, and I found a very well done example template. But the template was built using ReactJS and I wanted to have an easier time extending the functionality for my personal projects, so I took that template as a blueprint and create it using Blazor. I found doing the conversion to Blazor a learning process, in that Blazor does not have a very well established way to handle Server Rendered head tags and how a Azure Resource Manager template can be used to spin up personal resources spaces for new projects.

## Integration with the Contentful Headless CMS

As already stated the templates integrates with Contentful, which is a Headless Content Management System, giving the template a backing systems for all the content of the Blog. By using Contentful you can write your blog posts in the Contentful system and with the built in hooks of the Template automatically trigger updates to your deployed site. The template includes the plumbing and does all the heavy lifting with the SDK behind a service layer, making it easier to focus on expanding the site while still having access to the data of your Contentful space.

## Out of the Box Theme

With the default starter you get a fully responsive theme using CSS, with customization available through the use of CSS Variables. Using the CSS variables you are able to easily change the color scheme of the whole site and with a little it of customization you could even create a light/dark theme toggle.

<a href="/image/Posts/2021-05-29/Theme.png" target="_blank">
    <img src="/image/Posts/2021-05-29/Theme.png" title="This image show the default theme the starter theme will be displayed in." />
</a>

## Vercel Image creator for Open Graph Images

With the template there was a cool feature of dynamic Open Graph images added, the theme uses a publicly exposed Open Graph image generator. This uses Vercel to create an image based on URL Query parameters that can then be used by services like Twitter for Tweet cards.

> If you are going to use this site in any compactly please spin up your own Open Graph generator, you can get the source from the <a href="https://github.com/canhorn/og-image">Blazor OG Image Repository</a>.

<a href="/image/Posts/2021-05-29/VercelOpenGraph.png" target="_blank">
    <img src="/image/Posts/2021-05-29/VercelOpenGraph.png" title="This image was generated for the home page of the Template using the Vercel Open Graph image generator." />
</a>

## Server Rendered Blazor SEO

One of the biggest feature, that most Blazor application might be lacking, is the ability to generate SEO related meta tags on Server render. This makes it so you can post any of the URLs from this deployed site to any service that support Open Graph tags and get the correct metadata about the site. The built in system will also update the tags as a user navigates around the application after the first request.

<a href="/image/Posts/2021-05-29/SEOHtml.png" target="_blank">
    <img src="/image/Posts/2021-05-29/SEOHtml.png" title="Show the generated html meta tags that were generated server side." />
</a>

## Localization Setup/Implementation

Another feature of the template is Localization is enabled by default for the site, a localization toggle is not currently implemented, but support should be very easy to implement using already existing pattern for Blazor. The pages the have values requiring localization are already in the default resource file, and a base component is created that will help with the inject of the Localizer, that can then be used by extension points.

## RSS Feeds

The template includes an RSS feed generator, that will include the core pages and the published blog posts. The feed URL is also included in the head of the site making it easy for existing tools to pickup and consume.

<a href="/image/Posts/2021-05-29/RSSFeed.png" target="_blank">
    <img src="/image/Posts/2021-05-29/RSSFeed.png" title="An example image of the generated RSS feed supplied by the default template." />
</a>

## Site Map Generation

The template also include a Site Map generator, that will include the core pages and published blog posts. The SiteMap is fully compliant for used with Google and Bing web master tools.

<a href="/image/Posts/2021-05-29/SiteMap.png" target="_blank">
    <img src="/image/Posts/2021-05-29/SiteMap.png" title="Show the generated Site Map from the default template." />
</a>

## Deploy to Azure

The template also includes an ARM template that can be used to deploy the repository to Azure. The template will guide you through the creation of the necessary resources to deploy the template as a App Service in Azure. It also ties the repository to the App Service Deployment Center, this can be used to manually deploy the application, with Azure handling the building of the application. This makes it really easy to make changes to the application code, and just trigger the changes in Azure to build and deploy your changes.

<a href="/image/Posts/2021-05-29/DeployToAzure.png" target="_blank">
    <img src="/image/Posts/2021-05-29/DeployToAzure.png" title="Shows the form that can be filled out to deploy the template to Azure." />
</a>

## Cache Busting (WebHooks)

One of the bigger features is cache busting which is built into this template. When you register the WebHook <code>/webhook/cache-buster</code> with Contentful, you will get automatic updates triggered as they happen right to your deployed site. This makes it easy to curate your content in Contentful, schedule it for publish at a later date, and only have to worry about Contentful for all the management of the post.
