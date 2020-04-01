# Setup iSCSI storage on cluster nodes

For nodes to work with Longhorn storage they need to have local storage available.

Raspberry Pis typically only have the micro-SD card storage available, which is not ideal.  If possible you should attach an USB3 disk, but if this is not an option you can share storage from another system to Raspberry Pis over TCP using iSCSI.

iSCSI shares storage as a block device rather than sharing a file system, such as systems as Samba or NFS.

Setting up an iSCSI share requires setting up the server (or target) and setting up the client (or initiator).  It is assumed that the server is a Linux based machine.

## Setting up the server

The server will share a disk for each client that needs storage.  There are 2 options for sharing storage:

- A block device attached to the server, such as a disk
- A file, created on the server which will be the disk image attached to the client.  The file is created to be the size of the disk on the client.

Performance of block devices will be better than using files.  However, I am using files to allow multiple files to be created on a single disk to be attached to each raspberry pi.

Before starting to configure the server you need to know the initiator names for each client that needs to attach to the server.  The name for each client can be found in file **/etc/iscsi/initiatorname.iscsi**.  If you are using the default names, that are set at boot then you have to have the nodes up and running and part of the K3OS cluster to be able to identify the names.  The other option is to set a name at startup (using the config.yaml file).

Initiator names are usually of the form: **iqn.<year created>-<month created>.<reverse host name>-<name>**, such as *iqn.2016-04.com.open-iscsi:b21fffe60cb*.  Setting a name is covered in the client setup section.

On the disk hosting the storage for each client we need to:

- create a backing store
- create the iSCSI target
- create a LUN (**L**ogical **UN**it of storage)
- create an ACL (Access Control List)

All these actions are performed in a tool **targetcli** run as root ```sudo targetcli``` then within the targetcli utility, set the global parameter auto_add_mapped_luns to false: ```set global auto_add_mapped_luns=False```

### Create a backing store

In the targetcli tool enter command ```backstores/fileio create rpi1 /mnt/ssd/iSCSI/rpi1 100G write_back=false```, where

- rpi1 is the name of the backing store
- /mnt/ssd/iSCSI/rpi1 is the file image to create
- 100G is the size of the share you want to create

Customise these values to your requirements and system configuration

If you want to use block storage instead of a file image for the backing store the command is ```backstores/block create name=block dev=/dev/sdc```, where **/dev/sdc** is the disk to share over iSCSI

Use the **ls** command in the targetcli tool to look at the configuration created.

Repeat this step for all the clients that you will share a file image with.

### Create the iSCSI target

Still in the targetcli tool move to the iSCSI path, enter **iscsi/** at the prompt.

**This step only needs to be performed once, the first time you set up a share**.  Create the target with the command ```create```.  This will generate an IQN for the target.  If you want to specify one to be more meaningful to your app, you can specify the name in the create command:  ```create iqn.2020-03.home.bik8s:target```

### Create a LUN

Move to the target portal group (TPG) bu entering the name of the target, followed buy /tpg1/:  ```iqn.2020-03.home.bik8s:target/tpg1/```

Create the LUN with command ```luns/ create /backstores/fileio/rpi1```, wherecd  ***rpi1** is the name of the backing store create above.  Run this command for all backing stores.

### Create an Access Control List (ACL) for the LUN

This action only needs to be done after the previous sections have been completed for all the clients, as the 
Still in the targetcli utility, enter the acl path for the LUN created in the last section ```acls/```

Create the ACL with command ```create iqn.2020-03.home.bik8s.bi-k3os-rpi1.initiator01```, where *iqn.2020-03.home.bik8s.bi-k3os-rpi1.initiator01* is the iqn of the client system, which can be found in location **/etc/iscsi/initiatorname.iscsi** on the client.  You can use the generated iqn or set your own client iqn (as explained in the Setting up the K3OS client section below)

