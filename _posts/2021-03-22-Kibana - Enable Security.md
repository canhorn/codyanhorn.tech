---
layout: post
author: Cody Merritt Anhorn
title: Kibana - Enable Security
categories: [blog]
tags: [Kibana, Elasticsearch]
---

In this article I will go over how I personally enable security, for user login and application connections, on my Kibana instances. 

## Details

While setting up Logging for the services that will be deployed on the Game Development Platform, I wanted a way to expose a public facing Kibana. To do this I wanted to make sure that Kibana was secure, making it so only permitted users have access to the Kibana Dashboard, by enabling the Security layer of Kibana/Elasticsearch.

This turned out to be very easy, using the all in one ECK Operator, with a kubectl apply and some setup configurations we can easily create an Elasticsearch cluster. This cluster is security first, so by default not much configuration is needed to get going. As for my configuration I wanted to expose the <code>elasticsearch-es-elastic-user</code> secret to the platform, this exposes the elastic (superuser) password for easier integration connections. 

The Kibana configuration was also pretty straight forward, since I wanted to fully manage it in another namespace, with a Deployment and Ingress configuration I was able to setup Kibana. As part of the configuration for Kibana I specify the Elasticsearch User and Password, just injecting in the secret we exposed earlier from Elasticsearch. Below I have include the configurations and details about them for reference.

To limit the permissions of an application that is interfacing with the Elasticsearch cluster I create a <code>platform_user</code> that restricts the permissions based on the index. I do this setup as part of the manual steps phase of the platform setup, since this only has to be done at startup or explicitly when needed. A secret is created for the <code>platform_user</code> password and setup the same way the <code>elasticsearch-es-elastic-user</code> is setup.

## Example

### Setup the Kubernetes Cluster

Here we are going to setup the Cluster with the all in one ECK Operator.

~~~ bash
kubectl apply -f https://download.elastic.co/downloads/eck/1.4.0/all-in-one.yaml
~~~

### elasticsearch-setup.yml

Here we create the namespace for Elasticsearch and create an empty secret for <code>elasticsearch-es-elastic-user</code>, this exposes the secret to allowed namespace using reflector. Since the Operator will update any already existing, not deleting the annotation that make reflector function.

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
        reflector.v1.k8s.emberstack.com/reflection-allowed-namespaces: 'core-namespace'
type: Opaque
data:

~~~

### elasticsearch-create.yml

Using the main default configuration, some updates to work with my environment, and disabling the selfSignedCertificate because I manage that with another service in my infrastructure.

~~~ yml
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

### kibana-setup.yml

The below configuration for Kubernetes is for a publicly exposed Kibana deployment, it includes the ConfigMap for the settings, these are required for interfacing with Elasticsearch and making sure the service is exposed to the public. You will notice the <code>{{your-apex-domain}}</code> parameter, this is just the root domain that your cluster is hosted on or exposed from. Most of the configuration is a standard deployment, service, and ingress for Kibana, the expanded areas are the secret mounting for the elastic user and the secret creation for the <code>platform_user</code>.

The mounting of the secret for the elastic user is done so we do not have to know the password and just get it from the secrets vault of Kubernetes. The <code>platform_user</code> password, in the form of a secret, is exposed here. The platform user password is used by other services that do not need super user permissions.

~~~ yml
apiVersion: v1
kind: ConfigMap
metadata:
    name: kibana-config
    namespace: monitoring-namespace
data:
    kibana.yml: |-
        server.name: kibana.{{your-apex-domain}}
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
    namespace: monitoring-namespace
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
                      - name: ELASTICSEARCH_URL
                        value: 'http://elasticsearch-es-http.elasticsearch:9200'
                      - name: ELASTICSEARCH_USER
                        value: 'elastic'
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
    namespace: monitoring-namespace
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
    namespace: monitoring-namespace
spec:
    rules:
        - host: kibana.{{your-apex-domain}}
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
              - kibana.{{your-apex-domain}}
          secretName: kibana.{{your-apex-domain}}
---
apiVersion: v1
kind: Secret
metadata:
    name: elasticsearch-users
    namespace: monitoring-namespace
    annotations:
        reflector.v1.k8s.emberstack.com/reflection-auto-enabled: 'true'
        reflector.v1.k8s.emberstack.com/reflection-allowed: 'true'
        reflector.v1.k8s.emberstack.com/reflection-allowed-namespaces: 'core-namespace'
type: Opaque
data:
    platform_user: U3VwZXIkZWNyZXRQYSQkV29yZA==

~~~
