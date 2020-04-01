# Cluster storage

Kubernetes NFS storage provider to access NFS shares on the network, NAS or other Linux server with storage attached.

Options to investigate:

- Longhorn - Provides redundancy through replication, but needs to access locally attached storage, can it be used in conjunction with NFS on raspberry Pi?
- Longhorn with iSCSI mounted disks on raspberry pis a better option? - Only amd64 architecture images available
