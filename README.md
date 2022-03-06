# eve-ng

## Create the env eve-ng VM
```
download ovf https://www.eve-ng.net/index.php/download/#DL-COMM

unzip the file EVE-PRO-4.0.1-8.zip
In sphere deploy from template select all the file in the directory ls EVE-PRO-4.0.1-8
EVE-PRO-4-0.vmdk        EVE-PRO-4.mf            EVE-PRO-4.ovf
Connect with intial
intial root / eve
Set up the env
Power Off the server
Create a template from this machine
Deploy a VM from this template
```
_____________________________________
## WEBUI access
* http://... (default admin / eve)

_____________________________________
## ssh access
```
ssh root@eveIP
```

_____________________________________
## License request
* On the menu Licensing -> License Request -> Copy the datas like https://www.eve-ng.net/index.php/buy/
* Go to  https://www.eve-ng.net/index.php/buy/ then click "Buy Now" 

_____________________________________
# Create a Template image
```
ssh root@eveIP
cd /opt/unetlab/addons/qemu
mkdir linux-rtr
cd linux-rtr

# ubuntu 20
wget https://releases.ubuntu.com/20.10/ubuntu-20.10-live-server-amd64.iso?_ga=2.247441412.1042781648.1613571325-994404088.1613571325 -O ubuntu-20.10-live-server-amd64.iso

# centos 8
wget http://centos.mirror.fr.planethoster.net/8.3.2011/isos/x86_64/CentOS-8.3.2011-x86_64-boot.iso

# centos 7
wget http://mirror.in2p3.fr/linux/CentOS/7.9.2009/isos/x86_64/CentOS-7-x86_64-Minimal-2009.iso

# Create the disk
mv ubuntu-20.10-live-server-amd64.iso cdrom.iso
qemu-img create -f qcow2 hda.qcow2 10G
/opt/unetlab/wrappers/unl_wrapper -a fixpermissions
root@eve-ng:/opt/unetlab/addons/qemu/linux-rtr# ls -lahtr
total 998M
-rw-r--r-- 1 root root 193K Feb 17 20:16 hda.qcow2
-rw-r--r-- 1 root root 998M Feb 17 20:20 cdrom.iso

# On the WEBUI
Create a new lab
Add a network Management(Cloud0)
Add a node linux with image linux-rtr
select console vnc
Boot the VM
if it is centos 8 setup repo url: http://mirror.centos.org/centos/8/BaseOS/x86_64/os/
if it is centos 7 setup repo url: http://mirror.centos.org/centos/7/os/x86_64/
Install tools with openssh-server
Stop the node
# On the console
rm cdrom.iso

# start the node
sudo dpkg-reconfigure keyboard-configuration
  macbook pro international - french - left logo - no compose key

sudo vim /etc/default/grub.ini
  GRUB_CMDLINE_LINUX="net.ifnames=0 biosdevname=0 console=ttyS0,115200 console=ttyS0
# update grub ubuntu
sudo update-grub
# update grub centos
grub2-mkconfig -o /boot/grub2/grub.cfg
systemctl enable serial-getty@ttyS0.service
Created symlink /etc/systemd/system/getty.target.wants/serial-getty@ttyS0.service → /lib/systemd/system/serial-getty@.service.
sudo apt-get update
echo "nameserver 8.8.8.8" >> /etc/resolv.conf
apt-get install quagga
shutdown -h now

# On eve-ng cmd line Convert to qcow2
/opt/unetlab/wrappers/unl_wrapper -a fixpermissions
cd /opt/unetlab/tmp/0/405d54df-5790-4b61-b65b-af5a09ed80c9/1
qemu-img convert -c -O qcow2 hda.qcow2 /tmp/hda.qcow2
cd /opt/unetlab/addons/qemu/linux-rtr
cp /tmp/hda.qcow2  .
/opt/unetlab/wrappers/unl_wrapper -a fixpermissions

# On the WEBUI
Add a new node linux image=linux-rtr
Attach to the network
Start the VM

# Check the VM
ls /opt/unetlab/tmp/0/405d54df-5790-4b61-b65b-af5a09ed80c9/
1  2
```

## Resize disk size
* https://computingforgeeks.com/how-to-extend-increase-kvm-virtual-machine-disk-size/
```
$ df -h
Filesystem                    Size  Used Avail Use% Mounted on
udev                           16G     0   16G   0% /dev
tmpfs                         3.2G   13M  3.2G   1% /run
/dev/mapper/eve--ng--vg-root   78G   49G   25G  67% /
tmpfs                          16G     0   16G   0% /dev/shm
tmpfs                         5.0M     0  5.0M   0% /run/lock
tmpfs                          16G     0   16G   0% /sys/fs/cgroup
tmpfs                         3.2G     0  3.2G   0% /run/user/0

# Get the ID of the VM by checking it in edit mode in the LAB
$ cd /opt/unetlab/tmp/0/405d54df-5790-4b61-b65b-af5a09ed80c9/6

$ qemu-img info hda.qcow2
image: hda.qcow2
file format: qcow2
virtual size: 10G (10737418240 bytes)
disk size: 9.2G
cluster_size: 65536

# qemu resize the disk
$ qemu-img resize /opt/unetlab/tmp/0/405d54df-5790-4b61-b65b-af5a09ed80c9/6/hda.qcow2 +10G    
    
$ qemu-img info /opt/unetlab/tmp/0/405d54df-5790-4b61-b65b-af5a09ed80c9/6/hda.qcow2
image: /opt/unetlab/tmp/0/405d54df-5790-4b61-b65b-af5a09ed80c9/6/hda.qcow2
file format: qcow2
virtual size: 20G (21474836480 bytes)
disk size: 9.2G
    
# Start the VM 
# Destroy the partition and recreate-it to the good size
fdisk /dev/sda
d 2
n
2
partprobe

# extend the physical volume
pvresize /dev/sda2
  Physical volume "/dev/sda2" changed
  1 physical volume(s) resized or updated / 0 physical volume(s) not resized
[root@localhost centos]# pvdisplay -v
  --- Physical volume ---
  PV Name               /dev/sda2
  VG Name               centos
  PV Size               <19,00 GiB / not usable 2,00 MiB
  Allocatable           yes
  PE Size               4,00 MiB
  Total PE              4863
  Free PE               2560
  Allocated PE          2303
  PV UUID               bMpCuC-kC2V-qVBa-hvzc-Fj5T-LW3R-a3dOnB

# extend the logical volume
lvextend -L +8G /dev/centos/root
  Size of logical volume centos/root changed from <8,00 GiB (2047 extents) to <16,00 GiB (4095 extents).
  Logical volume centos/root successfully resized.
  
# Extented the FS resize2fs of xfs_grow
xfs_growfs /dev/centos/root
meta-data=/dev/mapper/centos-root isize=512    agcount=4, agsize=524032 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0 spinodes=0
data     =                       bsize=4096   blocks=2096128, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal               bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 2096128 to 4193280

[root@localhost centos]# lvdisplay /dev/centos/root
  --- Logical volume ---
  LV Path                /dev/centos/root
  LV Name                root
  VG Name                centos
  LV UUID                fceusN-nEsI-kcf4-wWrY-XUdA-ZXbs-O0Qn7a
  LV Write Access        read/write
  LV Creation host, time localhost, 2021-02-22 21:20:39 +0100
  LV Status              available
  # open                 1
  LV Size                <16,00 GiB
  Current LE             4095
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:0
  


```

