# LeakTracer for Android NDK

## Original Repository
* [README](https://github.com/fredericgermain/LeakTracer/blob/master/README)
* License (library): LGPLv2.1+
* License (manual and tools): GPLv2+

## Porting to Android
* use NDK API `_Unwind_Backtrace` to catch more accurate backtrace
* automatically caculate symbols offset by retreiving module base ahead (easier for analyze)
* use NDK toolchains to convert leak address to line in source code

## Usage

### Makefile
Use this library as ***static link***. Source compilation or prebuilt as static library.

#### Android.mk
* source compilation
	
```make
LOCAL_C_INCLUDES += $(LOCAL_PATH)/libleaktracer/include/
LOCAL_SRC_FILES += $(LOCAL_PATH)/libleaktracer/src/AllocationHandlers.cpp \
                   $(LOCAL_PATH)/libleaktracer/src/MemoryTrace.cpp
```
    
* prebuild
    
```make
LOCAL_C_INCLUDES := $(LOCAL_PATH)/libleaktracer/include/
LOCAL_SRC_FILES := $(LOCAL_PATH)/libleaktracer/src/AllocationHandlers.cpp \
                   $(LOCAL_PATH)/libleaktracer/src/MemoryTrace.cpp
LOCAL_EXPORT_C_INCLUDES := $(LOCAL_C_INCLUDES)
include $(BUILD_STATIC_LIBRARY)
```

#### Application.mk

change to debug compilation

```make
APP_OPTIM := debug
```

### Native code

```cpp
#include "MemoryTrace.hpp"

void somePlaceToStart()
{
	// starts monitoring memory allocations in all threads
	leaktracer::MemoryTrace::GetInstance().startMonitoringAllThreads();
	// or starts monitoring memory allocations in current thread
	// leaktracer::MemoryTrace::GetInstance().startMonitoringThisThread();
}

void somePlaceToStop()
{
	leaktracer::MemoryTrace::GetInstance().stopAllMonitoring();
	// just make sure you have the very permission (android.permission.WRITE_EXTERNAL_STORAGE)
	// to save leak log on SDCard
	leaktracer::MemoryTrace::GetInstance().writeLeaksToFile("/mnt/sdcard/test.leak");
}

```

### Parsing leak file

* add NDK_ROOT to your environment specifing Android NDK location root (used by leak-analyze-addr2line script)
    
* parsing leak log on ternimal:

```bash
adb pull /mnt/sdcard/test.leak /tmp/
./helpers/leak-analyze-addr2line ~/proj.android/native/obj/local/armeabi/libtest.so /tmp/test.leak
```
    
* if everything's fine, you should see output like this:
    
```
28 bytes lost in 1 blocks (one of them allocated at 25841.648999), from following call stack:
    ~/proj.android/native/./../../src/leaktracer/include/MemoryTrace.hpp:364
    ~/proj.android/native/./../../src/leaktracer/include/MemoryTrace.hpp:414
    ~/proj.android/native/./../../src/leaktracer/src/AllocationHandlers.cpp:29
    /Volumes/Android/buildbot/out_dirs/aosp-ndk-r11-release/build/tmp/build-42939/build-gnustl/static-armeabithumb-4.9/include/bits/basic_string.h:204
    ~/ndk/sources/cxx-stl/gnu-libstdc++/4.9/include/bits/basic_string.tcc:138
    /Volumes/Android/buildbot/out_dirs/aosp-ndk-r11-release/build/tmp/build-42939/build-gnustl/static-armeabithumb-4.9/include/bits/basic_string.h:275
    ~/xlab/framework/platform/android/jni/xxJNIReflection.cpp:68 (discriminator 9)
    ~/xlab/framework/platform/android/jni/xxJNIReflection.cpp:49

23 bytes lost in 1 blocks (one of them allocated at 25841.654000), from following call stack:
    ~/proj.android/native/./../../src/leaktracer/include/MemoryTrace.hpp:364
    ~/proj.android/native/./../../src/leaktracer/include/MemoryTrace.hpp:414
    ~/proj.android/native/./../../src/leaktracer/src/AllocationHandlers.cpp:29
    /Volumes/Android/buildbot/out_dirs/aosp-ndk-r11-release/build/tmp/build-42939/build-gnustl/static-armeabithumb-4.9/include/bits/basic_string.h:204
    ~/ndk/sources/cxx-stl/gnu-libstdc++/4.9/include/bits/basic_string.tcc:138
    /Volumes/Android/buildbot/out_dirs/aosp-ndk-r11-release/build/tmp/build-42939/build-gnustl/static-armeabithumb-4.9/include/bits/basic_string.h:275
    ~/proj.android/native/./../../src/android/VPNController-android.cpp:87 (discriminator 12)
    ~/proj.android/native/./../../src/XXControlCenter.cpp:296
```

## Notice
* leak-analyze-addr2line script use NDK toolchain v4.9 by default. feel free to modify `$toolchains_version` definition
* leak-analyze-addr2line script handle object file with fully debug symbols (normally located at `obj/local/` after each ndk-build)
