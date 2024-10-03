Supported WiFi hardware:
------------------------
In theory wfb-ng can support any wifi hardware that supports packet injection and monitor modes.
But injection quality in different drivers are **very different**. So now we officially support only cards on Realtek **RTL8812au** and **RTL8812eu** chipsets.
Please note that 8811*, 8812bu, 8812cu or 8814au are different cards and not supported by author. They may work but it at your own risk.

 - **RTL8812au**. (stable) 802.11ac capable. [**Requires external patched driver!**](https://github.com/svpcom/rtl8812au)  System was tested with ALPHA AWUS036ACH on both sides in 5GHz mode.
 - **RTL8812eu**. (stable) 802.11ac capable. [**Requires external patched driver!**](https://github.com/svpcom/rtl8812eu) System was tested with [LB-LINK's BL-M8812EU2 module](https://www.lb-link.com/product_36_183.html)
 - **Atheros AR9350**. (beta) 802.11ac capable. Should work out of box, but requires kernel patches **to control TX power** (by default it will use max power). System was tested with [TP-Link CPE510](https://openwrt.org/toh/tp-link/cpe510), but should work with all similar devices.

##### NOTE:
- For Atheros SoC you need to use [cluster mode (beta)](../Distributed-operation) due to CPU, Flash and RAM limits.
- If you want only RX then in theory any card with monitoring mode will be suitable. For examle you can try your OpenWRT-enabled router.
- AC180 (aka Big Rookie) is not recommended anymore - it has bad RF design (**dies after several days of continuous use**) and bad SNR on RX (-20dB compared to BL-M8812EU2). Use BL-M8812EU2 as replacement.

Not supported, but **may** works:
---------------------------------
The following Atheros chipsets should work (not used by author):

 -  Atheros AR9271, Atheros AR9280, Atheros AR9287 and other **ath9k** **usb** cards for 2.4GHz band. They requires special **firmware** and kernel patches for injection support.
 -  ~~Ralink RT28xx family. Cheap, but doesn't produced anymore. System was tested with ALPHA AWUS051NH v2 as TX and array of RT5572 OEM cards as RX in 5GHz mode.~~  **Ralink packet injection is broken in latest 5.x kernels** (injection became too slow and eats 100% cpu)