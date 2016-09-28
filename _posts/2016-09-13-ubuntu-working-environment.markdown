---
layout: post
title: Ubuntu working environment
---

* [port
forwarding](https://help.ubuntu.com/community/SSH/OpenSSH/PortForwarding) socks
tunel `ssh -C -D 1080 server` gui `ssh -X server` remote `ssh -R
5900:localhost:5900 guest@joes-pc` local
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

* webcam does not work on chrome but works in firefox

~~~
sudo apt-get install libv4l-0
sudo mv /opt/google/talkplugin/GoogleTalkPlugin /opt/google/talkplugin/GoogleTalkPlugin.real 
cat > /opt/google/talkplugin/GoogleTalkPlugin <<HERE_DOC
#!/bin/bash
LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libv4l/v4l1compat.so /opt/google/talkplugin/GoogleTalkPlugin.real
HERE_DOC
~~~

* [chrome scrambled](https://code.google.com/p/chromium/issues/detail?id=375957) can be solved with `sudo amdconfig --initial`
* chrome flickers on resize [link](http://askubuntu.com/questions/279088/google-chrome-flickering) solution is to disable (uncheck) **Use hardware acceleration when available** in System Settings -> Show Advanced -> (at bottom, search for this option) (or `--disable-gpu` in `/usr/share/applications/chromium-browser.desktop`). I try to disable only composition `--blacklist-accelerated-compositing` but still is enabled. Check that features is software only on <chrome://gpu/>.

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

# Chrome Developer Tools

## Plugins

* [user agent switcher](https://github.com/chrispederick/user-agent-switcher/) 

## Developer tools

* [network tab filter
  request](https://developers.google.com/web/tools/chrome-devtools/profile/network-performance/resource-loading#filter-requests)
  will filter by filename containg the string `posts`. But it also supports
  keywords with autosuggestions. Examples: `domain:*.com`, `larger-than:1K`, 
* to clear autofill suggestions you can use keyboard shortcut. First select
  suggestion with UP or Down arrows than press Shift + Delete
  [answer](https://support.google.com/chrome/answer/142893?p=settings_autofill&rd=1)
* to search file in google developer tools you can open console window (with
  ESC) than on three dots, open dropdown menu and find *Search*
* to stop on some page that redirects immediatelly, you can go `Sources` tab and
  *Event Listener Breakpoints* and *Load -> beforeunload* or *Script -> Script
  first statement*

