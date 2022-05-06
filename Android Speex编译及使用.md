# Android Speex编译及使用

@Author:明明不说话

@Statement:博客内容纯属个人观点，欢迎转载，转载请注明，谢谢

Speex是一套主要针对语音的开源免费，无专利保护的音频压缩格式。Speex工程着力于通过提供一个可以替代高性能语音编解码来降低语音应用输入门槛 。另外，相对于其它编解码器，Speex也很适合网络应用，在网络应用上有着自己独特的优势。同时，Speex还是GNU工程的一部分，在改版的BSD协议中得到了很好的支持。它完全是C语言实现的，所以它具有很好的移植性。所以在Android当中具有很好的使用性。可以在它的官网上下载需要的源码来操作。

Android当中移植Speex需要以下几个步骤：

## 1.NDK配置 ##

NDK怎么配置和操作可以网上搜索，一大堆资料来告诉大家怎么操作的。现在越来越多的Android应用项目需要JNI代码操作，所以懂得NDK操作还是非常必要的。（最后能够在命令行执行出ndk-build，就说明NDK基本配置成功了）

## 2.Speex编译 ##

Speex编译生成连接库给Android有很多种方法，可以通过MinGW编译,也可以通过在linux下编译，这些方法可以网上搜索吧。我这里采用的是将Speex在Android工程中编译成静态库.a的方式，这样该Speex可以很方便的在不同项目当中使用，而且方便其他项目管理，如果Speex有更改，则重新将Speex编译程静态库，然后替换到相应项目中就行了。

下面是libSpeex.a库的编译生成过程：

环境

windows + Android studio + NDK10

- a. 去Speex官网下载最新Speex。
- b. 随便找一个android工程，新建一个也可以，在它的工程下创建jni目录
- c.把speex源码目录下的libspeex和include目录及其子目录文件全部拷贝到jni目录下
- d. 在jni中创建Android.mk
- 
Android.mk的写法如下：

```
LOCAL_PATH := $(call my-dir)

include $(CLEAR_VARS)

LOCAL_MODULE:= libspeex
LOCAL_CFLAGS = -DFIXED_POINT -DUSE_KISS_FFT -DEXPORT="" -UHAVE_CONFIG_H
LOCAL_C_INCLUDES := $(LOCAL_PATH)/include

LOCAL_SRC_FILES :=\
libspeex/bits.c \
libspeex/buffer.c \
libspeex/cb_search.c \
libspeex/exc_10_16_table.c \
libspeex/exc_10_32_table.c \
libspeex/exc_20_32_table.c \
libspeex/exc_5_256_table.c \
libspeex/exc_5_64_table.c \
libspeex/exc_8_128_table.c \
libspeex/fftwrap.c \
libspeex/filterbank.c \
libspeex/filters.c \
libspeex/gain_table.c \
libspeex/gain_table_lbr.c \
libspeex/hexc_10_32_table.c \
libspeex/hexc_table.c \
libspeex/high_lsp_tables.c \
libspeex/jitter.c \
libspeex/kiss_fft.c \
libspeex/kiss_fftr.c \
libspeex/lpc.c \
libspeex/lsp.c \
libspeex/lsp_tables_nb.c \
libspeex/ltp.c \
libspeex/mdf.c \
libspeex/modes.c \
libspeex/modes_wb.c \
libspeex/nb_celp.c \
libspeex/preprocess.c \
libspeex/quant_lsp.c \
libspeex/resample.c \
libspeex/sb_celp.c \
libspeex/scal.c \
libspeex/smallft.c \
libspeex/speex.c \
libspeex/speex_callbacks.c \
libspeex/speex_header.c \
libspeex/stereo.c \
libspeex/vbr.c \
libspeex/vq.c \
libspeex/window.c \

#创建static静态库，动态库为BUILD_SHARED_LIBRARY
include $(BUILD_STATIC_LIBRARY)    
```

e. 在jni目录下新增Application.mk文件

Application.mk的编写如下，可以根据需要的硬件平台来配置，如arm，arm-v7,x86，mips等。

	APP_ABI := armeabi armeabi-v7a    

