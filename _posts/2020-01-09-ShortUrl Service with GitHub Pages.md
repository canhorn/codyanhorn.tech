---
layout: post
author: Cody Merritt Anhorn
title: ShortUrl Service with GitHub Pages
categories: [blog]
tags: [github]
---

Did you know with GitHub page you can create a simple ShortUrl server?

With the power of Jekyll, the static website generator, you can. Jekyll is what GitHub uses by default for generating static websites when you setup the GitHub pages feature. This gives you a simple static websites that gets updated, pretty quickly, from a commit.

This post assumes you already have GitHub Pages enabled for your site.

To achieve our goal of creating simple ShortUrls we are going to use the layouts feature of Jekyll. Using layouts we can create a simple abstraction for the logic of navigating the user to any site, but also having it branded to our domain.

## How to do it

We will need to two files, one for the layout and another for the ShortUrl. Make sure the {{"{{ content "}}}}, stated below in ***_layouts/shorturl.html***, is above the &lt;script&gt; block. In our ShortUrl instances file, example below in ***s/twitter.md***, you can see with very few lines we create a url to wherever we want.

After creating those files and pushing up the files to your GitHub Pages repository it will deploy the pages to either a Url you setup or GitHub will give you a custom domain for you to use. 

We can now access our ShortUrl like so: 
~~~
<Your Domain>/s/twitter.
~~~

Here is an example structure: <a href="https://projecteventhorizon.com/s/twitter">https://projecteventhorizon.com/s/twitter</a>

### Some Example Code

***_layouts/shorturl.html***
~~~ html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8" />
        <meta name="robots" content="noindex" />
        <title>{{"{{ page.title "}}}}</title>
    </head>

    <body>
        {{"{{ content "}}}}  <!-- Here is where our content will be injected into the page. -->
        <script>
            if (url) {
                window.location = url;
            }
        </script>
    </body>
</html>
~~~

***s/twitter.md***
~~~ html
---
layout: shorturl
title: ShortUrl - Your Twitter
---

<script>
    const url = 'https://twitter.com/CodyAnhorn';
</script>
~~~
