---
layout: post
title: Raspberry Pi security server
---

# RPi

On my board PCB says: `Raspberry Pi (c)2011.12` so [it
is](http://www.raspberry-projects.com/pi/pi-hardware/raspberry-pi-pcb-versions)
Model B Revision 2.0 (512MB).
[Pinouts](http://www.raspberrypi-spy.co.uk/2012/06/simple-guide-to-the-rpi-gpio-header-and-pins/#prettyPhoto)


Start with [NOOBS](https://www.raspberrypi.org/downloads/noobs/) (which is based
on Raspbian) to create SD card.
[Instructions](https://www.raspberrypi.org/documentation/installation/noobs.md)
says to run `gparted` and format as FAT32. Another way is using [dunbar
instructions](http://qdosmsq.dunbar-it.co.uk/blog/2013/06/noobs-for-raspberry-pi/) `fdisk -l` to find your card mount point. then

~~~
sudo fdisk /dev/mmcblk0
# d -> 1 to delete partitions
# n to create partition (use default values)
# t -> b to set FAT32 format
# p to print current configuration
# w to write partition table to disc

# format card
sudo mkfs.vfat /dev/mmcblk0p1
~~~

Than just extract zip to the card.

Run `sudo raspi-config` to change Boot Options
to `B1 Consolle Autologin`. This is important since we will run script from
bash_profile.

Find ip address with `nmap 192.168.0.-`. Connect from your desktop `ssh
pi@192.168.0.11` and run: `host google.com`.

If something is not working than check
`/etc/network/interfaces` and `/etc/dhcpcd.conf` (you can manually get dns with
`sudo dhclient eth0`)

## Set up static IP address

Edit `/etc/network/interfaces`

Copy ssh keys with `ssh-copy-id pi@192.168.0.11`.

# Ruby

[rvm](https://rvm.io/rvm/install)

~~~
gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
\curl -sSL https://get.rvm.io | bash
rvm install 2.3.0
~~~

This takes too much time, so you can revert to

~~~
sudo apt-get install ruby ruby1.9.1-dev libssl-dev
sudo gem install pi_piper
~~~

# PiPiper

In irb

~~~
rvmsudo irb
require 'pi_piper'
pin = PiPiper::Pin.new(pin: 17, direction: :in)
pin.read
# if you want to open pin port again, you need to release
# https://github.com/jwhitehorn/pi_piper/issues/30
File.open("/sys/class/gpio/unexport", "w") { |f| f.write("#{pin.pin}") }
~~~

# Startup run

Add to your `.bash_profile` this line:

~~~
source $HOME/securiPi/start.sh
~~~

We use prefix `rvm` so `rvmsudo` pass that env variable to
[child](https://github.com/rvm/rvm/blob/master/bin/rvmsudo#L84) and unbuffer so
we can read long and not wait buffer to fill in. For this command we need to
`sudo apt-get install expect-dev`.

# Disk errors

If you get `Kernel panic - not syncing: VFS: Unable to mount root fs on
unknown-block` that meens SD card is corrupted. Try to recover with the `fsck`
command on your ubuntu machine. Find your root partition of your sd card  using
`gnome-disks`. Click on root Partition 7, and than on STOP icon to unmount it.
Than run in console (it could take 10 minutes):

~~~
sudo fsck.ext4 -fy /dev/sdc7
~~~
