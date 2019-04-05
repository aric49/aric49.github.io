---
layout: post
title: HackSaw ToolKit for Container Troubleshooting
subtitle: Troubleshoot Container instances from the inside-out. 
gh-badge: [star, fork, follow]
---
Containers are designed to be lightweight and fast to deploy.  Most authors of Dockerfiles opt for writing containers that contain as few packages as possible to streamline filesize and reduce dependencies.  As a result, many times you ned up with a container running in the wild that consists of a base image, application code, perhaps a few libraries and not much else. Often times, when these containers fail to resolve DNS properly, or stop hitting endpoints external to themselves, most administrators opt to pop into a shell on the container so they can troubleshoot from the same perspective as the application itself.  Most of the time, simple troubleshooting tools such as traceroute, ping, nslookup, and dig don't exist in the application container, for good reason.   These tools are extraneous and only increase the filesize and overhead of the container images built and pushed. Many admins (including myself), often find themselves `apt-get install`ing, or `apk add`ing these troubleshooting packages on the fly inside running containers. Normally, installing these packages is seen as harmless, since most of the time you'll end up killing and spinning up a new containers once the troubleshooting session is completed. However, when modifying a running container you are actually introducing variables, unaccounted for packages, and violating the principles of immutability of containers. This may sometimes lead to inconsistencies with how the application running inside the container behaves, giving a false sense of functionality. Moreover, when you deploy an updated version of that container, your troubleshooting tools are gone forcing you to install them again if the issue returns. After all, who can remember that installing ping using apt requires you to install the `inetutils-ping` package? 

To address this issue, I had the idea to write a toolkit container that has many of these common troubleshooting tools pre-installed and ready to run at a moments notice.   My goal for this project is to create a container that can be quickly deployed in Kubernetes or standard Docker as a one-liner and be torn down when the troubleshooting session is finished. This allows developers and admins to re-deploy their application container without distrubing the troubleshooting workflow within the toolkit container. Ellaborating on the "toolkit" metaphor futher, I decided to name this project, "HackSaw". Since it's a Tool (saw) for Hacking deployments.

HackSaw contains common preinstalled tools since as: `curl, nslookup, dig, ping, hping, nmap, netcat, openssl, tmux, git`. From the moment the container is launched, the goal is that the user has toolkit available with the most common troubleshooting tools for problem solving.  I have even started work on writing preset scripts for calling the Kubernetes API from directly inside the container itself, although this is still a work in progress.

To get started with HackSaw, you can spin it up locally inside Kubernetes or Docker using the following commands:

**Docker**:

```
docker run -it quay.io/aric49/hacksaw:latest
```

**Kubernetes**:

```
kubectl run hacksaw -it --namespace=your-namespace --image quay.io/aric49/hacksaw:latest
```

Once, it's running, open a shell into the container and start debugging:

**Docker**:

```
$ docker exec -it cocky_chatterjee /bin/sh
/workdir # ping google.com
PING google.com (172.217.2.46) 56(84) bytes of data.
64 bytes from atl14s78-in-f14.1e100.net (172.217.2.46): icmp_seq=1 ttl=53 time=7.62 ms
^X64 bytes from atl14s78-in-f14.1e100.net (172.217.2.46): icmp_seq=2 ttl=53 time=7.67 ms
64 bytes from atl14s78-in-f14.1e100.net (172.217.2.46): icmp_seq=3 ttl=53 time=7.98 ms
^C
--- google.com ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 6ms
rtt min/avg/max/mdev = 7.619/7.757/7.983/0.176 ms

```

**Kubernetes**:

```
$ kubectl exec -it --namespace hacksaw hacksaw-d7867ccbf-ck4hp -- /bin/sh 
/workdir # dig kubernetes.default.svc.cluster.local

; <<>> DiG 9.12.3 <<>> kubernetes.default.svc.cluster.local
;; global options: +cmd
;; Got answer:
;; WARNING: .local is reserved for Multicast DNS
;; You are currently testing what happens when an mDNS query is leaked to DNS
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 51584
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;kubernetes.default.svc.cluster.local. IN A

;; ANSWER SECTION:
kubernetes.default.svc.cluster.local. 18 IN A   10.43.0.1

;; Query time: 1 msec
;; SERVER: 10.43.0.10#53(10.43.0.10)
;; WHEN: Fri Apr 05 15:54:09 UTC 2019
;; MSG SIZE  rcvd: 81
```

Feel free to checkout my repo at [GitHub](https://github.com/aric49/hacksaw)

*Happy Hacking!*