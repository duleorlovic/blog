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
* `⌘ space` spotlight search

You can disabke any special key for any app. From System Preferences ->
Keyboard -> Shortcuts -> App Chortcuts -> +  than select the app and write exact
name and add shortcut uncluding ⌥  key:
* I disable Minimize since I do not need it, so add Minimize and random keyboard
  combination like `⌥ ⌘ M`.
* for iTerm I do not need Clear Buffer so now it is `⌘ ⌥ M`

From [Mac keyboard shortcuts from
support](https://support.apple.com/en-us/HT201236)
* `⌘ H` hide front app
* `⌘ ⌥ H` to show only front app
* `Fn-delete` is forward del since that key does not exists on macbook air

Click on link by holding ⌘ or ⌥ or ctrl will open new tab (with shift it will
focus that new tab), download and open context menu (ctrl click also works for
selected text).

# Update packages

Install homebrew
Update bash: `brew install bash`
[link](https://gist.github.com/xuhdev/8b1b16fb802f6870729038ce3789568f)

~~~
brew install coreutils findutils gnu-tar gnu-sed gawk gnutls gnu-indent gnu-getopt
brew info coreutils

brew install bash-completion
~~~

# iTerm2

Do `⌘ ,` to open Preferences -> Keys -> Key mappings and add `⌘ [` and `⌘ ]` to
map scroll page down and up.

# AppleScript

You can run scripts inside bash with `osascript -e 'display dialog "Hi"'` or for
shell scripts (you need to `chmod +x filename.txt`

~~~
#!/usr/bin/osascript
display dialog "Hi"
~~~



