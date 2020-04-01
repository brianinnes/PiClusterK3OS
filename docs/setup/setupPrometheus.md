# Prometheus

Prometheus is a monitoring tool.  To install it on your cluster enter the following commands:

**NOTE: Not all images are multi-arch, so this will not work without some modification**

```bash
kubectl create namespace monitoring
helm install prometheus-operator stable/prometheus-operator --namespace monitoring
```
