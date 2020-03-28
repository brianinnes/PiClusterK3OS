# Install the dashboard into the k3s cluster

## Deploy the dashboard

```kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta8/aio/deploy/recommended.yaml```

## Add a dashboard user

```
kubectl apply -f manifests/dashboard/dashboard_serviceAccount.yaml
kubectl apply -f manifests/dashboard/dashboard_clusterRoleBinding.yaml
kubectl apply -f manifests/dashboard/dashboard_ingress.yaml
```

# Traefik ingress

You can find the IP addresses for services using command ```kubectl get services --all-namespaces```.  Look at the external IP column to get the traefik external IP address.  This IP address should be from the range you configured in the Metal Load Balancer.

The ingress uses the host name to route traffic to the correct service, so you need to set up your DNS server or use your system local name resolution capability (/etc/hosts on Linux and MacOS) to map hostname to the external IP address of the traefik service, e.g.

```text
192.168.0.201   traefik.bik3s.home
192.168.0.201   k3s.bik3s.home
```

Then you can access the dashboards using:

- Kubernetes dashboard : ```https://k3s.bik3s.home```
- Traefik dashboard : ```http://traefik.bik3s.home```

# Access the bearer token to login into the dashboard

```kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')```
