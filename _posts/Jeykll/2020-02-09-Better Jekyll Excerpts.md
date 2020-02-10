---
layout: post
author: Cody Merritt Anhorn
title: Better Jekyll Excerpts
---

I recently found that my blog post listing page was not correctly splitting on paragraph. The default behavior should of caused the post except to be the first paragraph, but for one post it was not correctly splitting. To fix this it was an easy fix by using the 'excerpt_separator' metadata property.

Below is an expert example I used, this will tell Jekyll to exactly where I wanted the post to break and show, cutting down the size of the post list page.

~~~
---
except_separator: <!--more-->
---
~~~