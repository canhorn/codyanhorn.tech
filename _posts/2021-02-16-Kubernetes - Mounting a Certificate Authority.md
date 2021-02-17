---
layout: post
author: Cody Merritt Anhorn
title: Kubernetes - Mounting a Certificate Authority
categories: [blog, kubernetes]
tags: [Kubernetes]
---

This article will go over adding a Certificate Authority to your Kubernetes cluster and mounting it into a container, I found that I needed this to setup local development with full SSL support using self-signed certificates.

## How To

Its actually quite easy, the only rough point is with the container and where to mount the Certificate Authority file. Based on the base linux you might have to adjust the location you mount the Certificate Authority file.

### Generator or Retrieve your ca.pem file

Make sure it's the Certificate and not the Private Key, below is an example of a CA file.

~~~
-----BEGIN CERTIFICATE-----
MIIE4DCCA0igAwIBAgIRAKOASBCmVbT/8wxIuKJMwQIwDQYJKoZIhvcNAQELBQAw
gYcxHjAcBgNVBAoTFW1rY2VydCBkZXZlbG9wbWVudCBDQTEuMCwGA1UECwwlREVT
S1RPUC1BM0RBTk85XGNvZHlhQERFU0tUT1AtQTNEQU5POTE1MDMGA1UEAwwsbWtj
ZXJ0IERFU0tUT1AtQTNEQU5POVxjb2R5YUBERVNLVE9QLUEzREFOTzkwHhcNMjAx
MDE1MjA1MjI3WhcNMzAxMDE1MjA1MjI3WjCBhzEeMBwGA1UEChMVbWtjZXJ0IGRl
dmVsb3BtZW50IENBMS4wLAYDVQQLDCVERVNLVE9QLUEzREFOTzlcY29keWFAREVT
S1RPUC1BM0RBTk85MTUwMwYDVQQDDCxta2NlcnQgREVTS1RPUC1BM0RBTk85XGNv
ZHlhQERFU0tUT1AtQTNEQU5POTCCAaIwDQYJKoZIhvcNAQEBBQADggGPADCCAYoC
ggGBAJwlmzuXvE3YJGn1REjLrbJQegoGOrFd4izxRMPpcmzYOZmz62ygQK1cN7rH
W+BhKWBNdKkDeJ+K5h3bo3XOKg7uljlwq/YugY5nWM+uZIBxx+tE7Ge217nFwtKq
7rwQQhKasjqZJR+e7vfbSfUiOljv2F6Ah6BZMZ/OPwwQfPznCV/tBW8LhpFW5RQn
3Ff9E3/j0tjWvvAM3mNd3Sn4umhdqFviSn2PIBRuQdCPfWE2hq8RyVCdW42bTg7+
fAQmt22qlOLv2pMUbCbkPUchO9vPhRfE3F5Gpiw0Y7fZr3U3q8M8+4guvX1o543n
AiZv5xqwmMzkm0hxhIDtpbHlGSvbRqOrFGU20wR0iOgnKySN/SNMFgkRL1zbk4iB
qjHkZyrwzb2h71QVjFQAAj2SYnEaRwRc/kyZ+tEomuZ2s/sfwqVwHHLGrP2nThAP
dLRUyp9nWZQN9BMej9A7YlCksQlvuqYq0mBUIiF9mD4FUMMIsY8JiR3fANcM4PjZ
d/HvVwIDAQABo0UwQzAOBgNVHQ8BAf8EBAMCAgQwEgYDVR0TAQH/BAgwBgEB/wIB
ADAdBgNVHQ4EFgQULFYN9UGKw7zRIzKtICGRn4yUonIwDQYJKoZIhvcNAQELBQAD
ggGBAJnFyA6KzrjO3akLqm32dudZ2gFe4/zh5vWChqs0Fcmdu1poGvRzQEGFbG2Z
V1dgtUPXwypUHQxx3Ezw3ty4V9dslKRjyEDsfJc91FXVjJrKL2j+A7ehgD2QKrQb
8DNgm/uggKBmTbOvvpcH+RlkrCFx9rLdgVBSAb5go9hmseBtz5AdBq6Qn1VOFUUv
UO9EOceO65eYkiiBxEKuqYyYsp1DOECLFk3DFh+WfqsTq17qN2ZfkI0b0R7pzjke
1tpgbooiRRXogsTm3kEx0V8GhDRtuzavVbhi0KDV11WeYY2ziSTCHSjLDP+kk/oe
ALzk3v2VVa9Et8Amfic3vjD4Cl3AqqzvuD7H97WrvPRjpzZ8gIaZpzRil2TfBN+I
tO+8Vd0Q850BC8Xp7QWeE19vak0XnK9+cC1IaHyJC5DDFU/2oeotRGRbiy6THZ6o
MTc0DxHH1grZ/CBaSaAmMQDViqUInT/j7x/HVXus0oLUPk4raz8Q/KxxiPbM2Uft
gRz7ug==
-----END CERTIFICATE-----
~~~

### We will now base64 encode it

This will put it into a format we can put into Kubernetes, below is a linux command that you can use to encode a files contents.

~~~ bash
base64 ./rootCA.pem > rootCA.pem.base64

# Using the fancy Windows Sub-System for Linux
wsl base64 ./rootCA.pem > rootCA.pem.base64

~~~

Take the base64 encoded string and replace the <code>(Base64 Encoded CA)</code> string in a YML file like below. You will have to update the namespace, your-namespace, to make sure it matches where you will running your Deployments.

***domain-ca-secret.yaml***  
~~~ yml
apiVersion: v1
kind: Secret
metadata:
  name: domain-ca-secret
  namespace: your-namespace
type: Opaque
data:
  ca.pem: (Base64 Encoded CA)

~~~

### Add the Secret to Kubernetes

Running the ***<code>kubectl apply -f ./domain-ca-secret.yaml</code>*** command, will add your CA to Kubernetes allowing for you to mount it into our Deployment containers.

### Create a Kubernetes Deployment

Below is an example of mounting the CA into a Deployment container. The section to point out in the configuration is the ***<code>mountPath: /etc/ssl/certs/domain-ca-cert.pem</code>***, this is the location and filename that the CA will be available in the container. By mounting this into the certs folder of the container/system will allow for application and the OS to correctly load the CA for validating SSL certs for internet requests. 

***ca-deployment.yaml***
~~~ yml
apiVersion: apps/v1
kind: Deployment
metadata:
    name: ca-injected-container
    namespace: your-namespace
spec:
    replicas: 1
    selector:
        matchLabels:
            app: ca-injected-container
    template:
        metadata:
            labels:
                app: ca-injected-container
        spec:
            containers:
                - name: ca-injected-container
                  image: dockerhub/your-container:latest
                  ports:
                      - containerPort: 80
                  volumeMounts:
                      - name: domain-ca
                        mountPath: /etc/ssl/certs/domain-ca-cert.pem
                        subPath: ca.pem
                        readOnly: false
            volumes:
                - name: domain-ca
                  secret:
                      secretName: domain-ca-secret

~~~

### Add the Secret to Kubernetes

Running the ***<code>kubectl apply -f ./ca-deployment.yaml</code>*** command, will create your Deployment and the container, if you want to check that the CA was correctly mounted you can Terminal into the container and run the command below.

~~~ bash
# (Optional) Install Curl, easy install if not installed
apt update; apt -y upgrade; apt -y install curl

# Run the curl command against any domain that would need the CA for validation.
# Notice the location and filename are from the volumeMounts configuration of the Deployment.
curl --cacert /etc/ssl/certs/domain-ca-cert.pem --verbose https://domain.com

~~~
