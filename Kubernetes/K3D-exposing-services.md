---
title: K3D Exposing Services
description: Guide for exposing different types of services in a K3D cluster
published: true
date: 2023-03-10T01:07:42.156Z
tags: 
editor: markdown
dateCreated: 2023-03-10T01:07:42.156Z
---

# K3D Exposing Services
## HTTP Services

1. Create the cluster and expose ports 80 and 443 to the loadbalancer (host:container)
```k3d cluster create test-cluster --agents 1 --servers 1 ---port '80:80@loadbalancer' ---port '443:443@loadbalancer'```

1. Use IngressRoute to define HTTP routes for traefik

## Non-HTTP Services
1. Create the cluster and expose custom ports to the loadbalancer (host:container)
```k3d cluster create test-cluster --agents 1 --servers 1 ---port '5432:5432@loadbalancer'```
1. Add the custom port to the traefik deployment
1. Add an entrypoint to traefik for that port
1. Add the custom port to the traefik service
1. Use IngressRouteTCP to define TCP routing rules
