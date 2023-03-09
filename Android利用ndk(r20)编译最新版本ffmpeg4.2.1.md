# Android利用ndk(r20)编译最新版本ffmpeg4.2.1

# 前言

编译**ffmpeg**真是太痛苦了，尤其是网上能搜到的所有同类文章皆告诉我一个道理--不要用最新版本的**NDK**去编译最新版的**FFMPEG**。但作为一个喜新厌旧的程序员，怎么能够忍受用这么旧的版本呢！故，我花了**1.6天**的工作时间成功编译了目前最新版的**ffmpeg**(当前官网为**4.2.1**)，而且用的是最新版的**ndk**（当前为**r20**）

# 准备工作

本来打算用windows编译的，但是看了网上的教程。。。。放弃了，转战Linux。
 准备以下编译环境：

1. VMware Workstation

   ![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/9/19/16d480a91d4d747c~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.awebp)

2. Ubuntu

   （PS：刚整了这虚拟系统的时候有点好奇，各种折腾，搞语言包，搞输入法，搞各种没用的软件，然后成功把系统搞崩了，又删掉重新安装了(*´ﾟ∀ﾟ｀)ﾉ ）

   ![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/9/19/16d480ba40bbd2a2~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.awebp)

3. 下载最新的ffmpeg压缩文件

   （ubuntu系统自带FireFox浏览器）

   ffmpeg4.2.1

   ![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/9/19/16d480fff3b5a63c~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.awebp)

4. 下载最新的ndk

   （ubuntu系统自带FireFox浏览器）

   NDKr20

   ![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/9/19/16d481136f826c8b~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.awebp)

5. 解压下载的两个文件

   /home/junt/Documents/android-ndk-r20

   /home/junt/Documents/ffmpeg-4.2.1

   ![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/9/19/16d4813d16d4bc7a~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.awebp)

# 开始编译

这里前面一部分都是老套路，很多文章里都有，简单带过

### 替换最终生成的so文件名

这里文件的编辑可以用（vim编辑的话需要先安装vim）

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/9/19/16d481c23b571ffa~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.awebp)



```makefile
# 将ffmpeg-4.2.1目录中configure 文件中的：
SLIBNAME_WITH_MAJOR='$(SLIBNAME).$(LIBMAJOR)' 
LIB_INSTALL_EXTRA_CMD='?(RANLIB) "$(LIBDIR)/$(LIBNAME)"' 
SLIB_INSTALL_NAME='$(SLIBNAME_WITH_VERSION)' 
SLIB_INSTALL_LINKS='$(SLIBNAME_WITH_MAJOR) $(SLIBNAME)'

#替换为：
SLIBNAME_WITH_MAJOR='$(SLIBPREF)$(FULLNAME)-$(LIBMAJOR)$(SLIBSUF)'
LIB_INSTALL_EXTRA_CMD='?(RANLIB) "$(LIBDIR)/$(LIBNAME)"'
SLIB_INSTALL_NAME='$(SLIBNAME_WITH_MAJOR)'
SLIB_INSTALL_LINKS='$(SLIBNAME)'
复制代码
```

### 创建编译脚本文件

右击ffmpeg-4.2.1文件夹中的空白处-Open in Terminal

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/9/19/16d481ec3ab609a9~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.awebp)

然后利用**touch**命令创建一个**Shell**脚本文件用来进行编译工作

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/9/19/16d481f440415bae~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.awebp)

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/9/19/16d4820bbd2a23d1~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.awebp)

将以下**shell**代码粘贴到**build_android.sh**文件中



