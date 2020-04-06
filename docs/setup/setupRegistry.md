# Private registry

This section will setup a local registry to hold container images.

```bash
helm template \
  --set image.tag=2.7.1 \
  --set ingress.enabled=true \
  --set ingress.hosts[0]=registry.bik8s.home \
  --set persistence.enabled=true \
  stable/docker-registry > ./registry.yaml
```

Allow cluster to pull from registry

on each node create a file **/etc/rancher/k3s/registries.yaml** with content.  To do this you need to modify **/var/lib/rancher/k3os/config.yaml and create or add to the write_files section:

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
