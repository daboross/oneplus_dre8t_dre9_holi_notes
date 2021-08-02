Trying to port TWRP
===================


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
