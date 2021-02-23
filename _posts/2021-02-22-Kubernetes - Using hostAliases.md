---
layout: post
author: Cody Merritt Anhorn
title: Kubernetes - Using hostAliases
categories: [blog, kubernetes]
tags: [Kubernetes]
---

This article goes over how to update the /etc/hosts on your deployment pods with custom entries.

## Details

Adding values to the <code>hostAliases</code> section of a deployment I am able to redirect requests to my identity server located on my main machine, since a  back plane network request causes issues with the url structure and SSL resolution. This has the added benefit of behaving more like a production environment as well.

Checkout **<a href="https://kubernetes.io/docs/concepts/services-networking/add-entries-to-pod-etc-hosts-with-host-aliases/" rel="noopener" target="_blank">Adding entries to Pod /etc/hosts with HostAliases</a>**, the official documentation, for more details.

~~~ yml
apiVersion: apps/v1
kind: Deployment
metadata:
    name: platform-service
spec:
    replicas: 1
        spec:
            hostAliases:
                - ip: 192.168.1.130
                  hostnames:
                      - identity-server.com
            containers:
                - name: platform-service
                  image: cloudplatform:latest
                  ports:
                      - containerPort: 80
~~~