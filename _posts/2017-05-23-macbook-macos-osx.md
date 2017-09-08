---
layout: post
title: MacBook MacOS OSX
---

# Shortcuts

Option (ALT) key is ⌥
Command cmd key is ⌘
Shift key is ⇧ 
Return key is ↵ 
Control (CTRL) key is ^

From terminal toolbar:
* `⌘ N` new window
* `⌘ T` new tab

From System Preferences -> Keyboard -> Shortcuts
* -> Spotlight
  * `⌘ space` spotlight search
  * `⌥  ⌘ space` finder
* -> Mission control
  * `^ up` mission control
  * `^ down` application windows in all workspaces, use arrow to focus, use tab
    to select another application and view it's windows

You can disable any special key for any app. From System Preferences ->
Keyboard -> Shortcuts -> App Chortcuts -> +  than select the app and write exact
name and add shortcut uncluding ⌥  key:
* I disable Minimize since I do not need it, so add Minimize and random keyboard
  combination like `⌥ ⌘ M`.
* for iTerm I do not need Clear Buffer so now it is `⌘ ⌥ M`
* Mission control -> Move left space I remaped to `^ ⌘ h` (also right space
  `^ ⌘ l`)

From [Mac keyboard shortcuts from
support](https://support.apple.com/en-us/HT201236)
* `⌘ H` hide front app
* `⌘ ⌥ H` to show only front app
* `Fn-delete` is forward del since that key does not exists on macbook air

Click on link by holding ⌘ or ⌥ or ctrl will open new tab (with shift it will
focus that new tab), download and open context menu (ctrl click also works for
selected text).

`⌘ shift 3` and `⌘ shift 4` to create screenshots for entire and selected area.
Press space after that to select window. Screen shots will be on desktop.

To show home folder in Finder, go to Finder Preferences -> Sidebar and enable
home folder in sidebar.

I added to `.bash_profile`:

* `alias ls='ls -Gp'` so I can see difference between files and folders

To enable ssh server you need to go "System Preferences -> Sharing -> enable
Remote login"

Position windows using [spectacle](https://github.com/eczarny/spectacle)
[video](https://www.youtube.com/watch?v=k1lmd2T5Z2A).
Comparison of all
[os-x-windoiw-manager](https://css-tricks.com/os-x-window-manager-apps/).
I found interestin
[hammerspoon](http://www.hammerspoon.org/go/)
[blog](http://thume.ca/2016/07/16/advanced-hackery-with-the-hammerspoon-window-manager/)

# Karabiner Elements to change keys

Since I like Alt + ` to switch to next window inside same app. You can try to
choose English PC in System Preferences->Keyboard->Input Sources. This way keys
(§ and `)  were replaced,  but also single quote key, so better solution is to
use karabiner. For Sierra 10.12.5 there is new version
<https://github.com/tekezo/Karabiner-Elements>

My `.config/karabiner/karabiner.json` file looks:

~~~
{
  "profiles": [
    {
      "simple_modifications": {
        "caps_lock": "left_control",
        "non_us_backslash": "grave_accent_and_tilde"
      }
    }
  ]
}
~~~

# Update packages

Install homebrew
Update bash: `brew install bash`
[link](https://gist.github.com/xuhdev/8b1b16fb802f6870729038ce3789568f)

~~~
brew install coreutils findutils gnu-tar gnu-sed gawk gnutls gnu-indent gnu-getopt
brew info coreutils

brew install bash-completion
~~~

Install [meld for mac](https://yousseb.github.io/meld/)

# iTerm2

You can maps pageup in iterm.  Do `⌘ ,` to open Preferences -> Keys -> Key
mappings and add `⌘ [` and `⌘ ]` to map scroll page down and up. But that does
not work well for vim since it show previous page in terminal buffer (not in
vim) so for vim use `^b` and `^f` (`^d` and `^u` for half down up)

# Work spaces

With `fn F3` (or swipe up with three fingers) you can create new spaces (any
number of workspaces). You can switch between them with left/right swipe with
three fingers (you can see more animations on System Preferences -> Trackpad.

Full screen apps open their own space.

# AppleScript

You can run scripts inside bash with `osascript -e 'display dialog "Hi"'` or for
shell scripts (you need to `chmod +x filename.txt`

~~~
#!/usr/bin/osascript
display dialog "Hi"
~~~

# Rails

~~~
brew unlink imagemagick
brew install imagemagick@6 && brew link imagemagick@6 --force

gem install puma -v 2.11.3 -- --with-cppflags=-I/usr/local/opt/openssl/include --with-ldflags=-L/usr/local/opt/openssl/lib

brew install qt
~~~

# Mysql

`brew install mysql` will will use `/tmp/mysql.sock` so you can create link to
ubuntu folder of mysql socket file

~~~
sudo mkdir -p /var/run/mysqld
sudo ln -s /tmp/mysql.sock /var/run/mysqld/mysqld.sock
~~~

# DNS server dnsmasq for .dev domains

<https://passingcuriosity.com/2013/dnsmasq-dev-osx/>

~~~
brew install dnsmasq
# read suggested commands and apply them
cp /usr/local/opt/dnsmasq/dnsmasq.conf.example /usr/local/etc/dnsmasq.conf
sudo brew services start dnsmasq

# in /usr/local/etc/dnsmasq.conf
address=/dev/127.0.0.1

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
# Check that .dev names work
ping -c 1 this.is.a.test.dev
~~~

# Android USB Thethering

Download <http://www.joshuawise.com/horndis> driver (right click open with) to
enable usb tethering from android phone (wireless thethering will consume
batery). I needed to restart mac to find my phone in network preferences.
