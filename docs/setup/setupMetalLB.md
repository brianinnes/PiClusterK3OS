# Install MetalLB load balancer

Run the following commands to install and configure the Metal Load Balancer

The latest online script misses the configuration to create the required name space, so run the following to create the namespace:
```bash
kubectl apply -f -<<EOF
apiVersion: v1
kind: Namespace
metadata:
  labels:
    app: metallb
  name: metallb-system
EOF
```

```bash
  kubectl apply -f https://raw.githubusercontent.com/google/metallb/v0.9.2/manifests/metallb.yaml  
  openssl rand -base64 128 | kubectl create secret generic -n metallb-system memberlist --from-literal=secretkey=-
```

create file metalLB_configMap.yaml with the content shown below.  You should change the IP address range to match your local network.  Use addresses that will not be handed out by your DHCP server, but are accessible from your local laptop or workstation:

``` yaml
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 192.168.0.201-192.168.0.240
```