```ini
#!/bin/bash
NDK=/home/junt/Documents/android-ndk-r20
TOOLCHAIN=$NDK/toolchains/llvm/prebuilt/linux-x86_64/
API=29

function build_android
{
echo "Compiling FFmpeg for $CPU"
./configure \
    --prefix=$PREFIX \
    --disable-neon \
    --disable-hwaccels \
    --disable-gpl \
    --disable-postproc \
    --enable-shared \
    --enable-jni \
    --disable-mediacodec \
    --disable-decoder=h264_mediacodec \
    --disable-static \
    --disable-doc \
    --disable-ffmpeg \
    --disable-ffplay \
    --disable-ffprobe \
    --disable-avdevice \
    --disable-doc \
    --disable-symver \
    --cross-prefix=$CROSS_PREFIX \
    --target-os=android \
    --arch=$ARCH \
    --cpu=$CPU \
    --cc=$CC
    --cxx=$CXX
    --enable-cross-compile \
    --sysroot=$SYSROOT \
    --extra-cflags="-Os -fpic $OPTIMIZE_CFLAGS" \
    --extra-ldflags="$ADDI_LDFLAGS" \
    $ADDITIONAL_CONFIGURE_FLAG
make clean
make
make install
echo "The Compilation of FFmpeg for $CPU is completed"
}

#armv8-a
ARCH=arm64
CPU=armv8-a
CC=$TOOLCHAIN/bin/aarch64-linux-android$API-clang
CXX=$TOOLCHAIN/bin/aarch64-linux-android$API-clang++
SYSROOT=$NDK/toolchains/llvm/prebuilt/linux-x86_64/sysroot
CROSS_PREFIX=$TOOLCHAIN/bin/aarch64-linux-android-
PREFIX=$(pwd)/android/$CPU
OPTIMIZE_CFLAGS="-march=$CPU"
build_android

#armv7-a
ARCH=arm
CPU=armv7-a
CC=$TOOLCHAIN/bin/armv7a-linux-androideabi$API-clang
CXX=$TOOLCHAIN/bin/armv7a-linux-androideabi$API-clang++
SYSROOT=$NDK/toolchains/llvm/prebuilt/linux-x86_64/sysroot
CROSS_PREFIX=$TOOLCHAIN/bin/arm-linux-androideabi-
PREFIX=$(pwd)/android/$CPU
OPTIMIZE_CFLAGS="-mfloat-abi=softfp -mfpu=vfp -marm -march=$CPU "
build_android

#x86
ARCH=x86
CPU=x86
CC=$TOOLCHAIN/bin/i686-linux-android$API-clang
CXX=$TOOLCHAIN/bin/i686-linux-android$API-clang++
SYSROOT=$NDK/toolchains/llvm/prebuilt/linux-x86_64/sysroot
CROSS_PREFIX=$TOOLCHAIN/bin/i686-linux-android-
PREFIX=$(pwd)/android/$CPU
OPTIMIZE_CFLAGS="-march=i686 -mtune=intel -mssse3 -mfpmath=sse -m32"
build_android

#x86_64
ARCH=x86_64
CPU=x86-64
CC=$TOOLCHAIN/bin/x86_64-linux-android$API-clang
CXX=$TOOLCHAIN/bin/x86_64-linux-android$API-clang++
SYSROOT=$NDK/toolchains/llvm/prebuilt/linux-x86_64/sysroot
CROSS_PREFIX=$TOOLCHAIN/bin/x86_64-linux-android-
PREFIX=$(pwd)/android/$CPU
OPTIMIZE_CFLAGS="-march=$CPU -msse4.2 -mpopcnt -m64 -mtune=intel"
build_android
复制代码
```

### 编译

在ffmpeg-4.2.1文件夹中打开终端

1. 赋予文件读写权限

   ![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/9/19/16d48232ec9dac8e~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.awebp)

2. 执行编译文件

   ![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/9/19/16d48244906eb5f0~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.awebp)

### 编译完成查看文件

在/ffmpeg-4.2.1/android/目录下



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/9/19/16d4825937f79bc0~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.awebp)





![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/9/19/16d4825da2f4f90f~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.awebp)





![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/9/19/16d482855b8ff633~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.awebp)



# 对于build_android.sh文件的说明