then map a LUN to the ACL with command ```iqn.2020-03.home.bik8s.bi-k3os-rpi1.initiator01/ create 0 0```, which maps LUN0 to what the client initiator will see as LUN0.  The second pi initiator should use create 0 1, which maps LUN1 to what the client will see as LUN0, etc...

Create an ACL for each initiator client.

I am not configuring authentication on the iSCSI connections, as my setup is in a private network within my home, but you may want to set up authentication.

You can see the config by returning to the base path ```cd /``` and using the ```ls``` command.

With 4 clients, this is what my configuration looks like:

```text
o- / ......................................................................................................................... [...]
  o- backstores .............................................................................................................. [...]
  | o- block .................................................................................................. [Storage Objects: 0]
  | o- fileio ................................................................................................. [Storage Objects: 4]
  | | o- rpi1 ................................................................ [/mnt/ssd/iSCSI/rpi1 (100.0GiB) write-thru activated]
  | | o- rpi2 ................................................................ [/mnt/ssd/iSCSI/rpi2 (100.0GiB) write-thru activated]
  | | o- rpi3 ................................................................ [/mnt/ssd/iSCSI/rpi3 (100.0GiB) write-thru activated]
  | | o- rpi4 ................................................................ [/mnt/ssd/iSCSI/rpi4 (100.0GiB) write-thru activated]
  | o- pscsi .................................................................................................. [Storage Objects: 0]
  | o- ramdisk ................................................................................................ [Storage Objects: 0]
  o- iscsi ............................................................................................................ [Targets: 1]
  | o- iqn.2020-03.home.bik8s:target ..................................................................................... [TPGs: 1]
  |   o- tpg1 ............................................................................................... [no-gen-acls, no-auth]
  |     o- acls .......................................................................................................... [ACLs: 4]
  |     | o- iqn.2020-03.home.bik8s.bi-k3os-rpi1.initiator01 ...................................................... [Mapped LUNs: 1]
  |     | | o- mapped_lun0 ................................................................................. [lun0 fileio/rpi1 (rw)]
  |     | o- iqn.2020-03.home.bik8s.bi-k3os-rpi2.initiator01 ...................................................... [Mapped LUNs: 1]
  |     | | o- mapped_lun0 ................................................................................. [lun1 fileio/rpi2 (rw)]
  |     | o- iqn.2020-03.home.bik8s.bi-k3os-rpi3.initiator01 ...................................................... [Mapped LUNs: 1]
  |     | | o- mapped_lun0 ................................................................................. [lun2 fileio/rpi3 (rw)]
  |     | o- iqn.2020-03.home.bik8s.bi-k3os-rpi4.initiator01 ...................................................... [Mapped LUNs: 1]
  |     |   o- mapped_lun0 ................................................................................. [lun3 fileio/rpi4 (rw)]
  |     o- luns .......................................................................................................... [LUNs: 4]
  |     | o- lun0 .............................................................................. [fileio/rpi1 (/mnt/ssd/iSCSI/rpi1)]
  |     | o- lun1 .............................................................................. [fileio/rpi2 (/mnt/ssd/iSCSI/rpi2)]
  |     | o- lun2 .............................................................................. [fileio/rpi3 (/mnt/ssd/iSCSI/rpi3)]
  |     | o- lun3 .............................................................................. [fileio/rpi4 (/mnt/ssd/iSCSI/rpi4)]
  |     o- portals .................................................................................................... [Portals: 1]
  |       o- 0.0.0.0:3260 ..................................................................................................... [OK]
  o- loopback ......................................................................................................... [Targets: 0]
  o- vhost ............................................................................................................ [Targets: 0]
```

Once you are satisfied your configuration is correct then you should save the configurate with command ```saveconfig``` then exit the targetcli tool with command ```exit```

Reboot the server to make the changes live or restart the service.

## Setting up the K3OS client

Setting up the client requires the following actions:

- Configure iSCSI
- attach the iSCSI share
- create a file system on the attached iSCSI device (1 time action the first time the disk is attached)
- mount the file system

## Configure iSCSI

