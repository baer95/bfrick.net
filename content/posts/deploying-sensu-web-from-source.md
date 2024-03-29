+++ 
draft = false
date = 2021-03-23T11:09:31+02:00
title = "Deploying sensu/web from source"
description = ""
slug = "deploying-sensu-web-from-source"
author = "Bernhard Frick"
tags = ["nginx", "docker", "sensu"]
categories = ["monitoring", "containers"]
externalLink = ""
series = []
+++

This is a guide to deploying https://github.com/sensu/web from source using Docker and nginx as a reverse proxy.

## Why?

**Limitations**

Sensu does already offer a Docker Image for Sensu Enterprise at https://hub.docker.com/r/sensu/sensu, complete with
`sensu-agent`, `sensu-backend` and an included dashboard. However, the free version of Sensu Enterprise is limited to
100 agents. To support more agents, one has to compile `sensu/sensu-go` from source, which means that the web dashboard
also has to be deployed separately.

Building `sensu/sensu-go` from source is covered here: https://github.com/sensu/sensu-go#building-from-source.

**Resource usage**

The [deployment guide](https://github.com/sensu/web/blob/master/INSTALL.md) recommends running sensu/web in production
by executing:
```shell
node scripts serve
```
which starts an express.js server. It watches the source code for changes, recompiles the changed files on-the-fly and
hot-reloads the webpage to greatly facilitate local development.

However, this is by far not a production-ready deployment for the following reasons:

* Takes ~5 minutes to compile and start
* Consumes ~1-2 GB of RAM
* Watches ~130.000 files for changes to re-compile and hot-reload - the guide mentions this as an unresolved bug and
  provides a workaround to increase the maximum number of file watches in the OS

Serving static files using nginx is a lot less resource-intensive: the Docker image size is reduced to 71 MB from ~2 GB
for an image with node and all dependencies, and the running container consumes almost no memory or CPU resources at all
compared to running a full node/express/webpack application.

## How?

**Static build**

When running the dev server, `sensu/web` proxies http requests to certain paths to the `sensu-backend` API, which can be
configured through the environment variable `API_URL`.

When building a static website, there is no server-side processing that could proxy requests to the `sensu-backend` API,
so we need nginx to do that. We need nginx anyway to serve the static files, so no problem here.

**Nginx Reverse Proxy**

We are going to replace the dev server with an nginx reverse proxy. The configuration of the dev server as seen in
https://github.com/sensu/web/blob/master/scripts/serve.js#L25 is used as a basis for the nginx configuration.

As seen in line 25, requests to `/auth` `/graphql` and `/api` need to be forwarded to the `sensu-backend` API. All other
requests should be forwarded to the deployment of `sensu/web`.

Deploy an nginx server at http://sensu.example.com with the following configuration:

```nginx configuration
upstream sensu-backend {
    server sensu-backend.example.com:8080;
}

upstream sensu-web {
    server sensu-web.example.com:5000;
}

server {
    listen 80;
    server_name sensu.example.com;

    location ~ ^/(auth|graphql|api) {
        proxy_pass http://sensu-backend;
    }

    location / {
        proxy_pass https://sensu-web;
    }
}
```

**The Dockerfile**

Since `sensu/web` is a SPA, we need to ensure that all incoming http requests are rewritten to `/index.html` as seen in
line 32 of the dev server by the use of the node package `connect-history-api-fallback`. For that, we use another custom
nginx configuration that we supply during the docker build. Here we also enable compression and caching for static
assets to enable faster load times:

```nginx configuration
server {
    listen 5000;
    server_name localhost;

    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
        error_page 404 = /index.html;
    }

    location ~ ^/(static)/ {
        gzip_static on;
        gzip_types text/xml text/css text/javascript application/x-javascript;
        expires max;
    }
}
```

We are using a multi-stage Dockerfile to clone the source, install dependencies, build the static site and then create a
docker image based on `nginx:alpine` that just contains the static build output:

```dockerfile
FROM node:14-alpine as build

WORKDIR /home/node/sensu/web

ENV NODE_ENV="production"

ARG VERSION

RUN apk add git && \
    git clone --depth 1 --branch "${VERSION}" https://github.com/sensu/web.git . && \
    yarn install --network-timeout 100000 && \
    yarn build

FROM nginx:alpine

COPY --from=build /home/node/sensu/web/build/app/ /usr/share/nginx/html

COPY ./etc/nginx/ /etc/nginx/

EXPOSE 5000
```

**Deployment**

Build the Dockerfile:
```shell
docker build --build-arg VERSION="v1.1.0" --tag "sensu-web-docker:v1.1.0" .
```

Run the image:
```shell
docker run -d --rm -p 5000:5000 sensu-web-docker:v1.1.0
```

## Is it working?

We are running a Sensu Go Cluster with 5 `sensu-backend` nodes and ~700 agents to monitor all servers related to a
production application with ~2M monthly active users. Our deployment of `sensu/web` is running on an Openshift cluster,
using the exact configuration described above, and it just works.

---

The code can be found here: https://github.com/baer95/sensu-web-docker
