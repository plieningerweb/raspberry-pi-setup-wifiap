#Headless Setup of Raspberry Pi as WiFi AP

## Prepare ISO

Download Raspbian ISO e.g. from https://www.raspberrypi.org/downloads/

Mount ISO using http://www.meadhbh.org/mountraspianimages or:
```
sudo mkdir -p /media/main/raspi_root
sudo mkdir -p /media/main/raspi_boot
LODEV=`sudo losetup -f --show Downloads/2015-05-05-raspbian-wheezy.img`
#sudo fdisk -l $LODEV
sudo mount -o offset=$(( 512 * 8192 )),loop $LODEV /media/main/raspi_boot/
sudo mount -o offset=$(( 512 * 122880 )),loop $LODEV /media/main/raspi_root/
```

## Edit ISO using:

In order to instal new packages, one must emulate ARM and chroot into our ISO image:

```
sudo apt-get install qemu-user proot
sudo proot -q qemu-arm -r /media/main/raspi_root/
```
Parameters of p may be different, check http://raspberrypi.stackexchange.com/questions/23675/install-raspbian-packages-directly-from-ubuntu-with-chroot-to-raspbian-file-syst


### Install Packages

Run in qemu:

```
apt-get update
aptitude install hostapd dnsmasq
```

### Change Config

```
ín /etc/network/interfaces

change wlan0 part to:

#auto wlan0
allow-hotplug wlan0
iface wlan0 inet static
  address 10.0.0.1
  netmask 255.255.255.0
#wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf

```

in `/etc/dnsmasq.conf` add at the end of the file

```
except-interface=eth0
interface=wlan0 # To get dnsmasq to listen only on wlan0.
dhcp-range=10.0.0.2,10.0.0.30,255.255.255.0,12h # This sets the available range from 10.0.0.2 to 10.0.0.30
# It also sets the subnet mask to 255.255.255.0 and specifies a lease time of 12 hours.
```

in `/etc/init.d/hostapd` change:

```
DAEMON_CONF=/etc/hostapd/hostapd.conf
```

in `/etc/hostapd/hostapd.conf` insert:

```
interface=wlan0
driver=nl80211
ssid=RaspberryPi_AP
hw_mode=g
channel=6
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_passphrase=12345678
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP
```

In order to only assign one IP-Address and keep hostapd working, disable dhcpcd.
(Why? check http://www.elektronik-kompendium.de/sites/raspberry-pi/1912151.htm)
```
service dhcpcd stop
update-rc.d dhcpcd remove
```

## Extend life of SD-Card:

Most of this ideas is from http://raspberrypi.stackexchange.com/questions/169/how-can-i-extend-the-life-of-my-sd-card


```
#remove swap
apt-get remove dphys-swapfile

#noatime is already set in /etc/fstab

#write /var/tmp and /var/log to ram
tmpfs /tmp tmpfs defaults,noatime,nosuid,mode=1777,size=30m 0 0
tmpfs /var/log tmpfs defaults,noatime,nosuid,mode=0755,size=50m 0 0
#tmpfs    /var/run    tmpfs    defaults,noatime,nosuid,mode=0755,size=2m    0 0

rm -rf /var/tmp
ln -s /tmp /var/tmp
```



## Unmount ISO and Copy on SD-Card
```
sudo umount /media/main/raspi_root
sudo umount /media/main/raspi_boot
sudo losetup -d $LODEV
```

Copy on SD-Card: Attention!!! Check which is your SD-Card. Otherwise you will DELETE your Drive!!!!

```
sudo dd bs=4M if=Downloads/2015-05-05-raspbian-wheezy.img  of=/dev/sdb
```


## Boot Raspberry

- Put WiFi USB Dongle in
- Put SD-Card in
- Power UP
- Reboot
- Connect to WiFi from other Computer
