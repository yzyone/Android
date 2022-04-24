# mac 环境编译FFmpeg

本篇文章主要来源自[https://blog.csdn.net/qqqq245425070/article/details/90041475](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fqqqq245425070%2Farticle%2Fdetails%2F90041475)，修改了文章中一些错误，并附上自己遇到的一些问题的解决方案

# 编译流程

step1:下载FFmpeg库和NDK库  
（1）我在usr目录下建立了一个ndk文件夹  
（2）然后进入ndk文件夹，FFmpeg官网找到下载地址，再在命令行输入 “wget [https://www.ffmpeg.org/releases/ffmpeg-4.0.2.tar.gz”](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.ffmpeg.org%2Freleases%2Fffmpeg-4.0.2.tar.gz%25E2%2580%259D)  
（3）解压文件夹 “tar -xzvf ffmpeg-4.0.2.tar.gz”  
（4）下载NDK，输入命  
[https://blog.csdn.net/momo0853/article/details/73898066](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fmomo0853%2Farticle%2Fdetails%2F73898066)

（5）解压ndk包 “unzip android-ndk-r14b-linux-x86_64.zip”，如果没有安装unzip，系统会提示，按照系统提示安装就好了，最后会解压生成文件夹android-ndk-r14b  
step2：NDK安装  
将android-ndk-r14b加入环境变量

step3：配置解压后的文件夹ffmpeg-4.0.2中的configure，更改下方内容：  
修改前：

```bash
SLIBNAME_WITH_MAJOR='$(SLIBNAME).$(LIBMAJOR)'
LIB_INSTALL_EXTRA_CMD='$$(RANLIB) "$(LIBDIR)/$(LIBNAME)"'
SLIB_INSTALL_NAME='$(SLIBNAME_WITH_VERSION)'
SLIB_INSTALL_LINKS='$(SLIBNAME_WITH_MAJOR) $(SLIBNAME)'
```

修改后：

```bash
SLIBNAME_WITH_MAJOR='$(SLIBPREF)$(FULLNAME)-$(LIBMAJOR)$(SLIBSUF)'
LIB_INSTALL_EXTRA_CMD='$$(RANLIB) "$(LIBDIR)/$(LIBNAME)"'
SLIB_INSTALL_NAME='$(SLIBNAME_WITH_MAJOR)'
SLIB_INSTALL_LINKS='$(SLIBNAME)'
```

这一步的主要目的是生成Android能够使用的 名称-版本.so文件的格式，不然的话生成的是Linux上使用库，Android不能用。  
step3：编写Android编译的脚本  
在解压后的文件夹ffmpeg-4.0.2中，新建build_android.sh文件（使用的shell命令“touch build_android.sh”）  
输入命令“vim build_android.sh”，点击Enter打开文件，在文件中粘贴下面内容，(注意修改NDK的地址)：

```bash
#!/bin/bash
export NDK_HOME=/Users/zamp/Documents/ffmpeg/android-ndk-r14b
export PLATFORM_VERSION=android-9
function build
{
    echo "start build ffmpeg for $ARCH"
    ./configure --target-os=linux \
    --prefix=$PREFIX --arch=$ARCH \
    --disable-doc \
    --enable-shared \
    --disable-static \
    --disable-yasm \
    --disable-asm \
    --disable-symver \
    --enable-gpl \
    --disable-ffmpeg \
    --disable-ffplay \
    --disable-ffprobe \
    --disable-ffserver \
    --cross-prefix=$CROSS_COMPILE \
    --enable-cross-compile \
    --sysroot=$SYSROOT \
    --enable-small \
  --disable-linux-perf \
    --extra-cflags="-Os -fpic $ADDI_CFLAGS" \
    --extra-ldflags="$ADDI_LDFLAGS" \
    $ADDITIONAL_CONFIGURE_FLAG
    make clean
    make
    make install
    echo "build ffmpeg for $ARCH finished"
}

#arm
ARCH=arm
CPU=arm
PREFIX=$(pwd)/android/$ARCH
TOOLCHAIN=$NDK_HOME/toolchains/arm-linux-androideabi-4.9/prebuilt/darwin-x86_64
CROSS_COMPILE=$TOOLCHAIN/bin/arm-linux-androideabi-
ADDI_CFLAGS="-marm"
SYSROOT=$NDK_HOME/platforms/$PLATFORM_VERSION/arch-$ARCH/
build

# #x86
# ARCH=x86
# CPU=x86
# PREFIX=$(pwd)/android/$ARCH
# TOOLCHAIN=$NDK_HOME/toolchains/x86-4.9/prebuilt/darwin-x86_64
# CROSS_COMPILE=$TOOLCHAIN/bin/i686-linux-android-
# ADDI_CFLAGS="-march=i686 -mtune=intel -mssse3 -mfpmath=sse -m32"
# SYSROOT=$NDK_HOME/platforms/$PLATFORM_VERSION/arch-$ARCH/
# build
```

–disable-ffmpeg意为禁用ffmpeg工具，编译时不编译出ffmpeg工具，–enable-ffmpeg为启用，但是configure文件配置有个特点，FFmpeg的默认的配置不是以show_help方法中的配置配置的，而是以前缀disable or enable取反配置的，也就是FFmpeg中各属性默认的配置把show_help中各个配置的前缀取反即可。  
step4：编译ffmpeg  
1）给ndk里的所有文件设置权限 chmod -R 777 ndk  
2）执行 ./build_android.sh

编译成功后文件会在/usr/ndk/ffmpeg-4.0.2/android路径目录下。实际上这个脚本执行完，会编译出现多个.so文件,我们使用的取/usr/ndk/ffmpeg-4.0.2/android/arm/lib中的大版本号的，就是名字里面有数字的。

# 减少包体积

如果你的APP比较在意包的大小，在编译时，我们可以针对自己需要的功能来进行配置，更改bash脚本（也就是我们创建的build_android.sh文件），加入配置：

–disable-everything

会把下面的组件不加入编译：

```bash
Individual component options:--disable-everything     
disable all components listed below--disable-encoder=NAME  
 disable encoder NAME
--enable-encoder=NAME  
 enable encoder NAME
--disable-encoders       
disable all encoders--disable-decoder=NAME   
disable decoder NAME
--enable-decoder=NAME   
enable decoder NAME--disable-decoders       
disable all decoders--disable-hwaccel=NAME   
disable hwaccel NAME--enable-hwaccel=NAME   
enable hwaccel NAME--disable-hwaccels       
disable all hwaccels--disable-muxer=NAME     
disable muxer NAME--enable-muxer=NAME     
enable muxer NAME--disable-muxers         
disable all muxers--disable-demuxer=NAME   
disable demuxer NAME--enable-demuxer=NAME   
enable demuxer NAME--disable-demuxers      
disable all demuxers--enable-parser=NAME     
enable parser NAME--disable-parser=NAME   
disable parser NAME--disable-parsers       
disable all parsers--enable-bsf=NAME       
enable bitstream filter NAME--disable-bsf=NAME       
disable bitstream filter NAME--disable-bsfs           
disable all bitstream filters--enable-protocol=NAME   
enable protocol NAME--disable-protocol=NAME 
disable protocol NAME--disable-protocols     
disable all protocols--enable-indev=NAME     
enable input device NAME--disable-indev=NAME     
disable input device NAME--disable-indevs         
disable input devices--enable-outdev=NAME     
enable output device NAME--disable-outdev=NAME   
disable output device NAME--disable-outdevs       
disable output devices--disable-devices       
disable all devices--enable-filter=NAME     
enable filter NAME--disable-filter=NAME   
disable filter NAME--disable-filters       
disable all filters
```

如果只加入–disable-everything那你编译出来的东西，里面几乎什么都没有了，所以你要在build_android.sh加入要编译的组件，比如：

```bash
#!/bin/bash
export TMPDIR=$(pwd)/ffmpegtemp #这句很重要，不然会报错 unable to create temporary file in
# NDK的路径，根据自己的安装位置进行设置
NDK=/softdata/android-sdk-macosx/android-ndk-r16b
# 编译针对的平台，可以根据自己的需求进行设置
# 这里选择最低支持android-14, arm架构，生成的so库是放在
# libs/armeabi文件夹下的，若针对x86架构，要选择arch-x86
PLATFORM=$NDK/platforms/android-14/arch-arm
# 工具链的路径，根据编译的平台不同而不同
# arm-linux-androideabi-4.9与上面设置的PLATFORM对应，4.9为工具的版本号，
# 根据自己安装的NDK版本来确定，一般使用最新的版本
TOOLCHAIN=$NDK/toolchains/arm-linux-androideabi-4.9/prebuilt/darwin-x86_64

function build_one
{
./configure \
    --prefix=$PREFIX \
    --target-os=linux \
    --cross-prefix=$TOOLCHAIN/bin/arm-linux-androideabi- \
    --arch=arm \
    --sysroot=$PLATFORM \
    --extra-cflags="-I$PLATFORM/usr/include" \
    --cc=$TOOLCHAIN/bin/arm-linux-androideabi-gcc \
    --nm=$TOOLCHAIN/bin/arm-linux-androideabi-nm \
    --enable-shared \
    --enable-runtime-cpudetect \
    --enable-gpl \
    --enable-small \
    --enable-cross-compile \
    --disable-debug \
    --disable-static \
    --disable-doc \
    --disable-asm \
    --disable-ffmpeg \
    --disable-ffplay \
    --disable-ffprobe \
    --disable-ffserver \
    --disable-postproc \
    --disable-avdevice \
    --disable-symver \
    --disable-stripping \

$ADDITIONAL_CONFIGURE_FLAG
sed -i '' 's/HAVE_LRINT 0/HAVE_LRINT 1/g' config.h
sed -i '' 's/HAVE_LRINTF 0/HAVE_LRINTF 1/g' config.h
sed -i '' 's/HAVE_ROUND 0/HAVE_ROUND 1/g' config.h
sed -i '' 's/HAVE_ROUNDF 0/HAVE_ROUNDF 1/g' config.h
sed -i '' 's/HAVE_TRUNC 0/HAVE_TRUNC 1/g' config.h
sed -i '' 's/HAVE_TRUNCF 0/HAVE_TRUNCF 1/g' config.h
sed -i '' 's/HAVE_CBRT 0/HAVE_CBRT 1/g' config.h
sed -i '' 's/HAVE_RINT 0/HAVE_RINT 1/g' config.h

make clean
make -j4
make install

}

# arm v7vfp
CPU=armv7-a
OPTIMIZE_CFLAGS="-mfloat-abi=softfp -mfpu=vfp -marm -march=$CPU "
PREFIX=./android/$CPU-vfp
ADDITIONAL_CONFIGURE_FLAG=
build_one
```

# 遇到的坑

如果在执行build_android.sh时，如果提示没有权限“Permission Denied”，这是因为这个文件还没有被标记为可执行文件，先执行chmod a+x build_android.sh，再执行./build_android.sh就可以了。

(一)  
上面演示的build_android.sh 脚本中使用的ndk版本是r15c,那在FFmpeg 4.1版本中,你应该会遇到这个error:

libavformat/udp.c: In function ‘udp_set_multicast_sources’:  
libavformat/udp.c:290:28: error: request for member ‘s_addr’ in something not a structure or union  
网上很少关于这个错误的描述,官方的回复也没看出来啥子有用的价值.

[https://trac.ffmpeg.org/ticket/7741](https://links.jianshu.com/go?to=https%3A%2F%2Ftrac.ffmpeg.org%2Fticket%2F7741)

解决方法:  
有两种解决方案  
1.ndk版本升到r17c  
2.如果不想升ndk版本的,那就修改libavformat/udp.c 文件,把报错的相关代码注释掉就好.前提是你的项目中用不到这块功能.  
在低版本的ndk应该都会有这个问题,我有试过r9b版本的,也有这个问题.

(二)  
如果使用ndk版本是r17c的话,那上面的问题就不会遇到,但是会遇到新的问题.

CC libavdevice/alldevices.o  
In file included from ./libavformat/internal.h:24:0,  
from libavdevice/alldevices.c:23:  
/opt/andorid_sdk/adt-bundle-linux-x86-20131030/android-ndk-r17c/toolchains/arm-linux-androideabi-4.9/prebuilt/linux-x86_64/lib/gcc/arm-linux-androideabi/4.9.x/include/stdint.h:9:26: fatal error: stdint.h: No such file or directory  
出现这个错误是因为NDK r17c版本将头文件和库文件进行了分离，我们指定的sysroot文件夹下只有库文件，而头文件放在了NDK目录下的sysroot内.  
解决方法: 需在–extra-cflags中添加 “-isysroot $NDK/sysroot”

还有有关汇编的头文件也进行了分离.  
解决方法:根据目标平台进行指定 “-I$NDK/sysroot/usr/include/arm-linux-androideabi”，将 “arm-linux-androideabi” 改为需要的平台就可以

修改之后的build_android.sh :

```
# !/bin/bash

set -x  
API=14  
NDK=/opt/andorid_sdk/adt-bundle-linux-x86-20131030/android-ndk-r17c  
SYSROOT=N D K / p l a t f o r m s / a n d r o i d − NDK/platforms/android-NDK/platforms/android−API/arch-arm/  
TOOLCHAIN=KaTeX parse error: Expected '}', got '\ ' at position 100: … { ./configure \̲ ̲ --prefix=PREFIX  
–disable-shared  
–enable-static  
–disable-doc  
–disable-ffplay  
–disable-ffprobe  
–disable-symver  
–disable-ffmpeg  
–cc=KaTeX parse error: Expected 'EOF', got '\ ' at position 41: …ndroideabi-gcc \̲ ̲ --cross-prefi…TOOLCHAIN/bin/arm-linux-androideabi-  
–target-os=linux  
–arch=arm  
–enable-cross-compile  
–sysroot=KaTeX parse error: Expected 'EOF', got '\ ' at position 9: SYSROOT \̲ ̲ --extra-cflag…NDK/sysroot/usr/include/arm-linux-androideabi -isysroot $NDK/sysroot -fPIC -DANDROID -D__thumb__ -mthumb -Wfatal-errors -Wno-deprecated -mfloat-abi=softfp -marm -march=armv7-a"  
–enable-neon

make clean all  
make  
make install  
}  
CPU=armv7-a  
PREFIX=( p w d ) / a n d r o i d / (pwd)/android/(pwd)/android/CPU  
function_one  
```



(三)

```
libavcodec/aaccoder.c: In function ‘search_for_ms’:  
libavcodec/aaccoder.c:803:25: error: expected identifier or ‘(’ before numeric constant  
int B0 = 0, B1 = 0;  
^  
```

这是由于定义冲突导致的一个error,和ndk  
版本有关,我在使用r15c和r9b版本的时候都没有遇到这个问题.  
解决方法:修改libavcodec/aaccoder.c 文件 B0改成b0(ps:就是把int型变量名改一下,避免冲突,名字随便起)

(四)

```
libavcodec/hevc_mvs.c: In function ‘derive_spatial_merge_candidates’:  
libavcodec/hevc_mvs.c:208:15: error: ‘y0000000’ undeclared (first use in this function)  
((y ## v) >> s->ps.sps->log2_min_pu_size))  
^  
```

解决方法:将libavcodec/hevc_mvs.c文件的变量B0改成b0，xB0改成xb0，yB0改成yb0

(五)

```
libavcodec/opus_pvq.c: In function ‘quant_band_template’:  
libavcodec/opus_pvq.c:498:9: error: expected identifier or ‘(’ before numeric constant  
int B0 = blocks;  
^  
```

解决方法：将libavcodec/opus_pvq.c文件的变量B0改成b0

(六)  

chmod: -R: No such file or directory 各种文件无法创建  
先执行 ./configure  
如果报nasm/yasm not found or too old. ，则安装yasm  
方法：  
1）下载：[yasm的下载链接](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.tortall.net%2Fprojects%2Fyasm%2Freleases%2Fyasm-1.3.0.tar.gz)  
2）解压：把下载下来的压缩包进行解压  
3）切换路径： cd yasm-1.3.0  
4）执行配置： ./configure  
5）编译：make  
6）安装：make install（提示：Permission denied，就执行sudo make install）  
（七）fatal error: linux/perf_event.h: No such file or directory  
ffmpeg 的android交叉编译 ，报这个错误最简单的方式是

--disable-linux-perf

编译后的so库请自行取用：  
[https://gitee.com/zhangxingzx0/ff-so](https://links.jianshu.com/go?to=https%3A%2F%2Fgitee.com%2Fzhangxingzx0%2Fff-so)

作者：zx111  
链接：https://www.jianshu.com/p/ca3db5f164b1  
来源：简书  
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
