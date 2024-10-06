Check USB wiring
----------------
**Proper usb wiring is very important!**

Cheap / thin / unshielded USB cables may issue strange behavior like a lot of FEC errors and lost packets!

Always connect wifi card `+5V` wire to the BEC (not to USB +5V). Use at least 5A BEC for card power!

Add >= 470uF low ESR capacitor (like ESC has, for example Panasinic EEUFR1V102) between power and ground to filter voltage spikes. Be aware of [ground loop](https://en.wikipedia.org/wiki/Ground_loop_%28electricity%29) when using several ground wires.

In any condition don't power up wifi card without antennas!

How to install WFB-NG with bidirectional mavlink telemetry and IPoWB
--------------------------------------------------------------------
1. Install [**patched** driver **v5.2.20**](https://github.com/svpcom/rtl8812au) for **realtek** cards.

   - Don't use **ralink (rt28xx)** cards with **5.x** kernels - they have **broken injection (became too slow)!**

   - Don't use driver **rtl8812au** version **v5.6.4.2** -- it has **low output power!**

   - **Check that stock realtek driver (if exists) was disabled!**

   To disable add it to the blacklist:
   ```
   cat > /etc/modprobe.d/wfb.conf <<EOF
   # blacklist stock module
   blacklist 88XXau
   blacklist 8812au
   blacklist rtl8812au
   blacklist rtl88x2bs
   # maximize output power, see note below
   options 88XXau_wfb rtw_tx_pwr_idx_override=30
   EOF
   ```
   **Note1:** I don't have RF power meter suitable for output power measurement, but via analyzing power consumption of alfa awus036ach card I've found that maximum current consumption is with `rtw_tx_pwr_idx_override=63` (~1.6А in pulse). But some users say that maximum transmit distance is with `rtw_tx_pwr_idx_override=45`. This may be due to nonlinear amplifier distortion or due to measurement errors. **You can burn your card if set high power without active cooling!**
 
   **Note2:** For "ac180 2W high power" card from aliexpress max value is **`rtw_tx_pwr_idx_override=30`**. Also it **requires** separate +5V power with BEC **>= 5A**, **Low ESR capacitor** and **active cooling** ! Without any of these requirements you will got unstable behavior and/or damage the card! Also this cards is not recommended anymore - it has bad RF design (**dies after several days of continuous use**) and have bad SNR on RX (-20dB compared to BL-M8812EU2). Use BL-M8812EU2 as replacement.

   rebuild initramfs (`update-initramfs -k all -u`) and reboot. Check with `ethtool -i wlanXX` that drivers version is empty (it will equal to kernel version for stock driver and empty for patched driver).  **NVIDIA Jetson has stock rtl8812au installed. You need to remove it!**

   Take a look at udev rules to maintain stable naming on GS. It may be convenient to create something like:

   ```
   cat > /etc/udev/rules.d/65-persistent-net.rules <<EOF
   SUBSYSTEM=="net",ACTION=="add",ATTR{address}=="50:2b:73:00:0b:01",NAME="wfb0"
   SUBSYSTEM=="net",ACTION=="add",ATTR{address}=="04:42:1a:3c:86:33",NAME="wfb1"
   EOF
   ```

   This will force naming for your NIC. Refer to udev rule documentation to create proper rules.
   Don't forget to use proper name like `wlan0` or `wfb0` for your GS interface.

2. Install python-twisted package. Build tgz, deb or rpm package (see README.md) according to your linux distro and install it.
3. Generate encryption keys for ground station and drone: `wfb_keygen`. You need to put `gs.key` to `/etc/gs.key` on the ground station and `drone.key` to `/etc/drone.key` on the drone. Mention that this script will regenerate key and delete old one. If you run it twice on GS, then you can lose your drone connectivity.
4. Add `net.core.bpf_jit_enable = 1` to /etc/sysctl.conf. Reload sysctl.
5. Create `/etc/wifibroadcast.cfg` with following content:
   common part for gs and drone:
   ```
   [common]
   wifi_channel = 161     # 161 -- radio channel @5825 MHz, range: 5815–5835 MHz, width 20MHz
                          # 1 -- radio channel @2412 Mhz, 
                          # see https://en.wikipedia.org/wiki/List_of_WLAN_channels for reference
   wifi_region = 'BO'     # Your country for CRDA (use BO or GY if you want max tx power)  
   ```

   **Please note that radio band (2.4 or 5.8 GHz) depends on your wifi adapter model and used antennas!**

   add to gs:
   ```
   [gs_mavlink]
   peer = 'connect://127.0.0.1:14550'  # outgoing connection
   # peer = 'listen://0.0.0.0:14550'   # incoming connection

   [gs_video]
   peer = 'connect://127.0.0.1:5600'  # outgoing connection for
                                      # video sink (QGroundControl on GS)
   ```
   add to drone:
   ```
   [drone_mavlink]
   # use autopilot connected to /dev/ttyUSB0 at 115200 baud:
   # peer = 'serial:ttyUSB0:115200'

   # Connect to autopilot via malink-router or mavlink-proxy:
   # peer = 'listen://0.0.0.0:14550'   # incoming connection
   # peer = 'connect://127.0.0.1:14550'  # outgoing connection

   [drone_video]
   peer = 'listen://0.0.0.0:5602'  # listen for video stream (gstreamer on drone)
   ```
   With this settings WFB-NG will listen on port 14550 on drone and connect to udp://127.0.0.1:14550 on GS.
   
   Select autopilot telemetry connection via uart or via udp.
   If you selected uart and its device is not available then **wfb-ng will not start!**

   Mention please that this is a configuration for UDP that do not offers `connections`, but if you launch QGroundControl and it will listen for incoming UDP packets, then it will be able to send back replies to UDP source. So we can speak about bidirectional connection.

   If you want to override default modulation type (MCS#1, long GI, 20MHz BW, STBC 1)
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
   See `wfb_ng/conf/master.cfg` for all available options and default values.

5. GS setup options:
     - Use QGroundControl (for video, telemetry and control)
     - Use QGroundControl (for telemetry and control) + [WifibroadcastOSD](https://github.com/svpcom/wfb-ng-osd) (for video and OSD) on the GS. You need to disable video display in QGroundcontrol to avoid conflict
6. Setup RTP video streaming on the drone to udp://127.0.0.1:5602 or redefine listen port in `/etc/wifibroadcast.cfg` (see `wfb_ng/conf/master.cfg` for reference)
7. **Disable wpa_supplicant and other daemons on WFB-NG wlan interface!** Use `ps uaxwwww | grep wlan` to check.

    Double check that card is in **unmanaged state** in nmcli output and `ifconfig wlanXX` **doesn't show any address and card state is down**.

   - Edit `/etc/default/wifibroadcast` and repace `wlan0` with proper wifi interface name. Also add to `/etc/NetworkManager/NetworkManager.conf` following section:
     ```
     [keyfile]
     unmanaged-devices=interface-name:wlan0
     ```

   - If available `/etc/dhcpcd.conf` then edit it and add:
     ```
     denyinterfaces wlan0
     ```
   to ignore WFB-NG interface.
8. Do `systemctl daemon-reload`, `systemctl start wifibroadcast@gs` on the GS and `systemctl start wifibroadcast@drone` on the drone.
9. Run `wfb-cli gs` on the GS side to monitor link state

   **Note:** For putty users don't forget to select: `Settings -> Window -> Translation -> Enable VT100 line drawing` checkbox before connect.
10. For IPoWB (IPv4 over Wifibroadcast tunnel) you need only tun/tap kernel driver (tun.ko).
    Tunnel will be configured and work out of box. Drone side will have ``10.5.0.2/24`` address and GS - ``10.5.0.1/24``.
    Please note that tunnel use **less efficient coding rate** (to minimize latency) than video and mavlink streams and use it only for low-bandwidth traffic (like ssh), not as general link for video and telemetry streams.
11. On the ground station you can specify **multiple nics** (in /etc/default/wifibroadcast) if you have **multiple adapters** with directed and/or omnidirected antennas. TX adapter will be selected using RSSI of RX signal. In case of rtl8812au cards with two antennas you can enable STBC to use both of them for tx simultaneously.
12. If you got errors like:
    ```
    2021-12-28 14:57:15+0700 [-] Log opened.
    2021-12-28 14:57:15+0700 [-] # iw reg set BO
    2021-12-28 14:57:15+0700 [-] # ifconfig wlan1 down
    2021-12-28 14:57:16+0700 [-] # iw dev wlan1 set monitor otherbss
    2021-12-28 14:57:16+0700 [-] SIOCSIFFLAGS: Operation not possible due to RF-kill
    2021-12-28 14:57:16+0700 [-] Stopping reactor due to fatal error: RC 255: ifconfig wlan1 up
    2021-12-28 14:57:16+0700 [-] Main loop terminated.
    2021-12-28 14:57:16+0700 [-] Exiting with code 1
    ```
    then you need to disable RFKill:
    ```
    rfkill list all
    sudo rfkill unblock all
    ```
