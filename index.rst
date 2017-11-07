Essential PH-1 (mata) FAQ
=================================================

* Bootloader can be unlocked via `fastboot flashing unlock` THIS WILL WIPE YOUR DATA
* You probaly want to `fastboot flashing unlock_critical` while you're at it. This allows updating firmware partitions through fastboot.
* The boot partitions on this device (boot_a and boot_b) are not your "traditional" boot partitions. On these, they contain a kernel and ramdisk like you're probably used to, but in this case the ramdisk is actually your recovery. The device uses a "system-as-root" layout where the system partition actually contains what would have been in your ramdisk.
