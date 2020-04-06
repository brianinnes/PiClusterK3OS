# Private registry

This section will setup a local registry to hold container images.  To add the registry to the cluster enter the following command:

```bash
kubectl apply -f manifests/registry/registry.yaml
```

You now need to allow the cluster nodes to pull from registry, which is currently insecure (self-signed TLS certificate)

On each node create a file **/etc/rancher/k3s/registries.yaml** with content.  To do this you need to modify **/var/lib/rancher/k3os/config.yaml** and create or add to the write_files section:

```yaml
write_files:
- encoding: ""
  content: |-
    mirrors:
      docker.io:
        endpoint:
          - "http://registry.bik8s.home"
  owner: root:root
  path: /etc/rancher/k3s/registries.yaml
  permissions: '0644'
```