为什么新版本的ffmpeg搭配新版本的ndk编译很容易出错呢？其实关键点主要还是新旧版本ndk中的交叉编译工具不一样导致的。比如旧版本（r17及之前）的ndk中的编译器用的是gcc，而网上大部分的同类文章中用的也是gcc，而新版本的ndk文件已经弃用gcc编译器改用clang了，所以照着网上的文章做当然编译不起来）

```ini
#!/bin/bash
NDK=/home/junt/Documents/android-ndk-r20
TOOLCHAIN=$NDK/toolchains/llvm/prebuilt/linux-x86_64/
#这里修改的是最低支持的android sdk版本（r20版本ndk中armv8a、x86_64最低支持21，armv7a、x86最低支持16）
API=29

function build_android
{
#相当于Android中Log.i
echo "Compiling FFmpeg for $CPU"
#调用同级目录下的configure文件
./configure \
#指定输出目录
    --prefix=$PREFIX \
#各种配置项，想详细了解的可以打开configure文件找到Help options:查看
    --disable-neon \
    --disable-hwaccels \
    --disable-gpl \
    --disable-postproc \
#配置跨平台编译，同时需要disable-static
    --enable-shared \
    --enable-jni \
    --disable-mediacodec \
    --disable-decoder=h264_mediacodec \
#配置跨平台编译，同时需enable-shared   
    --disable-static \
    --disable-doc \
    --disable-ffmpeg \
    --disable-ffplay \
    --disable-ffprobe \
    --disable-avdevice \
    --disable-doc \
    --disable-symver \
#关键点1.指定交叉编译工具目录
    --cross-prefix=$CROSS_PREFIX \
#关键点2.指定目标平台为android
    --target-os=android \
#关键点3.指定cpu类型
    --arch=$ARCH \
#关键点4.指定cpu架构
    --cpu=$CPU \
#超级关键点5.指定c语言编译器
    --cc=$CC
    --cxx=$CXX
#关键点6.开启交叉编译
    --enable-cross-compile \
#超级关键7.配置编译环境c语言的头文件环境
    --sysroot=$SYSROOT \
    --extra-cflags="-Os -fpic $OPTIMIZE_CFLAGS" \
    --extra-ldflags="$ADDI_LDFLAGS" \
    $ADDITIONAL_CONFIGURE_FLAG
make clean
make
make install
echo "The Compilation of FFmpeg for $CPU is completed"
}

#armv8-a
ARCH=arm64
CPU=armv8-a
#r20版本的ndk中所有的编译器都在/android-ndk-r20/toolchains/llvm/prebuilt/linux-x86_64/目录下（clang）
CC=$TOOLCHAIN/bin/aarch64-linux-android$API-clang
CXX=$TOOLCHAIN/bin/aarch64-linux-android$API-clang++
#头文件环境用的不是/android-ndk-r20/sysroot,而是编译器//android-ndk-r20/toolchains/llvm/prebuilt/linux-x86_64/sysroot
SYSROOT=$NDK/toolchains/llvm/prebuilt/linux-x86_64/sysroot
#交叉编译工具目录,对应关系如下(不明白的可以看下图)
# armv8a -> arm64 -> aarch64-linux-android-
# armv7a -> arm -> arm-linux-androideabi-
# x86 -> x86 -> i686-linux-android-
# x86_64 -> x86_64 -> x86_64-linux-android-
CROSS_PREFIX=$TOOLCHAIN/bin/aarch64-linux-android-
#输出目录
PREFIX=$(pwd)/android/$CPU
OPTIMIZE_CFLAGS="-march=$CPU"
#方法调用
build_android
复制代码
```



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/9/19/16d48405ea1f4203~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.awebp)



# 其他

其他的可能碰到的错误，比如无法创建文件，目录不存在等等大多都是权限以及系统缺少相关库导致的，百度一下大多都能解决。
 提示：这里编译出来的so文件是不能直接用的，还需要在AndroidStudio构建native方法，重新编译。

作者：xiaojigugu
链接：https://juejin.cn/post/6844903945496690696
来源：稀土掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。