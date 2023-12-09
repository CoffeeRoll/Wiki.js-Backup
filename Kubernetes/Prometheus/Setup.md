---
title: Prometheus Setup
description: K8s Monitoring 
published: true
date: 2023-12-09T19:03:18.144Z
tags: k8s, prometheus
editor: markdown
dateCreated: 2023-12-07T03:17:13.335Z
---

# [WIP] K8s Monitoring with Prometheus

## kube-prometheus-stack
https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack

This helm chart deploys a configures prometheus instance with grafana and metrics systems.
It includes many preconfigured dashboards

### Installing from Helm Chart
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

helm repo update

helm install prometheus prometheus-community/kube-prometheus-stack --namespace monitoring --create-namespace
```

### Installing from Local Helm Chart 

```
git clone https://github.com/prometheus-community/helm-charts.git

cd helm-charts/charts/kube-prometheus-stack

helm install prometheus . --namespace monitoring --create-namespace
```

### Configuring Values.yaml

#### Running from a subdomain

The only thing that needs to be updated when running grafana from a subdomain is the ingress

Below is a snippet from the values.yaml file fromnano  the helm chart.

```yaml
grafana:
  ingress:
    ## If true, Grafana Ingress will be created
    ##
    enabled: true       # << Ensure this is true

    ## IngressClassName for Grafana Ingress.
    ## Should be provided if Ingress is enable.
    ##
    ingressClassName: traefik    # << UPDATE THIS (Not sure if this is important though)

    ## Annotations for Grafana Ingress
    ##
    annotations: {}
      # kubernetes.io/ingress.class: nginx
      # kubernetes.io/tls-acme: "true"

    ## Labels to be added to the Ingress
    ##
    labels: {}

    ## Hostnames.
    ## Must be provided if Ingress is enable.
    ##
    # hosts:
    #   - grafana.domain.com
    hosts:
      - grafana.coffeeroll.dev  # << UPDATE THIS

    ## Path for grafana ingress
    path: /                     # << Must be /

    ## TLS configuration for grafana Ingress
    ## Secret must be manually created in the namespace
    ##
    tls: []
    # - secretName: grafana-general-tls
    #   hosts:
    #   - grafana.example.com

```

#### Running from a Sub Path

This is a snippet of the values.yaml that is found in the kube-prometheus-stack helm chart.
The piece that needs to be added is the "env" section to configure grafana. 
The "env" section s does not exist by default, but the values will be respected. 
The entire "grafana" section will be used to overrite values in the grafana chart. 

```yaml
...
## Using default values from https://github.com/grafana/helm-charts/blob/main/charts/grafana/values.yaml
##
grafana:
  enabled: true
  namespaceOverride: ""

  env:
    GF_DOMAIN: coffeeroll.dev
    GF_SERVER_ROOT_URL: https://coffeeroll.dev/grafana/
    GF_SERVER_SERVE_FROM_SUB_PATH: true
    GF_ENFORCE_DOMAIN: false
...
```

additional, the ingress section needs to be updated:

```yaml
grafana:
  ingress:
    ## If true, Grafana Ingress will be created
    ##
    enabled: true       # << Ensure this is true

    ## IngressClassName for Grafana Ingress.
    ## Should be provided if Ingress is enable.
    ##
    ingressClassName: traefik    # << UPDATE THIS (Not sure if this is important though)

    ## Annotations for Grafana Ingress
    ##
    annotations: {}
      # kubernetes.io/ingress.class: nginx
      # kubernetes.io/tls-acme: "true"

    ## Labels to be added to the Ingress
    ##
    labels: {}

    ## Hostnames.
    ## Must be provided if Ingress is enable.
    ##
    # hosts:
    #   - grafana.domain.com
    hosts:
      - coffeeroll.dev  # << UPDATE THIS

    ## Path for grafana ingress
    path: /grafana      # << UPDATE THIS

    ## TLS configuration for grafana Ingress
    ## Secret must be manually created in the namespace
    ##
    tls: []
    # - secretName: grafana-general-tls
    #   hosts:
    #   - grafana.example.com

```


### Other Notes

#### Configuring helm install without local chart

A values file can be provided that will be applied to the chart's values.yaml file. 
Values provided via the command line have priority over existing values.

```
helm install prometheus prometheus-community/kube-prometheus-stack --namespace monitoring --create-namespace -f grafana-values.yaml
```

grafana-values.yaml

```yaml
grafana:
  enabled: true
  namespaceOverride: ""

  env:
    GF_DOMAIN: coffeeroll.dev
    GF_SERVER_ROOT_URL: https://coffeeroll.dev/grafana/
    GF_SERVER_SERVE_FROM_SUB_PATH: true
    GF_ENFORCE_DOMAIN: false

  ingress:
    ## If true, Grafana Ingress will be created
    ##
    enabled: true       # << Ensure this is true

    ## IngressClassName for Grafana Ingress.
    ## Should be provided if Ingress is enable.
    ##
    ingressClassName: traefik    # << UPDATE THIS (Not sure if this is important though)

    ## Annotations for Grafana Ingress
    ##
    annotations: {}
      # kubernetes.io/ingress.class: nginx
      # kubernetes.io/tls-acme: "true"

    ## Labels to be added to the Ingress
    ##
    labels: {}

    ## Hostnames.
    ## Must be provided if Ingress is enable.
    ##
    # hosts:
    #   - grafana.domain.com
    hosts:
      - coffeeroll.dev  # << UPDATE THIS

    ## Path for grafana ingress
    path: /grafana      # << UPDATE THIS

    ## TLS configuration for grafana Ingress
    ## Secret must be manually created in the namespace
    ##
    tls: []
    # - secretName: grafana-general-tls
    #   hosts:
    #   - grafana.example.com
```


#### Traefik Ingress
the helm chart comes with a functional ingress, but it can also be exposed with an IngressRoute like

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: grafana-ingress
  namespace: monitoring
spec:
  entryPoints:
    - web
    - websecure
  routes:
  - match: Host(`grafana.my-site.com`)
    kind: Rule
    services:
    - name: prometheus-grafana
      port: 80
      kind: Service
```

