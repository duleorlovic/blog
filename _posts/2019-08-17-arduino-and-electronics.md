---
layout: post
---


# Arduino

You can download old Arduino (ver 1.8) to Programs, or you can use from Ubuntu
(ver 2.1).

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


# Solar food dehydrator

https://www.youtube.com/results?search_query=solar+food+dehydrator

# Air quality monitor

https://github.com/carldunham/air-quality-monitor/tree/cd/initial-implementation

