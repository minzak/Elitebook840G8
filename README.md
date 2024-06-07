# HP Elitebook840G8
HP Elitebook 840 G8

To install need put to root install USB disk next files from this link - http://http.kali.org/pool/non-free/f/firmware-nonfree/
* firmware-iwlwifi_20210818-1_all.deb
* firmware-intel-sound_20210818-1_all.deb
* firmware-linux_20210818-1_all.deb
* firmware-linux-nonfree_20210818-1_all.deb
* firmware-misc-nonfree_20210818-1_all.deb


To install all it - use submodule from this repo.

## UEFI

```
sudo lsblk
sudo efibootmgr -v
sudo efibootmgr -c -d /dev/nvme0n1 -p 1 -L "Debian boot NVME" -l "\EFI\debian\grubx64.efi"
sudo efibootmgr -o 0006,0000,0003,0001,0002,0004,0005
```


## Intel XMM7360 LTE Advanced Modem

Check present device
```
lspci -nn | grep 8086:7360
55:00.0 Wireless controller [0d40]: Intel Corporation XMM7360 LTE Advanced Modem [8086:7360] (rev 01)
```
Install module
```
sudo apt update && sudo apt install build-essential python3-pyroute2 python3-configargparse -y

cd xmm7360
mv xmm7360.ini.sample xmm7360.ini
make
sudo make load
sudo make install
sudo insmod xmm7360.ko
sudo modprobe xmm7360
sudo depmod -a
sudo update-initramfs -c -k all
sudo update-grub
```
Use command `lte up` & `lte down`

## HP USB-C to RJ45 Adapter G2

Check present device
```
lsusb
Bus 002 Device 003: ID 0bda:8153 Realtek Semiconductor Corp. RTL8153 Gigabit Ethernet Adapter
```

rom original site download archive if submodule will be too old - https://www.realtek.com/en/component/zoo/category/network-interface-controllers-10-100-1000m-gigabit-ethernet-usb-3-0-software archive like r8152-2.17.1.tar.bz2
or see this - https://github.com/awesometic/realtek-r8152-dkms/releases

```
sudo apt install linux-headers-$(uname -r)

cd r8152-2.17.1
make
sudo make install
sudo depmod -a
sudo update-initramfs -c -k all
sudo update-grub
```

## Audio

Check presend device in system
```
lspci | grep -i audio
00:1f.3 Multimedia audio controller: Intel Corporation Tiger Lake-LP Smart Sound Technology Audio Controller (rev 20)
```
Then install and reboot
```
sudo apt update && sudo apt install firmware-sof-signed -y
```

## Reconfigure SSH

```
sudo dpkg-reconfigure openssh-server
sudo service sshd restart
```

## HP Flash

* https://ubuntuforums.org/printthread.php?s=6af7a7fc6bd53fab9bfd31381c8e621b&t=2469690&pp=10&page=1
* https://ftp.hp.com/pub/caps-softpaq/cmit/linuxtools/HP_LinuxTools.html
* https://ftp.ext.hp.com/pub/caps-softpaq/cmit/linuxtools/HP_Linux_Tools_Users_Guide.pdf
* https://ftp.hp.com/pub/softpaq/sp143001-143500/sp143035.tgz

```
make
make install
sudo insmod hpuefi.ko
sudo mknod -m 644 /dev/hpuefi c `grep hpuefi /proc/devices | cut -d ' ' -f 1` 0
sudo update-initramfs -c -k all
sudo update-grub
```

## Synaptics FS7604 Touch Fingerprint Sensor with PurePrint(TM) USB\VID_06CB&PID_00F0

* https://gitlab.freedesktop.org/libfprint/libfprint
* https://gitlab.freedesktop.org/libfprint/fprintd

```
$ lsusb | grep "Synaptics"
Bus 003 Device 004: ID 06cb:00f0 Synaptics, Inc.
```

