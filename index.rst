
Installation
============

Prerequisites:
--------------

-  You want fastboot up and running. Don’t even attempt any of this if
   you don’t.

-  Unlocked Bootloader and Critical Partitions

Unlocking the Bootloader
========================

-  Bootloader can be unlocked via ``fastboot flashing unlock`` THIS WILL
   WIPE YOUR DATA

-  It is important you also ``fastboot flashing unlock_critical``. This
   allows updating firmware partitions through fastboot. Failure to do
   so has caused issues for users attempting to revert to stock.

Working with A/B Partitioning
=============================

The two boot partitions on this device, named ``boot_a`` and ``boot_b``
respectively, are not your “traditional” boot partitions. Both of these
partitions contain a kernel and ramdisk like you are probably used to.
The difference is that the ramdisk is now your recovery. This device
uses a "system-as-root" layout, with which the system partition now
contains what would have been the ramdisk.

The A/B partitioning scheme can be quite confusing to users. There are
actually two copies of many of the partitions

Slots
-----

The concept of 'slots' comes into play to determine whether you are
booting 'slot A' or 'slot B'. We can determine which slot is 'active' or
marked for booting via adb:

.. code:: 

    adb shell getprop ro.boot.slot_suffix

and via fastboot:

.. code:: 

    fastboot getvar current-slot

At any time we can switch to the next inactive slot using:

 ``fastboot set_active other``, or we can use:

 ``fastboot set_active [a,b]``, to manually switch to a specified slot.

The slots are designed to enable seamless system upgrades. Android can
install an update to the inactive slot while it is still running. Once
the update is completed, and optimized, Android will tell the system to
switch to the slot that was upgraded. This means, unless tampered with,
one slot will always be one version behind.

A/B Partitioning not only reduces 'first boot' time by performing dexopt
*before* rebooting, it also prevents boot failures from failed updates.
If Android fails to boot it will switch *back* to the previous slot so
that the next reboot will be back on the current, working, slot.

Installing LineageOS
====================

You will need to flash TWRP, the latest can be found here:
https://gg.gg/9ubmn Flash it to the **boot** partition with fastboot.

invisiblek's Lineage builds can be found here:

http://gg.gg/ai7lc

Roms are flashed the normal way through TWRP. More info and instructions
here:

https://gg.gg/9ubmz

**Note:** If you are changing roms you likely broke encryption and
*will* bootloop until you ``fastboot -w``

Common Issues
=============

Fingerprint Scanner Stopped Working
-----------------------------------

Chances are you have a firmware mismatch. Typically checking your
build.prop for the build version and flashing the matching firmware
package will fix this.

 Firmware packages here: https://gg.gg/9ubnb

If you move data between devices or change encryption. You may end up
with a nonworking fingerprint scanner after restoring data from a twrp
backup. If this occurs, you can flush the recorded fingerprint data by
deleting ``/data/system/users/0/fpdata`` and
``/data/system/users/0/settings_fingerprint.xml``. You should now be
able to re-enroll your fingerprints.

I Cant Boot After Restoring a Backup
------------------------------------

This is usually caused by the fact that you had to flash TWRP in order
to get in to TWRP. Therefore, when you did your backup, it actually
backed up the TWRP install instead of your ROM’s boot.img. The kernel
included with TWRP cannot boot Android.

I recommend swapping slots **before** flashing and booting into TWRP.
This will flash twrp onto your older, non-booting, slot.
``fastboot set_active other`` will switch slots for you.

Once you get to TWRP, go to the reboot menu, change to the other
(original) slot, and finally perform your backup. Make sure you always
decrypt your data, in TWRP, **before** backing up. It’s unlikely backing
up the encrypted FDE data is viable to restore. (Tinfoilers can always
encrypt the twrp backup that is generated with the check of a box).

No Touch in TWRP
----------------

Some day you may end up with a firmware that doesn’t play well with
twrp. Either no touch screen, or no data decryption.

One quick way to flash a zip in this situation is to put the device in
sideload mode via an adb command:

 ``adb shell twrp sideload`` then flash your zip using adb sideload:
 ``adb sideload myawesomezip.zip``

Broken Crypto
-------------

If you get an error about ``ExtractTarFork()`` its plausible that you
have a problem with crypto. It is likely that what is currently on your
device is unhappy with how things were backed up.

The best way I’ve found to fix this is to perform a data wipe in
fastboot via ``fastboot -w``, then boot into your ROM (with clean data),
go through the setup wizard and set a lockscreen password/pin
(preferably matching what you had before), then reboot into twrp and
restoring your data.

