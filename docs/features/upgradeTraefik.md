# Ingress

K3OS comes with Traefik, which works well as a cluster ingress, however the version shipped is not current and the V2.x releases of Traefik support additional options, such as UDP routing, so need to investigate updating to the latest release of Traefik or using a different ingress technology

Options

- Remove bundled Traefik and replace with latest version
- nginx ingress

## Investigation notes:

Option to switch out Traefik version to latest:

Current config:

```yaml
apiVersion: helm.cattle.io/v1
kind: HelmChart
metadata:
  name: traefik
  namespace: kube-system
spec:
  chart: https://%{KUBERNETES_API}%/static/charts/traefik-1.81.0.tgz
  set:
    rbac.enabled: "true"
    ssl.enabled: "true"
    metrics.prometheus.enabled: "true"
    kubernetes.ingressEndpoint.useDefaultPublishedService: "true"
    dashboard.enabled: "true"
    dashboard.domain: "traefik.bik3s.home"
    ssl.insecureSkipVerify: "true
```

Test config:

```yaml
apiVersion: helm.cattle.io/v1
kind: HelmChart
metadata:
  name: traefik
  namespace: kube-system
spec:
  chart: traefik/traefik
  repo: https://containous.github.io/traefik-helm-chart
  targetNamespace: kube-system
  set:
    rbac.enabled: "true"
    ssl.enabled: "true"
    metrics.prometheus.enabled: "true"
    kubernetes.ingressEndpoint.useDefaultPublishedService: "true"
    dashboard.enabled: "true"
    dashboard.domain: "traefik.bik3s.home"
    ssl.insecureSkipVerify: "true"
```