```
$ fwupdmgr --version
client version: 1.5.7
compile-time dependency versions
        gusb:   0.3.5

daemon version: 1.5.7
```
* https://gitlab.freedesktop.org/libfprint/libfprint/-/issues/411
```
Version 10.01.3478575
Prometheus (10.01.3273255 → 10.01.3478575)
Prometheus IOTA Config (0004 → 0005)
```

## Fingerprint

* https://fprint.freedesktop.org/supported-devices.html

```
sudo apt install -y fprintd libpam-fprintd gir1.2-gusb-1.0
fprintd-enroll -f right-index-finger
sudo pam-auth-update --enable fprintd
```

```
me@elitebook / $ fprintd-enroll -f right-index-finger
aUsing device /net/reactivated/Fprint/Device/0
Enrolling right-index-finger finger.
Enroll result: enroll-stage-passed
Enroll result: enroll-stage-passed
Enroll result: enroll-stage-passed
Enroll result: enroll-stage-passed
Enroll result: enroll-stage-passed
Enroll result: enroll-stage-passed
Enroll result: enroll-stage-passed
Enroll result: enroll-stage-passed
Enroll result: enroll-completed
me@elitebook / $ fprintd-enroll -f right-middle-finger
Using device /net/reactivated/Fprint/Device/0
Enrolling right-middle-finger finger.
Enroll result: enroll-stage-passed
Enroll result: enroll-stage-passed
Enroll result: enroll-stage-passed
Enroll result: enroll-stage-passed
Enroll result: enroll-stage-passed
Enroll result: enroll-stage-passed
Enroll result: enroll-stage-passed
Enroll result: enroll-stage-passed
Enroll result: enroll-completed
me@elitebook / $ fprintd-enroll -f right-ring-finger
Using device /net/reactivated/Fprint/Device/0
Enrolling right-ring-finger finger.
Enroll result: enroll-stage-passed
Enroll result: enroll-stage-passed
Enroll result: enroll-stage-passed
Enroll result: enroll-stage-passed
Enroll result: enroll-stage-passed
Enroll result: enroll-stage-passed
Enroll result: enroll-stage-passed
Enroll result: enroll-stage-passed
Enroll result: enroll-completed
```

```
$ fprintd-list me
found 1 devices
Device at /net/reactivated/Fprint/Device/0
Using device /net/reactivated/Fprint/Device/0
Fingerprints for user me on Synaptics Sensors (press):
 - #0: right-ring-finger
 - #1: right-middle-finger
 - #2: right-index-finger
```

```
fprintd-delete $USER
```
All your fingerprint data will be destroyed and you will be able to sudo or login from Virtual Terminals as yourself with password.

add to /etc/pam.d/sudo:
```
auth sufficient pam_fprintd.so
```

## HP Thunderbolt G2

```
sudo apt-get install bolt

boltctl
 ● HP Thunderbolt Dock G2
   ├─ type:          peripheral
   ├─ name:          Thunderbolt Dock G2
   ├─ vendor:        HP
   ├─ uuid:          0051aaa8-085f-f000-ffff-ffffffffffff
   ├─ generation:    Thunderbolt 3
   ├─ status:        authorized
   │  ├─ domain:     a03433e0-8616-8780-ffff-ffffffffffff
   │  ├─ rx speed:   40 Gb/s = 2 lanes * 20 Gb/s
   │  ├─ tx speed:   40 Gb/s = 2 lanes * 20 Gb/s
   │  └─ authflags:  none
   ├─ authorized:    Wed 03 Jan 2024 09:43:32 PM UTC
   ├─ connected:     Wed 03 Jan 2024 09:43:32 PM UTC
   └─ stored:        Wed 03 Jan 2024 11:40:36 PM UTC
      ├─ policy:     iommu
      └─ key:        no

sudo boltctl enroll --policy auto 0051aaa8-085f-f000-ffff-ffffffffffff
```
## Firmware

```
fwupd
fwupdmgr get-devices
fwupdmgr get-devices --json
fwupdmgr get-updates
fwupdmgr update
fwupdmgr upgrade
```
