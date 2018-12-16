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
   listen = None
   connect = 14550
   ```
   With this settings WFB will connect to udp://127.0.0.1:14550 on drone and GS.
   Configure mavlink-router to listen on 127.0.0.1:14550 on the drone and use QGroundControl on the GS.
   See `telemetry/conf/master.cfg` for all available options and default values.
5. Edit `/etc/default/wifibroadcast` and repace `wlan0` with proper wifi interface name. Also add to `/etc/NetworkManager/NetworkManager.conf` following section:
   ```
   [keyfile]
   unmanaged-devices=interface-name:wlan0
   ```
   to ignore WFB interface.
6. Do `systemctl daemon-reload`, `systemctl start wifibroadcast@gs` on the GS and `systemctl start wifibroadcast@drone` on the drone.
7. Run `wfb-cli` on GS to monitor link state