f. 在jni/include/speex/目录下speex_config_types.h.xx文件的xx后缀去掉，编辑内容如下

```
#ifndef __SPEEX_TYPES_H__    
#define __SPEEX_TYPES_H__    
pedef short spx_int16_t;    
typedef unsigned short spx_uint16_t;    
typedef int spx_int32_t;    
typedef unsigned int spx_uint32_t;    
#endif   
```

g. 然后打开cmd命令行，定位到前面创建的jni目录，执行ndk-build，则会生成相应的libSpeex.a（这里面不仅仅包含encode和decode，同时还包含降噪等操作，因为这里是整个项目全部编译了）

## 3.libSpeex.a的使用 ##

将libSpeex.a拷贝到需要的工程目录中，然后将libSpeex下的.h文件和include中的.h文件全部拷贝到jni目录中，然后编写自己的Speex Native文件，可以是C或者C++的

Speex初始化

```
**ps:示例代码，仅供参考**
 preprocessState = speex_preprocess_state_init(RECORDER_FRAMES, SL_SAMPLINGRATE_16/1000);
    int denoise = 1, noiseSuppress = -25;
    speex_preprocess_ctl(preprocessState, SPEEX_PREPROCESS_SET_DENOISE, &denoise); //降噪
    speex_preprocess_ctl(preprocessState, SPEEX_PREPROCESS_SET_NOISE_SUPPRESS, &noiseSuppress); //设置噪声的
    //        int agc = 1, q=15000;
    ////        //actually default is 8000(0,32768),here make it louder for voice is not loudy enough by default. 8000
    //        speex_preprocess_ctl(preprocessState, SPEEX_PREPROCESS_SET_AGC, &agc);//增益
    //        speex_preprocess_ctl(preprocessState, SPEEX_PREPROCESS_SET_AGC_LEVEL,&q);

   int vad = 1;
   int vadProbStart = 99;
   int vadProbContinue = 99;
   speex_preprocess_ctl(preprocessState, SPEEX_PREPROCESS_SET_VAD, &vad); //静音检测
   speex_preprocess_ctl(preprocessState, SPEEX_PREPROCESS_SET_PROB_START , &vadProbStart); //Set probability required for the VAD to go from silence to voice
   speex_preprocess_ctl(preprocessState, SPEEX_PREPROCESS_SET_PROB_CONTINUE, &vadProbContinue); //Set probability

   //init speex encoder
   speex_bits_init(&ebits);
   enc_state = speex_encoder_init(&speex_wb_mode);
   speex_encoder_ctl(enc_state, SPEEX_SET_QUALITY, &compression);
   speex_encoder_ctl(enc_state, SPEEX_GET_FRAME_SIZE, &enc_frame_size);
   speex_encoder_ctl(enc_state, SPEEX_GET_SAMPLING_RATE, &sample_rate);
   pcm_buffer = malloc(enc_frame_size * sizeof(short));
   output_buffer = malloc(enc_frame_size * sizeof(char));
```

Speex编码

	speex_encode_int(enc_state, pcm_buffer, &ebits);

Speex解码

	speex_bits_read_from(&dbits, rtmp_pakt.m_body + 1, rtmp_pakt.m_nBodySize - 1);
	speex_decode_int(dec_state, &dbits, output_buffer);

ps:使用speex_preprocess注意

在arch.h当中需要定义是否浮点运算，还是定点运算，这取决于cpu，这里是我选择定点运算FIXED_POINT

```
#define FIXED_POINT  //需要自己定义一个FIXED_POINT   or  FLOATING_POINT
/* A couple test to catch stupid option combinations */
#ifdef FIXED_POINT
#ifdef FLOATING_POINT
```

基本上Speex在Android上的使用就是这样。可以下载[demo查看](http://download.csdn.net/detail/hack__zsmj/9320235)。Q&A，

Thanks!

————————————————

版权声明：本文为CSDN博主「HACK__ZSMJ」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。

原文链接：https://blog.csdn.net/HACK__ZSMJ/article/details/50135899