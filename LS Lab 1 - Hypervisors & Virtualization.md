---
tags: Large Systems
---
:::success
# LS Lab 1 - Hypervisors & Virtualization
Name: Ivan Okhotnikov
:::


## Task 1 - Choose virtualization technology:
:::warning
- Linux KVM (default choice, but guest configs are XML and harder to synchronize among the farm...)
- Community XEN (harder to install and run, but there are some benefits that you can try to explore…)
- XCP-NG XEN on nested virtualization (no team, will be alone for the whole lab)
- XenServer on nested virtualization (no team, will be alone for the whole lab )
- VMware vSphere (be creative and explore advanced options due it's easier to configure)
- Microsoft HyperV (you need to be very creative there due it's pretty easier to configure)
- Otherwise exotic choices & presentation are possible, discuss it first with TA
:::

## Implementation:
:::info
I'll choose a Linux KVM
:::

## Task 2 - Local implementation:
:::warning
Agree on which host system version you’re going to work on with you teammate. Different OS and verions may work, and will make task 2 even more interesting. But if you don’t want to take any risks, check `uname −r` and `lsb_release −a` first. Proceed with the following steps **INDIVIDUALLY**:
everybody needs to know how to setup a single and true virtualization host.
- **Host install** – Install the hypervisor and host tools on a physical machine (make sure VT/AMD-V is enabled).
- **Guest install** – Install a guest with a local virtual disk with whatever method fits best, just to validate that you get a VM up and running. If using XEN, be clear and provide details on what type of guest you chose.
- **Sparse-file virtual disk** – Install another guest hard and DIY way. Create a SPARSE file (either with dd seek or QCOW2, for example), mount it in a folder and use debootstrap to get a Ubuntu Server system over there, quick & dirty. For VMware/HyperV users, try to do something similar.
- **Network** – Setup the network manually (always show how you do it in the report). For Community XEN you need to setup a bridge. For KVM/libvirt, please get rid of the default setup and do it yourself. In other words, make your guests capable to obtain a DHCP lease from the SNE network (or any other local network in the case of remote work).
- **Text console** – Make sure you can reach the text console of the guest from the host. What configurations allows for both, the kernel and the userland system to show up there?
Eventually disable the graphical console.
- **Snapshot** – Proceed with a hot-snapshot, meaning while the guest is running, take it.
Attempt to make sure that the file system was properly dealt with…
:::

## Implementation:

:::warning
1. [Installing KVM](https://phoenixnap.com/kb/ubuntu-install-kvm)
2. [Creating a sparse file and installing the operating system on it](https://jeremy.geek.nz/tag/debootstrap/)
:::

:::info
To begin with, I will execute the commands:
```
uname -r
```
<center>
    
![](https://i.imgur.com/Hfd22eM.png)
Figure 1: The result of the command execution
</center>

```
lsb_release −a
```
<center>
    
![](https://i.imgur.com/fLFlAvm.png)
Figure 2: The result of the command execution
</center>

> - **Host install** – Install the hypervisor and host tools on a physical machine (make sure VT/AMD-V is enabled).


Installation is a simple step. It is enough to run the command
```
sudo apt install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils
```

After installation, it is desirable to make sure that kvm is installed normally, this is done by the command:

```
sudo kvm-ok
```
<center>
    
![](https://i.imgur.com/12Rt585.png)
Figure 3: Program output
</center>

Now I need to create users `libvirt` and `kvm`
It is necessary to do this, because without these users, problems may arise in the operation of virtual machines
```
sudo adduser st11 libvirt
sudo adduser st11 kvm
```

To view all virtual machines, there is a command:
```
virsh list --all
```
<center>
    
![](https://i.imgur.com/kt1UXOt.png)
Figure 4: List of all virtual machines on the host
</center>

Since I just installed everything, the list will be empty


> - **Guest install** – Install a guest with a local virtual disk with whatever method fits best, just to validate that you get a VM up and running. If using XEN, be clear and provide details on what type of guest you chose.

For easy work with the creation and management of virtual machines, I installed the program `virt-manager`:
```
sudo apt install virt-manager
```

<center>
    
![](https://i.imgur.com/aAcx5sv.png)
Figure 5: Program interface
</center>

To create a guest operating system, I clicked on the create button, selected `Local install media` to install from a ready-made 'iso' image

Then I selected an operating system image, allocated it memory, the number of CPUs, the size of the virtual disk and installed it (figure 6-8)
<center>

![](https://i.imgur.com/bIIFR50.png)
Figure 6: Window for selecting the type of operating system
![](https://i.imgur.com/kNeFRAJ.png)
Figure 7: The window for adding storage
![](https://i.imgur.com/FnHKKnb.png)
Figure 8: Image selection window in the file system
![](https://i.imgur.com/BOYI7xV.png)
Figure 9: Window for selecting memory and CPU settings for a virtual machine
![](https://i.imgur.com/XQZOhBp.png)
Figure 10: The window for creating a virtual disk (since there is no one for a regular iso)
![](https://i.imgur.com/9ohqP40.png)
Figure 11: A window with the final display of the virtual machine configuration and specifying the name
![](https://i.imgur.com/yJZAbXW.png) 
Figure 12: Displaying a virtual machine in the program interface
</center>


> - **Sparse-file virtual disk** – Install another guest hard and DIY way. Create a SPARSE file (either with dd seek or QCOW2, for example), mount it in a folder and use debootstrap to get a Ubuntu Server system over there, quick & dirty. For VMware/HyperV users, try to do something similar.


### Installation:
For this procedure I will need the `debootstrap` program
I'll install it
```
sudo apt install debootstrap
```

Now I will create a sparse file `ubuntu-sparse` using the `dd` command

```
dd if=/dev/zero of=ubuntu-sparse.img bs=1M count=1 seek=4095
```

So that I can communicate with this file as with a regular block device, I will create a loop device using the command:
```
sudo losetup --show -f ubuntu-sparse.img
```
The output of the program is very important because it contains the path to the loopback device in the `/dev` directory

This path will be useful to me in the future

<center>

![](https://i.imgur.com/HVOOHBX.png)
Figure 13: Creating a loop device
</center>


After creating a loop device, you need to create a partition table
```
sudo parted /dev/loop20 mklabel msdos
```

Creating the main partition that will fill the disk:
```
sudo parted /dev/loop20 mkpart pri ext2 0% 100%
```

Since I created the main section, the loop device is now available `/dev/loop20p`

Now I'm going to create a file system for the main partition:
```
sudo mkfs.ext4 /dev/loop20p1
```

Now I'll move on to mounting the main partition to my folder. This is necessary to work with the file system inside the partition
```
sudo mount /dev/loop20p1 /mnt
```

After mounting, you can start installing the operating system inside the mounted folder:

```
sudo debootstrap focal /mnt http://nz.archive.ubuntu.com/ubuntu
```
This process is not fast and you will need to wait for downloading, unpacking and configuration
The debootstrap program takes several parameters:
1 - the assembly of the operating system (I have it `focal`)
2 - the folder where the installation will be performed
3 - link to the operating system mirror

<center>

![](https://i.imgur.com/ghAo1ql.png)
Figure 14: debootstrap program output during operation
</center>

Now for further work I will need the /dev folder from my host inside another operating system
I will mount it inside using the command:

```
sudo mount -o bind /dev /mnt/dev
```

The most interesting part begins. 
**This is a very important step. Because if I don't do this, with further configuration commands I can break the startup of my host.**
Using the chroot <dir_name> command, I will open a shell for the /mnt folder
This will allow me to manipulate the mounted folder

```
sudo chroot /mnt
```

<center>
    
![](https://i.imgur.com/jhsG3qU.png)

Figure 15: Changing the operating system shell
</center>

Now I will mount the folders `/proc` and `/sys`
This is also necessary for further configuration. Since I will be installing packages and installing the GRUB loader

```
mount -t proc proc /proc
mount -t sysfs sysfs /sys
```

Now that I have a mounted /dev folder inside the shell, I can write the partition mount string to `/etc/fstab`
`blkid` - used to output the UUID of the main partition that I created at the very beginning
This is necessary to start the operating system

```
cat << EOF > /etc/fstab
UUID=$(blkid /dev/loop20p1 | cut -d\" -f2) / ext4 errors=remount-ro 0 1
EOF
```

Now I will add the repository sources with the command below (this is necessary for `apt` to work)

```
cat << EOF > /etc/apt/sources.list
deb http://nz.archive.ubuntu.com/ubuntu focal main restricted universe multiverse
deb http://nz.archive.ubuntu.com/ubuntu focal-updates main restricted universe multiverse
deb http://security.ubuntu.com/ubuntu focal-security main restricted universe multiverse
EOF
```

Now I'm updating the dependencies
```
apt update
apt -y dist-upgrade
```

It's time to install the necessary packages for the operating system to work.

```
apt -y install linux-image-generic grub2-common bridge-utils ethtool nano
```

To make my sparse file smaller, I will delete unused packages

```
apt clean
```

To prevent the grub loader from searching for operating systems on other disks, I will disable probing by deleting the file:

```
rm /etc/grub.d/30_os-prober
```

It remains a small matter. I update grub and install the bootloader on my loopback device `/dev/loop20`

```
update-grub
grub-install --force /dev/loop20
```

<center>
    
![](https://i.imgur.com/0gD2GYz.png)
Figure 16: installing GRUB
</center>

I will change the host name to add uniqueness

```
echo hardway > /etc/hostname
```

Now I will create a user `user` and assign a password to the user 'root`

```

groupadd admin
useradd -s /bin/bash -m -d /home/user -G admin user
passwd user
passwd root
```

<center>
    
![](https://i.imgur.com/eVmOn8i.png)
Figure 17: Changing the root password
</center>


Setup is complete, I exit the shell

```
exit
```

I will unmount all the folders that I used to configure inside the environment

```
sudo umount /mnt/{dev,proc,sys} /mnt
```

Since in task 3 I will need to transfer my image, I will compress it with the command:
```
sudo zerofree /dev/loop20p1
```

It remains only to remove the loop device
```
sudo losetup -d /dev/loop20
```

### Verification:
Now I will check my created sparse file
When creating a virtual machine, I will select from the list `Import existing disk image` (figure 18)
And I will choose my file (figure 19)
<center>
    
![](https://i.imgur.com/7G8z2o7.png)
![](https://i.imgur.com/R9YqKMb.png)
Figure 18,19: Creating a virtual machine
</center>


<center>

![](https://i.imgur.com/6XRF3LI.png)
Figure 20: Window of a running virtual machine based on a sparse file
</center>


> - **Network** – Setup the network manually (always show how you do it in the report). For Community XEN you need to setup a bridge. For KVM/libvirt, please get rid of the default setup and do it yourself. In other words, make your guests capable to obtain a DHCP lease from the SNE network (or any other local network in the case of remote work).

As an interface for the network, I chose the virbr0 interface, which was created back in the first laboratory CIA
This interface allows you to distribute addresses based on dhcp, where my workstation is the host (`192.168.122.1'`

<center>
    
![](https://i.imgur.com/ZZ57fuz.png)
Figure 21: Network Setup window in virt-manager
</center>

as for my virtual machines, I specified the following setting for them in the file `/etc/netplan/01-default.yaml`:

```
network:
  version: 2
  renderer: networkd
  ethernets:
    enp1s0:
      dhcp4: true
```


<!-- cat << EOF > /etc/netplan/01-default.yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp1s0:
      dhcp4: true
EOF -->

I will check that everything is working correctly:

<center>
    
![](https://i.imgur.com/EIMbcZ0.png)
Figure 22: Checking network operation in a virtual machine
</center>

> - **Text console** – Make sure you can reach the text console of the guest from the host. What configurations allows for both, the kernel and the userland system to show up there?
Eventually disable the graphical console.

To disable the GUI and enable working with the text console, I changed the settings in the configuration file `/etc/default/grub` for virtual machines (figure 23)

<center>
    
![](https://i.imgur.com/TS1zRvK.png)
Figure 23: Configuration `/etc/default/grub`
</center>

In it, I indicated that it is necessary to use the console by default. Also allowed serial connections with a speed of `115200`

After changing the file, I performed a GRUB update

```
sudo update-grub
```

<center>

![](https://i.imgur.com/wsMPCqc.png)
Figure 24: Checking that the text console is working
</center>

To disable the availability of the graphical console, you will need to comment out the lines in the virtual machine settings.

To do this, run the command:


```
sudo virsh edit guest2
```

Find the highlighted lines as in figure 25 and comment out as shown in figure 26

<center>

![](https://i.imgur.com/RzzhBcj.png)
Figure 25: Before editing the `guest2` VM configuration
![](https://i.imgur.com/Bl7BT5c.png)
Figure 26: After editing the `guest2` VM configuration
</center>


<center>
    
![](https://i.imgur.com/tYXVOsV.png)
Figure 27: Result
</center>

> - **Snapshot** – Proceed with a hot-snapshot, meaning while the guest is running, take it.

To create a snapshot, I used the command:

```
virsh snashot-create-as --domain guest --name guest-snapshot
```
In the command, you must specify the name of the virtual machine and the name of the snapshot

During execution, the virtual machine (if it is running) will be paused (figure 28) and after execution will be active again (figure 29)


<center>
    
![](https://i.imgur.com/TqWHSrE.png)
Figure 28: During the snapshot execution
![](https://i.imgur.com/vtMOY38.png)
Figure 29: After completing the snapshot
</center>
:::


## Task 3 - Cluster validation:
:::warning
Now as a team, choose which one of the two machines is also going to provide the shared storage for
virtual disks to live in. Set it up and share both ideally, guests virtual disks and configurations.
- Take team member 1’s favorite guest (virtual disk and configuration) and put it in the shared storage
- Take team member 2’s favorite guest (virtual disk and configuration) and put it in the shared storage
- Eventually fix the pathes in the configuration and validate that both guests run as well as before
- Now shut them down and run them on the other team member’s machine/host (coldmigration)
- Now don’t even shut them down while migrating… (hot-migration/live-migration)
You have to show a live DEMO of task 3 and bonuses. So, record a video of your process and upload among with report. By the way, you still need to include this task in your report as usual.
:::

## Implementation:
:::info
> - Take team member 1’s favorite guest (virtual disk and configuration) and put it in the shared storage
> - Take team member 2’s favorite guest (virtual disk and configuration) and put it in the shared storage

To implement directory mounting over ssh, I used the sshfs program

```
sudo apt-get install sshfs
```

After receiving the username and password from my partner, I created a folder for mounting and mounted the external folder of the partner to myself

```
mkdir ~/shared_storage
sudo sshfs -o allow_other,default_permissions shared_storage@10.1.1.67:/home/shared_storage ~/shared_storage
```


To allow external connections via ssh, I created a shared_storage user and assigned him a password (figure 30)
```
sudo adduser shared_storage
```
<center>

    
![](https://i.imgur.com/7d1tW65.png)
Figure 30: Creating a user
</center>


Now I have copied my virtual disks to the folder `/home/shared_storage` and specified the owner forcibly. This is very important, because without it my partner will not be able to work with the virtual disk

```
sudo chown -R libvirt-qemu:kvm /home/shared_storage
```


After a similar setup with a partner, I can start a virtual machine based on his virtual disk:

<center>

![](https://i.imgur.com/w8o9e0v.png)
Figure 31: The virtual disk I have running is my partner's `guest`
![](https://i.imgur.com/xJ9aN0W.png)
Figure 32: I have my partner's `guest2` virtual disk running
</center>



> - Eventually fix the pathes in the configuration and validate that both guests run as well as before

To do this, I went to the settings of the turned-off machine, added a new hard drive (indicated that it was virtual) and later deleted the old one

<center>

![](https://i.imgur.com/8ORycjc.png)
Figure 33: Windows of the virt-manager program for working with virtual machine settings

</center>
    
> - Now shut them down and run them on the other team member’s machine/host (coldmigration)

To check this, I added a text file (figure 34) and turned off the machine
Then I asked the partner to turn on the virtual machine on the same virtual disk and see the changes in the file (figure 35)

<center>
    
![](https://i.imgur.com/mzMAV0X.png)
Figure 34: The interface of the virtual machine I have with the command to write text to a file
</center>

<center>

![](https://i.imgur.com/6t0m8l1.png)
Figure 35: The interface of the partner's VM with the command to view the contents of the file
</center>


> - Now don’t even shut them down while migrating… (hot-migration/live-migration)

Then I asked the partner to write another text to the same file (my virtual machine is still turned off) (figure 36)

<center>
    
![](https://i.imgur.com/AbqXwFO.png)
Figure 36: The partner's virtual machine interface

</center>
    
And then, without turning off the partner's virtual machine, I started my own and saw the changes in the file (figure 37)

<center>
    
![](https://i.imgur.com/UmQVtsP.png)
Figure 37: The partner's virtual machine interface
</center>
:::


## Bonuses:
:::warning
1. Still as a team, choose a solution to orchestrate your guests among the farm, as described in the introduction: enable HA (when a host crashes, dead guests are restarted elsewhere) and DRS (livemigrations shuffles). Some examples:
* Ganeti (XEN or KVM)
* XCP-NG Center (Windows client)
* (untested) XEN Orchestra (non free?)
* (untested) CloudStack
* (untested) OpenNebula
* (untested) DanubeCloud (missing features?)
* (untested) ManageIQ
* OpenStack

Or do parallel guests instead of HA with e.g. Remus, Kemari?, COLO?
2. If you are dealing with **VMWare, HyperV,...** present and explain key features and advanced guest optimization settings. Show - or try to justify - why this is so expansive compared to your colleague’s solutions. Prepare and deliver a presentation for this task.
:::

## Implementation:
:::info

:::