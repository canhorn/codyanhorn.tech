---
layout: post
author: Cody Merritt Anhorn
title: Kubernetes - Docker Desktop and SSL Termination
categories: [blog]
tags: [Docker, Kubernetes, TLS, SSL, Nginx]
---

This article will go over a minimal configuration Kubernetes for SSL/TLS termination with Docker Desktop. This article is mostly for my self and not finding an tutorial/article that had all this information in a single spot.

This article will be a list of commands and configurations that can be used to setup Docker Desktop. It will also include resources that can be used to expand on what could is not in the article. 

## Table of Contents

To start off you can use this table of contents to jump to any of the section provided by this article.

<style>
    .table-of-contents__list {
        padding-left: 2rem;
    }
</style>

<ul class="table-of-contents__list">
    <li>
        <a href="#create-kubernetes-dashboard" title="Jump to the Creation of the Kubernetes Dashboard">
            Create Kubernetes Dashboard
        </a>
    </li>
    <li>
        <a href="#configure-tls-keys-and-ingress" title="Jump to the Configure TLS Keys and Ingress for TLS Termination">
            Configure TLS Keys and Ingress
        </a>
    </li>
    <li>
        <a href="#create-http-echo-services" title="Jump to the creation of our Service to use for Testing.">
            Create Http Echo Services
        </a>
    </li>
    <li>
        <a href="#check-that-it-works" title="Jump to the section that will validate our configuration works!">
            Check that it Works
        </a>
    </li>
    <li>
        <a href="#appendix" title="Checkout of supplemental content">Appendix</a>
        <ul class="table-of-contents__list">
            <li>
                <a href="#resource-links-used" title="Checkout some helpful Links I used to help create this article">Resource Links Used</a>
            </li>
            <li>
                <a href="#helpful-commands" title="Checkout some helpful commands to use with Kubernetes">Helpful Commands</a>
            </li>
            <li>
                <a href="#supplemental-file-content" title="This sections contains the contents of all the files used by the Article.">Supplemental File Content</a>
            </li>
        </ul>
    </li>
</ul>


## Create Kubernetes Dashboard

This section will be quick, showing how to get the Kubernetes Dashboard, a UX that can be accessed from the browser. Following the commands below expects you to have Kubernetes and Kubectl setup, and at least know a little bit of the CLI.

<p style="font-size: 0.8rem">
    Run the commands below to setup the Kubernetes Dashboard in Kubernetes. You can checkout to file content in the appendix below.
</p>

~~~ bash
# Create Base kubernetes-dashboard 
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-rc6/aio/deploy/recommended.yaml

# Service - Dashboard Service Account
kubectl create -f ./001_create-new-service-account.yml

# Service - Dashboard Cluster Role Binding
kubectl create -f ./002_create-cluster-role-binding.yml
~~~

You should now be able to open a browser and navigate to: <a href="http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#/overview?namespace=default" target="_blank">http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#/overview?namespace=default</a>. You will need to open the proxy to the Kubernetes cluster, the command is available below in <a href="#helpful-commands" title="Checkout some helpful commands to use with Kubernetes">Helpful Commands</a>.


## Configure TLS Keys and Ingress

Next we will add the secret key and certificate for our TLS to the Kubernetes cluster and also setup the Ingress for accessing our services eternally from the cluster. You will need to generate a cert and key for your domain, checkout the turorial here for generating them for the Nginx Ingress: <a href="https://kubernetes.github.io/ingress-nginx/examples/PREREQUISITES/" target="_blank">Prerequisites - NGINX Ingress Controller</a>

<p style="font-size: 0.8rem">
    Here we create a generic secret key called "k8s-test-ingress.com", to match the domain name we will be tieing our Ingress domain too. Example file content is located below in the <a href="#appendix" title="Checkout of supplemental content">Appendix</a>. We then use the deploy.yaml provided by the kubernetes/ingress-nginx repository to create a Cloud based Nginx Ingress.
</p>

~~~ bash
# Create a secret container for our TLS files
kubectl create secret generic k8s-test-ingress.com --from-file=tls.crt=k8s-test-ingress.com.crt --from-file=tls.key=k8s-test-ingress.com.key

# Using kubernetes/ingress-nginx to create Cloud Ingress
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.40.1/deploy/static/provider/cloud/deploy.yaml
~~~

## Create Http Echo Services

Here we create the HTTP echo server services, this allows us to see some content and verify our settings are correct. We can then create an Ingress, this ingress has settings that tie it back to our secret TLS settings, and a rule that will direct traffic to the HTTP services. Example file content is located below in the <a href="#appendix" title="Checkout of supplemental content">Appendix</a>. 

~~~ bash
# Http Echo Server Services
kubectl create -f ./003_create-http-service.yml

