# Metrics Server Installation & Fix Guide (Kubernetes)

This document explains how to install the Kubernetes **metrics-server**
and fix the common issue:

    False (MissingEndpoints)

which prevents `kubectl top` from working.

## 1. Install Metrics Server

``` bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

Verify status:

``` bash
kubectl get apiservice | grep metrics
```

If output shows:

    False (MissingEndpoints)

follow the fixes below.

## 2. Fix MissingEndpoints Issue

Local clusters (kubeadm, VMware, Minikube, Docker, etc.) often require
custom flags because kubelet uses self-signed certificates.

### 2.1 Add `--kubelet-insecure-tls`

``` bash
kubectl -n kube-system patch deployment metrics-server   --type='json'   -p='[
    {
      "op": "add",
      "path": "/spec/template/spec/containers/0/args/-",
      "value": "--kubelet-insecure-tls"
    }
  ]'
```

### 2.2 Add Preferred Kubelet Address Types

``` bash
kubectl -n kube-system patch deployment metrics-server   --type='json'   -p='[
    {
      "op": "add",
      "path": "/spec/template/spec/containers/0/args/-",
      "value": "--kubelet-preferred-address-types=InternalIP,Hostname,InternalDNS,ExternalDNS,ExternalIP"
    }
  ]'
```

## 3. Restart Metrics Server

``` bash
kubectl -n kube-system rollout restart deployment metrics-server
```

## 4. Verify Fix

``` bash
kubectl get apiservice | grep metrics
```

Expected result:

    v1beta1.metrics.k8s.io   True

## 5. Test Metrics

``` bash
kubectl top nodes
kubectl top pods -A
```

## 6. Troubleshooting (Optional)

Check logs:

``` bash
kubectl -n kube-system logs -l k8s-app=metrics-server
```
