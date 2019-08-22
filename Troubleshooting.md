If your video pipeline doesn't work, before opening new issue please
try following steps:

1. Ensure that NetworkManager is **disabled** or there is **line** in `/etc/NetworkManager/NetworkManager.conf`
**both on TX and RX**. Without this **rtl88xxau driver may not receive any packets** (tested on ubuntu-18.04):
```
[keyfile]
unmanaged-devices=interface-name:wlan0
```
where ``wlan0`` is name of your card

2. Run ``scripts/tx_standalone.sh wlan1`` on TX host

3. Run ``scripts/rx_standalone.sh wlan1`` on RX host

4. Run ``nc -lu 5600`` on RX host

5. Run ``echo test | nc -u 127.0.0.1 5600`` on TX host

If you got ``test`` message - debug video pipeline.

If you don't see ``test`` -- run tcpdump on TX ``tcpdump -i wlan1 -n -p`` and **repeat test** (i.e. run tcpdump after WFB scripts, but **before** `echo test`).

If card successfully tx - repeat tcpdump test on RX host.
