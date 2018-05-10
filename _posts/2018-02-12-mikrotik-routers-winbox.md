---
layout: post
---

# Winbox on Wine

<https://wiki.winehq.org/Ubuntu>

~~~
sudo dpkg --add-architecture i386
wget -nc https://dl.winehq.org/wine-builds/Release.key
sudo apt-key add Release.key
sudo apt-add-repository https://dl.winehq.org/wine-builds/ubuntu/
sudo apt-get update
sudo apt-get install --install-recommends winehq-stable
~~~

Run `winbox.exe` in wine and download plugins. Click on tab `Neighbors` and when
it finds it, use Login: admin and empty password.
I'm using WinBox v3.12.

On router there is RouterOS.  To upgrade, download routeros
https://mikrotik.com/download for your specific archicteture and drag and drop
(works in wine) to "Files" (it will appear on root) and simply reboot.
I'm using RouterOS v6.41.2

# Terminal and Scripts

~~~
# comment should start on beggining of the line
# move top level
/
# move up
..
# show available commands
?
# run other commands within current level
/ping 10.0.0.1
~~~

Item names can be used instead of item number and are more stable since some
other user can configure router in the same time or change scripts.

General menu commands:
https://wiki.mikrotik.com/wiki/Manual:Console#General_Commands

* `add` to create new record. You can use `copy-from`

  ~~~
  add address = 192.168.0.1 / 24 network = 192.168.0.0 broadcast = 192.168.0.255 interface = Local
  ~~~

* `remove <id>` remove record
* `disable <id>` and `enable <id>`
* `move`
* `edit <id> prop` any property/param in editor `edit pptp-out1 password`
* `set <id> prop=value` similar to edit, but for more properties. Use `!` to
  unset `:set 3 !password`. To disable you can use `set 3 disable=yes`.
* `get <id> prop` used for inline expressions `:put [get 1 address]`
* `find` filtering `:put [/interface find name~"ether"]` this will return array
  of `<id>`. It can be used for example
  to remove all globals you can `/system script environment remove [find]`,
  update specific item: `/system logging set [find topics="warning"]
  action=disk`
* `print file=my_file` print to file, `print detail` print in format
  `prop=value` (oposite to `print brief`), `print follow` like tail -f, `print
  from=my_property_name` print single item find by name (similar to `print where
  name=my_property_name`), `/ip route print where interface="ether1"` filtering
  used for print, `print dynamic` print dynamic items. `print static interval=1`
  print static items and refresh every 2 seconds
* `/export file=myfilename` will export all configurations. You can export only
  not default with `/export compact`, or `/export compact file=my_export` to
  save to a file `/dule.rsc` which you can download using ftp. If you are not in
  root, for example you are in `/ip` it will export only changes to `ip`. You
  can `/import myfilename.rsc` to load configurations. Add `verbose=yes` to see
  more info. If file name ends with auto `myfilename.auto.rsc` than it will be
  executed when uploaded using FTP.
  In rsc file you can indent all commands with `:%s/^\([as]\)/    \1/`

Keyboard shortcuts similar to linux: `<c-k>` clear to end of line, `<c-b>`
`<c-f>` back and forward one char, `<c-a>` `<c-e` jump to begging or end of
line.

Safe mode starts with `<c-x>`, and ends and keep all changes also with `<c-x>`.
twice `<c-x>` will empty safe mode action list so you can store more.
`<c-d>` undo all safe mode changes, while `/quit` does not.
You can rewiev all safe mode changes (while you are in safe mode) with `/system
history print`
If you want to paste some commands, but autocompletion triggers some errors, you
can go to Safe mode so autocompletion is triggered only on tab.

<https://wiki.mikrotik.com/wiki/Manual:Scripting#Global_commands>
Note that all commands that work with data, starts with `:`

* `:delay 3` do nothing for 3 seconds. Could be miliseconds `:delay 10ms`

In terminal you can set variables. Local variables only works inside `{ }`, They
are ignored if used at root of console. Always use snakeCase, so you do not
overlap with keywords. If you need to use underscore than wrap variable with
quotes.

