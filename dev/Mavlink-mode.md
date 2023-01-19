# Mavlink

WFB-NG offers first-class support for mavlink bidirectional connectivity between drone and GS.


Principal schema is following:

```
|----------------------------|
|      Flight controller     |
|   |---------------------|  |
|---| UART with telemetry |--|


|---| UART on Raspi       |--|
|   |---------------------|  |
|                            |
|   WFB on Drone             |
|----------------------------|


|----------------------------|
|  WFB on GS                 |
|   |----------------------| |
|---| UDP sink for mavlink |-|


|--| UDP listener          |-|
|  |-----------------------| |
|                            |
|      QGroundControl        |
|----------------------------|
```

After configuring this schema, you will immediately see drone connection in QGC.




# Delivery details

By default WFB-NG encapsulates one source UDP packet into one WiFi packet. But mavlink packets are very small (usually less than 100 bytes) and send them in separate packets produces too much overhead. So for mavlink packets there is default setting `[common] mavlink_agg_timeout = 0.1` which will aggragate mavlink packets while they less than radio_mtu but no longer than 100ms.

