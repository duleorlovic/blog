---
layout: post
---


# Arduino

To install I download tar.gz and run `./install.sh`

## Arduino Makefile

I use Arduino Makefile to compile and flash to arduino https://github.com/sudar/Arduino-Makefile

```
sudo apt-get install arduino-mk
```

It gives some files https://packages.ubuntu.com/xenial/all/arduino-mk/filelist
I could ignore path to arduino dir since I used system arduino

```
# ~/config/bashrc/arduino.sh
export ARDUINO_DIR=/usr/share/arduino
export ARDMK_DIR=/usr/share/arduino/
```

So for your project you have to define
```
# Makefile
BOARD_TAG    = uno
ARDUINO_PORT = /dev/ttyACM0
include $(ARDMK_DIR)/Arduino.mk
```
In vim you can run

```
make
make upload
make monitor
# you can exit with Ctrl+C or Ctrl+A + K (and y)
make show_boards
```

In case you have an error `avrdude: can't open config file
"/usr/../avrdude.conf": No such file or directory` than define in makefile
`AVRDUDE_CONF = /etc/avrdude.conf`

Ignore build files
```
#.gitignore
build-uno/
```

To exit from serial monitor (screen or minicom, which I could not setup to work
from makefile) you can `Ctrl + A and than K and than Y`

## Low cost tx rx

VirtualWire works fine on ATMEGA Follow installation
http://www.airspayce.com/mikem/arduino/VirtualWire/ and download zip and extract
to `sketchbook/libraries` in my case is `/home/orlovic/Arduino/libraries`. Than
you can find examples of it: File -> Examples -> VirtualWire -> transmitter
Pin 12 is for sending, Pin 11 for receiving.

But since it is not updated I used Manchester http://mchr3k.github.io/arduino-libs-manchester/

# GRBL drive step motor using g codes

To install grbl download and extract https://github.com/gnea/grbl/releases and
inside Arduino Sketch -> Include library -> Add zip library navigate to grbl
folder.
This will create `~/Arduino/libraries/grbl`
Then upload firmware example: Open File -> Examples -> grbl -> grblUpload


## UGS

To install UGS universal g code sender, go to 
https://winder.github.io/ugs_website/download/ and download to programs and link
to bin
```
ln -s /home/orlovic/Programs/ugsplatform-linux/bin/ugsplatform ~/Programs/bin/
ugsplatform
```
https://www.youtube.com/watch?v=cWKO8j_tv7A&t=195s
Set Machine -> Firmware settings -> $20 $21 $22 to 0 Soft hard limits and homing
cycle
In console

```
$$
$20=0
```


a4988 Driver https://mehatron.rs/arduino-a4988-drajver-step-motora-sa-hladnjakom
arduino modul v3 for a4988 drivers
https://mehatron.rs/arduino-drajver-cnc-shield-v3-za-a4988
https://www.youtube.com/watch?v=TMK_fLgpESQ

200 steps x 16 resolution = 3200 steps for one round (360 degres)
steps per milimeter is 83
400 steps for Z axis, ie 1mm is 45 degres.


# Solar food dehydrator

https://www.youtube.com/results?search_query=solar+food+dehydrator

# Air quality monitor

https://github.com/carldunham/air-quality-monitor/tree/cd/initial-implementation

