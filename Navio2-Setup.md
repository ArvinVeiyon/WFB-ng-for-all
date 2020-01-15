Navio2 setup
============

This guide will help you setup Wifibroadcast on Navio2 with bidirectional telemetry and video. This guide is based on the following references which were modified, expanded upon, and tested to work with the following setup:

- Drone running the 4.14.95 Navio2 image setup in the Arduplane configuration.
- Ground station running QGroundStation on Ubuntu.
- 2x Alfa AWUS036NHA Wifi cards. 

Please note that this guide hasn’t been thoroughly tested and flushed out yet, but it does work for me. :)

References:

- https://dev.px4.io/v1.9.0/en/qgc/video_streaming_wifi_broadcast.html
- https://github.com/svpcom/wifibroadcast/wiki/Install-from-scratch
- https://github.com/svpcom/wifibroadcast/wiki/Setup-HOWTO
- https://github.com/intel/mavlink-router
- https://www.instructables.com/id/Step-by-step-Guidance-to-Build-a-Drone-From-Scratc/
- https://community.emlid.com/t/estimation-of-feasibility-ez-wbc-on-navio2-rp3b/12274

And many, many various forums questions… :)

Drone setup
-----------

SSH into the drone. I like to use the ssh extension on VS code.

Download dependencies for wifibroadcast:

```
sudo apt-get install -y virtualenv fakeroot debhelper python-twisted \
libpcap-dev libsodium-dev python-pyroute2 python-future \
python-configparser python-all git python-setuptools
```

Download, build, and install wifibroadcast:

```
cd ~
git clone https://github.com/svpcom/wifibroadcast.git
cd wifibroadcast
make deb
cd deb_dist
sudo dpkg -i wifibroadcast*.deb
```

The Ubuntu 18.04 and the Navio2 4.14.95 image already have drivers for the Alpha AWUS036NHA. I’m not sure if they are the latest, but they work. Feel free to correct me or update them yourself. You can use the lsusb and usb-devices commands to check for drivers. I’m not sure if any patches are necessary to the kernel to adjust card tx power or other similar settings, but I didn’t do any. Suggestions are welcome here. See the following link for more info on people doing similar work: https://community.emlid.com/t/building-a-module/65/14

Run the following command, which will generate a drone.key and gs.key in your home directory: 

```
cd ~
wfb_keygen
```

You need to put the drone.key to /etc/drone.key and the gs.key to /etc/gs.key on the ground station. Since you are currently on the drone, use the following command to put the drone.key into /etc/. We’ll do the gs.key later.

```
sudo mv drone.key /etc/
```

Create the file `/etc/wifibroadcast.cfg` and give it the following contents:

```
[common]
wifi_channel = 1       # 161 -- radio channel @5825 MHz, range: 5815–5835 MHz, width 20MHz
                       # 1 -- radio channel @2412 Mhz, 
                       # see https://en.wikipedia.org/wiki/List_of_WLAN_channels for reference
wifi_region = 'BO'     # Your country for CRDA (use BO or GY if you want max tx power)

[drone_mavlink]
peer = 'listen://0.0.0.0:14550'   # incoming connection
#peer = 'connect://127.0.0.1:14550'  # outgoing connection

[drone_video]
peer = 'listen://0.0.0.0:5602'  # listen for video stream (gstreamer on drone)
```

Follow this guide until the “Connecting to the GCS” part to setup ardupilot:
https://docs.emlid.com/navio/common/ardupilot/installation-and-running/
They are using the arducopter in this example, but my setup is a plane. Thus, when they go to edit the /etc/default/arducopter file, edit instead the /etc/default/arduplane file and replace the `TELEM1="-A udp:127.0.0.1:14550"` with the following `TELEM1="-A udp:127.0.0.1:24550"`

This tells ardupilot to output the mavlink telemetry on port 24550.

Download and install mavlink-router:

```
git clone https://github.com/intel/mavlink-router.git
git submodule update --init --recursive
./autogen.sh && ./configure CFLAGS='-g -O2' --sysconfdir=/etc --localstatedir=/var --libdir=/usr/lib64 --prefix=/usr
make && sudo make install
```

Create the file `/etc/mavlink-router/main.conf` and give it the following contents:

```
[UdpEndpoint ardupilot]
Mode = Eavesdropping
Address = 127.0.0.1
Port = 24550

[UdpEndpoint wifibroadcast]
Mode = Normal
Address = 127.0.0.1
Port = 14550
```

