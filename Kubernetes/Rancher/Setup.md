---
title: Rancher Setup
description: Rancher Setup Notes
published: true
date: 2023-12-09T21:22:34.134Z
tags: 
editor: markdown
dateCreated: 2023-12-09T21:22:34.134Z
---

# Rancher Setup [WIP]

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

## Uninstalling Rancher

1. Uninstall rancher-webhook
```helm -n cattle-system uninstall rancher-webhook```

1. Uninstall rancher
```helm -n cattle-system uninstall rancher```