the iSCSI system is configured using the config file /etc/iscsi/iscsid.conf.  As this is in the /etc filesystem, the file will be reset on each boot.  To modify this file the /var/lib/rancher/k3os/config.yaml file needs to be used to configure the system on each boot.  Add a **write_files** section to the config.yaml file, if one doesn't already exist, then add the following content:

```yaml
write_files
- content: |
    iscsid.startup = /etc/init.d/iscsid start
    node.startup = automatic
    node.leading_login = No
    discovery.startup = automatic
    discovery.type = sendtargets
    discovery.sendtargets.address = <SERVER_IP>
    discovery.sendtargets.port = 3260
    node.session.timeo.replacement_timeout = 120
    node.conn[0].timeo.login_timeout = 15
    node.conn[0].timeo.logout_timeout = 15
    node.conn[0].timeo.noop_out_interval = 5
    node.conn[0].timeo.noop_out_timeout = 5
    node.session.err_timeo.abort_timeout = 15
    node.session.err_timeo.lu_reset_timeout = 30
    node.session.err_timeo.tgt_reset_timeout = 30
    node.session.initial_login_retry_max = 8
    node.session.cmds_max = 128
    node.session.queue_depth = 32
    node.session.xmit_thread_priority = -20
    node.session.iscsi.InitialR2T = No
    node.session.iscsi.ImmediateData = Yes
    node.session.iscsi.FirstBurstLength = 262144
    node.session.iscsi.MaxBurstLength = 16776192
    node.conn[0].iscsi.MaxRecvDataSegmentLength = 262144
    node.conn[0].iscsi.MaxXmitDataSegmentLength = 0
    discovery.sendtargets.iscsi.MaxRecvDataSegmentLength = 32768
    node.session.nr_sessions = 1
    node.session.iscsi.FastAbort = Yes
  owner: root:root
  path: /etc/iscsi/iscsid.conf
  permissions: '0644'
```

If you want to manually set an initiator name, then you need to add another file configuration in the config.yaml file to write the **/etc/iscsi/initiatorname.iscsi** file at boot:

```yaml
- content: InitiatorName=iqn.<IQN_FOR_THIS_PI>
  owner: root:root
  path: /etc/iscsi/initiatorname.iscsi
  permissions: '0644'
```

replacing **iqn.<IQN_FOR_THIS_PI>** with the initiator name you want to set for this pi

Reboot the pi to make the config live.

### Setup the disk

When the pi has restarted you need to do a 1 time setup for each iscsi mount.

Connect the iSCSI share by issuing the following commands:

```bash
sudo iscsiadm --mode discovery --type sendtargets --portal 192.168.201.1
sudo iscsiadm --mode node --targetname iqn.2020-03.home.bik8s:target --portal 192.168.201.1:3260 --login
```
You can verify you have the storage connected using command ```lsblk --scsi``` should show the attached disk

You now need to create a partition on the disk and then create a file system on the connected storage.

Issue the command ```sudo fdisk /dev/sda``` then once in the fdisk application use the following:

- select **n** to add a new partition
- select to create a primary partition and accept all the defaults
- select **w** to write the partition to the disk

Now create a file system on the disk using command ```sudo mkfs.ext4 /dev/sda1```

### Mounting the disk

Issue the following commands to mount the drive:

```bash
sudo mkdir /iscsi
sudo mount /dev/sda1 /iscsi
```

### Mounting the storage automatically at node boot

To allow the disk to be mounted at boot you need to edit the K3OS config file: ```sudo vi /var/lib/rancher/k3os/config.yaml```

In the run_cmd section add the following:

```yaml
run_cmd:
- "iscsiadm --mode discovery --type sendtargets --portal 192.168.201.1"
- "iscsiadm --mode node --targetname iqn.2020-03.home.bik8s:target --portal 192.168.201.1:3260 --login"
- "mkdir -p /iscsi"
- "mount /dev/sda1 /iscsi"
```

Change the targetname iqn to match your configured server target and the --portal to match your iSCSI target IP address.
