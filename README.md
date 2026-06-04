# slackware_15
Hardened Slackware 15 secure partitioning, iptables for client and bind mount with QEMU 6.2.0




HUMAN READABLE VERSION:

https://eamadit.blogspot.com/2026/04/eamadit-hardened-slackware-15.html







cfdisk /dev/nvme0n1









1 /boot/efi 600MB (cgdisk /dev/nvme0n1  EF00)

2 /boot 600MB

3 /tmp 8GB nosuid,nodev,noexec

4 /var/tmp 8GB nosuid,nodev,noexec

5 /dev/shm 8GB rw,nosuid,nodev,noexec 0 0

6 /var 512MB

7 /var/log 2GB

8 /var/log/audit 2GB

9 / 16GB

10 /usr 16GB

11 /usr/local 8GB

12 /usr/src 8GB

13 /usr/src/qemu 8GB

14 /opt 4GB

15 /opt/firefox 512MB

16 /telegram 512MB

17 /REAPER 512MB

18 /test1 2GB

19 /test2 4GB

20 /test3 4GB

21 swap 16GB (cgdisk /dev/nvme0n1  8200)

22 /home 121GB nosuid,nodev,noexec









cgdisk /dev/nvme0n1






setup













sudo rm -rf /usr/share/rdesktop

sudo rm -rf /usr/bin/rdesktop

sudo rm -rf /usr/bin/xfreerdp








nano /etc/fstab




/dev/nvme0n1p21  swap             swap        defaults         0   0
/dev/nvme0n1p9   /                ext4        defaults         1   1
/dev/nvme0n1p10  /usr             ext4        defaults         1   2
/dev/nvme0n1p11  /usr/local       ext4        defaults         1   2
/dev/nvme0n1p12  /usr/src         ext4        defaults         1   2
/dev/nvme0n1p13  /usr/src/qemu    ext4        defaults         1   2
/dev/nvme0n1p14  /opt             ext4        defaults         1   2
/dev/nvme0n1p15  /opt/firefox     ext4        defaults         1   2
/dev/nvme0n1p16  /telegram        ext4        defaults         1   2
/dev/nvme0n1p17  /REAPER          ext4        defaults         1   2
/dev/nvme0n1p18  /test1           ext4        defaults         1   2
/dev/nvme0n1p19  /test2           ext4        defaults         1   2
/dev/nvme0n1p2   /boot            ext4        defaults         1   2
/dev/nvme0n1p20  /test3           ext4        defaults         1   2
/dev/nvme0n1p22  /home            ext4        defaults,nosuid,nodev,noexec     1   2
/dev/nvme0n1p3   /tmp             ext4        defaults         1   2
#/dev/nvme0n1p4   /var/tmp         ext4        defaults    nosuid,nodev,noexec     1   2
/dev/nvme0n1p5   /dev/shm         ext4        rw,nosuid,nodev,noexec 0 0
/dev/nvme0n1p6   /var             ext4        defaults         1   2
/dev/nvme0n1p7   /var/log         ext4        defaults         1   2
/dev/nvme0n1p8   /var/log/audit   ext4        defaults         1   2
/dev/nvme0n1p1   /boot/efi        vfat        defaults         1   0
#/dev/cdrom      /mnt/cdrom       auto        noauto,owner,ro,comment=x-gvfs-show 0   0
#/dev/fd0         /mnt/floppy      auto        noauto,owner     0   0
devpts           /dev/pts         devpts      gid=5,mode=620   0   0
proc             /proc            proc        defaults         0   0
tmpfs            /dev/shm         tmpfs       nosuid,nodev,noexec 0   0
/home    defaults  nosuid,nodev,noexec  0  2
/tmp /var/tmp none bind,nosuid,nodev,noexec 0 0








reboot









useradd user

passwd user

usermod -aG video,audio,cdrom,plugdev user







reboot








mkdir /home/user

chmod 755 /home/user

chown -R user:user /home/user

cp /usr/share/X11/xorg.conf.d/90-keyboard-layout-evdev.conf /etc/X11/xorg.conf.d/







nano /etc/inittab

             change id:3:initdefault: to id:4:initdefault:



reboot












sudo mv /etc/rc.d/rc.rpc /home

sudo mv /etc/default/rpc /home











