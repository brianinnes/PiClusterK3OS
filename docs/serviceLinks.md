# Exposed services

This page provides links to services running on the cluster which are exposed via the ingress:

- [Kubernetes Dashboard](https://k3s.bik3s.home)
- [Traefik dashboard](http://traefik.bik3s.home/dashboard)
- [Longhorn storage dashboard](http://longhorn.bik3s.home)

To make the links work your local DNS server needs to be updated to point the host names to the IP address assigned to the traefik service by the cluster load balancer.  The find the IP address use command ```kubectl get svc --all-namespaces```

The other option is to add the resolution to the /etc/hosts file on mac or linux:

```text
192.168.0.202   traefik.bik3s.home
192.168.0.202   k3s.bik3s.home
192.168.0.202   longhorn.bik3s.home
```
