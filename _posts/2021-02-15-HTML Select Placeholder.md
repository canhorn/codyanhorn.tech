---
layout: post
author: Cody Merritt Anhorn
title: HTML Select Placeholder
categories: [blog]
tags: [.NET, C#, MediatR]
---

With HTML5 came a way to show a placeholder for a select form element, without giving the user an option to select it. This give developers an easy way to have a nice looking placeholder but not have it as an option for selection.

## Example

Below is an example and some code to go with it.

<hr />
<label style="font-weight: bold">NPC Type:</label>
<select name="npc-type">
    <option value="" disabled hidden selected>Select a Type for the NPC...</option>
    <option value="king">King</option>
    <option value="shop-keep">Shop Keep</option>
    <option value="citizen">Citizen</option>
    <option value="guard">Guard</option>
</select>

~~~ html
<label style="font-weight: bold">NPC Type:</label>
<select name="npc-type">
    <option value="" disabled hidden selected>Select a Type for the NPC...</option>
    <option value="king">King</option>
    <option value="shop-keep">Shop Keep</option>
    <option value="citizen">Citizen</option>
    <option value="guard">Guard</option>
</select>
~~~
