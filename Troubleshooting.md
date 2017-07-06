1. Run ``scripts/tx_standalone.sh wlan1`` on TX host
2. Run ``scripts/rx_standalone.sh wlan1`` on RX host
3. Run ``nc -u 5600`` on RX host
4. Run ``echo test | nc -u localhost 5600`` on TX host

If you got ``test`` message - debug video pipeline.
If you don't see ``test`` -- run tcpdump on TX ``tcpdump -i wlan1 -n -p`` and repeat test.
If card successfully tx - repeat tcpdump test on RX host.
