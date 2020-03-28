#K3OS setup on AMD64 architecture

As part of the cluster I need at least 1 amd64 node, this will be a multi-arch build node when the cluster is fully operational, but as part of the initial setup it is needed to run images not available for arm architectures.  

Kubernetes automatically adds the label **kubernetes.io/arch: arm** to each node, which can be used as a selector to ensure pods can be limited to a specific architecture when required.  The value of the label is the **runtime.GOARCH** as defined by the go language for the node architecture.

I plan to create multi-architecture builds of all containers that form part of the core infrastructure, but this is a secondary goal after the core setup.

Initially my amd64 node will be an old macbook pro laptop, but eventually I may switch this out for a small form factor amd64 system.

## Preparing for the install

When the system is initially setup it needs a configuration file.  The easiest way to supply this is from a web site.  This could be any http server you already have running, or you can use a public site, such as git or gist, however I temporarily setup a nginx web server using docker running on my laptop.  This is quick and easy and doesn't require me to upload the config file to a public server.

1. Create the config file, **config_amd64.yaml**.  You can copy and customise the sample below or look at the [configuration reference](https://github.com/rancher/k3os#configuration-reference) for additional options:

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
2. Make the config file available using docker ```docker run --name configsrv -v `pwd`:/usr/share/nginx/html -p 80:80 -d nginx``` This should be run from the directory containing the config file you created in step 1.
   - If port 80 is already in use on your workstation you can choose a different port, but changing the left side port number in the -p option, e.g. 8080:80 will expose the web server on port 8080 on your workstation.

## Install K3OS on the amd64 system

1. Download K3OS iso file from the [rancher k3os git repo](https://github.com/rancher/k3os/releases)
2. burn the iso to a USB key, insert the key in the target machine then reboot it into k3OS
3. on the k3OS command line enter ```sudo k3os install --config http://WORKSTATION_IP/config_amd64.yaml``` and follow the prompts to install to the laptop hard drive .  THIS WILL ERASE THE MAC DISK.  You want to select to install a server, accept all the default promps.  Select Yes to format the disk.  When the install has completed let the machine reboot and remove the USB key.  The laptop should boot into K3OS

After the system boots it should automatically register with the master node and become part of the cluster.  User ```kubectl get nodes``` to show the nodes available in your cluster

## Clean up

Once the node has been installed you can stop the docker image running the nginx web server for the config using command: ```docker kill configsrv && docker rm configsrv && docker rmi nginx```
