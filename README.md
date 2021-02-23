# eve-ng

Create the env
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

Go to the WEBUI
http://... (default admin / eve)

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
