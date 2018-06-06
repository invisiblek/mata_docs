
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

Invisiblek's Lineage builds can be found here:

https://updater.invisiblek.org/mata

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

Flashing magisk after a rom is a bit of a problem. The official magisk
zip ends up installing it to the currently booted slot. Typically
though, you’d want to be installing it to the inactive slot after
flashing a ROM zip (and thus switching to the slot the rom was installed
to).

I’ve made a hacked magisk zip that forces the flash to go to the
opposite slot that you are booted to in order to alleviate this
headache:
https://invisiblek.org/magisk/magisk\ *15.2*\ invisiblekhax.zip

Flash this after flashing your rom while you’re still in TWRP.

Removing the Red Verity Warning
-------------------------------

The red verity message that appears on modified systems and requires you
to hit the power button to boot can be cleared by fastboot flashing this
boot.img: https://download.invisiblek.org/mata/boot.fix.red.img

That image will reboot over and over again (you’ll never get anywhere)
but when it does, it’ll clear out that annoying red error. After
flashing it, boot normally once. You will still get the red error but it
will be cleared it the next reboot.

Back to Stock
=============

There is a tutorial on xda here:
https://forum.xda-developers.com/essential-phone/development/stock-7-1-1-nmj20d-t3701681

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

It may be desirable for the user to use a DNS server other than Google's. Prior to Android P this is easily done. Invisiblek demonstrates that you can make this change by echo'ing the changes to ``/data/local.prop`` from adb shell. You will need to be root for this.

In this example we will be setting the system dns server too Cloudflare's DNS ``1.1.1.1``, ``1.0.0.1``:

.. code::

    adb root # you must first have root
    adb shell "echo 'net.rmnet_data2.user_dns1=1.1.1.1; >> /data/local.prop"
    adb shell "echo 'net.rmnet_data2.user_dns2=1.0.0.1' >> /data/local.prop"
    adb shell chmod 600 /data/local.prop # make local.prop rw for the current owner
    adb reboot

Source: http://gg.gg/mataDNS
