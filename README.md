docker-vm
=========

If you run your Docker host as a virtual machine, this repo provides a quick & easy way to start it & connect to it (through ssh) from the computer that hosts the VM

## Usage
- Just run the applicable `docker` script
- If you're asked for your (sudo) password, it's for accessing the hosts file, where the address is updated (if it changed) so you can consistently connect to the VM by name (e.g. `http://docker` if you have a web server container running)
- Use `docker -h` if you don't want it to update your hosts file
- To stop the Docker VM: `docker stop`

## Current limitations
- Only VirtualBox VMs
- VM name must be `Docker`
- Only runs from bash (if you're on Windows, you should be good to go after installing [GitHub for Windows](https://windows.github.com/), and start `bash` from the Git Shell, though the path to `/etc/hosts` should probably be prefixed with `/c/windows/system32/drivers` then)

## Ubuntu Server on VirtualBox
This is how I set up my environment:

- On MacBook Pro Retina mid 2013
- Connected to the Internet w/ DHCP server in local network

### 1. Install VirtualBox
- [Download](https://www.virtualbox.org/wiki/Downloads)
- Double click

### 2. Create Virtual Machine
- [Download .iso file](http://www.ubuntu.com/download/server) (I had `ubuntu-14.04.1-server-amd64.iso`)
- In VirtualBox menu `Machine | New...`
- Type: Linux
- Version: Ubuntu (64 bit)
- 4096 MB RAM (or less ;-))
- Create new virtual disk now
- VDI (VirtualBox Disk Image)
- Dynamically allocated
- 50 GB (or less)
- Start the new machine & choose the downloaded ubuntu .iso file to install the OS
- Accept all default options, except:
- Host name: `docker`
- User name: same as host
- Partitioning method: Guided - use entire disk
- Write changes to disks? Yes
- Install security updates automatically
- Choose software to install: OpenSSH server

### 3. Install VirtualBox Guest Additions
- After reboot, login
- `$ sudo -i`
- `$ apt-get update`
- `$ apt-get -y upgrade`
- `$ apt-get -y install dkms`
- `$ reboot`
- After reboot, login
- `$ sudo -i`
- In VirtualBox menu (with Ubuntu server window focused) `Devices | Insert Guest Additions CD image...`
- `$ mount /dev/cdrom /media/cdrom`
- `$ cd /media/cdrom`
- `$ ./VBoxLinuxAdditions.run` - the last line will state "Could not find the X.Org or XFree86 Window System, skipping;" that's alright, since we have a windowless server

### 4. Configure networking
- Tell Ubuntu you want a second network adapter:
```
$ cat >> /etc/network/interfaces <<EOF

# The secondary network interface
auto eth1
iface eth1 inet dhcp
EOF
```
- In VirtualBox menu (with Ubuntu server window focused) `Machine | ACPI Shutdown`
- In the machine settings, choose Network tab
- Adapter 1:
  - Enable Network Adapter
  - Attached to: Host-only Adapter (this will let you reach the VM from your host computer)
- Adapter 2:
  - Enable Network Adapter
  - Attached to: NAT (this will let you reach the Internet from the VM)
- OK

### 5. Create Shared Folder
- In the machine settings, choose Shared Folders tab
- Click the tiny icon "Adds a new shared folder definition" and select Folder Path "Other..."
- Create a new directory on the host computer to share with the Ubuntu Server VM (existing directories may not get recognised by VirtualBox, especially "special" ones, like your home directory)
- Choose the newly created folder
- Folder name: `host`
- Select `Auto-mount` and `Make permanent`
- OK

### 6. Verify Network & Shared Folder
- Startup the VM & login
- `$ ip -f inet addr` - verify that the eth0 and eth1 interfaces list a (DHCP-supplied) address
- `$ ls -la /media` - verify that the shared folder `sf_host` is listed, and owned by root in the vboxsf group
- `$ sudo adduser <you> vboxsf` - this will add your user to the vboxsf group
- `$ exit`
- Login again
- `$ ls /media/sf_host` - verify you don't get "permission denied"

### 7. Install Docker
- `$ sudo -i`
- Optional: see the (Docker installation page)[https://docs.docker.com/installation/ubuntulinux/]
- `$ apt-get -y install docker.io`
- `$ ln -sf /usr/bin/docker.io /usr/local/bin/docker`
- `$ sed -i '$acomplete -F _docker docker' /etc/bash_completion.d/docker.io`
- `$ docker version` - verify that this reports the client & server versions of Docker

### 8. Optional: nsenter
- Optional: see the (nsenter readme)[https://github.com/jpetazzo/nsenter]
- `$ docker run --rm -v /usr/local/bin:/target jpetazzo/nsenter`

### 9. Optional: passwordless ssh
- On the host machine (where you'll run the ssh client to access the Docker VM) `$ ssh-keygen -t dsa` to generate a key, if not already done
- `$ cat ~/.ssh/id_dsa.pub | ssh <docker-vm-ip> "mkdir -p .ssh && cat >> ~/.ssh/authorized_keys"` - to list your key as an authorised key on the Docker VM (answer yes to the authenticity question & enter your password to connect)
- `$ ssh <docker-vm-ip>` - verify that you weren't asked for your password now
- `$ exit` - leave the ssh session

### 10. Install the start script
- `$ sudo cp ./VirtualBox/docker /usr/local/bin`
- Test drive:
```
$ docker
Starting Docker VM...............done
Welcome to Ubuntu 14.04.1 LTS (GNU/Linux 3.13.0-32-generic x86_64)

 * Documentation:  https://help.ubuntu.com/

  System information as of Thu Oct  2 21:14:02 CEST 2014

  System load: 0.0                Memory usage: 1%   Processes:       77
  Usage of /:  41.3% of 45.15GB   Swap usage:   0%   Users logged in: 0

  Graph this data and manage this system at:
    https://landscape.canonical.com/

Last login: Thu Oct  2 21:13:24 2014 from 192.168.59.3
wsf@docker:~$ sudo docker version
[sudo] password for wsf: 
Client version: 1.0.1
Client API version: 1.12
Go version (client): go1.2.1
Git commit (client): 990021a
Server version: 1.0.1
Server API version: 1.12
Go version (server): go1.2.1
Git commit (server): 990021a
wsf@docker:~$ exit
logout
Connection to 192.168.59.101 closed.
```


