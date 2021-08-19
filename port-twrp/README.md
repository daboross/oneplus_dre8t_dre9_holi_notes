Trying to port TWRP
===================

## Notes:

TWRP doesn't currently support weird extra partitions. See
https://gerrit.twrp.me/c/android_bootable_recovery/+/3009/1 for an active PR
for this.

The current build uses https://github.com/AOSPA/android_kernel_oneplus_sm8250.
However, the download from
https://github.com/OnePlusOSS/OpenSourceReleases/wiki/OnePlus-Nord-N200 should
include Linux kernel sources as Linux is GPL! So we should be able to compile
our own kernel.

So - some ideas on kernel source - the SM4350 hardware platform is shared
between other phones. Can we get a ROM dump of another one easier?

Idea - Nokia X20. Going to create a new folder for this.

### 2021-08-01:

Following guide at
https://forum.xda-developers.com/t/dev-how-to-compile-twrp-touch-recovery.1943625/

I'm going to start with
https://github.com/minimal-manifest-twrp/platform_manifest_twrp_aosp.

- Grabbing `platform_manifest_twrp_aosp` with `repo init` and `repo sync` in a
  new directory


so - things I'll likely need to set:

```
TW_THEME = "portrait_hdpi"
```

so... the guide has abandoned me. I don't understand what to do next, and it
seemingly requires me to already have a device manifest, or to have already
created it.

Reading
https://forum.xda-developers.com/t/how-to-building-twrp-from-source-with-goodies.4029449/
now.

So I think I need to create an "android_device_oneplus_holi" repo. There don't
seem to be any guides on creating these, and every repo seems to at one point
have been copied from a different one, so I'm going to do exactly that - start
with https://github.com/TeamWin/android_device_oneplus_fajita and create my own
android_device_oneplus_holi.

Let's try removing everything, and seeing how much we can get away with and
still build TWRP. Certainly, none of the vendor files are going to be the same.

So far, creating it has been fairly straightforward, albiet with many variables
left untouched since I don't understand them.

Now I'm making fstab.hardware, and using mount information from adb shell to
figure out where to mount what. .. And after making logs, it seems like
everything except 'op2' matches what's listedn from fajita's fstab.hardware, and
I don't know if anything else makes sense to include.


welp. let's try building it.

hey. it took removing the "include" statements that referred to an existing omni
rom for holi, and removing the "target_2nd" definitions for a non-64bit build,
but it built!

oh.the last step of 'mka adbd recoveryimage' fails with an error about
LOCAL_COPY_HEADERS:

```
$ mka adbd recoveryimage
============================================
PLATFORM_VERSION_CODENAME=REL
PLATFORM_VERSION=16.1.0
TARGET_PRODUCT=aosp_holi
TARGET_BUILD_VARIANT=eng
TARGET_BUILD_TYPE=release
TARGET_ARCH=arm64
TARGET_ARCH_VARIANT=armv8-2a
TARGET_CPU_VARIANT=cortex-a76
HOST_ARCH=x86_64
HOST_2ND_ARCH=x86
HOST_OS=linux
HOST_OS_EXTRA=Linux-5.13.0-0.rc6.45.vanilla.1.fc34.x86_64-x86_64-Fedora-34-(Workstation-Edition)
HOST_CROSS_OS=windows
HOST_CROSS_ARCH=x86
HOST_CROSS_2ND_ARCH=x86_64
HOST_BUILD_TYPE=release
BUILD_ID=RQ1A.210205.004
OUT_DIR=out
============================================
[ 73% 207/282] including device/oneplus/holi/Android.mk ...
FAILED:
device/oneplus/holi/gpt-utils/Android.mk: error: libgptutils: LOCAL_COPY_HEADERS is obsolete. See https://android.googlesource.com/platform/build/+/master/Changes.md#copy_headers
In file included from build/make/core/prebuilt.mk:53:
In file included from device/oneplus/holi/Android.mk:30:
In file included from device/oneplus/holi/gpt-utils/Android.mk:27:
build/make/core/shared_library.mk:60: error: done.
01:22:18 ckati failed with: exit status 1

#### failed to build some targets (36 seconds) ####
```

