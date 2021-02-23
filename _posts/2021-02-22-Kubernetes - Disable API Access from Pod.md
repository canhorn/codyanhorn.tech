---
layout: post
author: Cody Merritt Anhorn
title: Kubernetes - Disable API Access from Pod
categories: [blog, kubernetes]
tags: [Kubernetes]
---

This article will be short, it shows an example of how to disable a pods access to the Kubernetes API. By setting <code>automountServiceAccountToken: false</code> in the template spec for a Deployment you can disable API access to Kubernetes from the container.

~~~ yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: core-server
spec:
  replicas: 1
  selector:
    matchLabels:
      app: core-server
  template:
    metadata:
      labels:
        app: core-server
    spec:
      automountServiceAccountToken: false
      containers:
      - name: core-server
        image: canhorn/ehz-platform-server-core:latest
~~~