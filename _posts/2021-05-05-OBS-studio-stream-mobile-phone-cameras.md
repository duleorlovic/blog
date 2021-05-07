---
layout: post
---

# Install

Install https://obsproject.com/

```
sudo add-apt-repository ppa:obsproject/obs-studio
sudo apt-get update
sudo apt-get install obs-studio
```
# Stream from IP cameras

You can stream RTSP cameras by selecting "Media source" and unchecking "Local
file" and on `input` put url

For RTSP you can use media source but it is better in my case to use VLC player and use window source
Here is example url for two cameras:
rtsp://admin:admin@192.168.1.108:554/cam/realmonitor?channel=1&subtype=0&unicast=true&proto=Onvif
rtsp://admin:admin@192.168.1.168:80/0

# Stream from mobile Phone

IP Webcam Pavel Khlebovich
https://play.google.com/store/apps/details?id=com.pas.webcam&hl=en&gl=US

On app:
To keep working after locking the screen you need to add permission: Service
control, optional permissions,
To keep WiFi, you can disable batery optimisation for IP Webcam

In browser http://192.168.1.100:8080/ (keep this opened to check connection)
Crop only works when zoom is applied.
In advance settings set video resolution to be the same for all phones, 1280x720
(same as obs project -> settings -> video -> base canvas resolution)

## Stream remote screen recordings

You can stream from other computers using VNC (on windows install TinyVNC) and
on main comp use VNC viewer and stream that Window Capture (Xcomposite).

Stream from web use: https://github.com/bazukas/obs-linuxbrowser

# Scenes

Since changing scenes will disconnect and reconnect again, best way to switch
cameras is to move to top (on mac it is fn + cmd + left).
To preview cameras you can use right click and "Windowed projector" so you can
see all three cameras in three windows.
On each source you should use "Transform" and "Strech to screen". It is good to
have all cameras with same ratio width and height.
## Auto rotate scenes based on timer

Advances Scene switcher
https://obsproject.com/forum/resources/advanced-scene-switcher.395/ source
https://github.com/WarmUpTill/SceneSwitcher

To install download .so and copy
```
sudo cp ~/Downloads/SceneSwitcher/SceneSwitcher/Linux/advanced-scene-switcher.so /usr/lib/obs-plugins/
```

On linux there could be an error (Help -> Log files -> View current log)
```
09:13:53.109: os_dlopen(/usr//lib/obs-plugins/advanced-scene-switcher.so->/usr//lib/obs-plugins/advanced-scene-switcher.so): /usr/lib/x86_64-linux-gnu/libQt5Gui.so.5: version `Qt_5' not found (required by /usr//lib/obs-plugins/advanced-scene-switcher.so)
09:13:53.109:
09:13:53.109: Module '/usr//lib/obs-plugins/advanced-scene-switcher.so' not loaded
```

You can check which libraries qt is using `ldd $(which qtcreator)` (it is in
/usr/lib/x86_64-linux-gnu/libQt5Gui.so.5: but plugin is not using it).
I tried to compile plugin from source
```
cmake -DLIBOBS_INCLUDE_DIR=~/Programs/obs-studio/libobs -DLIBOBS_FRONTEND_INCLUDE_DIR=~/Programs/obs-studio/UI/obs-frontend-api/ -DLIBOBS_FRONTEND_API_LIB=/usr/lib/ -DCMAKE_INSTALL_PREFIX=/usr ..
make
```
but I receive error
```
Generating ui_advanced-scene-switcher.h
File '/home/orlovic/Programs/SceneSwitcher/src/headers/advanced-scene-switcher.ui' is not valid
AUTOUIC: error: process for ui_advanced-scene-switcher.h needed by
 "/home/orlovic/Programs/SceneSwitcher/src/headers/advanced-scene-switcher.hpp"
```

So I build from source using Debian-based Build Directions on
https://obsproject.com/wiki/install-instructions#linux

```
sudo apt-get install build-essential pkg-config cmake git-core checkinstall
sudo apt-get install libx11-dev libgl1-mesa-dev libvlc-dev libpulse-dev libxcomposite-dev libxinerama-dev libv4l-dev libudev-dev libfreetype6-dev libfontconfig1-dev qtbase5-dev libqt5x11extras5-dev libqt5svg5-dev libx264-dev libxcb-xinerama0-dev libxcb-shm0-dev libjack-jackd2-dev libcurl4-openssl-dev libluajit-5.1-dev swig python3-dev
```

Install ffmpeg from source

```
sudo apt-get install zlib1g-dev yasm
git clone --depth 1 git://source.ffmpeg.org/ffmpeg.git
cd ffmpeg
./configure --enable-shared --prefix=/usr
make -j4
sudo checkinstall --pkgname=FFmpeg --fstrans=no --backup=no \
        --pkgversion="$(date +%Y%m%d)-git" --deldoc=yes
#      dpkg -r ffmpeg

```

To find package which provide a header file
```
apt-file search X11/extensions/scrnsaver.h
```

So I installed also

```
sudo apt-get install libxss-dev
```

Build OBS

```
git clone --recursive https://github.com/obsproject/obs-studio.git
cd obs-studio
mkdir build && cd build
cmake -DUNIX_STRUCTURE=1 -DCMAKE_INSTALL_PREFIX=/usr ..
make -j4
sudo checkinstall --pkgname=obs-studio --fstrans=no --backup=no \
       --pkgversion="$(date +%Y%m%d)-git" --deldoc=yes
```

