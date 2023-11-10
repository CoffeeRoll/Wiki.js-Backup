---
title: Cluster Setup
description: K3D Cluster Setup Notes
published: true
date: 2023-11-10T05:36:48.126Z
tags: k3d, k8s
editor: markdown
dateCreated: 2023-09-02T19:15:57.048Z
---

# Cluster Setup
How to setup and configure a bare-bones k3d cluster with custom certificates 

## Installation

1. See [K3D Website](https://k3d.io)

## Cluster Creation

1. Create Cluster and expose http/https ports
```k3d cluster create my-cluster-name --agents 1 --servers 1 -p "80:80@loadbalancer" -p "443:443@loadbalancer"```

## Additional Configuration

1. See [Traefik Config](/Kubernetes/Traefik/Traefik-Config)