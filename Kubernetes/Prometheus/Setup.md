---
title: Prometheus Setup
description: K8s Monitoring 
published: true
date: 2023-12-07T03:17:13.335Z
tags: k8s, prometheus
editor: markdown
dateCreated: 2023-12-07T03:17:13.335Z
---

# [WIP] K8s Monitoring with Prometheus

## kube-prometheus-stack
https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack

This helm chart deploys a configures prometheus instance with grafana and metrics systems.
It includes many preconfigured dashboards

It can be installed with this script
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

helm repo update

helm install prometheus prometheus-community/kube-prometheus-stack --namespace monitoring --create-namespace
```