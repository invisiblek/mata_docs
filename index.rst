Essential PH-1 (mata) FAQ
=================================================

* You want fastboot up and running. Don't even attempt any of this if you don't.
* Bootloader can be unlocked via ``fastboot flashing unlock`` THIS WILL WIPE YOUR DATA
* You probably want to ``fastboot flashing unlock_critical`` while you're at it. This allows updating firmware partitions through fastboot.
* The boot partitions on this device (boot_a and boot_b) are not your "traditional" boot partitions. On these, they contain a kernel and ramdisk like you're probably used to, but in this case the ramdisk is actually your recovery. The device uses a "system-as-root" layout where the system partition actually contains what would have been in your ramdisk.
* Stock images (NMJ20D) can be found here: http://goo.gl/Zofzcb Flash them via fastboot.
* Early LineageOS build by invisiblek can be found here: https://goo.gl/mvZroX This is to be flashed with fastboot, follow the README inside the zip. Source here: https://gitlab.com/mata-dev
* TWRP can be downloaded from here: https://goo.gl/nNWykg but the touchscreen isn't working. ADB can be used to do various things. For instance ``adb shell twrp sideload`` will enable you to ``adb sideload something.zip``. Full TWRP command line reference is available here: https://twrp.me/faq/openrecoveryscript.html
