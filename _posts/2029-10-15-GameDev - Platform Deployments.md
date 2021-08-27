---
layout: post
author: Cody Merritt Anhorn
title: Game Dev Platform - Platform Deployments
categories: [blog, kubernetes]
tags: [Kubernetes]
---

This article will go over how the Cloud Platform creates a new Development Platform.

- Approve Registration
- Validation of Platform for User
- Create Identity Clients
- Create Game Platform Namespace (by User's Id)
- Create Game Platform Configuration
- Create Game Platform Services
  - Includes creation of Volumes
  - Details on how the services are separated, how they integrate with each other
- Game Platform Reporting

Token Replacement pre processing, this updates the configuration for the specific game platform being created.