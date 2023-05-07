---
layout: post
---


# Install

To install I download source tar.gz and run `./install.sh` or download binaries.
https://github.com/arduino/arduino-ide/releases

You should add user to `dialout` group (or try to install from ubuntu software
and installation will add your user to the group automatically, you just have to
log out)

## Arduino Makefile

I use Arduino Makefile to compile and flash to arduino https://github.com/sudar/Arduino-Makefile

```
# ubuntu
sudo apt-get install arduino-mk

# mac
brew tap sudar/arduino-mk
brew install arduino-mk
pip install pyserial
```

If your system is python3 than switch to python 2 with pyenv
PR https://github.com/sudar/Arduino-Makefile/pull/640 but can not find
python in Arduino.mk so you can switch back to python 2
```
eval "$(pyenv init -)"
pyenv shell 2.7.18
```
or to update the file
```
# vi $ARDMK_DIR/bin/ard-reset-arduino
#!/usr/bin/env python3
```

arduino-mk gives some mk files like Arduino.mk and you can find installation
folder with
```
brew --prefix arduino-mk
export ARDMK_DIR=`brew --prefix arduino-mk`
export ARDUINO_DIR=/Applications/Arduino.app/Contents/Java
export AVR_TOOLS_DIR=$ARDUINO_DIR/hardware/tools/avr
```
or put in bachrc ` ~/config/bashrc/arduino.sh`

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
make monitor # serial monitor using screen, press any key to send to serial
# you can exit with Ctrl+C or Ctrl+A + K (and y)
make show_boards
make show_submenu
```

In case you have an error `avrdude: can't open config file
"/usr/../avrdude.conf": No such file or directory` than define in makefile
`AVRDUDE_CONF = /etc/avrdude.conf`

In case of error `grep: /usr/share/arduino/hardware/arduino//boards.txt: No such
file or directory` than you need to export ARDUINO_DIR


Ignore build files
```
#.gitignore
build-uno/
```

Include libraries https://github.com/sudar/Arduino-Makefile#including-libraries


## Makefile for ESP8266 ESP32

https://github.com/plerup/makeEspArduino

```
git clone https://github.com/plerup/makeEspArduino.git
cd makeEspArduino

export ARDUINO_ROOT=~/Library/Arduino15

make -f makeEspArduino.mk DEMO=1 flash
make -f makeEspArduino.mk DEMO=1 clean
```

To debug Makefile you can echo variable value:
```
$(error   SKETCH is $(SKETCH))
```

I stuck with error
```
/Users/dule/Documents/Arduino/libraries/ArduinoIoTCloud/src/cbor/lib/tinycbor/src/cborparser.c:878:39: error: 'INT_MIN' undeclared (first use in this function)
  878 |         if (unlikely(v > (unsigned) -(INT_MIN + 1)))
      |                                       ^~~~~~~
/Users/dule/Documents/Arduino/libraries/ArduinoIoTCloud/src/cbor/lib/tinycbor/src/compilersupport_p.h:180:45: note: in definition of macro 'unlikely'
  180 | #  define unlikely(x)   __builtin_expect(!!(x), 0)
      |                                             ^
/Users/dule/Documents/Arduino/libraries/ArduinoIoTCloud/src/cbor/lib/tinycbor/src/cborparser.c:37:1: note: 'INT_MIN' is defined in header '<limits.h>'; did you forget to '#include <limits.h>'?
   36 | #include "cborinternal_p.h"
  +++ |+#include <limits.h>
```

so I end up using Arduino command line interface
https://arduino.github.io/arduino-cli/0.24/getting-started/

```
# generate completion
arduino-cli completion bash > ~/config/bashrc/arduino-cli.sh

# list installed boards
arduino-cli core list

# list ports
arduino-cli board list
# list board name and fqbn
arduino-cli board listall esp8266 | grep generic
Generic ESP8266 Module          esp8266:esp8266:generic

# compile
arduino-cli compile --fqbn  esp8266:esp8266:generic

# upload
arduino-cli upload --fqbn  esp8266:esp8266:generic -p /dev/cu.usbserial-13320

# serial monitor
arduino-cli monitor -p /dev/cu.usbserial-13320
```

## Arduino tips

`millis()` is unsigned long, it will go overflow (ie return to zero) after 50
days https://www.arduino.cc/reference/en/language/functions/time/millis/
Note that you should not use substraction since you might have what you do not
expect
```
unsigned long current = millis();


