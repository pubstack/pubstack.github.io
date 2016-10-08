---
layout: post
title: "How to un-brick a Sony Xperia S and install Oneofakind Android 6.0.1_r10"
author: "Carlos Camacho"
categories:
  - blog
tags:
  - hobbies
  - oneofakind
  - rom
  - flash
  - un-brick
commentIssueId: 17
---

Ok, after playing converting partitions to f2fs I made a huge mistake
and corrupted the partitions table of my mobile.. Yeahp, I messed it up.

![](/static/fist.gif)

The following notes are meant to be a reminder about how to follow this
process as it's a cumbersome and time consuming task 
to get the correct version of the firmware and tools needed.

A hard brick is the state of android device that occurs when your device
won’t boot at all i.e. no boot loop, no recovery and also not charging.

The steps in order to fix the phone will try to flash a new ROM and
upgrade it to Android 6.0.1.

# Prerequisites

* Download all the required files from [here](http://goo.gl/aBYq5w)
  (TWRP, flashtool, flashtool drivers, OEM firmware, root exploit and
  Oneofakind ROM).
* Have a PC near to you.
* A USB cable.
* 1 beer (Save it for the end of the tutorial).


# Part 1

The first part of the tutorial will get you a working
installation of Android 4.1 Jelly Bean in our bricked phone.

## Install Flashtool

Download and install Flashtool, this tool is in the package that
you should have downloaded from the prerequisites.

## Install flashtool drivers for your phone

In order to install the flashtool drivers you need to run some
previous steps as Windows won't let you install them as they are
not signed (Also inside the prerequisites package).

Run the following steps from your PC 

* Press the Windows key + R together and in the 'Run' box type
  `shutdown.exe /r /o /f /t 00`
* Now make the following selections to boot into
  the Start Up Setting Screen:
  Troubleshoot > Advanced options > Start Up Settings > Restart
* Then, when the machine restarts, select number 7 i.e.
  'Disable driver signature enforcement'. Your machine will start
  with Driver signing enforcement disabled until the next reboot.

Now you can install the Flashtool drivers.

* Select from the options to install, flashmode drivers and fastboot driver.

Windows will warn that the driver is not signed
and will require you to confirm the installation.
Once the installation is complete, reboot the machine.


## Start the Xperia S in flashmode

Switch off your Xperia S first, then
press and hold the `Volume Down` button.
Connect to your PC using a USB Cable
while holding down the `Volume Down`
button on your Xperia S. Your phone
should be in Flash Mode now and the
device's LED light should turn into Green.

## Flash the image using flashtool

With your phone in flashmode, open Flashtool
and then click on the lightning bolt.
Select the folder where you have the
firmware image `LT26i_6.2.B.1.96_World.ftf`
(downloaded from the prerequisites package).
Check all the items from wipe list and
uncheck all items from exlude list.
Click flash and wait until finish.

Reboot the phone and you should have
Android 4.1 Jelly Bean installed and ready to use,
but we still want to install Android 6.1,
so the first part of the tutorial is finished.

---

# Part 2

After having a working installation of Android
in our already un-bricked phone,
the next steps allow to upgrade the stantard
ROM to Oneofakind Android 6.0.1_r10.

## Unlock the phone boot loader

Follow the instructions from the official
[Sony website](http://developer.sonymobile.com/unlockbootloader/unlock-yourboot-loader/)

## Gain root access to the phone.

First in your mobile phone. 

*  Activate USB Debugging, Setting -> Developer Options

*  Activate Unknown Sources, Setting -> Security

Now open the application `RunMe.bat` within
Root_with_Restore_by_Bin4ry_v36
and select option 2. Your phone should be rooted and rebooted.

## Installing TWRP

Switch off your Xperia S first.
Press and hold the `Volume Up` button.
Connect to your PC using a USB Cable while
holding down the `Volume Up` button on your Xperia S.
Your Xperia S should be in Fast Mode now and the
device’s LED light should turn into Blue. Then run:

```
fastboot flash boot twrp-3.0.2-0-nozomi.img
```

At this point you should have the recovery installed.

## Re-partitioning

Let's start TWRP. Turn on the phone and the the Sony 
screen appears press the `Volume Up` button.

Go to Mount on TWRP gui (uncheck system, data, cache)
Open the terminal and run:

```
fdisk -l /dev/block/mmcblk0
```

Copy the output of the command to a file with your backup.

Interesting parts are the following lines:

```
/dev/block/mmcblk0p14 42945 261695 7000024 83 Linux
/dev/block/mmcblk0p15 261696 954240 22161424 83 Linux
```

It can be not exactly the same values for you depending
the size of your /data (p14) and /sdcard (p15), run:

```
fdisk /dev/block/mmcblk0
```

The following steps will merge p14 and 15 into one big partition for data,
do this or otherwise you won't be able to use the internal SD.

```
Command (m for help): p

Command (m for help): d
Partition number (1-15): 15

Command (m for help): d
Partition number (1-14): 14

Command (m for help): n
First cylinder (769-954240, default 769): 42945
Last cylinder or +size or +sizeM or +sizeK (42945-954240, default 954240): (just press enter if the default value is the good one) 
Using default value 954240

Command (m for help): t
Partition number (1-14): 14
Hex code (type L to list codes): 83

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table
```

Once re-partitioning done, do NOT do anything
else and just reboot the device (to be sure
that the partition table is take into account by the kernel)


Now we will convert /data and /cache to F2FS.
Ext4 is not supported anymore on nAOSProm. You don't
need to take care about the 16384 byte to reserve
for encryption. TWRP will do it for you.

## Install and run TWRP again

Switch off your Xperia S first.
Press and hold the `Volume Up` button
Connect to your PC using a USB Cable while holding down the `Volume Up` button on your Xperia S
Your phone should be in Fast Mode now and the device’s LED light should turn into Blue, then run

```
fastboot flash boot twrp-3.0.2-0-nozomi.img
```

Let's start TWRP. Turn on the phone and the the Sony 
screen appears press the `Volume Up` button.

Mount all the partitions.

From the menu click:

* Wipe -> Advanced Wipe -> select Data -> Repair or Change File system -> Change File System -> F2FS -> Swipe to Change

* Wipe -> Advanced Wipe -> select Cache -> Repair or Change File system -> Change File System -> F2FS -> Swipe to Change

Once done, again, do NOT do anything else and just reboot the device (required by TWRP).

Congratulation, if everything is fine you should be able to mount /cache and /data and to see a big /data volume around 28 GiB.

Boot the phone again in recovery mode (TWRP).

Start TWRP, mount all partitions and from your PC run:

```
adb push open_gapps-arm-6.0-pico-20161006.zip /sdcard/
adb push oneofakind_nozomi-27-Jan-2016.zip /sdcard/
```
This will load those files into your smartphone allowing
the installation of Oneofakind.

## Final installation of Oneofakind....


From TWRP menu click install and select the two images that 
we just have uploaded (oneofakind_nozomi-27-Jan-2016.zip and open_gapps-arm-6.0-pico-20161006.zip).

Click flash and wait until the installation is complete, in the mean while
open the beer (Last prerequisite) and drink it.

Cheers!