# Create HTTP Ingress
kubectl create -f ./004_create-http-ingress.yml
~~~

## Check that it Works

To check that the configuration works navigate to, <a href="https://k8s-test-ingress.com/" target="_blank">k8s-test-ingress.com</a>, make sure your hosts file is updated. Include a line for <code>127.0.0.1 k8s-test-ingress.com</code>, this will redirect our browser to our Kubernetes Cluster.

## Appendix

### Resource Links Used

<ul class="table-of-contents__list">
    <li>
        <a href="https://kubernetes.github.io/ingress-nginx/" target="_blank" title="The Nginx Ingress Controller Setup">
            Welcome - NGINX Ingress Controller
        </a> - The Nginx Ingress Controller Setup
    </li>
    <li>
        <a href="https://kubernetes.github.io/ingress-nginx/deploy/" target="_blank" title="The Nginx Ingress Controller Installation Guide">
            Installation Guide - NGINX Ingress Controller
        </a> - The Nginx Ingress Controller Installation Guide
    </li>
    <li>
        <a href="https://kubernetes.io/docs/tutorials/kubernetes-basics/" target="_blank" title="A basics Kubernetes tutorial">
            Learn Kubernetes Basics | Kubernetes
        </a> - A basics Kubernetes tutorial
    </li>
    <li>
        <a href="https://kubernetes.io/docs/concepts/services-networking/ingress/" target="_blank" title="Details about Services and Networking for Ingress Concepts">
            Ingress | Kubernetes
        </a> - Details about Services and Networking for Ingress Concepts
    </li>
    <li>
        <a href="https://kubernetes.io/blog/2020/05/21/wsl-docker-kubernetes-on-the-windows-desktop/" target="_blank" title="Setting up of the Kubernetes using Docker Desktop for Window">
            WSL+Docker: Kubernetes on the Windows Desktop
        </a> - Setting up of the Kubernetes using Docker Desktop for Window
    </li>
</ul>

### Helpful Commands

#### Kubectl Proxy Command 

<p style="font-size: 0.8rem">
    This command will open a proxy from your machine to the Kubernetes Cluster, usually if you have any issues also having this running will fix them.
</p>

~~~ bash
kubectl proxy
~~~

#### Kubectl Admin User Token Command

<p style="font-size: 0.8rem">
    You will need to run this command below and use it to get an Access Token for the Dashboard.
</p>

~~~ bash
kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')
~~~

<p style="font-size: 0.8rem">
    Example output from the above get access token command, you want the long encoded string from the Data > token: section.
</p>
~~~ bash
Name:         admin-user-token-hs6dt
Namespace:    kubernetes-dashboard
Labels:       <code>
Annotations:  kubernetes.io/service-account.name: admin-user
              kubernetes.io/service-account.uid: 3fcb5af8-e776-4f81-82e3-1a2882099983
Type:  kubernetes.io/service-account-token
Data
====
ca.crt:     1025 bytes
namespace:  20 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6ImV4NzV6R0ljck1uUklpOGxYVk91bmhfZmpxZVlYV0h4STdDUV9PSGQwbTAifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLWhzNmR0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiIzZmNiNWFmOC1lNzc2LTRmODEtODJlMy0xYTI4ODIwOTk5ODMiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZXJuZXRlcy1kYXNoYm9hcmQ6YWRtaW4tdXNlciJ9.D2rj1pA0cNXqoa61NOXfv8A4EsHRZkLIAUr3fbgmz6W7yi95X8Ldd1sx1ytUBRSs7B-SkyxiPNTojOsouvbKpJ28jPGvINxjV7WWU_F0QHZK5M6iM-rAiHP5ckR6c_Qy6cbwDpoW2Cpc_7CP3Q6R3CcJEOMtd-u0_c_ZAb48y8KnR78IkKZPjh0YIDDmZONqSs3O2ESKdi_sKzckQ--Dcnti8vwPXIvNDExc30sNfFK22FM0UIU-Oa_Ur4Q4X6EjaedahvwoUFKpj3crAlFPYylv7zwsOQvi8Et3OF2c6j9ldTN9xivOzX8iLkNVFJlY5a_DjNWt7Kop_YmvleFvBA
~~~

### Supplemental File Content

#### 001_create-new-service-account.yml file content
    
~~~ yml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
~~~

#### 002_create-cluster-role-binding.yml file content

~~~ yml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
~~~

#### 003_create-http-service.yml file content

~~~ yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: http-svc
spec:
  replicas: 1
  selector:
    matchLabels:
      app: http-svc
  template:
    metadata:
      labels:
        app: http-svc
    spec:
      containers:
      - name: http-svc
        image: gcr.io/kubernetes-e2e-test-images/echoserver:2.1
        ports:
        - containerPort: 8080
        env:
          - name: NODE_NAME
            valueFrom:
              fieldRef:
                fieldPath: spec.nodeName
          - name: POD_NAME
            valueFrom:
              fieldRef:
                fieldPath: metadata.name
          - name: POD_NAMESPACE
            valueFrom:
              fieldRef:
                fieldPath: metadata.namespace
          - name: POD_IP
            valueFrom:
              fieldRef:
                fieldPath: status.podIP

