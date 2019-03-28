---
layout: post
title: NGINX Reverse Proxy for Fun, Profit, and Load Balancing 
subtitle: NGINX is SO much more than a web server!
gh-badge: [star, fork, follow]
---
# NGINX Reverse Proxy for Fun, Profit, and Load Balancing 

At first glance, once may look at NGINX and immediately think of it as a simple HTTP web server for serving simple web requests or HTML sites.   Perhaps, if you've used NGINX in the past, you have fond memories (or nightmares) of the curly braces that define the various block definitions which make up the configuration syntax. NGINX is amazingly powerful language (yes, `language`)for defining web servers, load balancers, reverse proxies, and so many other use cases.  NGINX is fast and lightweight for installing directly on a server, yet nimble enough to be tossed into a `container` definition of a Kubernetes pods.  If you have spent any time in world of DevOps, automation, or infrastructure, I can almost gurantee there is a usecase for NGINX that you not have even thought of yet. 

I started using NGINX a few years ago as a sidecar container for containerized web applications.   Using Docker Compose and Docker Swarm, I could easily whip up a production-ready web application using Django or Ruby on Rails in just a matter of minutes.  I quickly discovered that with the NGINX configuration language so easy to write, I could start testing with a simple configuration like the one below, with relative robustness: 

```
config goes here
```

Essentially, NGINX would accept connections initially, and proxy requests to the container in the pod definition or service which is serving the actual content.   This provides a super fast way to quickly stand up web service for a development or staging environment without having to worry about exposing a misconfigured or non-production ready web server that might be built into your application's framework.   In Kubernetes world, a pod definition using an NGINX container to front your connectivity may look like the following:

```
```

Taking this example one step further, we could expand our NGINX configuration to include a liveliness or healthcheck endpoint. This would be useful for providing other dependent servies a quick touch-point with NGINX to ensure that the front-end is up and ready to accept traffic:

```

```

You could even create a reverse proxy endpoint in NGINX that would allow you to check the readiness of the web application, while continuing to only expose NGINX to incoming web requests:

```
```

In this last example, I was recently given the task at work to provide a way to integrate with a partner's web applcation. Suffice it to say, that we had a few limitions in the way their non-production environment was setup, so we were left having to send web requests using production host headers to compensate.   Using /etc/hosts file entries inside of a Kubernetes pod-spec, we could send traffic heading for `wwww.partner.com` towards an NGINX reverse proxy standalone container, which in turn was configured to point to their nonproduction environment using the following config:

```
```

NGINX allowed us to quickly work around environmental limitations that were impacting the velocity in which we could integrate with their platform.  Using the flexibility of NGINX, we could quickly standup a stable, repeatible, just-works solution that requires absolutely no fiddling. 
