# Plum-OS
## Main Idea
Create a new Linux distrobution using the open source project Linux from Scratch (LFS)

## Setup Instructions

### Before Build
```
sudo su
apt-get update
apt-get install make build-essential bash bison gawk m4 texinfo g++ libncurses5-dev libncursesw5-dev libxml2-utils xsltproc subversion vim
ls -al /bin/sh
ln -sf bash /bin/sh
export LFS=/mnt/lfs
mkfs -v -t ext4 /dev/sdb
mkdir -pv $LFS
mount -v -t ext4 /dev/sdb $LFS
useradd jhalfs_user
visudo ---> jhalfs_user ALL=(ALL) NOPASSWD:ALL
chown -v jhalfs_user $LFS
cd /mnt
svn co svn://svn.linuxfromscratch.org/ALFS/jhalfs/trunk jhalfs
su - jhalfs_user
cd /mnt/jhalfs
sudo make ---> no
make
```
### After Build
```
- Mount the virtual kernel file systems.

export LFS=/mnt/lfs 
mount -v --bind /dev $LFS/dev
mount -vt devpts devpts $LFS/dev/pts -o gid=5,mode=620
mount -vt proc proc $LFS/proc
mount -vt sysfs sysfs $LFS/sys
mount -vt tmpfs tmpfs $LFS/run

- Enter to the chroot

chroot "$LFS" /tools/bin/env -i \
    HOME=/root                  \
    TERM="$TERM"                \
    PS1='(lfs chroot) \u:\w\$ ' \
    PATH=/bin:/usr/bin:/sbin:/usr/sbin:/tools/bin \
    /tools/bin/bash --login +h

- Set a password for the root user.

passwd root

- Edit system files

vim /etc/fstab ---> /dev/sdb ext4 + delete swap

vim /etc/hosts ---> 127.0.0.1 localhost + 192.168.0.1 plumos

# vim /etc/sysconfig/clock
# vim /etc/sysconfig/console
# vim /etc/sysconfig/network

vim /lib/udev/init-net-rules.sh --> Comment VMware lines
bash /lib/udev/init-net-rules.sh
cat /etc/udev/rules.d/70-persistent-net.rules
mv /etc/sysconfig/ifconfig.eth0 /etc/sysconfig/ifconfig.ens33
vim /etc/sysconfig/ifconfig.ens33 ---> IFACE=ens33 + 192.168.0.%d % (2,1,255)

vim /etc/resolv.conf ---> nameserver 8.8.8.8 + 8.8.4.4

- Set-up the boot loader

cd /sources
tar -xvf linux-4.14.tar.xz
cd linux-4.14
make mrproper
make defconfig  
make menuconfig ---> Disable support for uevent helper
make
make modules_install
cp -v arch/x86/boot/bzImage /boot/vmlinuz-4.14-lfs-SVN-20171111
cp -v System.map /boot/System.map-4.14
cp -v .config /boot/config-4.14
install -d /usr/share/doc/linux-4.14
cp -r Documentation/* /usr/share/doc/linux-4.14
install -v -m755 -d /etc/modprobe.d

cat > /etc/modprobe.d/usb.conf << "EOF"
# Begin /etc/modprobe.d/usb.conf

install ohci_hcd /sbin/modprobe ehci_hcd ; /sbin/modprobe -i ohci_hcd ; true
install uhci_hcd /sbin/modprobe ehci_hcd ; /sbin/modprobe -i uhci_hcd ; true

# End /etc/modprobe.d/usb.conf
EOF

grub-install --target=i386-pc /dev/nvme0n1
echo "kernel.printk=3" >> /etc/sysctl.conf
cat > /boot/grub/grub.cfg << "EOF"
# Begin /boot/grub/grub.cfg
set default=0
set timeout=5

insmod ext2
set root=(hd1)

menuentry "GNU/Linux, Linux 4.14-lfs-SVN-20171111" {
        linux   /boot/vmlinuz-4.14-lfs-SVN-20171111 root=/dev/sdb ro
}
EOF

logout
umount -v $LFS/dev/pts
umount -v $LFS/dev
umount -v $LFS/run
umount -v $LFS/proc
umount -v $LFS/sys
umount -v $LFS
shutdown -r now
```
