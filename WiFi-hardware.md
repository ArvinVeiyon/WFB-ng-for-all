Supported WiFi hardware:
------------------------
In theory wfb-ng can support any wifi hardware that supports packet injection and monitor modes.
But injection quality in different drivers are **very different**. So now we officially support only cards on Realtek **RTL8812au** chipset.
Please note that 8811*, 8812bu, 8812cu or 8814au are different cards and not supported by author. They may work but it at your own risk.

RTL8812au [**Requires external patched driver!**](https://github.com/svpcom/rtl8812au).  System was tested with ALPHA AWUS036ACH on both sides in 5GHz mode.

Not supported, but **may** works:
---------------------------------
The following Atheros chipsets should work (not used by author):

 -  Atheros AR9271, Atheros AR9280, Atheros AR9287. They requires patched **firmware** for injection support.
 -  ~~Ralink RT28xx family. Cheap, but doesn't produced anymore. System was tested with ALPHA AWUS051NH v2 as TX and array of RT5572 OEM cards as RX in 5GHz mode.~~  **Ralink packet injection is broken in latest 5.x kernels** (injection became too slow and eats 100% cpu)