---

apiVersion: v1
kind: Service
metadata:
  name: http-svc
  labels:
    app: http-svc
spec:
  ports:
  - port: 80
    targetPort: 8080
    protocol: TCP
    name: http
  selector:
    app: http-svc
~~~

#### 004_create-http-ingress.yml file content

~~~ yml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  annotations:
    # Enable client certificate authentication
    # nginx.ingress.kubernetes.io/auth-tls-verify-client: "off"
    # Create the secret containing the trusted ca certificates
    # nginx.ingress.kubernetes.io/auth-tls-secret: "default/ca-secret"
    # Specify the verification depth in the client certificates chain
    # nginx.ingress.kubernetes.io/auth-tls-verify-depth: "1"
    # Specify an error page to be redirected to verification errors
    # nginx.ingress.kubernetes.io/auth-tls-error-page: "http://www.mysite.com/error-cert.html"
    # Specify if certificates are passed to upstream server
    nginx.ingress.kubernetes.io/auth-tls-pass-certificate-to-upstream: "true"
  name: nginx-test
  namespace: default
spec:
  rules:
  - host: k8s-test-ingress.com
    http:
      paths:
      - backend:
          serviceName: http-svc
          servicePort: 80
        path: /
  tls:
  - hosts:
    - k8s-test-ingress.com
    secretName: k8s-test-ingress.com
~~~


#### k8s-test-ingress.com.crt file content

~~~ text
-----BEGIN CERTIFICATE-----
MIIEVTCCAr2gAwIBAgIQfd5u56cRVZxIhH8h2Z2OXTANBgkqhkiG9w0BAQsFADB/
MR4wHAYDVQQKExVta2NlcnQgZGV2ZWxvcG1lbnQgQ0ExKjAoBgNVBAsMIUNPRFlB
TkhPUk4tUENcY29keWFAQ09EWUFOSE9STi1QQzExMC8GA1UEAwwobWtjZXJ0IENP
RFlBTkhPUk4tUENcY29keWFAQ09EWUFOSE9STi1QQzAeFw0yMDEwMDMxMzUzNDFa
Fw0zMDEwMDMxMzUzNDFaMFUxJzAlBgNVBAoTHm1rY2VydCBkZXZlbG9wbWVudCBj
ZXJ0aWZpY2F0ZTEqMCgGA1UECwwhQ09EWUFOSE9STi1QQ1xjb2R5YUBDT0RZQU5I
T1JOLVBDMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAveL782jb3WBE
rOELGNG+aOmaM65UYYmRQsk4mJVRA4mkcV+6CP7zAeSIR00A8QAwPEhGPWVxZnLk
hl5/l1Q74VZeOV+Li8qARbkWFoRDBRGZDEa0u4VhG+Bk6znOdxrh5o2pGZHE78KC
qHdB3FfCJMMpLLmcmPE0NtfV6ScD6MyGF8cHgR4ioR4BE0MJ2YW7wMKWvDESI9Ti
CAvzqs/4p8GofIPnJtkPPYNzyjyYECtieFKSRf+AEgwS3Khwf00WtVCFHR6fyLkM
OtW251uh99ebEgfledEayL1YzjSmPHVhErCMWdLy1sRT7faIzSkr3OfMZKQGFEl2
8ADnbnaXlQIDAQABo3cwdTAOBgNVHQ8BAf8EBAMCBaAwEwYDVR0lBAwwCgYIKwYB
BQUHAwEwDAYDVR0TAQH/BAIwADAfBgNVHSMEGDAWgBQ4XbzEiX1Fp8N/oRfDSJbZ
tGYC2DAfBgNVHREEGDAWghRrOHMtdGVzdC1pbmdyZXNzLmNvbTANBgkqhkiG9w0B
AQsFAAOCAYEAWhDpGlalkY7Jeswq7/5uaPVH76E77na57KOZDyx7mz1rRZyqP9RV
GOv8hg+hr5ACYZ0MW/2tjV5AKvXeligKhMHmLIzkOfCf2hKlnxmqxzrN14Qmr+a6
iuQbu9GB+oPh2jZgq1jZrKf1G8FZ2hDmD899Ro9F32DS+aWq+BljLuSFtk/HE6zK
oOtLCPQCIs4C/aeIXD/j6GgXErkMDBG0JmpROcmN6NhtoXPnwyQGpvSwRw3RVWLz
zpExR1nPzkqgp5PW09icBIUEPrJ9gsTq3yt9+g9NCcrNmRY7kVzzXYDkcknJnWbj
OzETGN9CCDxKTEZMiLcAHqq8EgWj6G/cCTzQac6BRDenOCTyrPu4kX7cPjBFXVHs
jceHXyj7Imgv830eC1SPZK1qag4+D+V4Wdun8wY8VVEEmz/3y/VZtJPin0zmTpMm
RJ6mF76zzjazatXHe7LdicMCKgE2dVvlqdna2MYqss5X/n1zM4Ca8U20RqZeqLP7
hD4QyQB4D9Jl
-----END CERTIFICATE-----
~~~

