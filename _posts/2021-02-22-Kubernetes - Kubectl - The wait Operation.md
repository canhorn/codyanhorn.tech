---
layout: post
author: Cody Merritt Anhorn
title: Kubernetes - Kubectl - The wait Operation
categories: [blog, kubernetes]
tags: [Kubernetes, Kubectl, Command Line]
---

This article will be a quick one, going over the <code>wait</code> operation for kubectl. It's relatively simple to use, but helps immeasurably helpful with automation scripts.

## Details

In my case I wanted to make my deployment scripts less error prone during a database migration job. I have a job that run in Kubernetes during a deployment that will run Database migrations, and with most database migrations you probably dont want to deploy a new application without having a fully updated database. To help facilitate the process of updating the database, but not deploying the application till it was done I used the <code>wait</code> operation to delay startup till it is done running.

Below is pretty much it, just using <code>wait</code>, this takes in the resources you are waiting for to finish. It also comes with a <code>--for</code> option that can be used to trigger a success on a condition, in my case I want the <code>wait</code> to trigger on a **complete condition**, this way if it does not in a certain type period **complete** it will trigger a figure of the job and not deploy the application. Checkout [Kubectl Operations](https://kubernetes.io/docs/reference/kubectl/overview/#operations) for more details about <code>wait</code> and other operations supported by Kubectl.

~~~ bash
kubectl wait --for=condition=complete job/auth-migrate-database-job
~~~
