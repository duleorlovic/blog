---
layout: post
title: MacBook MacOS OSX
---

Macbook Aur M1 2020
specification https://support.apple.com/kb/SP825?viewlocale=en_US&locale=en_US

# Shortcuts

Option (ALT) key is ⌥
Command cmd key is ⌘
Shift key is ⇧ 
Return key is ↵ 
Control (CTRL) key is ^

Since on mac keyboard `fn` key is at left edge, I use it as `ctrl` . Remap with
System preferences -> Keyboard -> Modifier Keys -> Globe key -> ^Control

Xkill on macOS is Force Quit Applications which can be started with ⌘ ⌥ Esc


To start inserting emoji use: Control + Command + Space

https://support.apple.com/en-gb/HT201236

In any window, or shell, you can:
* `⌘ n` new window
* `⌘ t` new tab, `⌘ w` close tab

From System Preferences -> Keyboard -> Shortcuts
* Spotlight
  * `⌘ space` spotlight search
  * `⌥  ⌘ space` finder search, inside finder you can open files with double click or `⌘ o` (enter just renames the file)
* Mission control
  * `^ up` mission control
  * `^ down` application windows in all workspaces, use arrow to focus, use tab
    to select another application and view it's windows
* Screenshot
  * `⌘ shift 3` and `⌘ shift 4` to create screenshots for entire and selected
    area. Press space after `⌘ shift 4` to select window. Screen shots will be
    on desktop but I changed to Downloads using `cmd shift 5` (record video).
    iphone screenshot use power button and home buttom (or volume up if home
    button does not exists)
    When you connect with usb cable, the Dusan's iPhone will show up in finder
    so you can browser files on phone from mac, if iCloud Photos is not used on
    a phone. If it is used than access with https://www.icloud.com

https://support.apple.com/en-gb/guide/terminal/trmlshtcts/mac
Typing Command-Full Stop (.) Dot Period is equivalent to entering Control-C on
the command line.