Applied a makeshift fix from
https://android.googlesource.com/platform/build/soong/+/master/docs/best_practices.md#Headers
and it fixed that error, at least!

Now I get:

```
$ mka adbd recoveryimage
============================================
PLATFORM_VERSION_CODENAME=REL
PLATFORM_VERSION=16.1.0
TARGET_PRODUCT=aosp_holi
TARGET_BUILD_VARIANT=eng
TARGET_BUILD_TYPE=release
TARGET_ARCH=arm64
TARGET_ARCH_VARIANT=armv8-2a
TARGET_CPU_VARIANT=cortex-a76
HOST_ARCH=x86_64
HOST_2ND_ARCH=x86
HOST_OS=linux
HOST_OS_EXTRA=Linux-5.13.0-0.rc6.45.vanilla.1.fc34.x86_64-x86_64-Fedora-34-(Workstation-Edition)
HOST_CROSS_OS=windows
HOST_CROSS_ARCH=x86
HOST_CROSS_2ND_ARCH=x86_64
HOST_BUILD_TYPE=release
BUILD_ID=RQ1A.210205.004
OUT_DIR=out
============================================
[ 99% 281/282] finishing build rules ...
FAILED:
build/make/core/main.mk:670: error:
OverlayHostTests.LOCAL_TARGET_REQUIRED_MODULES : illegal value
OverlayHostTests_NonPlatformSignatureOverlay : not a device module. If you want
to specify host modules to be required to be installed along with your host
module, add those module names to LOCAL_REQUIRED_MODULES instead.
01:29:54 ckati failed with: exit status 1

#### failed to build some targets (34 seconds) ####
```

