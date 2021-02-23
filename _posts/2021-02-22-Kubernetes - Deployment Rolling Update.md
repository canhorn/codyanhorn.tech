---
layout: post
author: Cody Merritt Anhorn
title: Kubernetes - Deployment Rolling Update
categories: [blog, kubernetes]
tags: [Kubernetes, Kubectl, Command Line, .NET]
---

In this article I will go over a few methods I use to restart my deployments in Kubernetes, without having to remove the whole deployment to do so.

## Details

When setting up deployment scripts, I research multiple ways to rollout update to my images to the deployments they were used in. The main way I use right now is the rolling restart command for scripted deployments, but I also update a deployment environment variable to "change" the deployment that will then trigger the pods to update.

The rolling restart is a better way, since kubernetes will cycle through the pods and restart them in a way you would expect. The environment variable change seems more like a hack, my main reason for using it is because the client I use does not support the API call yet.

## Examples

Use this command below to easily tell Kubernetes to start a rolling deployment on the <code>your-application</code> deployment. 

~~~ bash
kubectl rollout restart deploy your-application
~~~

Here is a CLI command for updating environment variables.

~~~ bash
kubectl set env deployment your-application DEPLOY_DATE="$(date)"
~~~
 
Also a .NET Kubernetes Client example snippet, has the added benefit of giving the pod a variables to access the time the deployment took place.

~~~ csharp
var yourNamespace = "your-namespace";
var deploymentName = "your-application";

var patchBody = new V1Deployment
{
    Spec = new V1DeploymentSpec
    {
        Template = new V1PodTemplateSpec
        {
            Spec = new V1PodSpec
            {
                Containers = new List<V1Container>
                {
                    new V1Container(
                        deploymentName,
                        env: new List<V1EnvVar>
                        {
                            new V1EnvVar
                            {
                                // Here is the env var we update every time we want to trigger a deployment update
                                Name = "DATE_DEPLOYED",
                                Value = DateTime.Now.ToString(),
                            }
                        }
                    ),
                }
            }
        }
    }
};
await _client.PatchNamespacedDeploymentAsync(
    new V1Patch(
        patchBody,
        PatchType.StrategicMergePatch
    ),
    deploymentName,
    yourNamespace,
    cancellationToken: cancellationToken
);
~~~


