docker-vm
=========

If you run your Docker host as a virtual machine, this repo provides a quick & easy way to start it & connect to it (through ssh) from the computer that hosts the VM

Current limitations:
- Only VirtualBox VMs
- VM name must be Docker
- Only runs from bash

## Ubuntu Server on VirtualBox
- On MacBook Pro Retina mid 2013
- Connected to the Internet w/ DHCP server in local network
### 1. Install VirtualBox
### 2. Create Virtual Machine
- Download .iso file from [Ubuntu website](http://www.ubuntu.com/download/server) (I had `ubuntu-14.04.1-server-amd64.iso`)
- In VirtualBox menu `Machine | New...`
- Type: Linux
- Version: Ubuntu (64 bit)
- 4096 MB RAM
- Create new virtual disk now
- VDI (VirtualBox Disk Image)
- Dynamically allocated
- 50 GB
- Start the new machine & choose the downloaded ubuntu .iso file
- Accept all default options during install, except:
- Host name: `docker`
- User name: same as host
- Partitioning method: Guided - use entire disk
- Write changes to disks? Yes
- Install security updates automatically
- Choose software to install: OpenSSH server
### 3. Install VirtualBox Guest Additions
- After reboot, login
- `$ sudo bash`
- `$ apt-get update`
- `$ apt-get -y upgrade`
- `$ apt-get -y install dkms`
- `$ reboot`
- After reboot, login
- `$ sudo bash`
- In VirtualBox menu (with Ubuntu server window focused) `Devices | Insert Guest Additions CD image...`
- `$ mount /dev/cdrom /media/cdrom`
- `$ cd /media/cdrom`
- `$ ./VBoxLinuxAdditions.run` - the last line will state "Could not find the X.Org or XFree86 Window System, skipping;" that's alright, since we have a windowless server
### 4. Create Shared Folder
- In VirtualBox menu (with Ubuntu server window focused) `Machine | ACPI Shutdown`
- In the machine settings, choose Network tab
- Attached to: Bridged Adapter
- OK
- In the machine settings, choose Shared Folders tab
- Click the tiny icon "Adds a new shared folder definition" and select Folder Path "Other..."
- Create a new folder on the host computer to share with the Ubuntu Server VM
- Choose the newly created folder
- Folder name: `host`
- Select `Auto-mount` and `Make permanent`
- OK
- Startup the VM & login
- `$ ip -f inet addr` - verify that the eth0 interface lists a (DHCP-supplied) address
- `$ ls -la /media` - verify that the shared folder `sf_host` is listed, and owned by root in the vboxsf group
- optional: `$ sudo adduser <you> vboxsf` - this will add your user to the vboxsf group
- optional: `$ exit`
- optional: Login again
- optional: `$ ls /media/sf_host` - verify you don't get "permission denied"
### 5. Install Docker
- `$ sudo bash`
- optional: see the (Docker installation page)[https://docs.docker.com/installation/ubuntulinux/]
- `$ apt-get -y install docker.io`
- `$ ln -sf /usr/bin/docker.io /usr/local/bin/docker`
- `$ sed -i '$acomplete -F _docker docker' /etc/bash_completion.d/docker.io`
- `$ docker version` - verify that this reports the client & server versions of Docker
### 6. optional: nsenter
- optional: see the (nsenter readme)[https://github.com/jpetazzo/nsenter]
- `$ docker run --rm -v /usr/local/bin:/target jpetazzo/nsenter`
### 7. optional: passwordless ssh
- 

