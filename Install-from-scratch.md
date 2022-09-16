1. Install libsodium
You can download and build sources from https://download.libsodium.org/doc/installation#integrity-checking
or install binary packages from you linux distro.

   To install from sources:
   ```
   git clone https://github.com/jedisct1/libsodium --branch stable
   cd libsodium
   ./configure
   make && make check
   sudo make install
   ```

2. Install wifibroadcast

   install deps:
   ```
   apt-get install virtualenv fakeroot debhelper
   apt-get install python-twisted libpcap-dev libsodium-dev python-pyroute2 python-future python-configparser python-all
   ```

   Build:

   ```
   cd ~
   git clone https://github.com/svpcom/wifibroadcast.git --branch stable
   cd wifibroadcast
   make deb
   ```

   Install:
   ```
   cd deb_dist
   dpkg -i wifibroadcast*.deb
   ```

3. Configure:

   Please follow wiki here:
   https://github.com/svpcom/wifibroadcast/wiki/Setup-HOWTO

   Create `/etc/wifibroadcast.cfg`
   modify the wifi device(s) to use in `/etc/default/wifibroadcast`

4. Troubleshooting

   **Check your wiring.** You may run into all kinds of strange problems, crashes, unstabilites, packetloss, bad blocks and glitches if you don't! **YOU NEED TO SUPPLY BOTH THE ARM BOARD AND THE WIFI CARDS WITH STABLE 5 VOLTS!**

   -  use short wires
   -  use wires with proper gauge, 20AWG (0.5mm²) is recommended
   -  solder everything, avoid any USB cables or plugs, often they are of questionable quality, wires are too thin and connectors fit too losely and can't cope with vibrations/shocks
   -  If you don't want or can solder directly to the card (for example if solder pads are very small), use a a USB cable and cut it off right behind the connector, then solder thicker wires to it (or directly to the board). Another alternative is to buy DIY USB connectors and solder cables to the conenctors directly.
   -  use a quality BEC which can handle the load, depending on number and type of cards and arm board used, current consumption can be as high as 3A for short peak periods.
   - consider adding low-ESR electrolytic caps near the wifi cards and Pi
   - do not use USB power banks, often they're of questionable quality and cannot deliver the advertised current
   - do not use the micro USB socket for power supply, apart from the drawbacks of the micro USB connector itself, on the Pi1/2/3 (as well as Pi1A+ and Pi3A+) there is a **polyfuse in the current-path which lowers voltage and limits current** which can lead to issues and instabilitues. If you want to power the Pi through the testpads at the bottom, make sure you use the testpads that are after the polyfuse (search Raspberry forum for details)
   - power through the **GPIO pins**

   **Check for EMI interference**

   USB2 bus frequency is 480MHz so 433MHz **high-power** telemetry radio (like SiK radio) can cause WiFi card disconnects. Need to use shielded cables and copper foil.

   **Check that you WiFi card and driver support selected band and/or channel**

   Old Atheros cards are 2.4GHz only. Some cards don't support arbitrary subsets of WiFi channels in 5.8GHz band.

   **Check wifibroadcast logs**

   Most of error messages are self-descriptive and help you to fix error quickly. Use following commands for drone:
   ```
   systemctl status wifibroadcast@drone
   journalctl -xu wifibroadcast@drone
   ```
   and for gs:
   ```
   systemctl status wifibroadcast@gs
   journalctl -xu wifibroadcast@gs
   ```