# Set up an ARM64 Raspberry Pi 4 cluster node

1. Download the [ubuntu](https://ubuntu.com/download/raspberry-pi) 19.10.1 64-bit image for Raspberry Pi 4
2. Uncompress the image and flash to a micro SD card
3. Boot the pi
4. ssh into the system user **ubuntu** and password **ubuntu**, you will be prompted to change the password, so enter a new password.  You will be disconnected, so ssh in again using the new password
5. Update ubuntu ```sudo apt update && sudo apt upgrade -y```
6. create config file **config.yaml**.  You can look at the configuration reference for additional configuration options:

   ```bash
   cat <<EOF | tee config.yaml
   ssh_authorized_keys:
   - ssh-rsa AAAAB3NzaC1yc2EAAAADAQ...
   hostname: bi-k3os-rpi1
   k3os:
     data_sources:
     - cdrom
     dns_nameservers:
     - 192.168.201.1
     - 192.168.0.4
     ntp_servers:
     - 0.uk.pool.ntp.org
     - 1.europe.pool.ntp.org
     password: password
     server_url: https://SERVER:6443
     token: TOKEN_VALUE
     labels:
       plan.upgrade.cattle.io/k3os-latest: enabled
   EOF
   ```

    Copy your ssh public key from ~/.ssh/id_rsa.pub on your workstation (this is needed to be able to ssh into the machine) and add it as the ssh_authorized_keys entry in the config file.  You may also want to change the hostname and password
7. Edit file **nobtcmd.txt** to enable cgroups needed for containers: ```sudo vi /boot/firmware/nobtcmd.txt``` and add ```cgroup_memory=1 cgroup_enable=memory``` to the end of the first line.  There should only be a single line of content in this file.  
  *Note: It is important that you do this step and reboot before booting the overlay, otherwise the cmdline.txt file is not used once the overlay takes over*
8. Reboot to make the kernel command line changes active ```sudo reboot```
9. Overlay K3OS and add the config then reboot into K3OS

    ```bash
    curl -sfL https://github.com/rancher/k3os/releases/download/v0.9.1/k3os-rootfs-arm64.tar.gz | sudo tar zxvf - --strip-components=1 -C /
    sudo cp config.yaml /k3os/system/config.yaml
    sync
    sudo reboot
    ```