With `fn F3` (or swipe up with three fingers) you can create new spaces (any
number of workspaces) desktops. You can switch between them with left/right
swipe with three fingers and you can drag and drop to different workspaces.
Full screen apps open their own space.
From [Mac keyboard shortcuts from
support](https://support.apple.com/en-us/HT201236)
* `⌘ H` hide front app
* `⌘ ⌥ H` to show only front app
* `Fn-delete` is forward del since that key does not exists on macbook air

Click on link by holding ⌘ or ⌥ or ctrl will open new tab (with shift it will
focus that new tab), download and open context menu (ctrl click also works for
selected text).

To use apple keyboard on ubuntu I tried to remap Fn key, but that is not
possible https://askubuntu.com/questions/370944/remap-fn-key-to-insert-key-on-apple-aluminium-keyboard

To connect two monitors on one port you need adapter that supports mst dual external displays https://www.reddit.com/r/UsbCHardware/comments/jmaa2t/anyway_to_use_two_monitors_on_a_laptop_with_one/
For M1 is it not enough to use two ports with two adapters. you need to use usb
A to hdmi adapter that is DisplayLink compatible:
https://www.youtube.com/watch?v=y_WHjdiqCMc
driver https://www.synaptics.com/products/displaylink-graphics/downloads/macos
list
https://www.synaptics.com/products/displaylink-graphics/displaylink-products-list?field_displaylink_category_value=usb_adapters
long description
https://www.macworld.co.uk/how-to/how-connect-two-or-more-external-displays-apple-silicon-m1-mac-3799794/

Another solution using magiclink cable
https://www.youtube.com/watch?v=GPWuABthnkE

To show home folder in Finder, go to Finder Preferences -> Sidebar and enable
home folder in sidebar.
Inside Finder Go menu you can hold ⌥  key to show `Library` folder. It is by
default hidden from user. To permanently enable showing it, go to home folder in
finder app, right click on empty space and choose 'Show View Options' from
dropdown. Than select 'Show Library Folder'.
Some app data can be
`~/Library/Containers/com.mydomain.myaoo/Data/Library/Application%20Support/myapp/`
When you open the app using ⌘ + space than it is usually from Applications
folder, but it could be from Downloads. You can check where app is location by
right clicking on app icon in Dock than Options -> Show in finder

I added to `.bash_profile`:

* `alias ls='ls -Gp'` so I can see difference between files and folders

To enable ssh server you need to go "System Preferences -> Sharing -> enable
Remote login"

To rename hostname go to System preferences -> Sharing

Position windows using [spectacle](https://github.com/eczarny/spectacle)
[video](https://www.youtube.com/watch?v=k1lmd2T5Z2A).
Comparison of all
[os-x-windoiw-manager](https://css-tricks.com/os-x-window-manager-apps/).
I found interesting
[hammerspoon](http://www.hammerspoon.org/go/)
[blog](http://thume.ca/2016/07/16/advanced-hackery-with-the-hammerspoon-window-manager/)

# Karabiner Elements to change keys

Former name for this app was KeyRemap4MacBook.
https://github.com/pqrs-org/Karabiner-Elements

Install instructions https://karabiner-elements.pqrs.org/docs/getting-started/installation/
after allowing karabiner_oberver you need to manually add also the
karabiner_grabber by selecting Machintos HD -> /Library/Application
Support/org.pqrs/Karabiner-Elements/bin/karabiner_grabber
https://github.com/pqrs-org/Karabiner-Elements/issues/1867#issuecomment-498484832

I use following scripts for complex modifications
https://github.com/pqrs-org/KE-complex_modifications
You can find this site in Karabiner -> Complex rules -> Add rule -> Import more
rules from the internet
* semicolon and colon https://superuser.com/a/1705236/877698
* curly braces and square brackets https://apple.stackexchange.com/a/437073/449651
https://ke-complex-modifications.pqrs.org/#exchange_square_brackets_and_curly_brackets
* Remap section sign (§) from British Keyboard to US's backtick + plus minus (±)
  to tilde (~) grave_accent_and_tilde
https://ke-complex-modifications.pqrs.org/#section_sign_to_backtick
* Exchange single and double quote
https://ke-complex-modifications.pqrs.org/#exchange_single_and_double_quote
* Exchange numbers and symbols (1234567890 and !@#$%^&*())
  https://ke-complex-modifications.pqrs.org/#exchange_numbers_and_symbols
* Left ctrl + hjkl to arrow keys Vim
https://ke-complex-modifications.pqrs.org/#ctrl_plus_hjkl_to_arrow_keys
but I change to caps_lock

* Caps Lock Vim Movements (rev 2)
https://ke-complex-modifications.pqrs.org/#capslock_vim_movements
* custom rule similar to single and double quote, I created for pipe and
  backslash. I just duplicated existing rule

Changes are visible immediatelly, but I do not use links, I copy from config
~~~
cp ~/.config/karabiner/karabiner.json ~/config/.config/karabiner/karabiner.json
cp ~/config/.config/karabiner/karabiner.json ~/.config/karabiner/karabiner.json
~~~

I added my config to enable bash alt + b/f back forward one word. I need to
enable in Termial -> Preferences -> Profiles -> Keyboard -> Use Option as Meta
key
Also remaped cmd+. to cmd+\ since default keyboard shortcut is Break but not in
menu bar, so now I remap to cmd+\ and use another automator to
activateWindowDotByBackslash and bind to cmd+\
https://stackoverflow.com/questions/71347942/change-command-dot-break-keyboard-shortcut-in-terminal-mac-which-is-not-in/71347943#71347943
```
                "rules": [
                    {
                        "description": "Change left_command+b/f to alt+b/f and left_control [/] to page up/down",
                        "manipulators": [
                            {
                                "conditions": [
                                    {
                                        "bundle_identifiers": [
                                            "^com\\.apple\\.Terminal$"
                                        ],
                                        "type": "frontmost_application_if"
                                    }
                                ],
                                "from": {
                                    "key_code": "period",
                                    "modifiers": {
                                        "mandatory": [
                                            "left_command"
                                        ]
                                    }
                                },
                                "to": [
                                    {
                                        "key_code": "backslash",
                                        "modifiers": [
                                            "left_command"
                                        ]
                                    }
                                ],
                                "type": "basic"
                            },
                            {
                                "conditions": [
                                    {
                                        "bundle_identifiers": [
                                            "^com\\.apple\\.Terminal$"
                                        ],
                                        "type": "frontmost_application_if"
                                    }
                                ],
                                "from": {
                                    "key_code": "b",
                                    "modifiers": {
                                        "mandatory": [
                                            "left_command"
                                        ],
                                        "optional": [
                                            "any"
                                        ]
                                    }
                                },
                                "to": [
                                    {
                                        "key_code": "b",
                                        "modifiers": [
                                            "option"
                                        ]
                                    }
                                ],
                                "type": "basic"
                            },
                            {
                                "conditions": [
                                    {
                                        "bundle_identifiers": [
                                            "^com\\.apple\\.Terminal$"
                                        ],
                                        "type": "frontmost_application_if"
                                    }
                                ],
                                "from": {
                                    "key_code": "f",
                                    "modifiers": {
                                        "mandatory": [
                                            "left_command"
                                        ],
                                        "optional": [
                                            "any"
                                        ]
                                    }
                                },
                                "to": [
                                    {
                                        "key_code": "f",
                                        "modifiers": [
                                            "option"
                                        ]
                                    }
                                ],
                                "type": "basic"
                            },
                            {
                                "from": {
                                    "key_code": "open_bracket",
                                    "modifiers": {
                                        "mandatory": [
                                            "fn"
                                        ],
                                        "optional": [
                                            "any"
                                        ]
                                    }
                                },
                                "to": [
                                    {
                                        "key_code": "page_down"
                                    }
                                ],
                                "type": "basic"
                            },
                            {
                                "from": {
                                    "key_code": "close_bracket",
                                    "modifiers": {
                                        "mandatory": [
                                            "fn"
                                        ],
                                        "optional": [
                                            "any"
                                        ]
                                    }
                                },
                                "to": [
                                    {
                                        "key_code": "page_up"
                                    }
                                ],
                                "type": "basic"
                            }
                        ]
                    },
```

# Update packages

Install homebrew
Update bash: `brew install bash`
[link](https://gist.github.com/xuhdev/8b1b16fb802f6870729038ce3789568f)

~~~
brew install coreutils findutils gnu-tar gnu-sed gawk gnutls gnu-indent gnu-getopt
brew info coreutils

brew install bash-completion
~~~

Edit Terminal background I tried to be as it is on ubuntu
Chose Pro profile on Terminal -> Preferences -> Profiles
https://medium.com/@json_singh/ubuntu-like-terminal-in-mac-bash-9afe37b09aa
```
brew install bash

sudo vi /etc/shells
# add
/opt/homebrew/bin/bash

# and now changing the shell will succeed
chsh -s /opt/homebrew/bin/bash
```
Also change font to Monospace Regular style (you can see it on ubuntu terminall
Preferences -> Default -> Text) and size 14.
You can also use Ubuntu Mono Regular https://design.ubuntu.com/font/.
Copy ttf file to Libraries (in finder use menu item Go -> Library).

Also in terminal profiles Pro select: When the shell exists: "Close if the shell
exited cleanly".

Install [meld for mac](https://yousseb.github.io/meld/)

On high sierra you need to run <https://github.com/yousseb/meld/issues/50>

~~~
unlink /Applications/Meld.app/Contents/Frameworks/libz.1.dylib
~~~

Install java JRE from jre-9.0.4_osx-x64_bin.dmg (not tar.gz)
http://www.oracle.com/technetwork/java/javase/downloads/index.html
Double click will install.
You can check if installed using https://java.com/en/download/installed.jsp in
browser.

Install vim

~~~
brew install vim --override-system-vi
# if you receive error
# xcrun: error: invalid active developer path (/Library/Developer/CommandLineTools), missing xcrun at: /Library/Developer/CommandLineTools/usr/bin/xcrun
xcode-select --install
~~~

Instal pip

~~~
sudo easy_install-3.6 pip
pip3 install awscli  --user --upgrade
~~~

# iTerm2

You can maps pageup in iterm.  Do `⌘ ,` to open Preferences -> Keys -> Key
mappings and add `⌘ [` and `⌘ ]` to map scroll page down and up. But that does
not work well for vim since it show previous page in terminal buffer (not in
vim) so for vim use `^b` and `^f` (`^d` and `^u` for half down up)

# Tips

* new disks like sd card will be mounted under `/Volumes` instead of `/media` or
  `/mnt`


# AppleScript

You can run and edit scripts in `Script Editor`. Open `Dictionary` in File->Open
Dictionary or drag application icon to Script Editor application icon.

https://www.youtube.com/watch?v=EAZlFptgEPQ

~~~
# comments
-- this is comment
# this is also comment
{* multi 
line
comment *}

# continuation character Option l
dialog "This is long line" ¬
buttons { "Great", "OK" }

# Boolean
true, false
# Text
"Hi"
# List
{ 1, "Duke" }
# Record
{name: "Duke", age: 35}

# Operators
"a" & "b"

# variables
set myName to "John"
copy 33 to myAge
~~~

Send keys
```
" you can use app Key codes to find a code
key code 36
keystroke "ASD"
# you can use keystroke to send shortcuts
https://eastmanreference.com/complete-list-of-applescript-key-codes
delay 1
```

Statements can include all above. Simple statement is single line, multiline is
compound statements
~~~
tell application "Finder"
  -- by position index, first/last or front/back, name
  get the name of window 2
  -- same as get the name of second window
  -- same as get the name of 2nd window
  -- same as get the name of the window after the front window
  -- same as get the name of window "Trash"

  -- name can not be updated, but other (like index) can be updated
  set the index of the last window to 1
  set toolbar visible of the front Finder window to true
  set the sidebar width of  the second Finder window to 240
  set the position of the front Finder window to {94, 134}
  set the bounds of the front Finder window to {24, 96, 524, 396}
  set the target of the front Finder window to home

  open window "Trash"
  -- same as select window "Trash"
  tell the front window
    set the current view to flow view
    set the bounds to {528, 116, 1016, 674}
  end tell

  close window "Trash"
  set savedName to name of front window
  set windowRef to a reference to window 1
  set windowId to theWindow's id
  -- same as set windowId to id of the first window

end tell
~~~

If conditional with type casting

~~~
if (myVar as string) is equal to "asd" then
end if
~~~

If you want to check the type of variable

~~~
if class of myVar is string then
end if
~~~

If you got exception than wrap with try to do exceptions handling

~~~
try
    set a to a as number
    display dialog "Yes! It's a number!"
end try

-- or

try
  set windowId to do shell script "defaults read com.duleorlovic.windowShortCuts " & key
on error
  display dialog "Please create window shortcut for key "
end try
~~~

Get user input

~~~
set theResponse to display dialog "Key (h, j, k, l, semicolon, m, comma, dot, slash, u, i) ?" default answer "" with icon note buttons {"Cancel", "Continue"} default button "Continue"
if button returned of theResponse is equal to "Cancel" then
 error number -128
end if
set key to text returned of theResponse
~~~

Sample programs
https://en.wikibooks.org/wiki/AppleScript_Programming/Sample_Programs

Command is series of words that request an action. Command is directed to a
target.
Choose file

~~~
    set xfile to choose file
    set xpath to POSIX path of xfile
~~~

Debugging with `display dialog "message"`. Use double quotes, not single quote.

Array list iterate loop

~~~
set theList {1, 2, 3}
set first as item 1 of theList
set first to item 1 of theList
~~~

~~~
on getPositionOfItemInList(theItem, theList)
    repeat with a from 1 to count of theList
        if item a of theList is theItem then return a
    end repeat
    return 0
end getPositionOfItemInList

set theList to {"Sal", "Ben", "David", "Chris", "Jen", "Lizzie", "Maddie", "Lillie"}
getPositionOfItemInList("Maddie", theList)
~~~

String manipulation https://developer.apple.com/library/content/documentation/LanguagesUtilities/Conceptual/MacAutomationScriptingGuide/ManipulateText.html

Split string
```
set AppleScript's text item delimiters to "-"
set theTextItems to every text item of windowNameId
set AppleScript's text item delimiters to ""
set windowId to item 2 of the theTextItems
```

You can run scripts inside bash with `osascript -e 'display dialog "Hi"'` or for
shell scripts (you need to `chmod +x filename.txt`

~~~
#!/usr/bin/osascript
display dialog "Hi"
~~~

You can also call scpt file

~~~
osascript ~/myscript.scpt
~~~

TODO: https://developer.apple.com/library/content/documentation/AppleScript/Conceptual/AppleScriptLangGuide/conceptual/ASLR_fundamentals.html#//apple_ref/doc/uid/TP40000983-CH218-SW1

# Activate window

iTerm supports hotkeys https://www.iterm2.com/documentation-hotkey.html
but it is not generic. I rather write window id and hotkey combination in user
store using a `b` or `b h` commands

~~~
do shell script "defaults write com.myname.myapp foo bar"
set myValue to do shell script "defaults read com.myname.myapp foo"
~~~

To assign keyboard shortcuts you need to create a service in Automator and
create shortcut in System Preferences -> Keyboard -> Shortcuts -> Services
That is available in all apps, you can find them on main menu using app name on
top left position -> Services. If there is already some shortcut than you need
to remap it to another key, I use same with a shift.

Create service for each key, for example activateWindowL
(`/home/orlovic/Library/Services/activateWindowL.workflow`).

~~~
on run {input, parameters}
    -- note that mac path is using colons instead of slash
    run script file "Macintosh HD:Users:dule:config:bashrc:mac_scripts:mac_activate_window.scpt" with parameters {"l"}
    return input
end run
~~~

Or you can copy all
```
cp -r ~/config/bashrc/mac_scripts/Library/Services/* ~/Library/Services/
# copy back the changes so we can save them in repo
cp -r ~/Library/Services/* ~/config/bashrc/mac_scripts/Library/Services/
```
For Input Sources I changed keyboard shortcut key `ctrl+space` *Select the
previous input source* to `shift+ctrl+space` (so I do not accidentally hold ctrl
and space) and change kayboard language.
You can disable any special key for any app so we can use it for our navigation.
From System Preferences -> Keyboard -> Shortcuts -> App Chortcuts -> +  than
select the app and write exact name and add shortcut uncluding ⌥  key:
* for iTerm I do not need Clear Buffer so now it is `⌘ ⌥ M`
* Mission control -> Move left space I remaped to `^ ⌘ h` (also right space
  `^ ⌘ l`)
* to disable hide front app, it has to be done for each app, for example, `Hide
  Terminal` should be remaped to cmd + shift + h . You can find this menu item
  in first menu dropdown.
here is the list of all mappings:
* All Applications: Minimise `⇧ ⌘ J`
* Finder: Connect to Server `⇧ ⌘ K`
* Terminal: Clear to Previous Mark `⇧ ⌘ L`, Jump to Selection `⇧ ⌘ J`, Clear to
  Start `⇧ ⌘ K`, Hide Terminal `⇧ ⌘ H`
* Google Chrome: Hide Google Chrome `⇧ ⌘ H`, Jump to Selection `⇧ ⌘ J`, Open
  Location... `⇧ ⌘ L`
* Preview: Rotate Left `⇧ ⌘ L`
* Firefox: Downloads `⇧ ⌘ J`, Hide Firefox  `⇧ ⌘ H`


![Mac keyboard shortcuts]({{ site.baseurl }}/assets/posts/mac keyboard shortcuts.png)

My custom keyboard shortcuts
* Mission Controller -> Move left a space fn + cmd + h and Move right a space fn
  + cmd + l

![Overwrite app shortcuts]({{ site.baseurl }}/assets/posts/overwrite app shortcuts.png)

To get foremost window

https://apple.stackexchange.com/questions/117421/how-do-i-focus-a-specific-window-with-applescript-without-doing-an-activate-and
https://stackoverflow.com/questions/10366003/applescript-google-chrome-activate-a-certain-window/34375804#34375804
http://tom.scogland.com/blog/2013/06/08/mac-raise-window-by-title/
https://macosxautomation.com/applescript/firsttutorial/index.html chap 3 name
property
~~~
# open last window
do shell script "open -a Google\\ Chrome"
~~~

~~~
tell application "iTerm"
       activate
       set theWindow to the first item of ¬
               (get the windows whose name is "2. bash")
       if index of theWindow is not 1 then
               set index to 1
               set visible to false
               set visible to true
       end if
end tell
~~~

~~~
tell application "System Events" to tell process "iTerm"
       perform action "AXRaise" of (first window whose name contains "2.")
end tell
~~~

# Rails

~~~
brew unlink imagemagick
brew install imagemagick@6 && brew link imagemagick@6 --force
brew install qt
~~~

# Mysql

`brew install mysql` will will use `/tmp/mysql.sock` so you can create link to
ubuntu folder of mysql socket file

~~~
sudo mkdir -p /var/run/mysqld
sudo ln -s /tmp/mysql.sock /var/run/mysqld/mysqld.sock
~~~

To start mysql server you can run 
```
mysql.server start
```
To find location of mysql config files

```
brew --prefix mysql
ls  -la $(brew --prefix mysql)
find /opt/homebrew/Cellar/mysql/8.0.28_1 -name "*.cnf"
vi /opt/homebrew/Cellar/mysql/8.0.28_1/.bottle/etc/my.cnf
```
To permanently change socket file location you can update

~~~
# find all places where mysql is searching for my.cnf
mysql --help|grep cnf

sudo vi ~/.my.cnf
[client]
socket=/var/run/mysqld/mysqld.sock
~~~

To see error log
```
tail -f /usr/local/var/mysql/mac.local.err
```

Usually enable permissions for `sudo chown _mysql /usr/local/var/mysql/*` but it
is easier to enable for all
```
sudo chmod -R 777 /usr/local/var/mysql/
sudo chmod 777  /var/run/mysqld/
```

# Errors

On Mac I got error for libsassc
~~~
Caused by:
LoadError: Could not open library '/Users/dule/.rvm/gems/ruby-2.6.5/gems/sassc-2.2.1/lib/sassc/libsass.bundle': dlopen(/Users/dule/.rvm/gems/ruby-2.6.5/gems/sassc-2.2.1/lib/sassc/libsass.bundle, 5): no suitable image found.  Did find:
	/Users/dule/.rvm/gems/ruby-2.6.5/gems/sassc-2.2.1/lib/sassc/libsass.bundle: load commands not in a segment
	/Users/dule/.rvm/gems/ruby-2.6.5/gems/sassc-2.2.1/lib/sassc/libsass.bundle: stat() failed with errno=25
~~~
which I solved with
https://github.com/sass/sassc-ruby/issues/146#issuecomment-534668663
~~~
gem uninstall sassc
gem install sassc -- --disable-march-tune-native
~~~


For error
```
Could not find MIME type database in the following locations:
```
I use
https://stackoverflow.com/questions/69248078/mimemagic-install-error-could-not-find-mime-type-database-in-the-following-loc
```
brew install shared-mime-info
```

For error libv8 therubyracer
```
../src/utils.h:33:10: fatal error: 'climits' file not found

```
I used https://github.com/rubyjs/libv8/issues/312#issuecomment-807104369
to install old x86 version of v8
(they also suggest to upgrade to ruby 2.7.1
https://github.com/rubyjs/libv8/issues/312#issuecomment-1023055391 )
We can also replace therubyracer with mini_racer
https://github.com/rubyjs/libv8/issues/282#issuecomment-778845927 and
https://github.com/rubyjs/libv8/issues/309
```
brew install v8
gem install libv8 -v '3.16.14.19' -- --with-system-v8

gem install libv8-node -v '16.10.0.0' -- --with-system-v8

pyenv global 3.9.1

gem install mini_racer -- --with-v8-dir=`brew --prefix v8`
gem install therubyracer -v '0.12.3' -- --with-v8-dir=`brew --prefix v8`
```

for error
```
checking for -lmysqlclient... no
-----
mysql client is missing. You may need to 'brew install mysql' or 'port install
mysql', and try again.
```
I solved with
```
brew install mysql
We've installed your MySQL database without a root password. To secure it run:
    mysql_secure_installation

MySQL is configured to only allow connections from localhost by default

To connect run:
    mysql -uroot

To restart mysql after an upgrade:
  brew services restart mysql
Or, if you don't want/need a background service you can just run:
  /opt/homebrew/opt/mysql/bin/mysqld_safe --datadir=/opt/homebrew/var/mysql
```

For error
```
ld: library not found for zstd
```
I solved https://stackoverflow.com/a/69722047/287166
```
bundle config --local build.mysql2 "--with-opt-dir="$(brew --prefix zstd)""
```

For error
```
ERROR:  Error installing jekyll:
	ERROR: Failed to build gem native extension.

    current directory: /Users/dule/.rvm/gems/ruby-3.0.2/gems/eventmachine-1.2.7/ext
compiling binder.cpp
In file included from binder.cpp:20:
./project.h:119:10: fatal error: 'openssl/ssl.h' file not found
#include <openssl/ssl.h>

# or
ruby 3.1.0 error: incomplete definition of type 'struct TS_verify_ctx'
# or
rvm install 3 ossl_pkey_rsa.c:950:5: error: use of undeclared identifier 'RSA_SSLV23_PADDING'
```

RVM NEEDS PKD_CONFIG_PATH set to old openssl using
```
PKG_CONFIG_PATH="$(brew --prefix openssl@1.1)/lib/pkgconfig" rvm reinstall 2.6.8 --with-out-ext=fiddle
# or
PKG_CONFIG_PATH="$(brew --prefix openssl@1.1)/lib/pkgconfig" bundle
```

```
brew info openssl
If you need to have openssl@1.1 first in your PATH run:
  echo 'export PATH="$(brew --prefix openssl@1.1)/bin:$PATH"' >> ~/.zshrc

For compilers to find openssl@1.1 you may need to set:
export LDFLAGS="-L$(brew --prefix openssl@3)/lib"
export CPPFLAGS="-I$(brew --prefix openssl@3)/include"

For pkg-config to find openssl@1.1 you may need to set:
  export PKG_CONFIG_PATH="$(brew --prefix openssl@1.1)/lib/pkgconfig"

```

for grpc https://github.com/grpc/grpc/issues/30976#issuecomment-1299550028
you can either
```
# recompile ruby

# gem install
gem install grpc -v 1.48.0 -- --with-ldflags="-Wl,-undefined,dynamic_lookup"

# bundle config
bundle config build.grpc --with-ldflags="-Wl,-undefined,dynamic_lookup"
```

when you also need to install ffi than join path with colon `:`
ffi error
```
install ruby closure.c:264:14: error: implicit declaration of function 'ffi_prep_closure' is invalid in C99 [-Werror,-Wimplicit-function-declaration]
```
so I solved with steps for rvm
https://github.com/ffi/ffi/issues/869#issuecomment-810890178
https://github.com/ffi/ffi/issues/869#issuecomment-1233000037
```
export LDFLAGS="-L/opt/homebrew/opt/libffi/lib"
export CPPFLAGS="-I/opt/homebrew/opt/libffi/include"
export PKG_CONFIG_PATH="/opt/homebrew/opt/libffi/lib/pkgconfig:$(brew --prefix openssl@1.1)/lib/pkgconfig"
export RUBY_CFLAGS=-DUSE_FFI_CLOSURE_ALLOC
bundle update ffi
# or
rvm reinstall 2.6.8
```

No need for --with-openssl-dir since it does not help
```
--with-openssl-dir=`brew --prefix openssl@1.1`
```
I solved with
```
# uninstall old version global and bundle
bundle remove eventmachine
gem uninstall -aIx eventmachine 

# configure
# this will raise error for C extension
# bundle config build.eventmachine --with-cppflags=-I$(brew --prefix openssl)/include
bundle config build.eventmachine --with-openssl-dir=$(brew --prefix openssl@1.1)
bundle install
# or just a gem
gem install eventmachine -- --with-cppflags=-I$(brew --prefix openssl)/include
# or set the system
brew link --force openssl
```

```
export RUBY_CONFIGURE_OPTS="--with-openssl-dir=`brew --prefix openssl@1.1`"
```

To install old ruby 2.6.7 for error
```
vm.c:2295:9: error: implicit declaration of function 'rb_native_mutex_destroy' is invalid in C99 
```
we need
```
CFLAGS="-Wno-error=implicit-function-declaration" rbenv install 2.6.7
```
but ruby 2.6.7 EOL so can not install on m1, and you should use 2.7.0

For error
```
closure.c:264:14: error: implicit declaration of function 'ffi_prep_closure' is invalid in C99
```
you can use
```
RUBY_CFLAGS=-DUSE_FFI_CLOSURE_ALLOC rbenv install 2.7.0
```

Puma 5.6.4 has problems with ssl
https://github.com/puma/puma/issues/2839#issuecomment-1086147152
```
LoadError: dlopen(/Users/dule/.rvm/gems/ruby-3.0.2/gems/puma-5.6.4/lib/puma/puma_http11.bundle, 0x0009): symbol not found in flat namespace '_SSL_get1_peer_certificate' - /Users/dule/.rvm/gems/ruby-3.0.2/gems/puma-5.6.4/lib/puma/puma_http11.bundle
```
which I solved with
```
gem uninstall puma
DISABLE_SSL=1 bundle

# or for new apps
DISABLE_SSL=1 rails new
```
Also you can compile with flags

```
gem install puma -v 5.6.5 -- --with-cppflags=-I`brew --prefix openssl@3`/include --with-ldflags=-L`brew --prefix openssl@3`/lib
```


For event machine
https://github.com/eventmachine/eventmachine/issues/960
```
/Users/dule/.rvm/gems/ruby-3.0.2/gems/bootsnap-1.11.1/lib/bootsnap/load_path_cache/core_ext/kernel_require.rb:30:in `require': dlopen(/Users/dule/.rvm/gems/ruby-3.0.2/gems/eventmachine-1.2.7/lib/rubyeventmachine.bundle, 0x0009): symbol not found in flat namespace '_SSL_get1_peer_certificate' - /Users/dule/.rvm/gems/ruby-3.0.2/gems/eventmachine-1.2.7/lib/rubyeventmachine.bundle (LoadError)
```
I solved with
```
# Gemfile
gem 'eventmachine', github: "eventmachine/eventmachine", branch: 'master'
```

For error
```
Unable to load the EventMachine C extension; To use the pure-ruby reactor, require 'em/pure_ruby'
```
I have no solutions.

For error
```
-bash: fork: retry: Resource temporarily unavailable
```
you have reached maximum number of proccesses
```
ulimit -u
```

For error
```
 xcodebuild[92386:809018] [MT] DVTPlugInLoading: Failed to load code for plug-in com.apple.dt.IDESimulatorAvailability (/Applications/Xcode.app/Contents/PlugIns/IDESimulatorAvailability.ideplugin), error = Error Domain=NSCocoaErrorDomain Code=3588 "dlopen(/Applications/Xcode.app/Contents/PlugIns/IDESimulatorAvailability.ideplugin/Contents/MacOS/IDESimulatorAvailability, 0x0109): Symbol not found: (_OBJC_CLASS_$_SimDiskImage)
```
I solved by starting xcode and it will install some development tools.

# Android USB Thethering

Download <http://www.joshuawise.com/horndis> driver (right click open with) to
enable usb tethering from android phone (wireless thethering will consume
batery). I needed to restart mac to find my phone in network preferences.

# Defaults

Mac user defaults are preferences for user system configurations

~~~
defaults read com.mydomain.myapp
defaults read com.mydomain.myapp "MyKey"
defaults write com.mydomain.myapp "MyKey" myvalue
defaults delete com.mydomain.myapp
~~~


# Logs

You can see logs using `console` application. You can attach iPhone and see logs
for it. Filter in `Search`, follow last logs with `Now` button.

# Closed lid

To keep working after lid is closed you can use app https://github.com/semaja2/InsomniaX and disable Idle Sleep and Disable Lid Sleep

Open new VLC in separate window using
~~~
/Applications/VLC.app/Contents/MacOS/VLC &
~~~

# Tips

Re-install node

```
brew uninstall --ignore-dependencies node icu4c
brew install node
brew link --overwrite node
```

Reinstall node-gyp https://github.com/nodejs/node-gyp/issues/1694
https://github.com/nodejs/node-gyp/issues/809
```
rm package-lock.json && rm -rf node_modules && rm -rf ~/.node-gyp
yarn upgrade
yarn
```

Reinstall postgres and postgis

```
brew update; brew reinstall postgresql; brew reinstall postgis
```

To start postgres
```
brew services restart postgresql
ps auxwww | grep postgres
psql
```

Create `postgres` user
```
createuser -s postgres
```

see logs with

```
pg_ctl -D /usr/local/var/postgres -l /usr/local/var/postgres/server.log start
# upgrade datatabases
rm -rf /usr/local/var/postgres && initdb /usr/local/var/postgres -E utf8
```

# Docker

Download docker desktop 4.5.0
https://docs.docker.com/desktop/mac/release-notes/

reinstall with https://github.com/docker/for-mac/issues/6145#issuecomment-1076397390
https://github.com/docker/for-mac/issues/6324#issuecomment-1143953963
https://stackoverflow.com/questions/44346109/how-to-easily-install-and-uninstall-docker-on-macos/65468254#65468254
```
rm -rf ~/Library/Caches/com.docker.docker
rm -rf ~/.docker
rm -rf ~/Library/Group Containers/group.com.docker

brew install --cask docker

open --background -a Docker
```
To move dataFolder you can create a symlink to external storage
```
ls -s /Volumes/eksterni/docker_containers/Docker.raw /Users/dule/Library/Containers/com.docker.docker/Data/vms/0/data/Docker.raw
```
Change config does not help since for some reason docker will not start
```
vi ~/Library/Group\ Containers/group.com.docker/settings.json
# "filesharingDirectories": [
  # eksterni does not work, not sure why, probably because it is usb
  "dataFolder": "/Volumes/eksterni/docker_containers/DockerDesktop",
  # sshfs also does not work
  "dataFolder": "/Users/dule/rails_main/docker_data",
  # local folders works
  "dataFolder": "/Users/dule/docker_data",
  "dataFolder": "/Users/dule/Library/Containers/com.docker.docker/Data/vms/0/data",
```

`docker-compose build` can raise an error
```
macos FileNotFoundError: [Errno 2] No such file or directory  During handling of the above exception, another exception occurred:
```
solution is to start `Docker Desktop`

If `curl` inside `Dockerfile` raise error
```
segmentation fault
```
solution is for M1 to define `platform: linux/amd64`
https://github.com/docker-library/php/issues/1176#issuecomment-901896821
```
# Dockerfile
version: '3.9'
services:
  web:
    platform: linux/amd64
    build: .
```

Starting rails can raise
```
web_1  | /usr/local/bundle/gems/rb-inotify-0.10.1/lib/rb-inotify/notifier.rb:69:in `initialize': Function not implemented - Failed to initialize inotify (Errno::ENOSYS)
web_1  | 	from /usr/local/bundle/gems/listen-3.7.0/lib/listen/adapter/linux.rb:29:in `new'
```
solution is to disable inotify file watcher
https://github.com/evilmartians/terraforming-rails/issues/34#issuecomment-872021786
```
# config/environments/development.rb
# config.file_watcher = ActiveSupport::EventedFileUpdateChecker
```
or update bin `docker-compose run web rake app:update:bin`
https://stackoverflow.com/questions/69773109/how-to-fix-function-not-implemented-failed-to-initialize-inotify-errnoenos


* to open iphone emulator from terminal you can try (note it does not contain
  app store)
```
open -a Simulator.app
```
* open android emulator android studio
```
/Volumes/eksterni/AndroidSDK/emulator/emulator -list-avds
/Volumes/eksterni/AndroidSDK/emulator/emulator @Pixel_3a_API_Tiramisu
/Volumes/eksterni/AndroidSDK/platform-tools/adb kill-server
```

Toggle software keyboard inside simulator with I/O -> Keyboard -> Toggle
software keyboard cmd + K
You can not send SMS to simulatork
https://help.apple.com/simulator/mac/current/#/devb0244142d

watch network requests
https://mdapp.medium.com/the-android-emulator-and-charles-proxy-a-love-story-595c23484e02

* safari show link on hover link preview at the bottom footer, Go to View =>
  Show Status Bar
* to take camera picture photo you can open *Photo Booth* app
* iphone icons https://developer.apple.com/sf-symbols/ https://github.com/cyanzhong/sf-symbols-online
* to install old python 2 you can not use 
```
brew install python@2
Warning: No available formula with the name "python@2". Did you mean ipython, bpython, jython or cython?
```
but you can install https://github.com/pyenv/pyenv
```
brew install pyenv
pyenv install 2.7.18
# pyenv global 2.7.18
pyenv versions

# install pyenv to .bashrc by copy paste script from
# PATH=$(pyenv root)/shims:$PATH
eval "$(pyenv init -)"
```

To set python version to current shell
```
pyenv shell 2.7.18
```

For error
```
lease use python3.9 or python3.8 or python3.7 or python3.6.
Node.js configure: Found Python 3.11.2...
````
use
https://gist.github.com/fernandoaleman/868b64cd60ab2d51ab24e7bf384da1ca?permalink_comment_id=4211086#gistcomment-4211086
```
pyenv global 3.9.1
```

* to mount ssh folder, instead of using brew
  ```
  brew install sshfs
  Error: sshfs has been disabled because it requires closed-source macFUSE!
  ```

  you can download
  https://github.com/osxfuse/sshfs/releases

  also https://github.com/osxfuse/osxfuse/releases

  ```
  sshfs orlovic@main:rails/ ~/rails_main/ -ocache=no -onolocalcaches -ovolname=ssh
  sshfs dule@trk:. ~/trk/ -ocache=no -onolocalcaches -ovolname=ssh
  ```

  unmount with
  ```
  umount -f ~/trk
  ```
* you can use Real VNC to connect to ubuntu (just enable Share -> Remove
  desktop) but it shows black screen
  Better is to use native vnc clint from Finder -> Go -> Connect to server or
  just `open vnc://user@machine`
  Note that on ubuntu you have to do login in order to unlock Keyring (for
  example f you have autologin enabled you will not be able to connect with VNC,
  it will just ask for password indefinitely, untill you open eg Chrome which
  will ask you to unlock keyring).
  One solution is to set empty Login password
  https://linuxconfig.org/how-to-disable-keyring-popup-on-ubuntu Search for
  Passwords > right click on Login > insert current password > insert blank
  password and confirm blank password. Now you do not need to unlock keyring.
  When vnc password is changed, than you will get error `Authentication failed
  to "trk"`. I do not know how to clear saved passwords for Finder.

