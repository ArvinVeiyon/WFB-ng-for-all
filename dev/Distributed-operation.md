# Distributed operation (PREVIEW)

WFB-NG supports distributed operation with multiple RX or (RX+TX) antennas on different hosts connected by IP network. You can use multiple hotspots connected
via ethernet and get "pseudo-cellular" network. For example you can gather video stream from moving transmitter in the forest without any wifi roaming overhead.

## Setup:

1. On the central node, the cluster topology should be set in `/etc/wifibroadcast.cfg`.  See `master.cfg` for reference.  You should specify node addresses, wifi card names and ssh keys (optional).
2. Only `wfb_rx` and `wfb_tx` binaries should be installed on the cluster nodes (except for the central master node) (for openwrt, wfb-ng and bash packages should be installed). Encryption keys and /etc/wifibroadcast.cfg **should not be installed** on them.
3. The cluster can be initialized in two modes: automatic (ssh) and manual
 a. For automatic mode, you only need to set the ssh key and user (usually root) under which ssh will work on cluster nodes without entering a password. If you use `ssh_agent`, then you do not need to add the key, but make sure that the agent runs under **root** since `wfb-server` needs to be launched under it (this is required to create an IP tunnel)
 b. For manual mode, you will need to generate wfb-ng initialization scripts for the cluster nodes. This is done via `wfb-server --profiles gs --gen-init X.X.X.X` where X.X.X.X is the **IP address** of the node. The resulting script needs to be copied to the node and run (for openwrt, save it as `/usr/sbin/wfb-ng.sh`, set executable permissions and do `/etc/init.d/wfb-ng enable` and `/etc/init.d/wfb-ng start`).
4. Run wfb-ng in cluster mode on the central node: `wfb-server --profiles gs --cluster ssh` or `wfb-server --profiles gs --cluster manual`. There is no cluster support in systemd scripts for wfb-ng yet, but you can add it yourself (this will only work for manual or ssh mode with keys)
5. Run `wfb-cli gs` on the central node and check that everything works. There will be very few differences from the single mode: instead of short antenna names like `X:Y` there should be `ADDR:X:Y`, where `ADDR` is the node IP address in hex format.

Differences between ssh and manual modes:

- In ssh mode, if an error occurs on one of the nodes (for example, something did not start or broke), the entire cluster will stop. Also, all services on the nodes will automatically stop if the central node is turned off.
- In manual mode, services on the cluster nodes work independently of the central node.

## **IMPORTANT**
- use only **IP addresses** (not FQDN names) for nodes! There is no separate check for this yet and this will lead to **undefined behavior** in the cluster logic!
- In manual mode you need to regenerate init scripts for **each node**  in case of any changes in wifibroadcast.cfg on the master node, because internal allocation of UDP ports will be different ant this will lead to **undefined behavior** in the cluster logic!
