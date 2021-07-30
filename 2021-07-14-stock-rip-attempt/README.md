# 2021-07-14 Stock rip attempt

Just unboxed my new OnePlus Nord N200 5G! I have not connected it to cellular
nor Wifi. The goal is to grab a completely stock rom image using fastboot.

## About Phone

About Phone data:

- Device name: OnePlus Nord N200 5G
- Regulartory labels: verification
    - Model DE2117
    - United States
        - FCC ID 2ABZ2-EF000
        - HAC Rating M3/T3
        - This device complies with part 15 of the FCC Rules. Operation is
           subject to the following two conditions:
        - (1) This device may not cause harmful interference, and
        - (2) this device must accept any interference received, including
          interference that may cause undesired operation.
    - Canada
        - IC:12739A-EF000
        - CAN ICES-003(B)/NMB-003(B)
- Android version: 11
    - Android version: 11
    - Android security update: 2021-04-01
    - Google Play system update: February 1, 2021
    - Baseband version: MPSS.HI.4.0.5.c1-00015-MANNAR_GEN_PACK-1.396706.7
    - Kernel version:
        - 5.4.61-gqki-gf149e3f
        - #1 Fri Apr 30 22:43:49 CST 2021
    - Build number: Oxygen OS 11.0.0.0.DE17AA
- Build number: Oxygen OS 11.0.0.0.DE17AA
    -
- Model: DE2117
- Legal information: Privacy policy, agreements, etc.
    - ...
- Status: Phone number, signal, etc.
    - ...
- Award: OxygenOS Contributors
    - ...
- Hardware Version: 10

I've also taken screenshots of the relevant pages. These are
./Screenshot_20210714-*.jpg

## ADB Backup

On second thought, this probably doesn't actually include anything useful.
Regardless, I've taken an adb backup with `adb backup -all`. It's at
`./backup.ab`.

Note: this is excluded from git, as it's a 4MB file and not likely useful.

## Recovery Code

Holding volume down while restarting with it plugged into a computer brought
me to what I believe is OnePlus's recovery. It does not have an adb interface.

I wiped user data, system data and cache. It then let me reboot to fastboot!

## Journey

- Took ./2021-07-14-1626305601-fastboot-rip.log
- Tried to enable OEM Unlocking. But it needs internet access to enable! Guess
  I'm connecting to the internet. NOT applying the OTA update, though.
- Enabled OEM Unlocking in the OS in Developer Settings
- fastboot flashing unlock worked!
- this opened a prompt on the device. follow that, and it'll reboot, erase,
  unlock, and reboot into the OS

Still, there doesn't seem to be any way from fastboot to fetch the images. And
to do it from the OS I would need root.
