# Certificate Manager

The certificate manager will issue certificates as needed by the cluster

To install the certificate manager follow the instructions:

```
kubectl create namespace cert-manager
kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v0.14.0/cert-manager.yaml
kubectl apply -f manifests/certManager/certManager_clusterIssuer.yaml
```