```

## Low cost tx rx

VirtualWire works fine on ATMEGA Follow installation
http://www.airspayce.com/mikem/arduino/VirtualWire/ and download zip and extract
to `sketchbook/libraries` in my case is `/home/orlovic/Arduino/libraries`. Than
you can find examples of it: File -> Examples -> VirtualWire -> transmitter
Pin 12 is for sending, Pin 11 for receiving.

But since it is not updated I used Manchester http://mchr3k.github.io/arduino-libs-manchester/

# ESP32

Add https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
to Preferences -> Additional Board Manager URLs
After installing new Tools -> Boards Manager -> esp32, you can select Tools ->
Board -> ESP32 Arduino -> DOIT ESP32 DEVKIT V1 and upload Blink example.
Port is `/dev/cu.usbserial-0001`

For ESP8266 based on
https://arduino-esp8266.readthedocs.io/en/latest/installing.html you need to add
https://arduino.esp8266.com/stable/package_esp8266com_index.json to Additional
boards manager urls, and install by Tools -> Boards Manager -> search for
esp8266. After that you can select Tools -> Board -> ESP8266
Note that port name is: `/dev/cu.usbserial-13310`

My model is NodeMCU 1.0 ESP-12E (older is 0.9 ESP-12)

Pinouts https://lastminuteengineers.com/esp8266-pinout-reference/

## Webserver

https://randomnerdtutorials.com/esp32-websocket-server-arduino/
https://randomnerdtutorials.com/esp32-relay-module-ac-web-server/

## Arduino cloud

Install iot cloud libraries https://docs.arduino.cc/software/ide-v1/tutorials/installing-libraries
by menu Sketch > Include library > Manage libraries > Bridge (by Arduino)

also if you have *ArduinoIoTCloud.h: No such file or directory* search for
ArduinoIoTCloud library and install it.

https://cloud.arduino.cc/home/
Create device
https://create.arduino.cc/iot/devices
select esp8266 or ESP32 or LoRaWan
use Genereic model
and save DEVICE_ID and SECRET_KEY

For device you can create thinks like bulbs, relays and add variables to that
thing.

Use firefox to connect to arduino agent.

Use dashboard to control variables.

You can download skecth and open in Arudino IDE but you need to set up
```
# arudino_secrets.h
#define SECRET_SSID "trk inovacije"
#define SECRET_OPTIONAL_PASS "asd"
#define SECRET_DEVICE_KEY "asd"
```
You should see thing based on secret key in serial log
```
Connection to "trk inovacije" failed
Retrying in  "500" milliseconds
Connected to "trk inovacije"
Connected to Arduino IoT Cloud
Thing ID: f24d631d-c36d-4a88-bede-05e21d3a23a8
onRelayDownChange relay_down=1
```
the source is `/Users/dule/Documents/Arduino/Relays_nov05a`

To connect to alexa we will follow next tutorial

## Alexa

Follow
https://iotcircuithub.com/arduino-iot-cloud-esp8266-alexa-home-automation/
but my echo can not find those switches on esp8266
Documentation says alexa should find switches
https://iotcircuithub.com/alexa-arduino-iot-cloud-smart-home-skill/
but I could not. Also I did not find any specific code or library for alexa,
only for Arduino IOT.

Another solution is to use https://github.com/vintlabs/fauxmoESP and follow
https://randomnerdtutorials.com/alexa-echo-with-esp32-and-esp8266/

Use Sketch -> Include library -> Manage libraries -> search fauxmo or install
libraries manually.
```
cd ~/Documents/Arduino/libraries
git clone git@github.com:me-no-dev/ESPAsyncWebServer.git
# for ESP32
git clone git@github.com:me-no-dev/AsyncTCP.git
# for ESP8266
git clone git@github.com:me-no-dev/ESPAsyncTCP.git
```
when I try example `fauxmoESP_Basic` my echo can not discover new devices. I
tried both ESP32 and ESP8266.


Another solution is to use https://github.com/Aircoookie/Espalexa
Example Examples -> Espalexa -> EspalexaBasic and echo recognizes those 3 lights
Note that once you recognize the light, you need to remove them so next time you
can recognize again.
I tried with ESP32 and it sometimes has a problem connecting to Wifi.
When I use mobile Wifi than it conects faster and more often.
Also not responding https://github.com/Aircoookie/Espalexa/issues/44
ESP8266 connects to both mobile wifi and cable wifi without problems.

You should start alexa on connect successfully. To listen on connection events
add callbacks:
```
bool alexaDeviceAdded = false;
void setup() {
  ArduinoIoTPreferredConnection.addCallback(NetworkConnectionEvent::CONNECTED, onNetworkConnect);
  ArduinoIoTPreferredConnection.addCallback(NetworkConnectionEvent::DISCONNECTED, onNetworkDisconnect);
  ArduinoIoTPreferredConnection.addCallback(NetworkConnectionEvent::ERROR, onNetworkError);
}

// https://github.com/arduino-libraries/Arduino_ConnectionHandler
void onNetworkConnect() {
  digitalWrite(LED_BUILTIN, ON_ACTIVE_LOW);
  Serial.print(">>>> CONNECTED to network. ");
  if (!alexaDeviceAdded) {
    Serial.println("espalexa.addDevice Light 1");
    espalexa.addDevice("Light 1", firstLightChanged);
    espalexa.begin();
  } else {
    Serial.println("Keep using old device");
  }
}

void onNetworkDisconnect() {
  digitalWrite(LED_BUILTIN, OFF_ACTIVE_LOW);

  Serial.println(">>>> DISCONNECTED from network");
}

void onNetworkError() {
  digitalWrite(LED_BUILTIN, OFF_ACTIVE_LOW);

  Serial.println(">>>> ERROR");
}
```


"Alexa, help me to get started with Light 1"
"Alexa, turn on Light 1"
"Alexa, turn off Light 1"
"Alexa, dim Light 1" # decrease by 50 from 255
"Alexa, increase light 1 by 100%" # increases by 100
"Alexa, brighten the Ligth 1" # increases by ~50


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

# CNC foam cutter

https://howtomechatronics.com/projects/arduino-cnc-foam-cutting-machine/
To prepare image you can use https://www.youtube.com/watch?v=ie5SqPaIaBw :
* gimp : clear background and use right click -> Colors -> Levels and move to
  right input level

# Solar food dehydrator

https://www.youtube.com/results?search_query=solar+food+dehydrator

# Air quality monitor

https://github.com/carldunham/air-quality-monitor/tree/cd/initial-implementation

# RF

NRF24L01
https://www.youtube.com/watch?v=rY8S9I2-zqs

Library: https://github.com/nRF24/RF24
https://www.youtube.com/watch?v=oIobOWXOJJg

# Motor change direction

Use two relays and two tasters to move engine forward and backward