I'm betting this is because I removed the references to omni_fajita.... since no
omni holi build exists :(

Ah! Now I'm getting somewhere! There are problems in device.mk:

```
$ lunch aosp_holi-eng
In file included from build/make/core/config.mk:291:
In file included from build/make/core/envsetup.mk:266:
In file included from device/oneplus/holi/aosp_holi.mk:41:
device/oneplus/holi/device.mk:27: error: PRODUCT_STATIC_BOOT_CONTROL_HAL is
obsolete. Use shared library module instead. See
https://android.googlesource.com/platform/build/+/master/Changes.md#PRODUCT_STATIC_BOOT_CONTROL_HAL.
01:46:32 dumpvars failed with: exit status 1
WARNING: Trying to fetch a device that's already there
Traceback (most recent call last):
  File
"/home/daboross/proj/lineage/android/twrp/vendor/twrp/build/tools/roomservice.py",
line 431, in <module>
    fetch_device(device)
      File
"/home/daboross/proj/lineage/android/twrp/vendor/twrp/build/tools/roomservice.py",
line 399, in fetch_device
    git_data = search_gerrit_for_device(device)
      File
"/home/daboross/proj/lineage/android/twrp/vendor/twrp/build/tools/roomservice.py",
line 86, in search_gerrit_for_device
    device_data = check_repo_exists(git_data, device)
      File
"/home/daboross/proj/lineage/android/twrp/vendor/twrp/build/tools/roomservice.py",
line 62, in check_repo_exists
    raise Exception("{device} not found,"
    Exception: holi not found,exiting roomservice
    In file included from build/make/core/config.mk:291:
    In file included from build/make/core/envsetup.mk:266:
    In file included from device/oneplus/holi/aosp_holi.mk:41:
    device/oneplus/holi/device.mk:27: error: PRODUCT_STATIC_BOOT_CONTROL_HAL is
obsolete. Use shared library module instead. See
https://android.googlesource.com/platform/build/+/master/Changes.md#PRODUCT_STATIC_BOOT_CONTROL_HAL.
01:46:33 dumpvars failed with: exit status 1

** Don't have a product spec for: 'aosp_holi'
** Do you have the right repo manifest?
```

Going to that link again, managed to fix things... probably. It builds at least
:shrug:.

Did a bunch more troubleshooting.

Now I've pulled in sm8350-common since there is no sm4350-common, and hey maybe
it's close enough? hopefully? *fingers crossed*

But now I need proprietary blobs. So I'm grabbing from a random repo on github!
https://github.com/TheMuppets/proprietary_vendor_oneplus

Cloning that, then copying sm8350-common into vendor/oneplus/.

Oof nope.

```
$ make clobber && lunch aosp_holi-eng && mka adbd recoveryimage
02:13:57 Entire build directory removed.

#### build completed successfully  ####

build/make/core/config.mk:888: error: BOARD_BUILD_SYSTEM_ROOT_IMAGE cannot be
true for devices with dynamic partitions.
02:13:59 dumpvars failed with: exit status 1
WARNING: Trying to fetch a device that's already there
Traceback (most recent call last):
  File
"/home/daboross/proj/lineage/android/twrp/vendor/twrp/build/tools/roomservice.py",
line 431, in <module>
    fetch_device(device)
      File
"/home/daboross/proj/lineage/android/twrp/vendor/twrp/build/tools/roomservice.py",
line 399, in fetch_device
    git_data = search_gerrit_for_device(device)
      File
"/home/daboross/proj/lineage/android/twrp/vendor/twrp/build/tools/roomservice.py",
line 86, in search_gerrit_for_device
    device_data = check_repo_exists(git_data, device)
      File
"/home/daboross/proj/lineage/android/twrp/vendor/twrp/build/tools/roomservice.py",
line 62, in check_repo_exists
    raise Exception("{device} not found,"
    Exception: holi not found,exiting roomservice
    build/make/core/config.mk:888: error: BOARD_BUILD_SYSTEM_ROOT_IMAGE cannot
be true for devices with dynamic partitions.
02:13:59 dumpvars failed with: exit status 1

** Don't have a product spec for: 'aosp_holi'
** Do you have the right repo manifest?
```

Giving up for now.

---

One more idea! It sounded a lot like I'm missing the kernel. Which would make
sense because I removed the line to include the prebuilt image.

Building with the prebuitl image - it all succeeds! But... it does not produce a
recovery.img file. A recovery folder, yes, but no .img per these instructions.

I wonder if there's a difference in what I need to execute, or if it's just
missing? not sure.

In any case there is an additional error which seems useful:


```
[ 99% 282/283] finishing build rules ...
vendor/twrp/build/tasks/kernel.mk:122: warning:
***************************************************************
vendor/twrp/build/tasks/kernel.mk:123: warning: * Using prebuilt kernel binary
instead of source              *
vendor/twrp/build/tasks/kernel.mk:124: warning: * THIS IS DEPRECATED, AND IS NOT
ADVISED.                     *
vendor/twrp/build/tasks/kernel.mk:125: warning: * Please configure your device
to download the kernel         *
vendor/twrp/build/tasks/kernel.mk:126: warning: * source repository to
vendor/twrp/build/tasks/kernel.mk:127: warning: * for more information
                             *
                             vendor/twrp/build/tasks/kernel.mk:128: warning:
***************************************************************
[100% 11642/11642] Install:
out/target/product/holi/recovery/root/system/bin/recovery
```

So, it would seem we aren't supposed to include a prebuilt binary anymore.


I wonder if there are any twrp-11 manifest repos I could steal off of...

OK.

Giving up again now, since it *just works* except there is no recovery.img

I'm going to try with a regular twrp omni-9.0 build next.

2021-08-03:

Got advice! I was doing a few things wrong:

A) `BOARD_USES_RECOVERY_AS_BOOT` was set, which means it generates boot.img
   instead of recovery.img
B) I should use a more up to date repo. Instead of
   https://github.com/TeamWin/android_device_oneplus_fajita, I'm now using
   https://github.com/theincognito-inc/android_device_oneplus_kebab.

Not taking a ton of notes on this new process. just porting changes. will try to
compile in a bit.

Notes:
- `AB_OTA_PARTITIONS` contains a few partitions - vendor, system, system_ext,
vbmeta, which don't seem to be a/b on my device.

Took log "2021-08-03-twrp-build-attempt-2". No success, still, but I might be
getting closer? I'm at least learning more.


2021-08-18:

Lots more unsuccessful attempts! Getting a lot of help from the TWRP zulip.
Critically, the compiler sources have been released, and so I'm working on
getting those compiled

Useful guides:
- https://github.com/nathanchance/android-kernel-clang
- https://forum.xda-developers.com/t/reference-how-to-compile-an-android-kernel.3627297/
