# FAQ

This document details lessons learn when working with K3OS and K3S

## Deleting a node

When deleting a node **kubectrl delete node**vi  doesn't remove the K3S password from the server (master) node, so if you want to add a new node with the same hostname as a previously deleted node, then you need to manually remove the K3S password from file node-passwd:

```bash
sudo -i
vi /var/lib/rancher/k3s/server/cred/node-passwd
```