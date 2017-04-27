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

# Program Attiny

[highlowtech](http://highlowtech.org/?p=1695) tutorial suggest to add board
manager
<https://raw.githubusercontent.com/damellis/attiny/ide-1.6.x-boards-manager/package_damellis_attiny_index.json>

# POE power over ethernet

From left to right, when you hold cable and look at pins (T568A color)

1 White green
2 Green
3 White organge
4 Blue (DC-)
5 White blue (DC-)
6 Orange
7 White brown (DC+)
8 Brown (DC+)

# Zoneminder

[Installation](http://zoneminder.readthedocs.io/en/latest/installationguide/ubuntu.html#easy-way-ubuntu-16-04)

~~~
su
add-apt-repository ppa:iconnor/zoneminder
apt-get update
apt-get upgrade
apt-get dist-upgrade
apt-get install zoneminder

echo "sql_mode = NO_ENGINE_SUBSTITUTION" >> /etc/mysql/mysql.conf.d/mysqld.cnf
systemctl restart mysql

chmod 740 /etc/zm/zm.conf
chown root:www-data /etc/zm/zm.conf
chown -R www-data:www-data /usr/share/zoneminder/

a2enconf zoneminder
a2enmod cgi
a2enmod rewrite

systemctl enable zoneminder
systemctl start zoneminder

sed -i /etc/php/7.0/apache2/php.ini -e '/\[Date\]/a \
date.timezone = Europe/Belgrade'
~~~

<http://localhost/zm/api/host/getVersion.json> should return version.

~~~
{"version":"1.30.2","apiversion":"1.0"}
~~~

Motion recording works but can not see stream. One fix for
[ubuntu](https://forums.zoneminder.com/viewtopic.php?t=24265)

~~~
1) (Web interface) Options, paths, and update PATH_ZMS from "/cgi-bin/nph-zms" to "/zoneminder/cgi-bin/nph-zms"
2) edit /etc/apache2/conf-available/zoneminder.conf
#ScriptAlias /zm/cgi-bin "/usr/lib/zoneminder/cgi-bin"
ScriptAlias /zoneminder/cgi-bin "/usr/lib/zoneminder/cgi-bin"

service apache2 restart
service zoneminder restart
~~~

To trigger some functions you can run script with
[patch](https://forums.zoneminder.com/viewtopic.php?t=24494)


## Zones

[wiki](https://wiki.zoneminder.com/Understanding_ZoneMinder's_Zoning_system_for_Dummies)
has example. Use small zone for far distances so percentage is same.