Firmware between N and O are finicky when it comes to crypto. I’m not
even sure I’ve even wrapped my head around all the ins and outs of
Oreo's crypto yet. If you’re backing up/restoring with O firmware, as of
(2017-11-22) you’re on your own.

Magisk
------

Requirements:

- Working Android Tools (fastboot/adb)
- Decrypted `/sdcard` (remove your lockscreen security and reboot)
- A copy of your boot.img
- Magisk 16.4 or greater (16.4 beta is the latest at the time of this writing)
- The latest version of TWRP_

Getting your boot.img:

- If you are on the stock rom/kernel you may simply use the boot.img the appropriate BTS.zip (http://gg.gg/mataBTS).
- If you are on Invisiblek's LineageOS builds, he offers a direct download_ of the boot image for 'pinned' builds.
- If you are using a custom kernel you may simply use it's .zip instead of the boot.img
- If you are not on a 'pinned' LOS, or you are using something else you will have to extract your boot.img_.

.. _download: https://updater2.invisiblek.org/mata
.. _boot.img: https://gg.gg/mataBTS/
.. _TWRP: https://gg.gg/9ubmn/

**Note:** It is important that you have completed the first time setup in order to Magisk to work. This cannot be applied to a fresh rom.

**Note 2:** You must *remove your lockscreen* (be decrypted) in order to complete this procedure. This is so you can access your twrp, magisk, and boot.img from /sdcard later.

Procedure:

If you are unfamiliar with how the slots work flashing magisk can seem cumbersome. This is because magisk patches the boot.img which we are actually replacing with TWRP. Therefore the process will look a bit like this:

Fastboot > flash twrp to active `boot` slot > boot twrp > flash boot.img to active `boot` slot > flash magisk to active `boot` slot.

- Before you reboot into fastboot to begin the process, make sure you have the required files on `/sdcard`.
- Reboot into fastboot, take note of your active slot, and flash twrp
- Reboot into recovery from fastboot
- Using TWRP, flash your boot.img to your `active` slot just like you would a rom. (If your boot is a raw boot.img, you must switch twrp into image mode from the 'choose zip' screen)
- Flash magisk *on top* of your boot.img in your active slot

You have now just patched the boot.img you flashed ontop of TWRP and your ROM should be able to boot.

Removing the Red Verity Warning
-------------------------------

The red verity message that appears on modified systems and requires you
to hit the power button to boot can be cleared by fastboot flashing this
boot.img: http://gg.gg/ai7m1

That image will reboot over and over again (you’ll never get anywhere)
but when it does, it’ll clear out that annoying red error. After
flashing it, boot normally once. You will still get the red error but it
will be cleared it the next reboot.

Here's a link to the patch that is included to perform this: http://gg.gg/ai7kb

Stock Update Notifications are Still Appearing
----------------------------------------------

You may still get stock Essential Update notifications even when on a custom rom. This is especially true is you're are one Android Security Bulliten (ASB) release behind. Update notifications can be annoying, and accidental taps can result in the device trying (and failing) to update only to reboot back into the installed custom rom.

These notifications can easily be disabled from Settings.

Browse to ``Settings -> Apps & notifications -> See all <x> apps -> Google Play Services -> App notifications``. 

At the bottom of this page you will see 'System Update'. This will stop you from seeing the stock rom update notifications until you switch roms or wipe data again.


Back to Stock
=============

There is a tutorial on xda here: http://gg.gg/mataBTS

Hidden features
===============

Invisiblek's Los has some hidden customizations that are made available through adb commands.

Add columns to QuickSettings
----------------------------

The number of columns are changed using the command ``setprop persist.qs_columns`` as ``adb root``

For example, if you wanted four Quick Settings columns you can run:

.. code::

    setprop persist.qs_columns 4

The default value is 3.

Change the System DNS Server
----------------------------

It may be desirable for the user to use a DNS server other than Google's. Prior to Android P this is not easily done. Invisiblek demonstrates that you can make this change by echo'ing the changes to ``/data/local.prop`` from adb shell. You will need to be root for this.

In this example we will be setting the system dns server too Cloudflare's DNS ``1.1.1.1``, ``1.0.0.1``:

.. code::

    adb root # you must first have root
    adb shell "echo 'net.rmnet_data2.user_dns1=1.1.1.1' >> /data/local.prop"
    adb shell "echo 'net.rmnet_data2.user_dns2=1.0.0.1' >> /data/local.prop"
    adb shell chmod 600 /data/local.prop # make local.prop rw for the current owner
    adb reboot

Source: http://gg.gg/mataDNS
