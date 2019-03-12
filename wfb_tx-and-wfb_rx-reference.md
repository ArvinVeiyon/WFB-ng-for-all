wfb_rx
-------
``` 
Local receiver: wfb_rx [-K rx_key] [-k RS_K] [-n RS_N] [-c client_addr] [-u client_port] [-p radio_port] [-l log_interval] interface1 [interface2] ...
Remote (forwarder): wfb_rx -f [-c client_addr] [-u client_port] [-p radio_port] interface1 [interface2] ...
Remote (aggregator): wfb_rx -a server_port [-K rx_key] [-k RS_K] [-n RS_N] [-c client_addr] [-u client_port] [-l log_interval]
Default: K='rx.key', k=8, n=12, connect=127.0.0.1:5600, radio_port=1, log_interval=1000
```

There are three modes of wfb_rx:
1. Local receiver -- I.e. packet capture and processing are done on the local host
2. Remote forwarder -- Only capture a packets
3. Remote aggregator -- Process (collect from multiple hosts) packets captured by forwarder. You need this mode if your wifi cards connected to multiple hosts.

- ``-K rx_key`` -- path to rx keypair, default is ``rx.key``
- ``-k RS_K`` -- Reed-Solomon parameter "k" -- default 8
- ``-n RS_N`` -- Reed-Solomon parameter "n" -- default 12.
  This means that FEC block size if 12 packets and up to 4 (12 - 8) can be recovered if lost
- ``-c client addr`` -- ipv4 address where to send UDP packets, default 127.0.0.1
- ``-u client_port`` -- udp port where to send UDP packets, default 5600
- ``-p radio_port`` -- (1-255) wifi stream id. Must be unique for each data stream (i.e. telemetry up, telemetry down, video down, etc)
- ``-l log_interval`` -- interval in ms (default 1000) to dump statistics. Used for UI
- ``-a server_port`` -- udp port to listen incoming stream from forwarders

wfb_tx
------
``` 
Usage: wfb_tx [-K tx_key] [-k RS_K] [-n RS_N] [-u udp_port] [-p radio_port] [-B bandwidth] [-G guard_interval] [-S stbc] [-L ldpc] [-M mcs_index] interface1 [interface2] ...
Default: K='tx.key', k=8, n=12, udp_port=5600, radio_port=1 bandwidth=20 guard_interval=long stbc=0 ldpc=0 mcs_index=1
Radio MTU: 1446
WFB version 19.2.16.44936-212f5c1d
```

- ``-K tx_key`` -- path to TX keypair. Default is ``rx.key``
- ``-k RS_K`` -- Reed-Solomon parameter "k" -- default 8
- ``-n RS_N`` -- Reed-Solomon parameter "n" -- default 12.
 This means that FEC block size is 12 packets and up to 4 (12 - 8) can be recovered if lost
- ``-u udp_port`` -- listen udp port for packets for TX via first interface, second - udp_port + 1, third - udp_port + 2, etc
- ``-p radio_port`` -- (1-255) wifi stream id. Must be unique for each data stream (i.e. telemetry up, telemetry down, video down, etc)
- ``-B 20|40`` -- 20 or 40 MHz -- wifi channel bandwidth, default 20
- ``-G short|long`` -- wifi guard interval short or long, default long
- ``-G 0|1`` -- 0 or 1 -- enable stbc, default 0
- ``-L 0|1`` -- 0 or 1 -- enable ldpc, default 0
- ``-M mcs_index`` -- 0 - 7, mcs modulation index, default 1
- Radio MTU: 1446 -- max size of data in UDP packet.