How to install WFB with bidirectional mavlink telemetry
-------------------------------------------------------
1. Patch kernel for **ralink** cards (use `patches/nanopi-neo2-4.14.52-crda-disable.patch`) or install [patched driver](https://github.com/svpcom/rtl8812au) for **realtek** cards.
2. Install python-twisted package. Build tgz, deb or rpm package (see README.md) according to your linux distro and install it.
3. Generate encryption keys for ground station and drone: `wfb_keygen`. You need to put `gs.key` to `/etc/gs.key` on the ground station and `drone.key` to `/etc/drone.key` on the drone.
4. Create `/etc/wifibroadcast.cfg` with following content:

   for gs:
   ```
   [gs_mavlink]
   listen = None      # udp port for incoming connection, conflicts with 'connect'
   connect = 14550    # udp port for outgoing connection, conflicts with 'listen'
   ```
   for drone:
   ```
   [drone_mavlink]
   listen = 14550
   connect = None
   ```
   With this settings WFB will listen on port 14550 on drone and connect to udp://127.0.0.1:14550 on GS.

   If you want to override default modulation type (MCS#1, short GI, 40MHz BW, STBC 1)
   you can do it for each stream. **Stream settings are independent**. You can use different modulation for each of them.
   For example:
   ```
   [drone_video]
   bandwidth = 20     # bandwidth 20 or 40 MHz
   short_gi = False   # use short GI or not
   stbc = 1           # stbc streams: 1, 2, 3 or 0 if unused
   mcs_index = 1      # mcs index

   [drone_mavlink]
   bandwidth = 20     # bandwidth 20 or 40 MHz
   short_gi = False   # use short GI or not
   stbc = 1           # stbc streams: 1, 2, 3 or 0 if unused
   mcs_index = 1      # mcs index

   [gs_mavlink]
   bandwidth = 20     # bandwidth 20 or 40 MHz
   short_gi = False   # use short GI or not
   stbc = 1           # stbc streams: 1, 2, 3 or 0 if unused
   mcs_index = 1      # mcs index
   ```
   
5. Configure [mavlink-router](https://github.com/intel/mavlink-router) to connect to 127.0.0.1:14550 on the drone:
   ```
   [UdpEndpoint wifibroadcast]
   Mode = Normal
   Address = 127.0.0.1
   Port = 14550
   ```
   and use QGroundControl on the GS.
   See `telemetry/conf/master.cfg` for all available options and default values.
6. Setup RTP video streaming on the drone to udp://127.0.0.1:5602 or redefine listen port in `/etc/wifibroadcast.cfg` (see `telemetry/conf/master.cfg` for reference)
7. Edit `/etc/default/wifibroadcast` and repace `wlan0` with proper wifi interface name. Also add to `/etc/NetworkManager/NetworkManager.conf` following section:
   ```
   [keyfile]
   unmanaged-devices=interface-name:wlan0
   ```
   to ignore WFB interface.
8. Do `systemctl daemon-reload`, `systemctl start wifibroadcast@gs` on the GS and `systemctl start wifibroadcast@drone` on the drone.
9. Run `wfb-cli` on GS to monitor link state