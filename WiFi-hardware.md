Supported WiFi hardware:
------------------------
My primary hardware targets are:
1. Realtek **RTL8812au**. 802.11ac capable. Easy to buy. [**Requires external patched driver!**](https://github.com/svpcom/rtl8812au)  System was tested with ALPHA AWUS036ACH on both sides in 5GHz mode.
2. ~~Ralink RT28xx family. Cheap, but doesn't produced anymore. System was tested with ALPHA AWUS051NH v2 as TX and array of RT5572 OEM cards as RX in 5GHz mode.~~  **Ralink cards are broken in latest 5.x kernels** (injection became too slow and eats 100% cpu)

Not supported, but **may** works:
---------------------------------
The following Atheros chipsets should work (not used by author):

 -  Atheros AR9271, Atheros AR9280, Atheros AR9287