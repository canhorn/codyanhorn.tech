---
layout: post
author: Cody Merritt Anhorn
title: Design Patterns - Composite Query Event
---

In CSQR sometimes you need to query a lot or dynamic of data across a distributed system. One pattern I use is to publish a notification event, this is an event multiple handlers can listen for and act on. I will go over some to the pros and cons of using this pattern, and the main scenario I use it in.

## Setup of Pattern