#### k8s-test-ingress.com.key file content

~~~ text
-----BEGIN PRIVATE KEY-----
MIIEvgIBADANBgkqhkiG9w0BAQEFAASCBKgwggSkAgEAAoIBAQC94vvzaNvdYESs
4QsY0b5o6ZozrlRhiZFCyTiYlVEDiaRxX7oI/vMB5IhHTQDxADA8SEY9ZXFmcuSG
Xn+XVDvhVl45X4uLyoBFuRYWhEMFEZkMRrS7hWEb4GTrOc53GuHmjakZkcTvwoKo
d0HcV8IkwyksuZyY8TQ219XpJwPozIYXxweBHiKhHgETQwnZhbvAwpa8MRIj1OII
C/Oqz/inwah8g+cm2Q89g3PKPJgQK2J4UpJF/4ASDBLcqHB/TRa1UIUdHp/IuQw6
1bbnW6H315sSB+V50RrIvVjONKY8dWESsIxZ0vLWxFPt9ojNKSvc58xkpAYUSXbw
AOdudpeVAgMBAAECggEATvW4Nzt/UcraQ9lAuA1f1FhvWtY/GGAQG2l5M1nD2mi+
DLb1sQ/zFPJnCN8oaQ7e//I94wHv6d4U0Rsyi0bnr4gunkTwzixispuIZ8yP4eez
cLtmJCQOgX0J+haLmFOBZFG743oOHnUPx+XvaSTFAmx1DrgQOjjysWuG8/DZW1AY
PfxMmrz/G5da4kgW9SgK1oTJbZtqeZonKxsIHZp6mUd7asWraje9pfUgnPtQMlH3
IGMl94cRLZdtA1WBpHK9tLAZkOCysZhLg7q/j/M9x20hJNUbUz7MPco14oqcaCdw
lE3BHkiEd9ywUzCvbYeJ3EGdJPAo+AhWa3/iud1U8QKBgQDD8HVKER8RraFb1Qiz
UrklYRXo/TTCLz/vcrNVw+ne7KE6SuHB6pZWEhQo3rKIbbzoPMYfHRfCC/8V/4v8
RDwx0ic5e/M+T4x7eVQ5c+Ha4pcr18/vxu/dDJpUtTJdTFZlp2J2BJWYiKUlnr/m
Rt77CjQsaKu3AwYRkJuiBFuTmwKBgQD4F5H4I5dEH4B+EwapEU+THkJ0O8PWA6TC
cA0F1Dy1ogHDNJd0G1pIxCbmDGF4AMzaU/N/HwNkGZWMmcCvqAfRd2DsVGHLMEAE
3PO5skZvBOqutguemJhGXycTwm5S1PSgwKdtgdN6PZPs5GCoLXjAjNqwpMh+yUHB
2MeMChmsjwKBgQCDBFCJPDWYLo/MxgBRWCkxE2ABMP6MMegAhqPf32aMh5GvVs7q
SaBI4UHVqkOa8jX03F5mA6AVQsxIu12xSzcs4ScVSwp1Sd/X2GM3H4OQRx9qU55Y
6J8PIhQ4PAj3VcoXovs1iw80tXooU6RhqFYFaudEIqFfeIggSp+mkF9jrQKBgFzg
vhXuhRfMz1bjmo+62laScoB/S16YuJrORiHThfLdttk0nCqsfs1VGdbv9aFuc0Kd
QUBrBOL5rQIZIYjhWCP0FEYFhSMDakQnz9rKQhIX/h/wKUNzjzZxMvjzxkeeUALK
fSPDAb/2w6VhDkqH03gKg2i0GBdXExYWKQZlVZ1DAoGBALdeOR5oncjn4DdR1b8D
V+ba0j2Zsr1e49HD2XxQLXwSMDJkC8P4+o/jeLI6xiH0f6Ay/+D/MDH3Zf346r5k
0OX/FomX9vAZJ2MUeueBwSTGsXkMRk8NMMhyUECPpWjz4cR+XCz+OkJxhnlKntWe
m6K+NiiYev8m+c3cRaMJzvp9
-----END PRIVATE KEY-----
~~~