---
layout: post
author: Cody Merritt Anhorn
title: Kubernetes - Exposing Kibana through an Ingress
categories: [blog, kubernetes]
tags: [Kubernetes, Kubectl, Kibana, Elasticsearch]
---

In this article I will go over the configuration I use that gives me a secure and publicly exposed Kibana backed by Elasticsearch hosted on Kubernetes. 

## Details

When setting up a secure Elasticsearch and Kibana on Kubernetes you need to take into consideration where and how these services are exposed/configured on the Kubernetes cluster. The below configuration will go over Elasticsearch and Kubana setup and configuration for easy setup/update/configuration. This should give you an idea of how Kibana can be configured to expose the service to the public. In the next sections I will go over how to setup Elasticsearch and Kibana, these should be complete enough to get started. But might be missing details on specifics, use my social contacts below if you have any comments or get stuck.


## Setup Elasticsearch

In this section we will setup an Elasticsearch cluster, that builds on top of the <a href="https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-deploy-eck.html" target="_blank">Elastic Cloud on Kubernetes</a> to get a running Elasticsearch cluster up and running.

Below are two sections, one to install ECK to kubernetes and the configuration to expose secrets to other namespaces. The bash lines use kubectl to install Reflector (for secret propagation) and ECK for easy Elasticsearch setup. The yml section is for kubernetes to do some pre setup for Elasticsearch, this way we can access the Elasicsearch elastic users password for interfacing with ES from Kibana. Using ECK we are able to easily create a Elasticsearch cluster.

~~~ bash
## This will help the Secrets propagate to allowed namespaces
kubectl apply -f https://github.com/emberstack/kubernetes-reflector/releases/latest/download/reflector.yaml

## Helps with the Management of Elasticsearch in the Cluster
kubectl apply -f https://download.elastic.co/downloads/eck/1.4.0/all-in-one.yaml
~~~

In the below kubernetes configuration we create a new namespace called 'elasticsearch' that will will create our new ECK managed Elasticsearch cluster. We have this setup prior to the Elasticsearch creation so we can setup some annotations for the 'elasticsearch-es-elastic-user' secret, that will help us to share the elastic users password between namespaces.

~~~ yml
apiVersion: v1
kind: Namespace
metadata:
    name: elasticsearch

---

apiVersion: v1
kind: Secret
metadata:
    name: elasticsearch-es-elastic-user
    namespace: elasticsearch
    annotations:
        reflector.v1.k8s.emberstack.com/reflection-auto-enabled: 'true'
        reflector.v1.k8s.emberstack.com/reflection-allowed: 'true'
        reflector.v1.k8s.emberstack.com/reflection-allowed-namespaces: 'core,cloud-platform,user-platform-[a-zA-Z0-9/-]*'
type: Opaque
data:

---

apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
    name: elasticsearch
    namespace: elasticsearch
spec:
    version: 7.11.1
    nodeSets:
        - name: default
          count: 2
          config:
              node.store.allow_mmap: false
    http:
        tls:
            selfSignedCertificate:
                disabled: true
~~~

## Setup Kibana

For our Kibana configuration we need to configure the Kibana instance with some settings that will be used to expose the server out of a localhost hosting. And the host, username, and password are used by Kibana to connect to our exposed Elasticsearch cluster. Most of the configuration is standard Kibana configuration, it includes inline comments to point out areas of interest, but the main one I will point out here is the <code>ELASTICSEARCH_PASSWORD</code> env variable, we are using the secret store to take our exposed secret from elasticsearch, so it if changes we can pick it up here.

The magic happens in the last configuration, the creation of an Ingress, exposing our Kibana to a host. This configuration uses the nginx ingress and letsencrypt for the SSL issuer, this configuration is out of scope for this article, you can checkout the article <a href="https://codyanhorn.tech/blog/2020/10/03/Kubernetes-Docker-Desktop-and-SSL-Termination.html">Kubernetes - Docker Desktop and SSL Termination</a> for more a way to setup host termination and SSL locally.

~~~ yml
apiVersion: v1
kind: ConfigMap
metadata:
    name: kibana-config
    namespace: monitoring
data:
    kibana.yml: |-
        server.name: kibana.external.host
        server.host: "0"
        elasticsearch:
          hosts: ${ELASTICSEARCH_URL}
          username: ${ELASTICSEARCH_USER}
          password: ${ELASTICSEARCH_PASSWORD}

---

apiVersion: apps/v1
kind: Deployment
metadata:
    name: kibana
    namespace: monitoring
spec:
    selector:
        matchLabels:
            app: kibana
    template:
        metadata:
            labels:
                app: kibana
        spec:
            containers:
                - name: kibana
                  image: docker.elastic.co/kibana/kibana:7.11.1
                  volumeMounts:
                      - name: kibana-config
                        mountPath: /usr/share/kibana/config/kibana.yml
                        readOnly: true
                        subPath: kibana.yml
                  env:
                      # We are connecting to an elasticsearch using the Kubernetes Network
                      # This is internal and not the ingress exposed service, so its more secure.
                      - name: ELASTICSEARCH_URL
                        value: 'http://elasticsearch-es-http.elasticsearch:9200'
                      - name: ELASTICSEARCH_USER
                        value: 'elastic'
                      # Here we inject the password for the elastic-user
                      # from the Secrets Store of Kubernetes 
                      - name: ELASTICSEARCH_PASSWORD
                        valueFrom:
                            secretKeyRef:
                                name: elasticsearch-es-elastic-user
                                key: elastic
                  resources:
                      limits:
                          cpu: 2
                          memory: 1.5Gi
                      requests:
                          cpu: 0.5
                          memory: 1Gi
                  ports:
                      - containerPort: 5601
                        name: http
                        protocol: TCP
            volumes:
                - name: kibana-config
                  configMap:
                      name: kibana-config

---

apiVersion: v1
kind: Service
metadata:
    name: kibana
    namespace: monitoring
    labels:
        app: kibana
spec:
    ports:
        - port: 5601
          targetPort: 5601
          protocol: TCP
          name: http
    selector:
        app: kibana
        
---

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
    annotations:
        kubernetes.io/ingress.class: nginx
        cert-manager.io/cluster-issuer: letsencrypt
    name: kibana-ingress
    namespace: ehz-core
spec:
    rules:
        - host: kibana.sandbox.ehzgames.studio
          http:
              paths:
                  - path: /
                    pathType: Prefix
                    backend:
                        service:
                            name: kibana
                            port:
                                number: 5601
    tls:
        - hosts:
              - kibana.sandbox.ehzgames.studio
          secretName: kibana.sandbox.ehzgames.studio
~~~