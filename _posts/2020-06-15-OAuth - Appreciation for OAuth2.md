---
layout: post
author: Cody Merritt Anhorn
title: OAuth - Appreciation for OAuth2
categories: [blog]
tags: [Online Security]
---

While implementing an integration with Tumblr I had to work against their OAuth 1.0a authorization API, I personally had never worked against an OAuth 1.x provider before so this was quite the learning experience for me. In this article I will go over the differences I found between OAuth 1/2 and some resources I found helpful.

## What Exactly is OAuth

> OAuth (Open Authorization) is an open standard authorization framework for token-based authorization on the internet. ... It acts as an intermediary on behalf of the end user, providing the third-party service with an access token that authorizes specific account information to be shared. **<a href="https://searchapparchitecture.techtarget.com/definition/OAuth" target="_blank">TechTarget.com - OAuth</a>**

This quote gives a good overview of what an OAuth is, a way for a user to give authorization to a third party service from a centralized provider. Using a authorization flow the user is able to authorize the resource provider with access, in the form of an access token, to the requesting service. By doing this they can deny access to private information about the user or help them get access to details on a resource server.

## OAuth 1.X

Since most of my experience is with OAuth2, specifically OpenId, I might have gaps or misrepresentations in this section. I only worked with OAuth1 very limitedly, but found it might be easy to accidentally sending the wrong credentials between the services involved.

OAuth1 had no clear distinction between the resource server and service provider, these were usually hosted on the same server and needed a security signature to be generated for every request. OAuth2 got around this by requiring SSL and the use of short-lived tokens, with the ability to layer on other security on top of the token. 

OAuth1 did not give a way to create authorization requests that did not have a browser, making it limited in the this day and age that has so many way necessary to authorize a user most would have to be done by hand rolling a personal/private authorization schema.

## OAuth 2 (OpenId)

Since I used OAuth2 in the confines of OpenId I will mostly speak against that. OpenId is an authentication protocol, meaning it mostly tracks the identity of the user making the request. With the ability to provide access tokens tied to a specific user, I find this gives the user more control over their access. I also find that the tools for OAuth2 are more mature and flushed out, compared to OAuth1 I only found a single library, forked, with issues that I would have to fork myself to fix. They have a HttpClient connection memory leak that would need to fixed before I would use it in any production capacity.

## Conclusion

This article was short, and more to give a location to brain dump what I worked on. The API I was working against was Tumblr and they seem to be dated in their authorization flow, but it did require me to role the authentication/authorization flows myself making it something I do not feel conformable using in any production capacity.

So to end I am grateful that I have tools that can be used to handle this, I like IdentityServer4 specifically for my Identity Management and using OAuth2 provided AccessToken's for all my resource authorization.


## Resources

**-** <a href="https://en.wikipedia.org/wiki/OAuth" target="_blank">Wikipedia OAuth</a>

**-** <a href="https://lifehacker.com/understanding-oauth-what-happens-when-you-log-into-a-s-5918086" target="_blank">Understanding OAuth: What Happens When You Log Into a Site with Google, Twitter, or Facebook</a>

**-** <a href="https://searchapparchitecture.techtarget.com/definition/OAuth" target="_blank">TechTarget.com - OAuth</a>

**-** <a href="https://identityserver.io/" target="_blank">Identity Server</a>

**-** <a href="https://openid.net/" target="_blank">OpenId</a>
