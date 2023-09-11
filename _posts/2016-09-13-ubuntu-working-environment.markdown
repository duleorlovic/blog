---
layout: post
title: Ubuntu working environment
---

# Ubuntu stuff

* upgrading
  ```
  do-release-upgrade
  ```
* find ubuntu and linux version

  ~~~
  lsb_release -a
  cat /proc/version
  uname -a
  ~~~

* taks screenshot: fullsize `Print`, currently active window `Alt+Print`, area
  `Shift+Print` (select window or drag area). images are automatically saved to
  ~/Pictures
  Use `ctrl` instead of alt if you want to copy to clipboard
* [port
forwarding](https://help.ubuntu.com/community/SSH/OpenSSH/PortForwarding)
```
ssh -L 8080:localhost:80 my.server.com
```
* tunel to external server so you do not need to open ports on your router and
  other can see your local env
  ```
  ssh -i $PEM_FILE -R 0.0.0.0:80:localhost:9292 ubuntu@52.202.201.113

  sudo vi /etc/ssh/sshd_config
  echo "GatewayPorts yes" >> /etc/ssh/sshd_config
  sudo service ssh restart

  # again connect using tunneling
  # make sure ports are opened in aws console

  ssh -i $PEM_FILE -R 0.0.0.0:80:localhost:9292 ubuntu@52.202.201.113
  ```
* gui ssh forwarding `ssh -X server` remote `ssh -R 5900:localhost:5900
guest@joes-pc` local
socks tunel `ssh -C -D 1080 server_url_or_ip`, than in firefox
<about:preferences#advanced> Networktab -> Settings choose "Manual proxy
configuration" and type SOCKS Host: localhost, and port 1080. Do not write
anyting in HTTP proxy... For Chrome or another connections you can use system
wide Ubuntu Settings -> Network -> Network Proxy -> Method: Manual -> Socks Host
localhost 1080 (HTTP Proxy is empty), Ignore Hosts: localhost, *.loc
than I can access cameras http://192.168.1.3:81/zm
For `curl` you need to export variable
```
export https_proxy=socks5://localhost:1080 http_proxy=socks5://localhost:1080
curl ifconfig.me
```
For ruby, you can check public ip address with
```
require "net/http"
Net::HTTP.get(URI("https://api.ipify.org"))
```
but ruby does not support proxy using socks. Here are some alternatives: polipo
http proxy using socks upstream, torsocks wrapper
https://superuser.com/questions/280129/http-proxy-over-ssh-not-socks
```
# /etc/polipo/config
# proxyAddress = "::1"
proxyPort = 8118
socksParentProxy = "localhost:1080"
socksProxyType = socks5
```
restart with
```
sudo /etc/init.d/polipo restart
```
for curl use
```
export https_proxy=localhost:8118 http_proxy=localhost:8118
```
but the best option is to change in system settings (it will add new env
variable) and open new terminals, curl and ruby works fine
```
echo "puts Net::HTTP.get(URI('http://ifconfig.me'))" | ruby -rnet/http
```

* to find my public ip address you can use what is my ip address
and other tools from https://github.com/chubin/awesome-console-services
```
# inline public ip address
curl l2.io/ip

# new line
curl eth0.me
dig +short myip.opendns.com @resolver1.opendns.com

# geolocation of ip address
curl ip-api.com
curl ip-api.com/8.8.8.8

# vicevi
curl https://icanhazdadjoke.com

# show a zoomable world map
telnet mapscii.me

# UNIX/Linux commands cheat sheets using curl (chubin/cheat.sh)
curl cheat.sh

# test upload download speed
curl -s https://raw.githubusercontent.com/sivel/speedtest-cli/master/speedtest.py | python -
```
ssh igrice https://github.com/chubin/awesome-console-services#entertainment-and-games
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

## Cups

Simple scan scanner for HP LasetJet MF1212nf MFP does not work if you add
  printer from gui. Better is to use:

  ~~~
  sudo apt install hplip # install hp-setup
  hp-setup -i
  # chose net
  # chose to download driver from hp site
  # no need to install fax printer
  ~~~

  On ubuntu 16 it works for 3.16.3 version
  ```
  HP Linux Imaging and Printing System (ver. 3.16.3)
  Printer/Fax Setup Utility ver. 9.0
  Plugin Download and Install Utility ver. 2.1
  Plugin Installer ver. 3.0
  ```

  On ubuntu 18 I used binary installation hplip-3.19.6.run from
  https://developers.hp.com/hp-linux-imaging-and-printing/gethplip
  but somehow `hp-plugin` does not work, so I used old version

To install hp-plugin I disabled aamor https://bugs.launchpad.net/hplip/+bug/1813768/comments/8
```
sudo apt-get install apparmor-utils
sudo aa-disable /usr/share/hplip/plugin.py
hp-plugin
```

To remove package and reinstall printer you can follow
https://askubuntu.com/questions/1056077/how-to-install-latest-hplip-on-my-ubuntu-to-support-my-hp-printer-and-or-scanner
```
sudo apt-get purge hplip hplip-data hplip-doc hplip-gui hpijs-ppds \
libsane-hpaio printer-driver-hpcups printer-driver-hpijs
sudo rm -rf /usr/share/hplip/
sudo apt-get autoremove
```

To remove hplip
```
sh hplip-3.19.8.run --noexec
cd hplip-3.19.8
sudo ./uninstall.py
sudo rm -rf /usr/share/hplip/
```

On ubuntu 18 I used official
```
sudo apt-get install hplip-gui
```


Enable automatic adding network printers
```
sudo systemctl start cups-browsed
sudo systemctl disable cups-browsed # remove from /etc/cups/cupsd.conf
sudo systemctl stop cups-browsed
```

To use HP color laserjet CP1215 you can use http://foo2hp.rkkda.com/
```
wget -O foo2zjs.tar.gz http://foo2zjs.rkkda.com/foo2zjs.tar.gz
tar zxf foo2zjs.tar.gz
cd foo2zjs
make
./getweb 1215      # Get HP LaserJet CP1215 .ICM files
sudo make install
sudo make cups
```
Go to <http://localhost:631/admin/> (your username and password) and configure
Color mode to Color instead of Monochrome. Instructions as screenshots
http://foo2hp.rkkda.com/cups/
Note that pdf usually use monochrome colors (unless after Ctrl+P under Advanced
tab Color Mode is Color) so use Writter to insert image and print document.


* recently I got dns error (I'm using 8.8.8.8). Following
  [post](http://askubuntu.com/questions/368435/how-do-i-fix-dns-resolving-which-doesnt-work-after-upgrading-to-ubuntu-13-10-s)
  I comment out `# dns=dnsmasq` from /etc/NetworkManager/NetworkManager.conf and
  `sudo restart network-manager`

* test sound using
  ```
  speaker-test -t wav -c 6
  ```
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
* disable url minimisation, go to
  chrome://flags/#omnibox-ui-hide-steady-state-url-trivial-subdomains and set
  Omnibox UI Hide Steady-State URL Path, Query, and Ref On Interaction to
  Disabled

## Chrome plugins and extensions

* [user agent switcher](https://github.com/chrispederick/user-agent-switcher/)
* to check which browsers was used by useragent string
[parse user agent strings](https://developers.whatismybrowser.com/useragents/parse)
* [railspanel](https://chrome.google.com/webstore/detail/railspanel/gjpfobpafnhjhbajcjgccbbdofdckggg)

## Chrome DNS and HSTS problem for .localhost and .dev

Since google owns `.dev` domain and it exists in hsts list, in chrome (no
firefox) request to `my-domain.dev` will be always https.
You can also use `lvh.me` since it always resolves to 127.0.0.1.
But for local development I use `.loc` with dnsmasq or `.localhost` also with
dnsmasq.
Old answer: __I did not choose to use `localhost` because I can not remap subdomain of localhost in /etc/hosts and use in google
chrome (firefox works fine) for some pathetic reason
<https://stackoverflow.com/questions/39666979/chrome-ignoring-hosts-file-for-subdomains-of-localhost>__
Now localhost works in Chrome but not in Firefox (about:config
network.dns.disableIPv6 set to false did not work for me).
When I change `loc` to `localhost` in `/etc/dnsmasq.d/dev-tld` than it works.

For all subdomains I use dnsmasq so I do not need to edit /etc/hosts for each
subdomain:

~~~
# /etc/dnsmasq.d/dev-tld
local=/localhost/
address=/localhost/127.0.0.1
~~~

restart with

~~~
sudo systemctl restart NetworkManager
sudo /etc/init.d/dnsmasq restart
~~~

Also follow [instructions](https://gist.github.com/marek-saji/6808114)

DNS server dnsmasq for .dev domains for osx <https://passingcuriosity.com/2013/dnsmasq-dev-osx/>
https://gist.github.com/ogrrd/5831371
https://kharysharpe.medium.com/automatic-local-domains-setting-up-dnsmasq-for-macos-high-sierra-using-homebrew-caf767157e43

~~~
brew install dnsmasq
# read suggested commands and apply them
# cp /usr/local/opt/dnsmasq/dnsmasq.conf.example /usr/local/etc/dnsmasq.conf
sudo brew services start dnsmasq

# in /usr/local/etc/dnsmasq.conf
# or /opt/homebrew/etc/dnsmasq.conf 
address=/localhost/127.0.0.1

sudo brew services restart dnsmasq

sudo mkdir -p /etc/resolver
sudo tee /etc/resolver/localhost >/dev/null <<EOF
nameserver 127.0.0.1
EOF
~~~

Test with

~~~
scutil --dns
# resolver #8
#   domain   : localhost
#   nameserver[0] : 127.0.0.1

# Make sure you haven not broken your DNS.
ping -c 1 www.google.com
# Check that .localhost names work
ping -c 1 this.is.a.test.localhost
~~~

Clear chrome dns cache on chrome://net-internals/#dns
and clear cookies.

You can check your domain name resolutions with mxtoolbox.com or domain settings
with command `nslookup` (dns resolution, dns lookup)
On web you can check on https://dns.google domain resolution for google

~~~
nslookup asd.localhost
# Server:		127.0.0.1
# Address:	127.0.0.1#53
#
# Name:	asd.localhost
# Address: 127.0.0.1

# To display the MX records, use the -query=mx option:
# you can show any records with
nslookup -query=any move-index.org
~~~

You can also use dig to find out dns lookup
```
dig +short trk.in.rs
```

Also host command
```
host trk.in.rs
```

You can see redirection url with
```
curl -I mydomain.com
```

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


Search
https://thoughtbot.com/blog/make-the-most-of-your-browser-s-address-bar

`^ str`, matches history with str, `# str` matches page titles

when strange characters are shown as squares on ubuntu
https://askubuntu.com/questions/1224125/font-characters-displayed-as-squares-in-ubuntu-18-04

```
rm -rf ~/.cache/fontconfig
sudo fc-cache -r -v
```

and restart computer (restarting firefox is also ok is you kill all firefox
sessions).

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

## Firefox WebIDE

You can remotelly debug a page on your phone. Start
https://developer.mozilla.org/en-US/docs/Tools/Remote_Debugging Enable Developer
mode on Android: Settings -> About Phone -> Build Number -> Tap 10 times and you
will see parent Setting menu -> Developer Options -> enable USB Debugging.

On desktop Firefox open Settings -> Web Developer -> WebIDE (shift+F8).
On android Firefox open Settings -> Advanced -> enable Remote Debugging via USB.

Android Firefox needs to confirm Incomming connection: Allow USB debugging
connection ?.  Also the app needs to be opened (in focus).

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

# Gimp

Cage transform https://www.youtube.com/watch?v=jL8TepHX0qE does not work in
latest gimp so use old ubuntu 14 in virtual box.

# FTP

To enable ftp write you need to uncomment `sudo vi /etc/vsftpd.conf`
`write_enable=YES` and restart `sudo service vsftpd restart`. You need to create
folder (owner should be trkftp user).

# Ngrok

Also https://localtunnel.github.io/www/

You can create local tunnel with ngrok.

When you signup you can generate authtoken in your `~/.ngrok2/ngrok.yml`.

You need to upgrade to use custom subdomains, but free domain is fine if you can
use some of your dns.

~~~
ngrok http 3002
~~~

In rails you need to allow

```
# config/environments/development.rb
  config.hosts << /[a-z0-9-.]+\.ngrok\.io/
```

# PDF

Merge pdf files

~~~
pdftk * cat output ../p.pdf
~~~

Convert images files to pdf. Some restrictions should be removed
https://askubuntu.com/questions/1081895/trouble-with-batch-conversion-of-png-to-pdf-using-convert
```
sudo mv /etc/ImageMagick-6/policy.xml /etc/ImageMagick-6/policy.xmlout
```

```
convert johny* nin.pdf
```


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

* resize mp4 video with avconv

~~~
avconv -i input.mp4 -s 640x480 -strict -2 output.mp4
for i in *.MP4; do avconv -i "$i" -strict -2 "resized/$i"; done
~~~

* convert dvd vob to mp4
  ```
  for f in *.VOB ; do ffmpeg -i "$f" -vcodec copy -strict experimental -c:a aac "${f%.*.VOB}.mp4"; done
  ```

* join videos using ffmpeg

  ```
  ffmpeg -i concat:"input1.mp4|input2.mp4" -fflags +genpts output.mp4

  # or
  # ls > textfile
  # edit file to contain:
  # file 'input1.mp4'
  # file 'input2.mp2'
  # ...
  ffmpeg -f concat -i textfile -fflags +genpts merged.mp4
  ffmpeg -f concat -i mylist.txt -c copy -safe 0 output.mp4
  ```

* slideshow using ffmpeg https://trac.ffmpeg.org/wiki/Slideshow
  ```
  # hold single image duration
  ffmpeg -loop 1 -i a00.jpg  -c:v libx264 -t 10 -pix_fmt yuv420p a00.mp4
  ```
* rotate video file using ffmpeg
  ```
  ffmpeg -i in.mov -vf "transpose=1" out.mov # 90 clockwise
  ffmpeg -i in.mov -vf "transpose=2" out.mov # 90 counter-clockwise
  ```
* cat video
  ```
  ffmpeg -ss 00:01:00 -i input.mp4 -to 00:02:00 -c copy output.mp4
  ```
* default gnome screenshot folder `dconf-editor` find
  `org->gnome->gnome-screenshot-> auto-save-directory ->
  file:///home/user/Download/`

* install Oracle Java instead of OpenJDK http://www.webupd8.org/2012/09/install-oracle-java-8-in-ubuntu-via-ppa.html

  ~~~
  sudo add-apt-repository ppa:webupd8team/java
  sudo apt-get update
  sudo apt-get install oracle-java8-installer
  sudo update-alternatives --config java
  java -version
  javac -version
  ~~~

* bluetooth

  https://forums.bunsenlabs.org/viewtopic.php?id=4375
  ~~~
  systemctl status bluetooth.service
  sudo systemctl restart bluetooth.service
  lsusb
  lspci -knn

  bluetoothctl
  list
  show
  agen on
  power on
  scan on
  ~~~

  My is

  ~~~
  04:00.0 Network controller [0280]: Qualcomm Atheros AR9462 Wireless Network Adapter [168c:0034] (rev 01)
    Subsystem: Lite-On Communications Inc AR9462 Wireless Network Adapter [11ad:6621]
  ~~~

  https://bugs.launchpad.net/ubuntu/+source/linux/+bug/1394368

  ~~~
  sudo -i
  echo "options ath9k btcoex_enable=1"  >  /etc/modprobe.d/ath9k.conf
  ~~~

* trim or mute vocals in mp3 files with audacity
* boot old computers
  * unetbootin http://unetbootin.github.io/
  * burn cd with mini.iso
    https://help.ubuntu.com/community/Installation/MinimalCD (since full iso is
    850MB > 700 MB CD capacity)

* find motherboard proccessor model with `sudo dmidecode | less`. To find type
  of memory run `sudo dmidecode --type 17` and look for `Type Detail` (for
  my server [Dell power Edge
  T110](https://www.dell.com/ky/business/p/poweredge-t110-2/pd)  it is `Type
  Detail: Synchronous Unbuffered (Unregistered)` and `Part Number`
  (M391B5273DH0-CK0) so it does not support registered/buffered memory
  [link](https://www.dell.com/community/PowerEdge-Hardware-General/PowerEdge-T110-RAM-compatibility/td-p/5816309)

# Remote access to services

To enable mysql service from remote you need to bind it to 0.0.0.0 in
configuration. To test on which interface is used port listening you can see
port use port in used port find port

```
sudo netstat -tupln | grep 3306

# see all connections
lsof -wni tcp:3000
# show puma process
lsof -t -i:3000 -sTCP:LISTEN
# so you can kill with
kill -9 $(lsof -t -i:3000 -sTCP:LISTEN)
```

You can try with nmap but it does not show for example neo4j 7474 port
```
nmap 127.0.0.1
nmap 192.168.3.2

# quick scan all devices, only ping scan
nmap -sn 192.168.0.0/24
# fast scan
nmap -F 192.168.0.0/24

# port scan, very slow
nmap -Pn 192.168.3.2
```

To start service on boot use

```
service --status-all

# to automatically boot
sudo update-rc.d neo4j defaults
# to remove from auto start
update-rc.d -f neo4j remove

# another way to set up to run at boot [+]

sudo systemctl enable neo4j
```

# Video processing

For youtube, video should be 16:9, ie for 1080p: 1920x1080, and for 720p:
1280x720.

Here are some tools
* Open Shot https://www.openshot.org/static/files/user-guide/introduction.html#features but better is KDEenlive
* http://www.blender.org for creating animations
* handbrake for video processing. I used for cropping, first to preview for
  720px height and use ffmpeg to crop window
  (original_width x 720) starting from position 0 x (top_pixel_handbrake) Ffmpeg
  generates smaller video than handbrake's output m4v file.

  ```
  ffmpeg -i dahua.ts -filter:v 'crop=2592:720:0:824' -strict -2 k.mp4
  ```
* kdenlive. When sound starts flickering, restart computer helps. Videos can be
  joined splited merged.

# ImageMagic rmagick

https://imagemagick.org/index.php https://github.com/rmagick/rmagick
Load image
```
require 'rmagick'
image = Magick::Image.read( 'demo.png' )
          .first
image = Magick::ImageList.new( 'demo.png' )
# to load just header block to find size of image
image = Magick::Image.ping( 'demo.png' ).first
image.display
# to return to ruby and not block the
pid = fork { draw.display }
# you can close from ruby
fork { sleep 30; Process.kill('SIGKILL', pid) }

# write output to different file
image.write file_name.split('.').insert(-2, 'proccessed').join('.')
```
To get a size of a image
```
image.columns # width
image.rows # height
```
To create gif
```
anim = ImageList.new("start.gif", "middle.gif", "finish.gif")
anim.write("animated.gif")
```
To resize but keep aspect ratio (ie to scale under constrains)
```
image = image.change_geometry("400") {|cols, rows, img| img.resize!(cols, rows)}
```
To write blue transparent box
```
draw = Magick::Draw.new
draw.stroke 'blue'
draw.fill_opacity 0
draw.rectangle 5,10, 50,100
draw.draw image
```

# TIPS

* to change default program open with file type, you can right click,
properties, set default.
another way is to edit `vi ~/.local/share/applications/defaults.list`
and run `sudo update-mime`
```
```
* read only usb, can't create file since usb is readonly https://askubuntu.com/questions/781223/physical-block-size-is-2048-bytes-but-linux-says-it-is-512-when-formatting-us

or error: 'The driver descriptor says the physical block size is 2048 bytes, but
Linux says it is 512 bytes'

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

* decrease font size in terminal with Ctrl - , increase Ctrl + and reset Ctrl 0
  (find them in manu Preferences -> Shortcuts)
* https://hluk.github.io/CopyQ/ for showing clipboard content in window
* open ms office docx files on https://onedrive.live.com/ to see comments
* to install rpm package use alien
  ```
  sudo apt-get install alien
  sudo alien --to-deb bluejeans-*.rpm
  sudo dpkg -i bluejeans_*.deb
  ```
* get image size information from command line
  ```
  file image.png
  MyPNG.png: PNG image, 681 x 345, 8-bit/color RGB, non-interlaced
  # get sizes of all images in a folder
  ls | xargs file
  # or
  identify image.png
  ```
* workspace grid is in one line, to use 2x2 matrix you can use this extension
  https://github.com/mzur/gnome-shell-wsmatrix
  Installation is using the side (browser extension and native host connector `$
  sudo apt-get install chrome-gnome-shell` for me, host connector is not
  detected in firefox)
  By default, only primary monitor windows are switching workspace, to swich all
  windows you should
  https://askubuntu.com/questions/1059479/dual-monitor-workspaces-in-ubuntu-18-04
  install *Gnome Tweaks* app or
  ```
  gsettings set org.gnome.mutter workspaces-only-on-primary false
  ```


* ~~~
  /dev/dsa1: UNEXPECTED INCONSISTENCY; RUN fsck MANUALLY.
          (i.e., without -a or -p options)
  fsck exited with status code 4
  done.
  Failure: File system check of the root filesystem failed
  The root filesystem on /dev/sda1 requires a manual fsck
  ~~~

  solution is to run `(initramfs) fsck -yf /dev/sda1`
* low resolution of second vga monitor is low 1024x768 . If you  need better
  resolution of thet Unknown Display you can use
  ```
  xrandr --listactivemonitors
  # find monitor id like VGA-0
  xrandr --addmode VGA-0 1920x1080
  ```
  to turn of monitors every night https://askubuntu.com/a/116806/40031
  ```
  sleep 1 && xset -display :0.0 dpms force off
  ```
* to check disk usage you can run to see folder size and disk size
  ```
  sudo du /* -s

  # to sort
  du -sm * | sort -nr
  ```
  to free up disk space from ubuntu clear clean you can remove /var/log/journal
  ```
  journalctl --disk-usage
  sudo vi /etc/systemd/journald.conf
  sudo journalctl --vacuum-size=50M
  ```
  and snapd https://www.linuxuprising.com/2019/04/how-to-remove-old-snap-versions-to-free.html
```
sudo snap set system refresh.retain=2
sudo rm -rf /var/lib/snapd/cache/*

cat > ~/remove-old-snaps.sh << 'HERE_DOC'
#!/bin/bash
# Removes old revisions of snaps
# CLOSE ALL SNAPS BEFORE RUNNING THIS
set -eu

LANG=en_US.UTF-8 snap list --all | awk '/disabled/{print $1, $3}' |
    while read snapname revision; do
        snap remove "$snapname" --revision="$revision"
    done
HERE_DOC

chmod +x ~/remove-old-snaps.sh
sudo ~/remove-old-snaps.sh
```
* to enable sent to devices you need to go on your phone Chrome -> three dots ->
  Recent tabs -> Sign in.
  Then on Chrome desktop you can click on Share icon in url bar -> Send to
  devices
* google drive spreadsheets countcoloredcells
  https://support.google.com/docs/thread/118955201/script-countcoloredcells?hl=en

# Multiseat

https://www.apalrd.net/posts/2022/multiseat_intro/
https://wiki.gentoo.org/wiki/Multiseat
https://wiki.archlinux.org/title/xorg_multiseat#top-page
https://wiki.debian.org/Multi_Seat_Debian_HOWTO

```
sudo apt install openssh-server
sudo apt install radentop
sudo vi /etc/gdm3/custom.conf
# uncomment WaylandEnable=false to switch to Xorg
```

Attach
```
sudo loginctl list-seats
sudo loginctl seat-status seat0
sudo loginctl attach seat1 /dev...
sudo loginctl list-seats
```
* enable remote desktop by going to Settings -> Sharing -> enable Remote Desktop
  then Enable Legacy VNC Protocol and set the password.
  Default is to prompt but you can change to password using dconf
  /org/gnome/desktop/remote-desktop/vnc/auth-method and uncheck Use default
  value and pick "password"
  To enable locked users to connect you can use extension
  https://extensions.gnome.org/extension/4338/allow-locked-remote-desktop/
  TODO still black screen when I connect from Vnc Viewer from mac
