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
    on desktop.

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
but I change to caps_lock:


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
You can check if installed using https://java.com/en/download/installed.jsp on
safari.

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
cp -r Library/Services/* ~/Library/Services/
# copy back the changes so we can save them in repo
cp -r ~/Library/Services/* Library/Services/
```
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

To start mysql server you can run `mysql.server start`
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

When I was installing ruby 2.6.6 I got an error
```
install ruby closure.c:264:14: error: implicit declaration of function 'ffi_prep_closure' is invalid in C99 [-Werror,-Wimplicit-function-declaration]
```
so I solved with steps for rvm
https://github.com/ffi/ffi/issues/869#issuecomment-810890178
```
brew info libffi
export LDFLAGS="-L/opt/homebrew/opt/libffi/lib"
export CPPFLAGS="-I/opt/homebrew/opt/libffi/include"
export PKG_CONFIG_PATH="/opt/homebrew/opt/libffi/lib/pkgconfig"
rvm install "ruby-2.6.6"
```

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

`docker-compose build` can raise an error
```
macos FileNotFoundError: [Errno 2] No such file or directory  During handling of the above exception, another exception occurred:
```
solution is to start `Docker Desktop` (and you can quit it)

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


* to open emulator from terminal you can try
```
open -a Simulator.app
```

