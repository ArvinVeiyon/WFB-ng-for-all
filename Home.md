## WFB-NG is an opensource digital FPV and telemetry system.

## Main features:
 - 1:1 map RTP to IEEE80211 packets for minimum latency (doesn't serialize to byte steam)
 - Smart FEC support (immediately yeild packet to video decoder if FEC pipeline without gaps)
 - Stream encryption and authentication ([libsodium](https://download.libsodium.org/doc/))
 - Distributed operation. It can gather data from cards on different hosts. So you don't limited to bandwidth of single USB bus.
 - Aggreagation of mavlink packets. Doesn't send wifi packet for every mavlink packet.
 - Enhanced [OSD](https://github.com/svpcom/wfb-ng-osd) for Raspberry PI (consume 10% CPU on PI Zero) or any other system which
   supports gstreamer (Linux X11, etc). Compatible with any screen resolution. Supports aspect correction for PAL to HD scaling.
 - Provides IPv4 tunnel for generic usage

## Quick Start
If you have Raspberry PI 3/3B/3B+ then you can try to use [preconfigured image](https://github.com/svpcom/wfb-ng/releases/download/wifibroadcast-22.09/wifibroadcast_22.09-rpi3.img.xz).

Just flash it into the SD-card (8Gb or larger), ssh to the board and follow instructions on the screen.

## [Setup HOWTO](Setup-HOWTO)
## Telegram group: (**wfb-ng support**) https://t.me/+uT7ziDLV4I5kZmUy
Please note, that it is only one official group.
## FAQ
Q: What is a difference from original wifibroadcast?

A: Original version of wifibroadcast use a byte-stream as input and splits it to packets of fixed size (1024 by default). If radio packet was lost and this is not corrected by FEC you'll got a hole at random (unexpected) place of stream. This is especially bad if data protocol is not resistent to (was not desired for) such random erasures. So i've rewrite it to use UDP as data source and pack one source UDP packet into one radio packet. Radio packets now have variable size depends on payload size. This is reduces a video latency a lot.

Q: What type of data can be transmitted using WFB-NG?

A: Any UDP with packet size <= 1466. For example x264 inside RTP or Mavlink.

Q: What are transmission guarancies?

A: WFB-NG use FEC (forward error correction) which can recover 4 lost packets from 12 packets block with default settings. You can tune it (both TX and RX simultaniuosly!) to fit your needs.

Q: Is only Raspberry PI supported?

A: WFB-NG is not tied to any GPU - it operates with UDP packets. But to get RTP stream you need a video encoder (with encode raw data from camera to x264 stream). In my case RPI is only used for video encoding (becase RPI Zero is too slow to do anything else) and all other tasks (including WFB-NG) are done by other board (NanoPI NEO2).

Q: I'm unable to setup WFB-NG and want immediate help!

A: [See License and Support](License-and-Support)

## Theory
[Analysis of Injection Capabilities and Media Access of IEEE 802.11 Hardware in Monitor Mode](https://github.com/svpcom/wfb-ng/blob/master/doc/Analysis%20of%20Injection%20Capabilities%20and%20Media%20Access%20of%20IEEE%20802.11%20Hardware%20in%20Monitor%20Mode.pdf)

## TODO
1. Write user docs. Pull requests are welcome.
2. Do a flight test with different cards/antennas.
3. Investigate how to set TX power without CRDA hacks.
4. Tune FEC for optimal latency/redundancy.
