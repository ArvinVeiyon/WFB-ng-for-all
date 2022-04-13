There is no "tx rate lock patch".
There are patches:
1. tx power lock (via CRDA).
2. ~~tx rate for injected packets (obsolete)~~
3. ~~tx timeslot patch (for one-way transmit only).~~

[1] is needed to fix tx power at max supported level and disable CRDA restrictions

[2] ~~enables API for specify bitrate when injecting packets. But for current ieee80211 stack is not sufficient to specify rate in injected header and you also need to set it via ``iw dev $WLAN set bitrates ht-mcs-5 1 sgi-5`` before tx start~~

[3] ~~removes most of tx timeslot guard intervals to maximize throughput. Use it **only** if you needed one-directional broadcast of video and telemetry. It is **not compatible** with two-way telemetry.  https://en.wikipedia.org/wiki/Extended_interframe_space~~

My HW setup is RT3572 (rt2800 driver) and RTL8812AU (https://github.com/svpcom/rtl8812au) in 5GHz band. If you have experience with other HW - pull requests are welcome.

**If you have recent mainline kernel (4.x) you don't need [2] (it already here). [1] and [3] are optional and needed only to increase transmit range and channel bandwidth.**