~~~
:global globalVar;
{
  :local localVar localValue;
  :set localVar 1;
  :put $localVar;
}
:global "var_with_underscore";
:global "var-with-dash";
~~~

Conditionals

~~~
:if () do={
}

# check if some record exists
:if ([:len [/system package find name="dhcp" !disabled]] != 0) do={
}
# check if ping did not timeout
:if ([:ping 123.123.123.123 count=5] = 0) do={
  :error "Ping timeout 5 times"
}
~~~

Various

~~~
:put message="some message"
# join long lines with \
# white space is not allowed around "="
~~~

Data types

* `num`
* `bool`
* `str` string can be concatenated with `"asd"."qwe"`. Lenght of string with
  `:put [:len "asdf"]`. Interpolate with `:put "a=$($a))"` or `:put "we have
  find $[ :len [/ip route find] ] routes"`
* `ip`
* `ip-prefix`
* `id` prefixed with `*`
* `time`
* `array` can be concatenated with comma `:put ({1;2;3}, 5)`. Can also use named
  elements `{1; a=2; "b3"=3; 4}` and than can be accessed with hash arrow but it
  needs to be grouped with brackets `:set ($a->"a") 22`. Positional elements can
  be accessed with `:put [:pick $a 0]`
  Arrays can be generated from string with `:local myArray [:toarray $myStr]`
* `nil`

You can convert data

~~~
#convert string to array
:local myStr "1,2,3,4,5";
:put [:typeof $myStr];
:local myArr [:toarray $myStr];
:put [:typeof $myArr]
~~~

Operators

* `in` does belongs `:put (1.1.1.1/32 in 1.0.0.0/8);`
* `[]` square brackets is command substitution `:put [ :len "my string" ];`.
  Only one line is allowed (commands can be separated with `;`). To use in menu
  commands wrap with a string `add address="$[ $ip . "1" ]"`. It is not allowed
  to have nested substitution.
* `()` round brackets sub expression or grouping operator `:put ( "value is " . (4+5));`

Find

`:find` there is also `find` command but it is only for filtering.
`:put [:find "asd" "s"];` return position of substring or array (in this
example `1` so if you pick to that position `s` will not be included), can
accept integer that means where to start (-1 is before start, 0 is to ignore
first).  If you want to know if it founds try this:

~~~
:if ([:len [:find "abcd" "x"]] > 0) do={:put "Found";} else={:put "Not Found";};
~~~

Pick

`:pick $myVar <start> <end>` return substring `:put [:pick "abcde" 1 3]` will
return `bc`. Start can be `-1` or `0` to pick from beggining.

Loops

`:for i from=1 to=10 step=2 do= {}` iterate for loop
`:do { } while=()` or `:while () do={ }` while loop
`:foreach i in=$myArray do= { }` iterate array elements

Functions

Functions can accept named `$argName` or unnamed arguments `$1, $2 ...`. Avoid
using parameters with same name as global variables.

~~~
:global myFunc do={
  :local sum ($a + $b)
  :return $sum
}

# call function with square brackets
:put [$myFunc a=1 b=2]
~~~

To call another function from inside function, you need to declare it

~~~
:global funcA do={ :return 5 }
:global funcB do={
  :global funcA;
  :return ([$funcA] + 4)
}
~~~

Parse can be used to define functions: `:global myFunc [:parse ":put hello!"];`

Logs and debug

<https://wiki.mikrotik.com/wiki/Manual:System/Log>
`:log warning "My message"` write to topic: `error`, `info` and `warning`. You
can define topis on `/system logging` and which action to perform. Actions could
be `memory` (shows in `/log print`), `echo` (show in console), `disk` (write to
file), `email` and `remote` (from webproxy to Kiwi server). Topics could be for
example `hotspot`, `pppoe`. To set `warnings` to write to file (anyway it will
show in `/log` just if we need to save for later use)

~~~
# set warning to disk
/system logging set [find topics="warning"] action=disk
:log warning "WAN started "
~~~

to see all logs on host you can run

