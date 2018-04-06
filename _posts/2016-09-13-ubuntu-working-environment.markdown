---
layout: post
title: Ubuntu working environment
---

# Ubuntu stuff

* taks screenshot: fullsize `Print`, currently active window `Alt+Print`, area
  `Shift+Print`. Use `ctrl` instead of alt if you want to copy to clipboard
* [port
forwarding](https://help.ubuntu.com/community/SSH/OpenSSH/PortForwarding) socks
tunel `ssh -C -D 1080 server_url_or_ip`, than in firefox
<about:preferences#advanced> Networktab -> Settings choose "Manual proxy
configuration" and type SOCKS Host: localhost, and port 1080. Do not write
anyting in HTTP proxy... 
* gui ssh forwarding `ssh -X server` remote `ssh -R 5900:localhost:5900
guest@joes-pc` local
* [v4l2loopback](https://github.com/umlaeute/v4l2loopback/wiki/Mplayer), after
`sudo make install` and `sudo modprobe v4l2loopback` we can stream some video
file to device `while true; do gst-launch-0.10 filesrc
location=~/Desktop/buck.ogv ! decodebin ! v4l2sink device=/dev/video1;done`
* dim screen adjust brightness `xrandr -q | grep " connected"` to find the
name, for example *DFP1*, than `xrandr --output DFP1 --brightness 0.5`. Here is
what I use `d 0.8` in
[my_bashrc.sh](https://github.com/duleorlovic/config/blob/master/my_bashrc.sh#L32)

~~~
# ~/.bashrc
function d {
  if [  -z $1 ]
  then
    echo Default is middle value: d 0.5
  fi
  xrandr --output DFP1 --brightness ${1:-0.5}
}
alias dim=d
~~~

there is [f.lux](https://justgetflux.com/) software that can automate screen
brighthness.

* webcam does not work on chrome but works in firefox

~~~
sudo apt-get install libv4l-0
sudo mv /opt/google/talkplugin/GoogleTalkPlugin /opt/google/talkplugin/GoogleTalkPlugin.real 
cat > /opt/google/talkplugin/GoogleTalkPlugin <<HERE_DOC
#!/bin/bash
LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libv4l/v4l1compat.so /opt/google/talkplugin/GoogleTalkPlugin.real
HERE_DOC
~~~

* Simple scan scanner for HP LasetJet MF1212nf MFP does not work if you add
  printer from gui. Better is to use:

  ~~~
  sudo apt install hplip # install hp-setup
  hp-setup -i
  # chose net
  # chose to download driver from hp site
  # no need to install fax printer
  ~~~

* recently I got dns error (I'm using 8.8.8.8). Following
  [post](http://askubuntu.com/questions/368435/how-do-i-fix-dns-resolving-which-doesnt-work-after-upgrading-to-ubuntu-13-10-s)
  I comment out `# dns=dnsmasq` from /etc/NetworkManager/NetworkManager.conf and
  `sudo restart network-manager`

* if `Bus 002 Device 003: ID 0d8c:013c C-Media Electronics, Inc. CM108 Audio
Controller` microphone does not work automatically (but you can see in sound
settings as *CM108 Audio Controller* than you need to comment out last line
`#options snd-usb-audio index=0 ` in `/etc/modprobe.d/alsa-base.conf`.

# Chrome chromium

* take screenshot with `ctrl+Shift + p` and search screenshot
* [chrome scrambled](https://code.google.com/p/chromium/issues/detail?id=375957)
  can be solved with `sudo amdconfig --initial`
* chrome flickers on resize
  [link](http://askubuntu.com/questions/279088/google-chrome-flickering)
  solution is to disable (uncheck) **Use hardware acceleration when available**
  in System Settings -> Show Advanced -> (at bottom, search for this option) (or
  `--disable-gpu` in `/usr/share/applications/chromium-browser.desktop`). I try
  to disable only composition `--blacklist-accelerated-compositing` but still is
  enabled. Check that features is software only on <chrome://gpu/>.
* you can open extension by going to <chrome://apps>
* enable some flags on <chrome://flags>
  * for example sound icon
* dns or other network <chrome://net-internals>
* T-rex game <chrome://network-error/-106>

## Chrome plugins and extensions

* [user agent switcher](https://github.com/chrispederick/user-agent-switcher/)
* to check which browsers was used by useragent string
[parse user agent strings](https://developers.whatismybrowser.com/useragents/parse)
* [railspanel](https://chrome.google.com/webstore/detail/railspanel/gjpfobpafnhjhbajcjgccbbdofdckggg)

## Chrome DNS and HSTS problem

Since google owns `.dev` domain and it exists in hsts list, in chrome (no
firefox) request to `my-domain.dev` will be always https. So for local
development I use `.local` with dnsmasq. I did not choose to use `localhost`
because I can not remap subdomain of localhost in /etc/hosts and use in google
chrome (firefox works fine)
<https://stackoverflow.com/questions/39666979/chrome-ignoring-hosts-file-for-subdomains-of-localhost>
For all subdomains I use dnsmasq so I do not need to edit /etc/hosts for each
subdomain;

~~~
# /etc/dnsmasq.d/dev-tld
local=/local/
address=/local/127.0.0.1
~~~

restart
~~~
sudo systemctl restart NetworkManager
sudo /etc/init.d/dnsmasq restart
~~~

Also follow [instructions](https://gist.github.com/marek-saji/6808114)

DNS server dnsmasq for .dev domains for osx <https://passingcuriosity.com/2013/dnsmasq-dev-osx/>

~~~
brew install dnsmasq
# read suggested commands and apply them
cp /usr/local/opt/dnsmasq/dnsmasq.conf.example /usr/local/etc/dnsmasq.conf
sudo brew services start dnsmasq

# in /usr/local/etc/dnsmasq.conf
address=/local/127.0.0.1

sudo brew services restart dnsmasq

sudo mkdir -p /etc/resolver
sudo tee /etc/resolver/dev >/dev/null <<EOF
nameserver 127.0.0.1
EOF
~~~

Test with

~~~
# Make sure you haven not broken your DNS.
ping -c 1 www.google.com
# Check that .local names work
ping -c 1 this.is.a.test.local
~~~

You can check your domain name resolutions with `nslookup`

~~~
nslookup asd.local
# Server:		127.0.0.1
# Address:	127.0.0.1#53
#
# Name:	asd.local
# Address: 127.0.0.1
~~~

On linux you can update `/etc/hosts` with you custom domain name and use it
instead of IP address.
On window it is `c:\Windows\System32\Drivers\etc`. Just edit with notepad and
save. No need to restart (maybe it will be cleared on restart).

## Chrome Developer tools

* open developer tools on last tab that was used is with `F12` or
  `ctrl+shift+i`. Switch to next and prev panel with `ctrl+]` and `ctrl+[`.
  `ctrl+shift+d` dock undock. Open drawer with `ESC`
* on drawer three dots, you can
  * to search file in google developer tools you can open console window (not
  console tab with console input, window can be opened on any tab with ESC key
  in dev tools). Than on three dots, open dropdown menu and find *Search*
  (keyboard shortcut is `ctrl+shift+f`). To search current panel use `ctrl+f`
* three dots -> Coverage can give how much of css or js is used on the page
* [network tab filter
  request](https://developers.google.com/web/tools/chrome-devtools/profile/network-performance/resource-loading#filter-requests)
  will filter by filename containg the string `posts`. But it also supports
  keywords with autosuggestions. Examples: `domain:*.com`, `larger-than:1K`, 
* to stop on some page that redirects immediatelly, you can go `Sources` tab and
  *Event Listener Breakpoints* and *Load -> beforeunload* or *Script -> Script
  first statement*.
  To stop on XHR/Fetch call you can set breakpoint on Sources -> XHR breakpoint
* to search all files `ctrl+o`
* to open **command menu** use `ctrl+shift+p`
  * `screenshot`
* to open **Console panel** you can use `ctrl+shift+j` (also on a page). You can
 use `monitor(function)`, `monitorEvents($0, "key");` `debug(function)` or
  `undebug(function)`. Usefull when you want to stop on some event listener
  `debug(getEventListeners($0).click[0].listener)` (you can do the same using
  Elements, Event Listeners tab, click, and jump to source where you want
  breakpoint). To show some value in log you can add Conditional breakport with
  `console.log(varname);` to use current selected element in console use `$0`
  (`$1` previous selected and so on), for xpath use `$x()`. Use `$_` for last
  returned value in console.
  [link](https://developers.google.com/web/tools/chrome-devtools/console/command-line-reference)
* to open **Elements panel** (if developer tools is not yet opened) is
  `ctrl+shift+c`. You can use `h` to hide element (`visibility: hidden`). Use
  `ctrl+f` to find element (`enter` to find next). You can use `.class` to
  locate by css class.

* `ctrl+Shift + m` to toggle mobile responsive view
* `ctrl+o` find filename
* `ctrl+shift+`
 * `^shift+c` to click and locate element
 * `^shift+f` to find string in all files
 * `^shift+o` find functions in the current js file
 * `^shift+p` chrome shortcuts

Enable experiments with <chrome://flags/#enable-devtools-experiments> and go to
Experiments menu in Settings (F1 or Ctrl+P+Settings)

To clear autofill suggestions you can use keyboard shortcut. First select
  suggestion with UP or Down arrows than press Shift + Delete
  [answer](https://support.google.com/chrome/answer/142893?p=settings_autofill&rd=1)

In console you can call

~~~
url="http://numbersapi.com/5/math"
await fetch(url)
await r.text()
~~~

Audit tab can give you broken best practices and explanations.

# Firefox

url suggestion does not use port number, so it is advised to disable it on
`about:config` for `browser.urlbar.autoFill` to false. That way only history
links will be provided, so you can navigate to them using tab.

## Firefox developer tools and navigations

like vim
* `ctrl+[` back
* `ctrl+]` forward
* `ctrl+f` find (`F3` and `shift+F3` find next and prev)
* `/` quick find, `'` find only in links
* `ctrl+j` focus search bar
* `ctrl+shift+i` toggle developer tools (`F12`), `ctrl+shift+k` console,
  `ctrl+shift+c` inspector, `shift+F7` style, `ctrl+shift+m` mobile responsive,
  `ctrl+shift+j` browser console
* `shift+f4` scratchpad and run with `ctrl+r`
* `shift+f2` developer toolbar
* `ctrl+i` page info (content-type, referring url, size...)
* `f7` carret browsing so you can navigate cursor with shift arrow and ctrl+c

# Opera

Download for ubuntu https://www.opera.com/
Download mobile browser Opera mobile emulator https://www.opera.com/developer/mobile-emulator

~~~
./opera-mobile-emulator
# error while loading shared libraries: libQtGui.so.4: cannot open shared object file: No such file or directory

# to find in which package this file belongs use
# apt-file search libQtGui.so

sudo apt install libqtwebkit4
./opera-mobile-emulator
# still same error
~~~

# Selenium

<https://github.com/SeleniumHQ/selenium/wiki/Ruby-Bindings>
Selenium for ruby use gem `selenium-webdriver`.
You also need executables for firefox `geckodriver` (just download from
<https://github.com/mozilla/geckodriver/releases> to `/usr/local/bin`)
and for chrome `chromedriver` (download from
<https://sites.google.com/a/chromium.org/chromedriver/downloads> to
`/usr/local/bin`) and also `gem 'chromedriver-helper'` is needed for client.
Make sure you have version of firefox and chrome that matches drivers.

You can control remote selenium server. Download
[selenium-server-standalone.jar](https://www.seleniumhq.org/download/) and run
`java -jar selenium-server-standalone.jar`. For error `Unsupported major.minor
version 52.0` you need to update java: 51 -> java7, 52 -> java8, 53 -> java9.

To test in `rails c` try

~~~
# chrome
driver = Selenium::WebDriver.for :remote, desired_capabilities: :chrome, url: "http://192.168.5.56:4444/wd/hub"
# same as
# driver = Selenium::WebDriver.for :chrome, url: "http://192.168.5.56:4444/wd/hub"
driver.navigate.to 'http://google.com' #=> nil

# firefox
# driver = Selenium::WebDriver.for :firefox, url: "http://192.168.5.56:4444/wd/hub"

# headless chrome
options = Selenium::WebDriver::Chrome::Options.new(
  args: %w[headless disable-gpu window-size=1024,768],
)
driver = Selenium::WebDriver.for :chrome, url: "http://192.168.5.56:4444/wd/hub", options: options
~~~

If there is a screen you can run both headless and not. If you run server from
ssh (there is no screen) than you can run headless or you can run server using X
virtual frame buffer

~~~
xvfb-run java -Dwebdriver.chrome.driver=/usr/local/bin/chromedriver -jar /usr/local/bin/selenium-server-standalone.jar
~~~

# Gimp

Cage transform https://www.youtube.com/watch?v=jL8TepHX0qE does not work in
latest gimp so use old ubuntu 14 in virtual box.

# FTP

To enable ftp write you need to uncomment `sudo vi /etc/vsftpd.conf`
`write_enable=YES` and restart `sudo service vsftpd restart`. You need to create
folder (owner should be trkftp user).

# Ngrok

You can create local tunnel with ngrok.

When you signup you can generate authtoken in your `~/.ngrok2/ngrok.yml`.

You need to upgrade to use custom subdomains, but free domain is fine if you can
use some of your dns.

~~~
ngrok http 3002
~~~

# TIPS

* to change default program open with file type, you can right click,
properties, set default.
* read only usb, can't create file since usb is readonly https://askubuntu.com/questions/781223/physical-block-size-is-2048-bytes-but-linux-says-it-is-512-when-formatting-us

~~~
df -Th
umount /media/duleusb
sudo fdisk -l
#
udisksctl unmount -b /dev/sdc1
sudo sgdisk --zap-all /dev/sdc
sudo sgdisk --new=1:0:0 --typecode=1:ef00 /dev/sdc
sudo mkfs.vfat -F32 /dev/sdc1
~~~

use `disks` application to erase and format usb flash drive.
If it is not mountable than try
[forum](https://ubuntuforums.org/showthread.php?t=2153643) but change number of
KB 1024, 2048 ...

~~~
sudo dd if=/dev/zero bs=4096 of=/dev/sdc
sudo apt-get install gparted
# Device -> Create partition table.
# format to fat32
~~~

* to backup external usb, you can sync files. With rsync files are not deleted,
just added, or overwritten if same filename. `cd
/media/orlovic/bf12a7e5-a5d4-4532-8612-a3984f90b56c/backup` and `rsync -r
/media/dusan/EKSTERNIUSB/ EXTERNIUSB`

* suspend recovery <https://wiki.ubuntu.com/DebuggingKernelSuspend>

~~~
# enable trace and log
sudo sh -c "sync && echo 1 > /sys/power/pm_trace && pm-suspend"

# enable log in command env
sudo PM_DEBUG=true pm-suspend

less /var/log/pm-suspend.log
~~~

* instead of double click .deb file to install you can use command line `sudo
dpkg -i /home/orlovic/Downloads/skypeforlinux-64.deb`


# PDF

Merge several pdf files

~~~
pdftk * cat output ../p.pdf
~~~

# Canon camera wifi

<http://www.testcams.com/airnef/> is used to download files from Canon camera.

# Network

`ifconfig` is a tool to change network configuration, but changes are not
persisted unless you save them in `/etc/network/interfaces`

~~~
ifconfig eth0
~~~

To configure default dateway gw you can use `route` command

~~~
sudo route add default gw 10.0.0.1 eth1
route -n
~~~

To configure DNS you can edit `/etc/resolv.conf`.
To purge all configuration you can `ip addr flush eth0`

To prevent Network manager to use connection as default route, you can edit
connections -> IPv4 Settings -> Routes -> check "Use this connection only for
resources on its network"

To remove docker bridge you can run `ip link del docker0`
