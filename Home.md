## Wifibroadcast

 - 1:1 map RTP to IEEE80211 packets for minimum latency (doesn't serialize to byte steam)
 - Smart FEC support (immediately yeild packet to video decoder if FEC pipeline without gaps)
 - Stream encryption and authentication ([libsodium](https://download.libsodium.org/doc/))
 - Distributed operation. It can gather data from cards on different hosts. So you don't limited to bandwidth of single USB bus.
 - Aggreagation of mavlink packets. Doesn't send wifi packet for every mavlink packet.
 - Enhanced [OSD](https://github.com/svpcom/wifibroadcast_osd for Raspberry PI) (consume 10% CPU on PI Zero)
   Compatible with any screen resolution. Supports aspect correction for PAL to HD scaling.

