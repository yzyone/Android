# ffmpeg for android编译全过程与遇到的问题

## 编译前准备

编译环境：Ubuntu16，可自行下载VMWare最新版并百度永久许可证或在服务器上安装Ubuntu

ffmpeg源码：[ffmpeg4.2.2](http://ffmpeg.org/)

NDK下载：[Android NDK r21e](https://developer.android.google.cn/ndk/downloads/index.html)

有条件的最好还是在Liunx平台下编译吧，Windows平台下编译坑更多，文章末尾有Github源码可自取

## 开始编译

1.解压NDK，执行 unzip android-ndk-r21e-liunx-x86_64.zip

如果提示没有unzip，执行此命令安装 sudo apt-get install unzip

2.解压ffmepg，tar -xvjf ffmpeg-4.2.2.tar.bz2

3.进入ffmpeg4.2.2目录，修改根目录下的 configure 文件

  搜索 LIB_INSTALL_EXTRA_CMD

```
SLIBNAME_WITH_MAJOR='$(SLIBNAME).$(LIBMAJOR)'
LIB_INSTALL_EXTRA_CMD='$$(RANLIB) "$(LIBDIR)/$(LIBNAME)"'
SLIB_INSTALL_NAME='$(SLIBNAME_WITH_VERSION)'
SLIB_INSTALL_LINKS='$(SLIBNAME_WITH_MAJOR) $(SLIBNAME)'
```

替换为

```
SLIBNAME_WITH_MAJOR='$(SLIBPREF)$(FULLNAME)-$(LIBMAJOR)$(SLIBSUF)'
LIB_INSTALL_EXTRA_CMD='$$(RANLIB)"$(LIBDIR)/$(LIBNAME)"'
SLIB_INSTALL_NAME='$(SLIBNAME_WITH_MAJOR)'
SLIB_INSTALL_LINKS='$(SLIBNAME)'
```

 此步骤主要是将so命名为Android通用的so名称

4.在ffmpeg下创建文件android_build.sh脚本文件，此脚本文件用于配置、执行编译，根据需求进行配置，网上的配置均有不同，以实际需要为准，将以下代码copy到android_build.sh脚本文件中，执行 sudo sh android_build.sh

```
#!/bin/bash
API=21
#arm64  x86 x86_64 <----> aarch64  i686  x86_64
ARCH=arm64
ARCH2=aarch64    
PREFIX=../out-ffmpeg/$ARCH
#此处路径替换为当前NDK路径
TOOLCHAIN=/home/jiang/ffmpeg/android-ndk-r21e-linux-x86_64/android-ndk-r21e/toolchains/llvm/prebuilt/linux-x86_64

build()
{
    #配置各个文件开关及NDK路径等
    ./configure \
    --prefix=$PREFIX \
    --disable-static \
    --enable-shared \
    --enable-small \
    --enable-gpl \
    --disable-doc \
    --disable-programs \
    --disable-avdevice \
    --enable-cross-compile \
    --target-os=android \
    --arch=$ARCH \
    --cc=$TOOLCHAIN/bin/$ARCH2-linux-android$API-clang \
    --cross-prefix=$TOOLCHAIN/bin/$ARCH2-linux-android-

    #清除上次编译
    make clean
    make -j4
    make install
}
#开始编译
build
```

[android_build.sh脚本文件自取](https://github.com/jhflovehqy/NewFFmpeg)

注：如果编译过程中出现错误（一般是在开头会有红色报错），部分需要安装其他库，具体可查阅

5.编译后会在ffmpeg4.2.2同级目录下生成out-ffmpeg文件，将out-ffmpeg导出到项目中

## Android Studio配置

**1.新建一个C++项目**

- 将编译完成后的include头文件导入到cpp文件中
- 将编译完成后的lib库文件导入到libs中

**2.配置build.gradle文件**

defaultConfig下增加

        externalNativeBuild {
            cmake {
                cppFlags '-frtti -fexceptions'
                abiFilters "arm64-v8a","armeabi-v7a"
            }
        }
abiFilters是指定当前项目所支持的CPU架构，一般来说有arm64-v8a（arm64位）、armeabi-v7a(arm32位)足够，大部分手机都是这两种架构之一，要完全兼容可能会导致APP体积增大

注意：如果你的Gradler版本足够高（大约>5.6.4），无须配置以下项，否则有可能报错

```
sourceSets {
     main {
         jniLibs.srcDirs = ["libs"]
     }
} 
```

**3.配置CMake**

由于android早已支持CMake，所以旧的android.mk配置此处不增加

```
#声明cmake版本号
cmake_minimum_required(VERSION 3.10.2)

#此处导入头文件目录
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include)

#将so库文件路径赋值ffmpeg_lib_dir，方便操作
set(ffmpeg_lib_dir ${CMAKE_CURRENT_SOURCE_DIR}/../../../libs/${ANDROID_ABI})

#项目名称
project("newffmpeg")


add_library( newffmpeg
        SHARED
        native-lib.cpp)


#初始化log库
find_library( log-lib
        log)

#江指定的源文件生成链接文件
add_library( avutil
        SHARED
        IMPORTED )

set_target_properties( avutil
        PROPERTIES IMPORTED_LOCATION
        ${ffmpeg_lib_dir}/libavutil.so )

add_library( swresample
        SHARED
        IMPORTED )
set_target_properties( swresample
        PROPERTIES IMPORTED_LOCATION
        ${ffmpeg_lib_dir}/libswresample.so )

add_library( avcodec
        SHARED
        IMPORTED )
set_target_properties( avcodec
        PROPERTIES IMPORTED_LOCATION
        ${ffmpeg_lib_dir}/libavcodec.so )

add_library( avfilter
        SHARED
        IMPORTED)
set_target_properties( avfilter
        PROPERTIES IMPORTED_LOCATION
        ${ffmpeg_lib_dir}/libavfilter.so )

add_library( swscale
        SHARED
        IMPORTED)
set_target_properties( swscale
        PROPERTIES IMPORTED_LOCATION
        ${ffmpeg_lib_dir}/libswscale.so )

add_library( avformat
        SHARED
        IMPORTED)
set_target_properties( avformat
        PROPERTIES IMPORTED_LOCATION
        ${ffmpeg_lib_dir}/libavformat.so )

add_library( postproc
        SHARED
        IMPORTED)
set_target_properties( postproc
        PROPERTIES IMPORTED_LOCATION
        ${ffmpeg_lib_dir}/libpostproc.so )

#将目标文件与库文件进行链接
target_link_libraries( # Specifies the target library.
        newffmpeg
        
        avutil
        swresample
        avcodec
        avfilter
        swscale
        avformat
        postproc
     
        ${log-lib}
        )
```

若未能链接到库文件，则检查路径是否正常（点击libs路径左侧菜单能正常展开说明路径正确） 

**4.测试ffmpeg**

在native-lib.cpp中增加或替换代码，注意JNI路径替换为你的包名路径或方法，在Java中调用，如能正常打印出配置信息，说明编译及导入完成

```
#include <jni.h>
#include <string>
#include <unistd.h>

extern "C" {
#include <libavcodec/avcodec.h>
#include <libavformat/avformat.h>
#include <libavfilter/avfilter.h>
#include <libavcodec/jni.h>

JNIEXPORT jstring JNICALL
Java_cn_jin_newffmpeg_MainActivity_getConfigurations(JNIEnv *env,
                                                   jobject  /* this */) {
    return env->NewStringUTF(avcodec_configuration());
}
}
```

**遇到的问题**

2 files found with path 'lib/arm64-v8a/libavcodec.so' from inputs:

```
2 files found with path 'lib/arm64-v8a/libavcodec.so' from inputs:

 - D:\ffmpeg\project\NewFFmpeg\app\build\intermediates\merged_jni_libs\debug\out\arm64-v8a\libavcodec.so
 - D:\ffmpeg\project\NewFFmpeg\app\build\intermediates\cxx\Debug\2xk41543\obj\arm64-v8a\libavcodec.so
If you are using jniLibs and CMake IMPORTED targets, see
https://developer.android.com/r/tools/jniLibs-vs-imported-targets
```

 解决办法：此处是由于在build.gradle中配置了jniLibs.srcDirs导致的文件冲突，gradle高版本已经不需要手动指定so库的路径，删除即可


D:/ffmpeg/project/FFmpegProject/app/src/main/cpp/../../main/jniLibs/arm64-v8a/libavcodec.so: error adding symbols: File in wrong format
clang++: error: linker command failed with exit code 1 (use -v to see invocation)

```
[1/1] Linking CXX shared library D:\ffmpeg\project\FFmpegProject\app\build\intermediates\cxx\Debug\2c676z6h\obj\arm64-v8a\libffmpeg.so
FAILED: D:/ffmpeg/project/FFmpegProject/app/build/intermediates/cxx/Debug/2c676z6h/obj/arm64-v8a/libffmpeg.so 
cmd.exe /C "cd . && C:\Users\as230\AppData\Local\Android\Sdk\ndk\21.4.7075529\toolchains\llvm\prebuilt\windows-x86_64\bin\clang++.exe --target=aarch64-none-linux-android21 --gcc-toolchain=C:/Users/as230/AppData/Local/Android/Sdk/ndk/21.4.7075529/toolchains/llvm/prebuilt/windows-x86_64 --sysroot=C:/Users/as230/AppData/Local/Android/Sdk/ndk/21.4.7075529/toolchains/llvm/prebuilt/windows-x86_64/sysroot -fPIC -g -DANDROID -fdata-sections -ffunction-sections -funwind-tables -fstack-protector-strong -no-canonical-prefixes -D_FORTIFY_SOURCE=2 -Wformat -Werror=format-security  -std=c++11 -frtti -fexceptions -O0 -fno-limit-debug-info  -Wl,--exclude-libs,libgcc.a -Wl,--exclude-libs,libgcc_real.a -Wl,--exclude-libs,libatomic.a -static-libstdc++ -Wl,--build-id -Wl,--fatal-warnings -Wl,--no-undefined -Qunused-arguments -shared -Wl,-soname,libffmpeg.so -o D:\ffmpeg\project\FFmpegProject\app\build\intermediates\cxx\Debug\2c676z6h\obj\arm64-v8a\libffmpeg.so CMakeFiles/ffmpeg.dir/native-lib.cpp.o  -llog D:/ffmpeg/project/FFmpegProject/app/src/main/cpp/../../main/jniLibs/arm64-v8a/libavcodec.so D:/ffmpeg/project/FFmpegProject/app/src/main/cpp/../../main/jniLibs/arm64-v8a/libavfilter.so D:/ffmpeg/project/FFmpegProject/app/src/main/cpp/../../main/jniLibs/arm64-v8a/libavformat.so D:/ffmpeg/project/FFmpegProject/app/src/main/cpp/../../main/jniLibs/arm64-v8a/libavutil.so D:/ffmpeg/project/FFmpegProject/app/src/main/cpp/../../main/jniLibs/arm64-v8a/libpostproc.so D:/ffmpeg/project/FFmpegProject/app/src/main/cpp/../../main/jniLibs/arm64-v8a/libswresample.so D:/ffmpeg/project/FFmpegProject/app/src/main/cpp/../../main/jniLibs/arm64-v8a/libswscale.so -latomic -lm && cd ."
D:/ffmpeg/project/FFmpegProject/app/src/main/cpp/../../main/jniLibs/arm64-v8a/libavcodec.so: error adding symbols: File in wrong format
clang++: error: linker command failed with exit code 1 (use -v to see invocation)
ninja: build stopped: subcommand failed.
```

出现动态库中的问题导致链接器命令失败大概率是两个原因

- 编译出来的so有问题，此时不要尝试，可下载其他已经编译好的so进行替换，如其他可正常运行则已经说明

- so库中依赖其他库文件，其他库文件未导入或者已导入未进行链接，此时应该检查一下cmake

java.lang.UnsatisfiedLinkError: dlopen failed: library “libxxx.so” not found 

此处错误是在运行之后出现的，原因是某些已经使用的库文件没有链接上，应该检查一下libs和cmkae，基本上能解决 

**源码**

[ffmpeg for android源码](https://github.com/jhflovehqy/NewFFmpeg)

————————————————

版权声明：本文为CSDN博主「家驹六月天」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。

原文链接：https://blog.csdn.net/lalallallalla/article/details/121470465