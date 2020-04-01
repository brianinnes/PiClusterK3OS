# Exposed services

This page provides links to services running on the cluster which are exposed via the ingress:

- [Kubernetes Dashboard](https://k3s.bik8s.home)
- [Traefik dashboard](http://traefik.bik8s.home/dashboard)
- [Git server](http://gitea.bik8s.home)

To make the links work your local DNS server needs to be updated to point the host names to the IP address assigned to the traefik service by the cluster load balancer.  The find the IP address use command ```kubectl get svc --all-namespaces```

The other option is to add the resolution to the /etc/hosts file on mac or linux:

```text
192.168.201.200   traefik.bik8s.home
192.168.201.200   k3s.bik8s.home
192.168.201.200   gitea.bik8s.home
```
