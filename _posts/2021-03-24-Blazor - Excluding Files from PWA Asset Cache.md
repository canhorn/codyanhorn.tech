---
layout: post
author: Cody Merritt Anhorn
title: Blazor - Excluding Files from PWA Asset Cache
categories: [blog, blazor]
tags: [.NET, C#, Blazor, Wasm, PWA]
---

In this quick article we will go over how to exclude a files from the Progressive Web Application asset cache.

## Why

As part of my deployments for my Blazor Wasm application, I wanted to support the PWA aspect but the way that the application gets its appsettings.json was causing issues with the SHA integrity check. Since I inject my own configuration, overriding the whole appsettings.json, this caused issues with the the SHA integrity check. This failure can cause the whole request tree to fail, so in turn causing the cache to become stale.

### Example

What I found is that the provided service-worker for a production environment has logic to exclude files from verification already. As shown below, we just add our own file regular expressions to the array <code>offlineAssetsExclude</code>.

~~~ javascript
// ... Removed for Brevity

const cacheNamePrefix = 'offline-cache-';
const cacheName = `${cacheNamePrefix}${self.assetsManifest.version}`;
const offlineAssetsInclude = [ /\.dll$/, /\.pdb$/, /\.wasm/, /\.html/, /\.js$/, /\.json$/, /\.css$/, /\.woff$/, /\.png$/, /\.jpe?g$/, /\.gif$/, /\.ico$/, /\.blat$/, /\.dat$/ ];
// Here is our updated with appsettings.json and version.js excluding.
const offlineAssetsExclude = [ /^service-worker\.js$/, /^appsettings\.json$/, /^version\.js$/ ];
// This is the Original
// const offlineAssetsExclude = [ /^service-worker\.js$/ ];

async function onInstall(event) {
    console.info('Service worker: Install');

    // Fetch and cache all matching items from the assets manifest
    const assetsRequests = self.assetsManifest.assets
        .filter(asset => offlineAssetsInclude.some(pattern => pattern.test(asset.url)))
        // Here is where the files are removed
        .filter(asset => !offlineAssetsExclude.some(pattern => pattern.test(asset.url)))
        .map(asset => new Request(asset.url, { integrity: asset.hash }));
    await caches.open(cacheName).then(cache => cache.addAll(assetsRequests));
}

// ... Removed for Brevity

~~~
