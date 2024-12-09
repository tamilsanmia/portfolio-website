---
layout: post
title:  "Install Prometheus Operator with Grafana"
date:   2024-07-18
category: DevOps
image: assets/img/blog/prometheus-grafana.png
author: TamilSelvan M
tags: DevOps
---


# Setting Up Prometheus Operator with Helm and Nginx Ingress


## Prerequisites
- Kubernetes cluster up and running
- `kubectl` configured to interact with your cluster
- `helm` installed
- Cert-manager and Nginx Ingress controller installed on the cluster

### 1. Create a new namespace for Prometheus Operator 

Create a new namespace called `monitoring`:
```sh
kubectl create namespace monitoring
```

### 2. Add the Prometheus Operator Helm repository

Add the Prometheus community Helm repository and update it:
```sh
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

### 3. Install the Prometheus Operator chart from the prometheus-community repository:

Install the Prometheus Operator chart from the prometheus-community repository into the monitoring namespace:

```sh
helm install prometheus prometheus-community/kube-prometheus-stack --namespace monitoring --create-namespace
```

### 4. Configure Nginx Ingress for Prometheus (optional)

Create an Ingress resource for Prometheus by creating a file named `prometheus-ingress.yml` with the following content:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: prometheus-ingress
  namespace: monitoring
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/rewrite-target: /

spec:
  ingressClassName: nginx
  rules:
  - host: prometheus.faveocloud.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: prometheus-kube-prometheus-prometheus
            port:
              number: 9090
  tls:
  - hosts:
    - prometheus.faveocloud.com
    secretName: prometheus-tls
```
Apply the Ingress resource
```sh
kubectl apply -f prometheus-ingress.yml
```

### 5. Configure Nginx Ingress for Grafana

Create an Ingress resource for Grafana by creating a file named `grafana-ingress.yml` with the following content:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: grafana-ingress
  namespace: monitoring
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/rewrite-target: /

spec:
  ingressClassName: nginx
  rules:
  - host: grafana.faveocloud.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: prometheus-grafana
            port:
              number: 80
  tls:
  - hosts:
    - grafana.faveocloud.com
    secretName: grafana-tls
```
Apply the Ingress resource
```sh
kubectl apply -f grafana-ingress.yml
```
### 6. Obtain Grafana Admin Password

Retrieve the Grafana admin password:

```sh
kubectl get secret --namespace monitoring prometheus-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```
### 7. Verify TLS Certificate Issuance

Verify that the TLS certificates for Prometheus and Grafana are issued correctly:

```sh
kubectl get certificates -n monitoring
kubectl describe certificate prometheus-tls -n monitoring
kubectl describe certificate grafana-tls -n monitoring
```
These steps should help you set up Prometheus and Grafana with Ingress and TLS certificates in your Kubernetes cluster.
