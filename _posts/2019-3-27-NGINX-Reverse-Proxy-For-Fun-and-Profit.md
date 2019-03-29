---
layout: post
title: NGINX Reverse Proxy for Fun, Profit, and Load Balancing
subtitle: NGINX is SO much more than a web server!
gh-badge: [star, fork, follow]
---
# NGINX Reverse Proxy for Fun, Profit, and Load Balancing

At first glance, once may look at NGINX and immediately think of it as a simple HTTP web server for serving simple web requests or HTML sites.   Perhaps, if you've used NGINX in the past, you have fond memories (or nightmares) of the curly braces that define the various block definitions which make up the configuration syntax. NGINX is amazingly powerful language (yes, `language`)for defining web servers, load balancers, reverse proxies, and so many other use cases.  NGINX is fast and lightweight for installing directly on a server, yet nimble enough to be tossed into a `container` definition of Kubernetes workloads.  If you have spent any time in world of DevOps, automation, or infrastructure, I can almost gurantee there is a usecase for NGINX  in your infrastructure.

I started using NGINX a few years ago as a sidecar container for containerized web applications.   Using Docker Compose and Docker Swarm, I could easily whip up a *somewhat * production-ready web application using Django or Ruby on Rails in just a matter of minutes.  I quickly discovered that with the NGINX configuration language so easy to write, I could start testing with a simple configuration like the one below, with relative robustness:

```
upstream app {
    server rails:3000;
}
server {
    listen 80;

    location / {
        proxy_pass http://app;
        proxy_set_header Host                   $host;
        proxy_set_header X-Real-IP              $remote_addr;
        proxy_set_header X-Forwarded-For        $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto      $scheme;
        proxy_set_header X-Forwarded-Host       $host;
        proxy_set_header X-Forwarded-Port       $server_port;
    }
}
```
To test this configuration locally, you could easily drop a quick and dirty nginx container inside your `docker-compose.yml`, volume mount the nginx configuration into `/etc/nginx/conf.d/proxy.conf` like so:

```
version: '3'
services:
  rails:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: rails
    command: bin/rails s -p 3000 -b 0.0.0.0
 nginx:
    ports:
      - 80:80
    image: nginx:stable-alpine
    container_name: nginx
    volumes:
      - ./nginx/conf.d:/etc/nginx/conf.d/proxy.conf
    depends_on:
      - rails
```

Essentially, this works by exposing NGINX on port 80 to accept the connections initially. NGINX would then proxy requests to the rails container in the pod definition or docker service.  This provides a super fast way to quickly stand up web service for a development or staging environment without having to worry about exposing a misconfigured or non-production ready web server that might be built into your application's framework.   In Kubernetes world, a pod definition using an NGINX container to front your connectivity may look like the following, after defining the NGINX configuration configmaps earlier in the pod manifest:

```
...
    spec:
      volumes:
        - name: nginx-config
          configMap:
            name: nginx-configmap
            items:
              - key: proxy.conf
                path: proxy.conf
...
      containers:
      #NGINX Front End
        - name: my-nginx
          image: nginx:stable-alpine
          imagePullPolicy: IfNotPresent
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
          volumeMounts:
            - name: nginx-config
              mountPath: /etc/nginx/conf.d/
        #Rails Container
        - name: my-rails
          image: my-rails:latest
          imagePullPolicy: IfNotPresent
          command: ['bin/rails', 's', '-p', '3000', '-b', '0.0.0.0']
          ports:
            - name: rails
              containerPort: 3000
              protocol: TCP
```

Taking this example one step further, we could expand our NGINX configuration to include a liveliness or healthcheck endpoint. This would be useful for providing other dependent servies a quick touch-point with NGINX to ensure that the front-end is up and ready to accept traffic:

```
upstream app {
    server rails:3000;
}
server {
    listen 80;

    location / {
        proxy_pass http://app;
        proxy_set_header Host                   $host;
        proxy_set_header X-Real-IP              $remote_addr;
        proxy_set_header X-Forwarded-For        $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto      $scheme;
        proxy_set_header X-Forwarded-Host       $host;
        proxy_set_header X-Forwarded-Port       $server_port;
    }

    location /healthcheck {
        return 200 'It Works!';
        default_type text/plain;
    }
}



```

Of course, the idea here is not that these NGINX configs are production ready, but can be implemented quickly to stand up POC versions of web applications or other services with ease. Out of the box, NGINX is a very powerful web server and only requires a few tweaks to turn this into a production ready configuration.

In this last example, I was recently given the task at work to provide a way to integrate with a partner's web applcation. Suffice it to say, that we had a few limitions in the way their non-production environment was setup, so we were left having to send web requests using production host headers to compensate. As a work around, I setup /etc/hosts file entries inside of our Kubernetes pods. These entries allowed us to route traffic destined for `www.partner.com` to a t2.micro AWS instance running a single NGINX container with the following configuration:

```
worker_processes auto;
worker_rlimit_nofile 40000;

events {
    worker_connections 8192;
}

stream {
    upstream servers_http {
        server nonprod.partner.com:80 max_fails=3 fail_timeout=5s;
    }
    server {
        listen     80;
        proxy_pass servers_http;
    }

    upstream servers_https {
        server nonprod.partner.com:443 max_fails=3 fail_timeout=5s;
    }
    server {
        listen     443;
        proxy_pass servers_https;
    }
}

```

NGINX reverese proxy allowed us to quickly work around environmental limitations that were impacting the velocity in which we could integrate with their platform.  Using the flexibility of NGINX, we could quickly standup a stable, repeatible, just-works solution that requires absolutely no fiddling. NGINX is an absolutely fantastic solution to address many common infrastructure and DevOps usecases. If you're new to NGINX, I would encourage you to give `nginx:stable` a `docker pull` today!