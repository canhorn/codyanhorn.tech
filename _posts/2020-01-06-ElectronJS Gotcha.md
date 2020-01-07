---
layout: post
author: Cody Merritt Anhorn
title: ElectronJS Gotcha
---

When creating an ElectronJS application and your trying to control the install location.
Make sure you have your package.json 'name' is valid. Below is the validation pattern, in my case I had an uppercase and so it was not installing into my expected directory.

Electron Builder uses the name to construct the location directory where it will be installed, so making sure this is valid will make sure it is installed in your expected location.

From VSCode package.json validation:
~~~
String does not match the pattern of "^(?:@[a-z0-9-~][a-z0-9-._~]*/)?[a-z0-9-~][a-z0-9-._~]*$".
~~~