---
title: Cluster Setup
description: K3D Cluster Setup Notes
published: true
date: 2023-09-05T23:25:55.923Z
tags: k3d, k8s
editor: markdown
dateCreated: 2023-09-02T19:15:57.048Z
---

# Cluster Setup
## K3D Setup

1. Create Cluster
```k3d cluster create test-cluster --agents 1 --servers 1 -p "80:80@loadbalancer" -p "443:443@loadbalancer"```

## TLS 
Still Researching 

### Self-signed (No CA)

1. Patch Traefik to allow self-signed certs
``` sudo kubectl patch deploy -n kube-system traefik --type='json' -p='[{"op": "add", "path": "/spec/template/spec/containers/0/args/-", "value": "--serversTransport.insecureSkipVerify=true"}]' ```

2. Create a secret for the TLS certs
```sudo kubectl -n kube-system create secret tls "ingress-tls" --key="ingress.key" --cert="ingress.crt"```

3. Create a TLSStore named "default" - Any other name will not work [See Traefik Docs](https://doc.traefik.io/traefik/routing/providers/kubernetes-crd/#kind-tlsstore)
  - List all tls secrets in the default store and Traefik will automatically serv the correct one based on the incomming request's SNI
  - Requests that do not match any given cert will be provided with the TRAEFIK DEFAULT CERT
```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: TLSStore
metadata:
  name: default
  namespace: kube-system

spec:
  certificates:
    - secretName: some-domain-tls-secret
    - secretName: some-other-domain-tls-secret
```

4. Apply IngressRoutes as normal 

### Self-signed (Custom CA)

1. Import CA
2. Unsure if differnt than with no-CA

### Let's Encrypt

1. See Traefik docs 
https://doc.traefik.io/traefik/https/acme/

## Rancher Setup

1. Create Namespace for Rancher
```kubectl create namespace cattle-system```

1. Create Secret for TLS
```kubectl -n cattle-system create secret tls tls-rancher --cert=certs/certs-output/tls.crt --key=certs/certs-output/tls.key```

1. Add Rancher Helm Repository
```helm repo add rancher-stable https://releases.rancher.com/server-charts/stable```

1. Install Rancher Chart
```helm install rancher rancher-<CHART_REPO>/rancher --namespace cattle-system --set hostname=<server.hostname> --set replicas=1 --set ingress.tls.source=secret```

1. Wait for Rancher to Start
```kubectl -n cattle-system rollout status deploy/rancher```

## Uninstalling

1. Uninstall rancher-webhook
```helm -n cattle-system uninstall rancher-webhook```

1. Uninstall rancher
```helm -n cattle-system uninstall rancher```