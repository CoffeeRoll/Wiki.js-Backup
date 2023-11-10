---
title: Cluster Setup
description: K3D Cluster Setup Notes
published: true
date: 2023-11-10T05:18:02.690Z
tags: k3d, k8s
editor: markdown
dateCreated: 2023-09-02T19:15:57.048Z
---

# Cluster Setup
How to setup and configure a bare-bones k3d cluster with custom certificates 

## K3D Setup

1. Create Cluster
```k3d cluster create test-cluster --agents 1 --servers 1 -p "80:80@loadbalancer" -p "443:443@loadbalancer"```

