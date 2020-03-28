# Setup K3OS on Raspberry Pi

1. Flash Raspbian Buster lite on a micro SD card
2. add file ssh in boot partition ```touch /Volumes/boot/ssh```
3. Boot the pi
4. Update raspbian ```sudo apt update && sudo apt upgrade -y```
5. sudo raspi-config
   - Advance Options -> Memory Split -> set to 16
   - select to reboot
6. switch into root user ```sudo -i```
7. create config file **config.yaml**.  You can look at the configuration reference for additional configuration options:

   ```bash
   cat <<EOF | tee config.yaml
   ssh_authorized_keys:
   - ssh-rsa AAAAB3NzaC1yc2EAAAADAQ...
   hostname: bi-rpi4-1
   k3os:
     data_sources:
     - cdrom
     dns_nameservers:
     - 192.168.0.4
     - 192.168.0.3
     ntp_servers:
     - 0.uk.pool.ntp.org
     - 1.europe.pool.ntp.org
     password: password
     labels:
       plan.upgrade.cattle.io/k3os-latest: enabled
   EOF
   ```

    Copy your ssh public key from ~/.ssh/id_rsa.pub on your workstation (this is needed to be able to ssh into the machine) and add it as the ssh_authorized_keys entry in the config file.  You may also want to change the hostname and password
8. Overlay K3OS and add the config then reboot into K3OS

    ```bash
    curl -sfL https://github.com/rancher/k3os/releases/download/v0.9.1/k3os-rootfs-arm.tar.gz | sudo tar zxvf - --strip-components=1 -C /
    sudo cp config.yaml /k3os/system/config.yaml
    sync
    sudo reboot -f
    ```

Once the system has rebooted you should be able to ssh into the server with user rancher.  You won't need a password as you added your key.

K3OS starts K3S with some standard services, which we don't want as we'll be installing different options during the configuration.  With the standard configured services you cannot modify the configuration as at reboot any changes will be overridden, so for the Traefik ingress we need to copy the standard configuration, then turn off the standard installed version.  We need Traefik configuration to enable the dashboard and also allow SSL connections to skip verification, as we are not using keys signed by a known issuing authority.

We will also turn off the local storage service and the limited load balancer service, as we will install different services which perform these roles.

Any yaml file in directory ```/var/lib/rancher/k3s/server/manifests``` is watched an changes automatically applied.

To make the changes described above enter the following commands:

   ```bash
   sudo cp /var/lib/rancher/k3s/server/manifests/traefik.yaml /var/lib/rancher/k3s/server/manifests/traefik-mod.yaml
   cat <<EOF | sudo tee /var/lib/rancher/k3os/config.yaml
   hostname: k3os-mac
   k3os:
     k3s_args:
     - server
       "--no-deploy=local-storage,traefik,servicelb"
   EOF
   cat <<EOF | sudo tee -a /var/lib/rancher/k3s/server/manifests/traefik-mod.yaml
       dashboard.enabled: "true"
       dashboard.domain: "traefik.otterburn.home"
       ssl.insecureSkipVerify: "true"
   EOF
   sudo rm /var/lib/rancher/k3s/server/manifests/traefik.yaml /var/lib/rancher/k3s/server/manifests/local-storage.yaml
   ```

Reboot the master node to make the changes live using  command ```sudo reboot```

## Setting up additional Raspberry Pis as arm32 nodes

You may want to add additional raspberry pis as cluster nodes.  To do this follow steps 1-8 above, but replace the config file with a version that contains the server URL and token.  You can look at the [configuration reference](https://github.com/rancher/k3os#configuration-reference) for additional options:

```yaml
ssh_authorized_keys:
- <WORKSTATION_SSH_PUBLIC_KEY>
hostname: bi-k3os-mac
k3os:
data_sources:
- cdrom
dns_nameservers:
- 192.168.0.4
- 192.168.0.3
ntp_servers:
- 0.uk.pool.ntp.org
- 1.europe.pool.ntp.org
password: PASSWORD
server_url: http://SERVER:6443
token: TOKEN_VALUE
labels:
  upgrade.cattle.io/k3os-latest: enabled
```

Where:
- WORKSTATION_SSH_PUBLIC_KEY is your ssh public key, on linux/mac this is stored in ~/.ssh/id_rsa.pub. It should start with something similar to *ssh-rsa AAAAB3NzaC1*
- PASSWORD is the password you want to assign to the rancher user - this is not going to be used as you use the ssh key when login to the system
- SERVER is the hostname or IP address of the master node of the cluster
- TOKEN_VALUE is the token from file **/var/lib/rancher/k3s/server/node-token** on the master node.  SSH into the master node and enter command ```sudo cat /var/lib/rancher/k3s/server/node-token``` to display it
