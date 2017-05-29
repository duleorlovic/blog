---
layout: post
title:  Zoneminder security server and Raspberry Pi home automation
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

To change apache port to 81 you can change:

~~~
# /etc/apache2/sites-enabled/000-default.conf
<VirtualHost *:81>

# /etc/apache2/ports.conf
Listen 81
~~~

To trigger some functions you can run script with
[patch](https://forums.zoneminder.com/viewtopic.php?t=24494)


## Zones

[wiki](https://wiki.zoneminder.com/Understanding_ZoneMinder's_Zoning_system_for_Dummies)
has example. Use small zone for far distances so percentage is same.

* set A is full zone pixels
* set B is Min/Max pixel Threshold: difference in color from previous
  * also Min/Max alarmed Area: check if set B is at least/most of set A in %
* set C is Filter Width/Height: check if 3x3 surround pixels (from set B) is
also different in colors
  * also Min/Max Filtered Area: check if C cover min/max of set A in %
* set D is Min/Max blobs: number of blobs
  * also Min/Max Blob area: check if cover at least/most of set A in %

To eliminate sun shadow, you can lower the *Max Alarmed Area* (50%) and use
*Overload Frame Ignore Count* to ignore next 3 frames in that case of overload.

To eliminate camera glitches, you can set in tab Source -> Monitor -> Buffers ->
Alarm Frame Count to 2.

Also you are using old camera in snapshot mode, zoneminder is pulling for new
images, you should limit Analysis FPS, Maximum FPS, and Alarm Maximum FPS in
Source -> Monitor -> General tab

# China IP cam

When I look at the source I see

~~~
var g_SoftWareVersion="V4.02.R12.00006531.10010.143300.00000";
var g_HardWareVersion="Unknown";
var g_mBuildTime="2016/9/13 16:50:12";
var g_SerialNo="fcccccdda05ad0a9";
var g_VideoInChannel=1;
var g_AlarmInChannel=2;
var g_AlarmOutChannel=1;
var g_AudioInChannel=1;
var g_DigChannel=0;


var g_channelNumber=1;
var g_user="admin";
var g_port="554";
 var g_address =document.location.hostname;

if (g_address == "")
{
//	g_address = "10.2.4.46";
}

var iLanguage=101;
var g_passWord="";
~~~

So to set zoneminder you can use "ONVIF" link at the top or manually select:

* Source Type -> Ffmpeg
* Source Path -> rtsp://admin:@192.168.1.11:554/user=admin_password=tlJwpbo6_channel=1_stream=0.sdp?real_stream
* Remote Method -> TCP
* Capture Width: 1280, Capture Height: 960

IPC-Model vendor is H264DVR, media port 34567, IP address: 192.168.1.11.
You can search, just type 192.168.1. (blank). Or you do not need to type
address, it will find cameras on other subnet mask.
You can change ip with "EditDevice"

![search]({{ site.base_url }}/assets/posts/china ip camera ip settings edit.png)
![search]({{ site.base_url }}/assets/posts/china ip camera ip settings edit address.png)


# Dahua IP Cam

IPC-HDW4300C default IP address 192.168.1.108 media port 3777 MAC
3c:ef:8c:a3:c9:6b, vendor Dahua, username: admin, password: admin. Also have
RTSP port 554.

Zoneminder ONVIF rtsp://admin:admin@192.168.3.5:554/cam/realmonitor?channel=1&subtype=0&unicast=true&proto=Onvif

# Lead acid

Charging is limited to 2.4V per cell (6x2.4 = 14.4v for 12v battery). Current is
10% of capacity, 0.1C (0.7A for 7Ah). Stop charging when current is below 3% of
capacity (0.21A for 7Ah), or after 16hours time (time to charge depends on current, if it is 10% than it need 10 hours).
Slow charging is 2.25V per cell (6x2.25 = 13.5V for 12v battery).
Best option is to start with 14.4V and decrease to 13.5V when battery is full.
Battery needs charge when it is below 12.5V in open circuit. Battery is fully
discarged when is it below 9.5V (do not go this far). Battery is around 2.1V per
cell when is fully charged (12.7V for 12V battery). Wait for 12hours after
charging to measure open circuit.
