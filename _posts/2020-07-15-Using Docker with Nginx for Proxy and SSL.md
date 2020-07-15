---
layout: post
author: Cody Merritt Anhorn
title: Using Docker with Nginx for a Locale Proxy and SSL
categories: [blog]
tags: [Docker, Nginx, Proxy, SSL]
---

This will be a quick article going over how to create a setup locally to have a Proxy server with SSL termination.

## Usage

I use it my self to make a secure platform environment for my game platform, this make my local development function very similar to how I will deploy it. Real urls, not working against IP's and having SSL in front of the URLs as well. This setup will require Docker and mkcert, I use Docker to host the nginx proxy server and I use mkcert to generate locally signed SSL certificates and it helps to manage the registration on my machine as well.


## Using mkcert create identity_server Cert

Start by following the install instruction provided by the [mkcert Github](https://github.com/FiloSottile/mkcert#installation) page. You need to install mkcert on your host machine so it can correctly setup your CA Root Certificates, these are used by your browser to verify SSL certificates. I have also included the steps for quick reference to help understand why each part needs to be done.

### Steps to Setup mkcert

#### Installs the mkcert CA Root Certificate locally, will be needed to tell your machine the SSL certs are verified.
~~~ bash
mkcert -install
~~~ 

#### Generates a single cert/key for all these domains, follow next steps to correctly format them.
The generated domains should also be added to your host file, pointing to your localhost domain, this will make your local development point to your proxy server.
~~~ bash
mkcert identity_server.com core_server.com zone_server.com client_server.com local_server.com
~~~

<br />
This will generate something similar to <code>identity_server.com+4.pem</code> and <code>identity_server.com+4-key.pem</code> based on how many domains you want to generate the certs for.

The generated certificate and key are NOT in the correct format to be used by the Nginx Proxy, mkcert will generate a single pem/key for all of the passed in domains. Below I have included a standard copy command that can be used to create the formatted domains, each domain will need to have its own file in the format shown below. After the copy, place the crt and key into the <code>./SSL-Proxy/certs</code> folder for your workspace, this will be used by Docker Compose to mount the certs, keys and CA Root Certificate into the Nginx/Proxy container.

<br />

#### Example Copy commands to generate correctly formatted Nginx cert/key
~~~ bash
cp ./identity_server.com+4.pem ./identity_server.com.crt
cp ./identity_server.com+4-key.pem ./identity_server.com.key
~~~
**Services:** ***identity_server.com***, ***core_server.com***, ***zone_server.com***, ***client_server.com***, ***local_server.com***

Move the generated CA Root Certificate into the certs folder, you can use the command below to figure out where mkcert created and placed your CA Root Certificate.

<br />
#### Where is my CA Root Certificate
~~~ bash
mkcert --CAROOT
>> /users/you/mkcert
~~~

Create a copy of the file and rename it to "rootCA.pem", then add it to this location in your workspace, <code>./SSL-Proxy/certs/rootCA.pem</code>. Docker Compose will mount this file, so it can be used by the proxy server to validate SSL requests.

<br />
#### Location to mount, used by Docker Compose up
~~~ bash
~/SSL-Proxy/certs/rootCA.pem
~~~

<br />

## Docker Compose Setup

Below is a docker-compose.yml file that shows of some configuration you can use to setup a container to be SSL terminated by the proxy server, how to setup the container to b also use the CA Root Certificate, setting up the "VIRTUAL_HOST" environment variable that will be used by the proxy server to route url requests to. The static IP address on the proxy server makes it easier to know what IP your proxy server it. This can come in handy when configuring your other containers and they need to be aware they are in a proxied request.

~~~ dockerfile
version: '3'
services:
    # Local Asset
    local:
        # This is needed to make sure the CA certs are updated so the backend calls to sites are
        #  successful, since a local CA is only available on the Host machine.
        entrypoint:
            [
                '/bin/bash',
                '-c',
                'update-ca-certificates; start-your-server.sh',
            ]
        image: your/server:latest
        environment:
            VIRTUAL_HOST: local_server.com
        ports:
            - 5555:80
        volumes:
            # This makes sure the container has the local rootCA, is registered on startup.
            - ./SSL-Proxy/certs/rootCA.pem:/usr/local/share/ca-certificates/ca.crt
    # SSL Proxy
    ssl-proxy:
        image: jwilder/nginx-proxy
        ports:
            - '80:80'
            - '443:443'
        volumes:
            - //var/run/docker.sock:/tmp/docker.sock:ro
            - ./SSL-Proxy/certs:/etc/nginx/certs
            - ./SSL-Proxy/proxy.conf:/etc/nginx/proxy.conf
        networks:
            default:
                ipv4_address: 172.20.0.200

networks:
    default:
        ipam:
            config:
                - subnet: 172.20.0.0/16
~~~

This is the configuration needed by the Nginx proxy server to correctly forward headers that the container can use to identify requests.

**Proxy Configuration**
~~~ bash
# HTTP 1.1 support
proxy_http_version 1.1;
proxy_buffering off;
proxy_set_header Host $http_host;
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection $proxy_connection;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $proxy_x_forwarded_proto;
proxy_set_header X-Forwarded-Ssl $proxy_x_forwarded_ssl;
proxy_set_header X-Forwarded-Port $proxy_x_forwarded_port;

# Mitigate httpoxy attack
proxy_set_header Proxy "";

# Custom Values Higher Buffer Size
proxy_buffer_size          128k;
proxy_buffers              4 256k;
proxy_busy_buffers_size    256k;
~~~

<hr />

If you have any problems or feedback, hit me up on [Twitter](https://twitter.com/CodyAnhorn) or my other contact details below.