---
title: Traefik Config
description: Traefik v2
published: true
date: 2023-12-07T02:33:07.038Z
tags: 
editor: markdown
dateCreated: 2023-11-10T05:29:10.772Z
---

# Traefik v2 Notes

## Important Disclaimers

Official documentation can be found here: https://doc.traefik.io/traefik/

- **Much of the example configurations use ```apiVersion: traefik.us/v1alpha1"``` but it should be ```traefik.containo.us/v1alpha1```**

- This documentation only covers how to configure Traefik after it is already running in a k3d/k3s cluster. 
Any yaml samples found here can be applied to a cluster with ```kubectl apply -f some-config.yaml```

## Traefik Dashboard

1. Enable the dashboard
``` kubectl patch deploy -n kube-system traefik --type='json' -p='[{"op": "add", "path": "/spec/template/spec/containers/0/args/-", "value": "--api.dashboard=true"}]' ```

1. Optionally, expose the Dashboard on HTTP
``` kubectl patch deploy -n kube-system traefik --type='json' -p='[{"op": "add", "path": "/spec/template/spec/containers/0/args/-", "value": "--api.insecure=true"}]' ```

1. Apply an IngressRoute to access the dashboard
```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: dashboard
  namespace: kube-system
spec:
  entryPoints:
    - web # <-- only works if api.insecure = true
    - websecure
  routes:
  - match: Host(`1.2.3.4`) && (PathPrefix(`/dashboard`) || PathPrefix(`/api`))
    kind: Rule
    services:
    - name: api@internal
      kind: TraefikService
```

## TLS 

### Self-signed or Provided Certs

1. Patch Traefik to allow self-signed certs
``` sudo kubectl patch deploy -n kube-system traefik --type='json' -p='[{"op": "add", "path": "/spec/template/spec/containers/0/args/-", "value": "--serversTransport.insecureSkipVerify=true"}]' ```

2. Create a secret for the TLS certs
```sudo kubectl -n kube-system create secret tls "some-domain-tls-secret" --key="path/to/cert.key" --cert="path/to/cert.crt"```

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


### Let's Encrypt

1. See Traefik docs 
https://doc.traefik.io/traefik/https/acme/

## Strict SNI

Configuring this will prevent Traefik from serving any request that does not have an SNI header. The TRAEFIK DEFAULT CERT will not be used for extraneous requests.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: TLSOption
metadata:
  name: default
  namespace: kube-system
spec:
  sniStrict: true
```

## Rancher Setup [WIP]

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

### Uninstalling Rancher

1. Uninstall rancher-webhook
```helm -n cattle-system uninstall rancher-webhook```

1. Uninstall rancher
```helm -n cattle-system uninstall rancher```