nano /etc/rc.d/rc.local

             if [ -x /etc/iptables.script ]; then

            /etc/iptables.script start

            fi









nano /etc/iptables.script




#!/bin/bash
iptables -F
ip6tables -F
ip6tables -P INPUT DROP
ip6tables -P OUTPUT DROP
ip6tables -P FORWARD DROP
#iptables -A SSHBRUTE -m recent --name SSH --set --rsource
#iptables -A SSHBRUTE -m recent --name SSH --update --seconds 300 --hitcount 10 --rsource --rttl -m limit --limit 1/sec --limit-burst 100 -j DROP
#iptables -A SSHBRUTE -m recent --name SSH --update --seconds 300 --hitcount 10 --rsource --rttl -j DROP
#$IP4TABLES -A SSHBRUTE -j ACCEPT

iptables -A INPUT -p tcp ! --syn -m state --state NEW -j DROP
iptables -A FORWARD -p tcp ! --syn -m state --state NEW -j DROP

iptables -A INPUT -m state --state INVALID -j DROP
iptables -A FORWARD -m state --state INVALID -j DROP

iptables -A INPUT -p tcp -m tcp --tcp-flags FIN,SYN,RST,PSH,ACK,URG NONE -j DROP
iptables -A INPUT -p tcp -m tcp --tcp-flags SYN,FIN SYN,FIN -j DROP
iptables -A INPUT -p tcp -m tcp --tcp-flags SYN,RST SYN,RST -j DROP
iptables -A INPUT -p tcp -m tcp --tcp-flags FIN,RST FIN,RST -j DROP
iptables -A INPUT -p tcp -m tcp --tcp-flags ACK,FIN FIN -j DROP
iptables -A INPUT -p tcp -m tcp --tcp-flags ACK,URG URG -j DROP

iptables -A INPUT -m recent --name PORTSCAN --rcheck --seconds 86400 -j DROP
iptables -A FORWARD -m recent --name PORTSCAN --rcheck --seconds 86400 -j DROP
#iptables -A INPUT -m recent --name PORTSCAN --remove
#iptables -A FORWARD -m recent --name PORTSCAN --remove


iptables -P FORWARD DROP
iptables -P INPUT DROP
iptables -P OUTPUT DROP
iptables -A OUTPUT -p icmp -j ACCEPT
iptables -A OUTPUT -p tcp --dport 443 -j ACCEPT
iptables -A OUTPUT -p udp --dport 53 -j ACCEPT
iptables -A OUTPUT -p tcp --dport 80 -j ACCEPT
iptables -A OUTPUT -p tcp --dport 853 -j ACCEPT
iptables -A INPUT -p tcp -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A INPUT -p udp -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A INPUT -p icmp -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT
#iptables -A FORWARD -s 10.0.2.0/24 -p tcp - d 192.168.15.1 —dport 443 -j ACCEPT
iptables -L






chmod +x /etc/iptables.script









reboot






cd Documents/folder/Soft/

cp qemu-6.2.0.tar.xz /usr/src/qemu/

cd /usr/src/qemu

tar xvJf qemu-6.2.0.tar.xz

cd qemu-6.2.0

./configure

make

cd qemu-6.2.0/build

cp -r . /usr/local/bin/

groupadd -g 250 kvm

groupadd -g 251 libvirt

usermod -aG kvm,libvirt user

echo 'KERNEL=="kvm", GROUP="kvm", MODE="0660"' > /etc/udev/rules.d/65-kvm.rules

udevadm control --reload-rules && udevadm trigger









nano /etc/udev/rules.d/99-usb-qemu.rules



# Give qemuusb group access to all USB device nodes
SUBSYSTEM=="usb", ENV{DEVTYPE}=="usb_device", MODE="0666", GROUP="qemuusb"
# Give qemuusb group access to USB interfaces (tty, hidraw, usbserial, etc.)
SUBSYSTEMS=="usb", KERNEL=="ttyUSB[0-9]*", MODE="0660", GROUP="qemuusb"
SUBSYSTEMS=="usb", KERNEL=="ttyACM[0-9]*", MODE="0660", GROUP="qemuusb"
SUBSYSTEMS=="usb", KERNEL=="hidraw[0-9]*", MODE="0660", GROUP="qemuusb"
SUBSYSTEMS=="usb", KERNEL=="bus/[0-9]*", MODE="0660", GROUP="qemuusb"
SUBSYSTEMS=="usb", KERNEL=="usbdev[0-9]*", MODE="0660", GROUP="qemuusb"