This is telling the mavlink router to route mavlink packets from ardupilot on port 24550 to wifibroadcast on 14550. I’m actually not sure if this is totally necessary, perhaps you could just have ardupilot output telemetry to 14550 and wifibroadcast pick it up, but I’m going with the suggested way from the guides.

Edit `/etc/default/wifibroadcast` and replace wlan0 with wifi interface name of the Alpha AWUS036NHA card. You can find this using ifconfig. For me, it was the one called wlx00XXXXXXXXXX, but it’s different depending on the card, obviously.

Add the following section `/etc/NetworkManager/NetworkManager.conf`, again replacing wlan0 with the wifi interface name of the Alpha AWUS036NHA card.

```
[keyfile]
unmanaged-devices=interface-name:wlan0
```

To make sure everything starts on boot, add the following 3 lines to your /etc/rc.local file, right above the `exit 0` line

```
sudo systemctl start wifibroadcast@drone
mavlink-routerd &
(raspivid --nopreview --awb auto -ih -t 0 -w 640 -h 360 -fps 49 -b 4000000 -g 147 -pf high -o - | gst-launch-1.0 fdsrc ! h264parse !  rtph264pay !  udpsink host=127.0.0.1 port=5602 &)
```

This will start the wifibroadcast, telemetry, and video on boot.

Ground station setup
--------------------

Download dependencies for wifibroadcast:

```
sudo apt-get install -y virtualenv fakeroot debhelper python-twisted libpcap-dev libsodium-dev python-pyroute2 python-future python-configparser python-all git python-setuptools
```

Download, build, and install wifibroadcast:

```
cd ~
git clone https://github.com/svpcom/wifibroadcast.git
cd wifibroadcast
make deb
cd deb_dist
sudo dpkg -i wifibroadcast*.deb
```

Create the file `/etc/wifibroadcast.cfg` and give it the following contents:

```
[common]
wifi_channel = 1       # 161 -- radio channel @5825 MHz, range: 5815–5835 MHz, width 20MHz
                       # 1 -- radio channel @2412 Mhz, 
                       # see https://en.wikipedia.org/wiki/List_of_WLAN_channels for reference
wifi_region = 'BO'     # Your country for CRDA (use BO or GY if you want max tx power)

[gs_mavlink]
peer = 'connect://127.0.0.1:14550'  # outgoing connection
# peer = 'listen://0.0.0.0:14550'   # incoming connection

[gs_video]
peer = 'connect://127.0.0.1:5600'  # outgoing connection for
                                   # video sink (QGroundControl on GS)
```

Edit `/etc/default/wifibroadcast` and replace wlan0 with wifi interface name of the Alpha AWUS036NHA card. You can find this using ifconfig. For me, it was the one called wlx00XXXXXXXXXX, but it’s different depending on the card, obviously.

Add the following section /etc/NetworkManager/NetworkManager.conf, again replacing wlan0 with the wifi interface name of the Alpha AWUS036NHA card.

```
[keyfile]
unmanaged-devices=interface-name:wlan0
```

Assuming you are on the same network as your drone, use scp to grab the gs.key file and put it in your /etc directory. Be sure to replace the `<insert ip of navio2 here>` with the corresponding IP:

```
scp pi@<insert ip of navio2 here>:/home/pi/gs.key ~
sudo mv gs.key /etc
```

Download and install QGroundControl for Ubuntu per this guide: 
https://docs.qgroundcontrol.com/en/getting_started/download_and_install.html

In addition, I made a desktop shortcut for it instead of starting it from the command line every time like they suggest, but that part is up to you.

Open up QGroundControl -> Application Settings -> Comm Links, and add a comm link of type UDP, listening on port 14550 with  the target host being 127.0.0.1. Set it to auto connect and save it.

Finished with that?

Reboot the drone and ground station. You’ve already setup the drone to start up everything necessary on boot. On the ground station, run the following command to start up wifibroadcast:

```
sudo systemctl start wifibroadcast@gs
```

Use the following command to monitor the link state on the ground station. You should see output describing how many packets have been received, rssi values, etc. If everything went correctly, you won’t see “Link lost!” or anything.

```
wfb-cli gs
```

Start up QGroundControl and it should automatically connect to your drone. Yay!