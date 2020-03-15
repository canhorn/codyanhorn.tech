---
layout: post
author: Cody Merritt Anhorn
title: Using calc with a SCSS Variable
categories: [blog]
tags: [scss]
---

To use a variable in the CSS calc use SCSS [Interpolation](https://sass-lang.com/documentation/interpolation), example below.

~~~ scss
$top-row-height: 3.5rem;

.quick-links {
    padding: 0.5em;
    height: calc(100% - #{$top-row-height});
}
~~~