~~~
ssh admin@$MIKROTIK_IP log print follow
~~~

`:put [:time { :delay 100ms }]` show time needed to execute command
`:environment print` show all variables and functions. They are defined as
records on `/system script environment print`
`:do {} on-error={ :put "my script failed" }` catch run time errors
`:error "Message"` stop executing the script

Run command in background and remove it
~~~
{
  :local pid [:execute {/interface print follow}]`
  :delay 5s
  :do { /system script job remove $pid } on-error={}
}
~~~

Scripts can be stored and triggered on event or by another
script. For example I use my helper functions and load them from other scripts.
To create use winbox (copy and paste) since `$`, `"` and `\ ` must be escaped
with preceding backslash `\ ` if you want to create using terminal `/system
script add name=helpers source=...`. Before use you need to run them

~~~
/system script run helpers
~~~

To replace mask with 1 you can

~~~
# helpers.script
# Add to /system scripts using Windbox
# if using terminal you need to escape $ " \
# before use you need to call
# /system script run helpers

# Strip Mark from IP
# :put [$stripMask 192.168.1.5/24]
# 192.168.1.5
# :put [$stripMask 192.168.1.5]
# 192.168.1.5
:global stripMask do={
  :local slashPosition [:find $1 "/"]
  if ([:len $slashPosition] > 0) do={
    :return [:pick $1 -1 $slashPosition ]
  } else= {
    :return $1
  }
}

# Replace Last Number in IP address
# :put [$replaceLastNumber 192.168.1.5/24 1]
# 192.168.1.1
# :put [$replaceLastNumber 192.168.1.5 "2"]
# 192.168.1.2
:global replaceLastNumber do={
  :local firstDot [:find $1 "." -1]
  :local secondDot [:find $1 "." $firstDot]
  :local thirdDot [:find $1 "." $secondDot]
  :return ([:pick $1 -1 $thirdDot] . "." . $2)
}
~~~


## Script to send system log to email

https://forum.mikrotik.com/viewtopic.php?t=29130

# Theory

To connect using IP you need to create IP -> Addresses -> New Address.
Also you need to setup routes with gateway that are provided by isp (or it is
your adsl router ip address).
https://wiki.mikrotik.com/wiki/Manual:IP/Route
Route with `dst-address=0.0.0.0/0` applies to every destination address.

Walled garden `/ip hotspot walled-garden` can be used to allow deny specific
host in http and https requests.

~~~
/ip hotspot walled-garden
add dst-host=:^www.example.com path=":/test\$"
~~~

`/ip hotspot walled-garden ip` can be used to allow deny specific host and IP
requests

~~~
/ip hotspot walled-garden ip
add action=accept disabled=no dst-host=www.paypalobject.com

# allow google dns and specific site
add action=accept disabled=no dst-address=8.8.8.8
add action=accept disabled=no dst-address=IpOfMySERVER
~~~

`dst-address` can be a [Destination IP address
list](https://wiki.mikrotik.com/wiki/Manual:IP/Firewall/Address_list) so you can
group them...

<https://wiki.mikrotik.com/wiki/Manual:Hotspot_HTTPS_example>
HTTPS can not be redirected since it starts in the browser and it creates a
socket destionation_ip:443/TCP and mikrotik intercepts that and can not provide
valid certificate for this url, so hotspot can't redirect https to login page.
Mikrotik can use certificate and redirect all https requests that are not on
HSTS list.

https://mikrotik.com/documentation/manual_2.6/IP/Hotspot.html
You can have two address pools (one for authenticated and one for non auth
users) and ARP feature could be set to `reply-only` to prevent network access
using static IP addresses.


# Connect to Mikrotik

You can FTP to Mikrotik `ftp 192.168.5.1` (username `admin` and blank password).

~~~
ssh-keygen -t dsa
ftp 192.168.5.1
# username: admin
# password: blank
> ls
> mkdir backups
> cd backups
> cd ..
> put local_file_name
> get remote_file_name
> exit
~~~

Or in one line you can use `lftp`

~~~
lftp -u admin, -e "cd hotspot;put doc/mikrotik/login.html;exit" 192.168.5.1
~~~

Also you can SSH
https://wiki.mikrotik.com/wiki/Use_SSH_to_execute_commands_(DSA_key_login)

~~~
# using winbox create new user: admin-ssh (password can be blank)
# https://askubuntu.com/a/885396/40031
/ip ssh set always-allow-password-login=yes
~~~

From your machine you can:

~~~
ssh -o HostKeyAlgorithms=+ssh-dss -o KexAlgorithms=diffie-hellman-group14-sha1 -l admin-ssh -i id 192.168.5.1

# or put in ~/.ssh/config
Host 192.168.5.1
  HostKeyAlgorithms ssh-dss
  KexAlgorithms diffie-hellman-group1-sha1

# so you can simply
ssh admin-ssh@192.168.5.1
~~~

Reset router with ssh

~~~
/system reset-configuration no-defaults=yes keep-users=yes run-after-reset=flash/wan.rsc
~~~

One short beep is that reseting started, normal beep is that actual reset
occured, and two beeps are ready.

Since there could be some initialization problems, you should add delay in rsc
files. To iterate command until it success you can try this script

~~~
# this succeed since we do not use network
/ip pool
add name=dhcp-pool-my-comp ranges=192.168.5.10-192.168.5.254

:local repeat true
:local counter 0
:while ($repeat) do={
  :do {
    # this raise error on boot, so repeat untill succeed
    /ip dhcp-server
    add address-pool=dhcp-pool-my-comp disabled=no interface=ether2 name=\
    dhcp-my-comp
    :set repeat false
  } on-error= {
    :delay 1
    :set counter ($counter + 1);
    :log warning "delay $counter"
    :if ($counter=50) do={ :set repeat false }
  }
}
~~~

# My hotspot testing

I have two ethernet cards: eth0 (Atheros, bottom port) and eth1 (Intel, upper
port). I connected my `eth0` to `ether2` and later `eth1` to `ether3`.
First port `ether1` on Mikrotik is used like WAN and connected to my ADSL router
<https://forum.mikrotik.com/viewtopic.php?t=60313#p308124>
Second mikrotik port `ether2` is my computer connection.

~~~
/ip address
# my wan is 192.168.2.1
add address=192.168.2.9/24 interface=ether1
# my comp is on 192.168.5.*
add address=192.168.5.1/24 interface=ether2

/ip pool
add name=dhcp-pool5 ranges=192.168.5.10-192.168.5.254
/ip dhcp-server
add address-pool=dhcp-pool5 disabled=no interface=ether2 name=dhcp5
/ip dhcp-server network
add address=192.168.5.0/24 dns-server=192.168.5.1 gateway=192.168.5.1

/ip dns
set allow-remote-requests=yes servers=8.8.8.8
/ip route
add distance=1 gateway=192.168.2.1

# IMPORTANT to masquerade if you are behind other routers
/ip firewall nat
add action=masquerade chain=srcnat out-interface=ether1
~~~

Third mikrotik port `ether3` is connected to my `eth1` ethernet and bridged to
Virtual Box.
Mikrotik is using Radius server PUBLIC_RADIUS_IP but inside VPN with
VPN_USERNAME and VPN_PASSWORD (so use PRIVATE_RADIUS_IP) on the mikrotik hosted
in the cloud MIKROTIK_IN_THE_CLOUD_IP. There could be also
PUBLIC_RADIUS_BACKUP_IP, and PRIVATE_RADIUS_BACKUP_IP. RADIUS_SECRET is needed
for communication.

~~~
:global PUBLIC_RADIUS_IP ....
:global PRIVATE_RADIUS_IP ....
:global PUBLIC_RADIUS_BACKUP_IP ....
:global PRIVATE_RADIUS_BACKUP_IP ....
:global VPN_USERNAME ....
:global VPN_PASSWORD ....
:global MIKROTIK_IN_THE_CLOUD_IP ....

# Interfaces -> + -> PPTP client -> Dial Out
# Connect To: MIKROTIK_IN_THE_CLOUD_IP, User: VPN_USERNAME, Password:
# VPN_PASSWORD, uncheck pap and chap, check mschap1 mschap2
/interface pptp-client
add allow=mschap1,mschap2 connect-to=$MIKROTIK_IN_THE_CLOUD_IP disabled=no \
  name=pptp-out1  password=$VPN_PASSWORD user=$VPN_USERNAME

# since there could be only one pptp client connection for specific username,
# check if connection established correctly

# IP -> Routes -> +
# Dst. Address: PRIVATE_RADIUS_IP_0/24 Gateway: pptp-out1
# Dst. Address: PRIVATE_RADIUS_BACKUP_IP_0/24 Gateway: pptp-out1
/ip route
add distance=1 dst-address=PRIVATE_RADIUS_IP_0/24 gateway=pptp-out1
add distance=1 dst-address=PRIVATE_RADIUS_BACKUP_IP_0/24 gateway=pptp-out1
# maybe also vpn private network

# IP -> Firewall -> NAT -> +
# Chain: srcnat, tab Action select masquerade
/ip firewall nat
add action=masquerade chain=srcnat out-interface=pptp-out1
~~~

If you want to ping from comp to hotspot user, user need to be logged in.
If it is logged in user can ping router `ping 10.5.50.1`, and other hotspot
users `ping 10.5.50.6` and host comp `ping 192.168.5.252`. Also from host you
can ping user.
If it not logged in, user can not ping router and host can not ping user.

If `/login` shows "Error 404: Not Found" that means you are connected with
ether2 and not ehter3 (hotspot)

If not logged in it can be byepassed with `/ip hotspot ip-binding add
address=10.5.50.5 type=bypassed`, but than it can not access login or status
pages `http://10.5.50.1/login`.
Or you can add Walled garden `/ip hotspot walled-garden ip add
dst-address=192.168.5.252 action=accept` and than it will be accessible in both
directions, from and to 192.168.5.252.
https://forum.mikrotik.com/viewtopic.php?t=88317

For Hotspot we allow ALLOW_SITE_IP and Google DNS
If www.google.com inside hotspot client does not work, it is because of DNS is
available but redirects to https, which is than redirected to login page, but
since we do not have proper certification, there is an error.
Try to use some of google IP address to see if it will redirect to hotspot login
page, or use `http://yahoo.com`

~~~
:global ALLOW_SITE_IP = ....

# IP -> Hotspot -> Hotspot setup
# HotSpot interface = ether3, Local Address of Network = 10.5.50.1/24
# check Masquerade Network, Address Pool of Network = 10.5.50.2-10.5.50.254,
# Select Certificate = none, IP Address of SMTP server = 0.0.0.0
# DNS Server = 8.8.8.8, DNS Name of local hotspot server = (empty)
# Hotspot user Name: admin, password: (empty)
/ip hotspot profile
add hotspot-address=10.5.50.1 login-by=cookie,http-pap name=hsprof1 \
    nas-port-type=ethernet use-radius=yes
/ip pool
add name=hs-pool-3 ranges=10.5.50.2-10.5.50.254
/ip dhcp-server
add address-pool=hs-pool-3 disabled=no interface=ether3 lease-time=1h name=dhcp1
/ip hotspot
add address-pool=hs-pool-3 disabled=no interface=ether3 name=hotspot1 profile=hsprof1
/ip address
add address=10.5.50.1/24 comment="hotspot network" interface=ether3
/ip dhcp-server network
add address=10.5.50.0/24 comment="hotspot network" gateway=10.5.50.1
/ip firewall filter
add action=passthrough chain=unused-hs-chain comment="place hotspot rules here" disabled=yes
/ip firewall nat
add action=passthrough chain=unused-hs-chain comment="place hotspot rules here" disabled=yes
add action=masquerade chain=srcnat comment="masquerade hotspot network" src-address=10.5.50.0/24
/ip hotspot user
add name=admin

# IP -> Hotspot -> Server Profiles -> hsprof
# -> Login : check HTTP PAP, uncheck HTTP CHAP
# -> RADIUS : check Use RADIUS, NAS Port Type: ethernet
# this properties: login-by, nas-port-type and use-radius already include above

# IP -> Hotspot -> Walled Garden IP List -> +
# Dst. Address = 8.8.8.8
# IP -> Hotspot -> Walled Garden IP List -> +
# Dst. Address = xceednet.com (or ip address for older RouterOS)
/ip hotspot walled-garden
add comment="place hotspot rules here" disabled=yes
/ip hotspot walled-garden ip
add action=accept disabled=no dst-address=8.8.8.8
add action=accept disabled=no dst-address=$ALLOW_SITE_IP

# Radius -> +
# check ppp, check hotspot, Address = PRIVATE_RADIUS_IP, Secret = RADIUS_SECRET,
# Timeout = 5000ms
# same for PRIVATE_RADIUS_BACKUP_IP
/radius
add address=$PRIVATE_RADIUS_IP secret=secret service=ppp,hotspot timeout=5s
add address=$PRIVATE_RADIUS_BACKUP_IP secret=secret service=ppp,hotspot timeout=5s
~~~

# Hotspot templates Html pages

https://wiki.mikrotik.com/wiki/Manual:Customizing_Hotspot

There are several pages that are rendered on mikrotik:

* `redirect.html` immediatelly redirect with http status (also with meta but
  I think that is not used). When I remove first two lines that perform
  http-status and http-header

  ~~~
  $(if http-status == 302)Hotspot redirect$(endif)
  $(if http-header == "Location")$(link-redirect)$(endif)
  ~~~

  than it will use some other default template which redirects. If I
  change `$(link-redirect)` to google, it will redirect to google. So you can
  not override this file to not perform redirect.

GET `/login`
* if user is not authenticated than renders `login.html` to ask user for
  username and password. Parameters on this url can be:
  * `username`, `password` (plain for PAP or md5 hash of chap-id, password and
  challenge  for CHAP)
  * `dst` original requested url before redirect
  * `popup` show status window on success login
  * `radius` some radius params
POST & GET `/login`
* if user is successfully authenticated or already authenticated than it
  renders `alogin.html` page. It redirects in javascript or meta to
  link-redirect (it is external page or http://10.5.50.1/status)
* if not successfull auth (wrong password or username) it renders `flogin.html`
  if exists or redirect with http status 302 to login

GET `/status`
* if auth user than render `status.html` show status with logout form that
  submit to `/logout` Form can have `erase-cookie` to erase cookie so user need
  to insert username/password again even cookie login is enabled
* if not auth user and if `fstatus.html` exists than use that template,
  otherwise use `redirect.html` to redirect to `/login`

GET `/logout`
* if auth user it will log out and render `logout.html` and provide a link to
 `/login` (refresh will not work since user is deauthenticated)
* if not auth user than redirect 302 to `/login` or use `flogout.html` if exists

* `error.html` show fatal errors

GET external page
* if user is not auth it renders `rlogin.html` if exists or redirect 302 to
  `/login?dst=target_page`. It can be used for external login

GET "/"on hotspot
* `rstatus.html` root on hotspot host for auth user,
* `radvert.html` when advertisement is due to be displayed,

You can have different pages for different `/ip hotspot profile print where
html-directory=hotspot`. You can create subfolder for translation (`lv`) and use
parametar `target=lv`. Other links will stay in that subfolder.

Links variables which can be used like `$(varName)`:
* `link-login` link to login page including original URL requested
  http://10.5.50.1/login?dst=http://www.example.com/
* `link-login-only` link to login page, not including original URL requested
  http://10.5.50.1/login
* `link-orig` original url requested http://yahoo.com
* `link-logout` link to logout http://10.5.50.1/logout
* `link-status` link to status

General variables:
* `logged-in` "yes" is user is logged in, otherwise "no"
* `mac` MAC of the user
* `username` name of the user
* `error` error message
* `hostname` DNS or IP address of hostspot servlet
* `server-address` hostspot server address with port
* `ssl-login` "yes" if https methods was used to access this page
* `interface-name` bridge or interface name
* `ip` ip address of the client
* `host-ip` client ip address from `/ip hotspot host`

Inside templates `$(if varName) ... $(elif varName) ... $(else) ... $(endif)`
or `$(if logged-in = 'yes')` can be used for conditionals.

You can POST to `/login.html` with username and password, and user will be
authenticated and redirected. You can also GET and user will be authenticated
but `/status.html` will be shown.
It's better to use POST with https.


If hotspot user after logout, goes to status page, if will be logged in again,
because of cookie. You need to erase cookie if you want user to type password
again.

~~~
<input type="hidden" name="erase-cookie" value="on">
~~~

or you can disable cookie login for hotspot profile `/ip hotspot profile set
default login-by=http-chap`

External login <https://wiki.mikrotik.com/wiki/HotSpot_external_login_page>

Note that form url should point to `/login` path
~~~
no
~~~

## Firewall

https://wiki.mikrotik.com/wiki/Manual:IP/Firewall
obsolete https://mikrotik.com/testdocs/ros/3.0/


`chain` is name of rule group. Rulles are taken from the chain in the order they
are listed and if packet matches the criteria than action is perfomed and no
more rules are processed in that chain (except for `passthrough` action), if a
packet has not matched any rule whith the chain then it is accepted.
  * `input` for connections to the router (destination is one of the router's
  addresses)
  * `forward` for connection going through router
  * `output` for connections from the router (originated the the router)
  * `my_name` for example `hotspot` for hotspot connections

`action` could be:
  * `accept` to allow packet to pass through and no more rules are applied
  * `redirect` replaces destionation port to `to-ports=64872` and destination
  address `to-addresses` to one of the routers local address.
  * `jump` with `jump-target=chain-name` jumps to the chain which can `return`
  usually with defined `hotspot` for example: `hotspot=from-client,!auth`, `to-client,auth`, `local-dst`
  * `return` jump back to the chain where the jump took place.
  * `drop` drop this packet without sending ICMP reject message
  * `reject` with `reject-with=tcp-reset` or `reject-with=icmp-net-prohibited`
  * `passthrough` continue rules on this chain (ignore current rule)
  * `log` add message to system log, same as when you check "Log" for other
  actions
  * `add-dst-to-address-list`, `add-src-to-address-list` add address to my-list
  `address-list=my-list`
  * `src-nat` replaces `src-address` with `to-addresses` and `to-ports`
  * `dst-nat` replaces `dst-address` with `to-addresses` and `to-ports`

`dst-address` ip address range ip/mask or ip-ip.
`dst-port` destination port
`dst-address-list` list
`protocol`: `tcp`

`hotspot` multiple choice: `auth,from client,to client,http,local dst`. There
are dynamic rules that `jump` to another chain:
  * `hs-unauth` for chain: `forward` and hotspot: `from client,!auth`
  * `hs-unauth-to` for chain: `forward` and hotspot: `!auth,to client`
  * `hs-input` for chain: `input` and hotspot: `from client`. which jumps to
  `pre-hs-input`
and at the end there is `reject` for `hs-unauth`, `hs-unauth-to`. If you want to
allow some trafic to (and from) add action `return` so it does not reach those
`reject` rules. You need to add twice for both incoming and outgoing packets.

You need to check both tables: Firewall Rules and NAT.
For Hotspot in NAT firewall table there are `redirect` to port 64872-64875.
https://wiki.mikrotik.com/wiki/Manual:Customizing_Hotspot#Firewall_customizations
  * 64872 port provides DNS service for Hotspot users
  `5 chain=hotspot action=redirect to-ports=64872 dst-port=53 protocol=tcp`
  * 64873 port is hotspot http servlet port (when user access router IP)
  `6 chain=hotspot action=redirect to-ports=64873 hotspot=local-dst dst-port=80
  protocol=tcp`
  * 64874 port is Walled Garden proxy server
  `12 D chain=hs-unauth action=redirect to-ports=64874 dst-port=80 protocol=tcp`
  * 64875 port is hotspot https servlet port
  `8 D chain=hotspot action=redirect to-ports=64875 hotspot=local-dst
  dst-port=443 protocol=tcp`

Redirection in NAT should be done in `dstnat` chain with `dst-nat` action, which
can accept `address-to=`

## IP Proxy

Proxy can be used for increasing web speed or for firewall, when we redirect
users to the proxy and show them local page.
https://wiki.mikrotik.com/wiki/Manual:IP/Proxy#Proxy_based_firewall_.E2.80.93_Access_List
For example when Radius assign user some ip UNAUTHORIZED_IP we can redirect all
trafic from that UNAUTHORIZED_IP to our hotspot page `unauthorized.html`

~~~
# assign address to interface
/ip address
add address=UNAUTHORIZED_IP_with_0_1/16 interface=ether3

# redirect all requests to 8080
/ip firewall nat
add action=redirect chain=dstnat protocol=tcp \
src-address=UNAUTHORIZED_IP_with_0_1/16 to-ports=8080

# enable ip proxy
/ip proxy
set enabled=yes

# redirect path
/ip proxy access
print detail
# you can redirect specific sites like facebook
add action=deny dst-host=www.facebook.com redirect-to=10.5.50.1/unauthorized.html
# you can allow specific sites
add action=allow dst-host=*.trk.in.rs
~~~

## HTTPS and redirect

https://wiki.mikrotik.com/wiki/Manual:Hotspot_HTTPS_example
Https use secure connection (SSL/TLS handshake) with a server so request to
https://www.google.com which is redirected at firewall level will not work with
your server/proxy/router.
Only think that user can do is to add Exception to install your certificate and
use transparent web proxy.

Even it is not possible to redirect https without warning, modern browsers
(firefox 58) will show login button (link to 10.5.50.1/login) for hsts and non
hsts site, so users can continue with login.

> Log in to network

> You must log in to this network before you can access the Internet.

> This site uses HTTP Strict Transport Security (HSTS) to specify that Firefox
> may only connect to it securely. As a result, it is not possible to add an
> exception for this certificate.

![https redirect non hsts site]({{ site.base_url }}/assets/posts/https%20redirect%20for%20non%20hsts%20site.png)
![https redirect hsts site]({{ site.base_url }}/assets/posts/https%20redirect%20for%20hsts%20site.png)

Lets encrypt can be used to generate certs.
https://www.ollegustafsson.com/en/letsencrypt-routeros/

~~~
# get acme
curl https://get.acme.sh | sh
# issue a cert
acme.sh --issue --dns -d router.mydomain.com
# check if record exists
host -t txt _acme-challenge.router.trk.in.rs
# renew when txt record is available
acme.sh --renew -d router.mydomain.com

~~~

https://github.com/gitpel/letsencrypt-routeros
https://github.com/Neilpang/acme.sh/pull/706/files

NAT
https://mikrotik.com/testdocs/ros/3.0/qos/filter.php
Nat is used to rewrite source (destination) ip address for source (and
destination) NATted network. `masquerade` is a form of source NAT where
`to-addresses` is not needed to be specified, outgoing interface is used.
`redirect` is a form of destination NAT where `to-addresses` is not used,
incoming interface is used.

For easier manipulation you can create lists

~~~
/ip firewall address-list
add list=my_site.com address=my_site.com
# at this moment, mikrotik will fetch ip from dns and save to address list
~~~


Mangle mark routing
https://www.youtube.com/watch?v=q13z9_dmmyA

Microtik youtube tutorials
https://www.youtube.com/user/rodrick4u/playlists

# Tips

* resolve hostname and update record
 https://wiki.mikrotik.com/wiki/Manual:Scripting-examples#Resolve_host-name

  ~~~
  :resolve www.google.com
  ~~~

* user scripts https://wiki.mikrotik.com/wiki/Scripts
* I got [rb 750 gr3](https://mikrotik.com/product/RB750Gr3) with 16MB RAM in
  flash.
