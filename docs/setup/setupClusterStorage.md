# Setup distributed block storage

The NFS storage provider works on both Raspberry Pis running a base Raspbian OS and also Intel/AMD workstations installed from the k3os ISO.

Longhorn is a distributed file service, which replicates the storage over multiple nodes - need to investigate if Longhorn is useable with replicas stored on NFS mounted volumes on Raspberry Pis.

## Longhorn storage provider

Longhorn is an open-source distributed block storage system which we will use for storage in the cluster.

On nodes without storage, or storage on micro-SD card, NFS will be used to provide the storage for longhorn.  

K3OS has NFS packages installed, so you can use them.  However, they are not enabled, so you need to provide the configuration to start service rpd.statd.  The configuration mounts an NFS share to /mnt then creates a directory for this node, with the hostname as the directory name.  A link is then created to /var/lib/longhorn, which is where longhorn stores data.

Add the following to the ```/var/lib/rancher/k3os/config.yaml``` file:

```text
boot_cmd:
- "mkdir -p /mnt"
- "rc-update add rpc.statd"
run_cmd:
- "mkdir -p /var/lib/nfs"
- "chown nobody:nogroup /var/lib/nfs"
- "rc-service rpc.statd start"
- "mount 192.168.201.1:/mnt/ssd /mnt"
- "mkdir -p /mnt/sysRoots/`hostname`"
- "ln -sfn /mnt/sysRoots/`hostname` /var/lib/longhorn"
```

To install follow the instructions here:

``` bash
kubectl apply -f https://raw.githubusercontent.com/longhorn/longhorn/master/deploy/longhorn.yaml
kubectl create -f manifests/longhorn/longhorn_storageClass.yaml
kubectl apply -f manifests/longhorn/longhorn_ingress.yaml
```
