# Knative

This section will install and configure Knative Serving.  Eventing will be left for another time

Unfortunately the Knative developers have yet to embrace multi-arch images, so they assume all deployments will be on the amd64 architecture.  This means that we need to constrain the images to amd64 nodes until multi-arch images are available

```bash
kubectl apply -f https://github.com/knative/serving/releases/download/v0.13.0/serving-crds.yaml
kubectl apply -f https://github.com/knative/serving/releases/download/v0.13.0/serving-core.yaml
```

To constrain the images to amd64 nodes, modify the deployments (this can be done in the Kubernetes dashboard or on the command line.).  Add the following to all the deployments in the **knative-serving** namespace

```yaml
nodeSelector:
  kubernetes.io/arch: amd64
```

## Enough Istio for Knative

As part of the Knative install there needs to be a networking layer installed.  Istio is the layer chosen for this project, as it appears to be the network layer of choice for most commercial deployments.

Run the following commands:

```bash
export ISTIO_VERSION=1.5.1
curl -L https://git.io/getLatestIstio | sh -
cd istio-${ISTIO_VERSION}
for i in install/kubernetes/helm/istio-init/files/crd*yaml; do kubectl apply -f $i; done
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Namespace
metadata:
  name: istio-system
  labels:
    istio-injection: disabled
EOF
helm template --namespace=istio-system \
  --set prometheus.enabled=false \
  --set mixer.enabled=false \
  --set mixer.policy.enabled=false \
  --set mixer.telemetry.enabled=false \
  --set pilot.sidecar=false \
  --set pilot.resources.requests.memory=128Mi \
  --set galley.enabled=false \
  --set global.useMCP=false \
  --set security.enabled=false \
  --set global.disablePolicyChecks=true \
  --set sidecarInjectorWebhook.enabled=false \
  --set global.proxy.autoInject=disabled \
  --set global.omitSidecarInjectorConfigMap=true \
  --set gateways.istio-ingressgateway.autoscaleMin=1 \
  --set gateways.istio-ingressgateway.autoscaleMax=2 \
  --set pilot.traceSampling=100 \
  --set global.mtls.auto=false \
  install/kubernetes/helm/istio \
  > ./istio-lean.yaml
kubectl apply -f istio-lean.yaml
```

Again we need to add a node selector to the Deployment to only target **amd64** architecture nodes, as Istio is only available for this architecture:

```yaml
nodeSelector:
  kubernetes.io/arch: amd64
```

Update the DNS config by running command ```kubectl get svc -nistio-system``` and noting the external IP address of the **istio-ingressgateway**.

Edit the config-domain ConfigMap in the **knative-serving** namespace and add the following in the data section:

```yaml
  192.168.201.201.xip.io: ''
```

Replacing the IP address with the external IP address of the **istio-ingressgateway**

## Add Knative Istio controller

Enter the following commands:

```bash
kubectl apply --filename https://github.com/knative/serving/releases/download/v0.13.0/serving-istio.yaml
```

and once again go and alter the deployment to constrain to **amd64** architecture nodes:

```yaml
nodeSelector:
  kubernetes.io/arch: amd64
```

Once everything is running setup the DNS for Knative with the following command:

```bash
kubectl apply --filename https://github.com/knative/serving/releases/download/v0.13.0/serving-default-domain.yaml
```

*Knative Eventing is not going to be installed at this point - it is left for a future exercise*