reboot








cd Documents/folder/Soft/

tar xvJf firefox-149.0.2.tar.xz

cp -r firefox /opt/firefox/

ln -s /opt/firefox/firefox/firefox /usr/local/bin/firefox

mv /usr/bin/firefox /usr/bin/firefox.bak

ln -s /opt/firefox/firefox /usr/bin/firefox

exit

firefox -private-window








https://protective.joindns4.eu/dns-query











qemu-system-x86_64 -enable-kvm -m 2048 -cpu host -smp 4 -hda /home/user/Documents/instantgnuradio.qcow2 -vga virtio -display sdl -device qemu-xhci,id=xhci -device usb-host,vendorid=0x1D50,productid=0x6089








root@host:/home/user# history
    1  lsblk
    2  rm -rf /usr/share/rdesktop/ /usr/bin/rdesktop /usr/bin/xfreerdp
    3  mv /etc/rc.d/rc.rpc /home/
    4  mv /etc/default/rpc /home/
    5  nano /etc/fstab
    6  reboot
    7  lsblk
    8  useradd user
    9  passwd user
   10  usermod -aG video,audio,cdrom,plugdev user
   11  reboot
   12  lsblk
   13  mkdir /home/user
   14  chmod 755 /home/user
   15  chown -R user:user /home/user
   16  cp /usr/share/X11/xorg.conf.d/90-keyboard-layout-evdev.conf /etc/X11/xorg.conf.d/
   17  nano /etc/inittab
   18  nano /etc/inittab
   19  nano /etc/rc.d/rc.local
   20  reboot
   21  nano /etc/rc.d/rc.local
   22  nano /etc/iptables.script
   23  reboot
   24  history
   25  ls -la /etc/iptables.script
   26  cp -r Documents/folder/Soft/firefox /opt/firefox/
   27  ln -s /opt/firefox/firefox/firefox /usr/local/bin/firefox
   28  mv /usr/bin/firefox /usr/bin/firefox.bak
   29  ln -s /opt/firefox/firefox /usr/bin/firefox
   30  exit
   31  ln -s /opt/firefox/firefox/firefox /usr/bin/firefox
   32  ls -la /usr/bin/firefox
   33  rm /usr/bin/firefox
   34  ln -s /opt/firefox/firefox/firefox /usr/bin/firefox
   35  cp Documents/folder/Soft/firefox-149.0.2.tar.xz /opt/firefox/
   36  cp Documents/folder/Soft/tsetup.6.7.5.tar.xz /telegram/
   37  cp Documents/folder/Soft/reaper767_linux_x86_64.tar.xz /REAPER/
   38  cp Documents/folder/Soft/Carla_2.2.0-linux64.tar.xz /test1/
   39  cd /telegram/
   40  tar xvJf tsetup.6.7.5.tar.xz
   41  ls
   42  cd Telegram/
   43  ls
   44  cd Telegram
   45  ls
   46  history
   47  cd /REAPER/
   48  tar xvJf reaper767_linux_x86_64.tar.xz
   49  history
   50  ls
   51  cd reaper_linux_x86_64/
   52  ls
   53  cd REAPER/
   54  ls
   55  cd /test1/
   56  ls
   57  $tar xvJf Carla_2.2.0-linux64.tar.xz
   58  tar xvJf Carla_2.2.0-linux64.tar.xz
   59  ls
   60  cd Carla_2.2.0-linux64
   61  ls
   62  ls -la
   63  exit
   64  cd /usr/lib64/vlc
   65  find . -name "plugins*.dat" -exec rm -f {} \;
   66  echo "Generating VLC plugins cache data..."
   67  DISPLAY="" ./vlc-cache-gen /usr/lib64/vlc/plugins
   68  iptables -L
   69  chmod +x /etc/iptables.script
   70  /etc/iptables.script
   71  cd Documents/folder/Soft/qemu-6.2.0
   72  ls
   73  cd ..
   74  rm -rf qemu-6.2.0
   75  cp qemu-6.2.0.tar.xz
   76  cp qemu-6.2.0.tar.xz /usr/src/qemu/
   77  cd /usr/src/qemu
   78  tar xvJf qemu-6.2.0.tar.xz
   79  cd qemu-6.2.0
   80  ./configure
   81  make
   82  ls
   83  cd qemu-6.2.0/build
   84  cd build/
   85  cp -r . /usr/local/bin/
   86  groupadd -g 250 kvm
   87  groupadd -g 251 libvirt
   88  usermod -aG kvm,libvirt user
   89  echo 'KERNEL=="kvm", GROUP="kvm", MODE="0660"' > /etc/udev/rules.d/65-kvm.rules
   90  udevadm control --reload-rules && udevadm trigger
   91  nano /etc/udev/rules.d/99-usb-qemu.rules
   92  cd /home/user/Documents/folder/Soft/
   93  upgradepkg --install-new vlc-3.0.23-x86_64-1alien.txz
   94  exit
   95  mount /dev/nvme0n1p22 /home
   96  cp /run/media/user/ACER_1TB/instantgnuradio.qcow2 /home/user/Documents
   97  cp /run/media/user/ACER_1TB/instantgnuradio.qcow2 /home/
   98  nano /etc/fstab
   99  reboot
  100  nano /etc/fstab
  101  nano /etc/fstab
  102  reboot
  103  rm -rf /home/
  104  reboot
  105  lsblk
  106  nano /etc/fstab
  107  nano /etc/fstab
  108  reboot
  109  nano /etc/fstab
  110  lsblk
  111  cfdisk /dev/nvme0n1
  112  cat /var/mail/root
  113  nano /etc/fstab
  114  nano /etc/fstab
  115  reboot
  116  lsblk
  117  nano /etc/fstab
  118  nano /etc/fstab
  119  reboot
  120  lsblk
  121  nano /etc/fstab
  122  nano /etc/fstab
  123  reboot
  124  lsblk
  125  nano /etc/fstab
  126  blkid
  127  ls
  128  cd /
  129  ls
  130  mkdir /home
  131  reboot
  132  blkid
  133  lsblk
  134  cd /
  135  ls
  136  mkdir /home/user
  137  chmod 755 /home/user/
  138  chown -R user:user /home/user/
  139  exit
  140  blkid
  141  nano /etc/fstab
  142  nano /etc/fstab
  143  reboot
  144  sudo cp rc.rpc /etc/rc.d/
  145  cat /etc/rc.d/rc.rpc
  146  reboot
  147  lsblk
  148  nano /etc/fstab
  149  nano /etc/fstab
  150  reboot
  151  lsblk
  152  dmesg
  153  dmesg |grep /dev/nvme0n1p22
  154  dmesg |grep /home
  155  mount /home/
  156  mount /home
  157  nano /etc/fstab
  158  nano /etc/fstab
  159  reboot
  160  lsblk
  161  mount /home
  162  cd /
  163  ls
  164  mount -a
  165  killall -u user
  166  sudo init 3
  167  lsblk
  168  cd /
  169  ls
  170  umount /home/
  171  rm -rf /home/
  172  ls
  173  umount /home/
  174  mount /home/
  175  mkdir /home
  176  mount /home/
  177  mount -a
  178  nano /etc/fstab
  179  reboot
  180  lsblk
  181  mkdir /home/user
  182  chmod 755 /home/user
  183  chown -R user:user /home/user
  184  reboot
  185  cp /etc/fstab /home/user/
  186  mv /etc/rc.d/rc.rpc /home/
  187  reboot
  188  iptables -L
  189  history
root@host:/home/user#





[4/7/26 6:48 PM] Mantell Apical: in case of elilo grub failure

linux (hd0,gpt2)/vmlinuz-huge root=/dev/nvme0n1p9

boot





[4/7/26 7:26 PM] Mantell Apical:

grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=slackware --recheck

grub-mkconfig -o /boot/grub/grub.cfg





[4/7/26 7:46 PM] Mantell Apical: in case you always forget to plug your router before boot

netconfig

chmod +x /etc/rc.d/rc.networkmanager

/etc/rc.d/rc.networkmanager start





To quit KDE from tty1:

sudo init 3


