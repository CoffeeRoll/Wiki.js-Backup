---
title: Exposing Services
description: K3D Exposing Services Notes
published: true
date: 2023-06-11T21:27:22.694Z
tags: k3d, k8s
editor: markdown
dateCreated: 2023-06-10T19:32:11.658Z
---

# Exposing Services
## HTTP Services

1. Create the cluster and expose ports 80 and 443 to the loadbalancer (host:container)
```k3d cluster create test-cluster --agents 1 --servers 1 ---port '80:80@loadbalancer' ---port '443:443@loadbalancer'```

1. Use IngressRoute to define HTTP routes for traefik

## Non-HTTP Services
1. Create the cluster and expose custom ports to the loadbalancer (host:container)
```k3d cluster create test-cluster --agents 1 --servers 1 ---port '5432:5432@loadbalancer'```
1. Ports can be opened on an existing cluster using 
```k3d cluster edit --port-add"```
For more info, refer to the docs: https://k3d.io/v5.0.1/usage/commands/k3d_cluster_edit/
1. Add the custom port to the traefik deployment
1. Add an entrypoint to traefik for that port
1. Add the custom port to the traefik service
1. Use IngressRouteTCP to define TCP routing rules
1. Use IngressRouteUDP to define UDP routing rules
