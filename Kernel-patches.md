There is no "tx rate lock patch".
There are patches:
1. tx power lock.
2. tx rate for injected packets
3. tx timeslot patch.

[1] is needed to fix tx power at max supported level and disable CRDA restrictions
[2] enables API for specify bitrate when injecting packets. But for current ieee80211 stack is not sufficient to specify rate in injected header and you also need to set it via ``iw dev $WLAN set bitrates ht-mcs-5 1 sgi-5`` before tx start
[3] removes most of tx timeslot guard intervals to maximize throughput. We use only one-directional broadcast and tx wouldn't listen for ACK's.  https://en.wikipedia.org/wiki/Extended_interframe_space

ez-wifibroadcast-1.4-kernel-4.4-patches.diff implements [1] and [3]
mac80211-radiotap-bitrate_mcs_rtscts.linux-4.4.patch implements [2]

Also there are AR9271 patches, but I've never try it because I don't have any atheros hardware. My HW setup is RT3572 (rt2800 driver) in 5GHz band.