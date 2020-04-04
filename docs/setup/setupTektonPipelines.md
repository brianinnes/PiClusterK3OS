# Tekton Pipelines

Like most Google originated projects, the tutorials and instructions on the open source site are polluted with Google commercial offerings!  It is also a single architecture project, so to make it work it containers need to be constrained to **amd64** architectures nodes.

To setup Tekton execute the following instructions:

```bash
kubectl apply -f https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml
```

To constrain the images to amd64 nodes, modify the deployments (this can be done in the Kubernetes dashboard or on the command line.).  Add the following to all the deployments in the **tekton-pipelines** namespace

```yaml
nodeSelector:
  kubernetes.io/arch: amd64
```

## Artiface Storage

Tekton needs storage for artifacts.  This can be a persistent volume or object storage.  For the K3OS cluster we already have persistent volumes setup.  Enter the commands below to use persistent volumes:

```bash
kubectl apply -f -<<EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: config-artifact-pvc
  namespace: tekton-pipelines
data:
  size: 10GiB
  storageClassName: default
EOF
```

## Tekton CLI

To work with Tekton you need to have the CLI installed.  To installed the CLI on your system follow the [instructions](https://github.com/tektoncd/cli) for your workstation/laptop OS.
