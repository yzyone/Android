Android使用LAME将pcm文件转mp3文件

---

lame介绍：LAME是一款开源的MP3编码器，被认为是中高比特率和VBR的最佳MP3编码器，质量和速度方面的改进仍在继续，可能使LAME成为仍在积极开发的唯一MP3编码器。使用lame进行mp3编码，需要了解一些NDK相关的知识，比如jni和cmake。

准备工作：

下载lame源码：https://lame.sourceforge.io/ 直接下载最新版本，本文使用的是3.100版本

android studio新建一个c++ support项目

集成lame源码

在cpp目录下，新建一个文件夹lamemp3，将下载的的源码文件中的\lame-3.100\libmp3lame文件夹下的所有.c和.h文件拷贝到lamemp3下并将\lame-3.100\include下的lame.h也拷贝进去，到这里，lame的源码都已经拷贝完成了，接下来，需要修改部分源码内容以及gradle配置参数。

1）
> 删除fft.c文件的47行的”include “vector/lame_intrin.h”“

2）
> 修改set_get.h文件的24行的#include“lame.h”

3）
> 将util.h文件的574行的”extern ieee754_float32_t fast_log2(ieee754_float32_t x);”
> 替换为 “extern float fast_log2(float x);”

需要修改app 下的build.gradle文件

```
android {
...
    defaultConfig {
    ...
        externalNativeBuild{
            cmake{
                cppFlags "-frtti -fexceptions"
                cFlags "-DSTDC_HEADERS"
            }
        }
    }
}
```

接下来，我们编辑cmake，编辑好以后，点击android studio的build下的refresh linked c++ project，关于cmake部分，看cmake文件注释部分，cmake编写方式不局限一种，取决于你对cmake相关方法的了解

```
# Sets the minimum version of CMake required to build the native library.
#设置构建本地库所需的最低cmake版本
cmake_minimum_required(VERSION 3.4.1)
#设置库文件导出目录
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/lameLibs/${ANDROID_ABI})
#批量导入c文件和cpp文件
file(GLOB lame lamemp3/*.c)
file(GLOB lame2 lamemp3/*.cpp)
#批量导入头文件
include_directories(${CMAKE_CURRENT_LIST_DIR}/lamemp3)
# 创建和给一个库命名，可以设置为静态库
# 或动态库，并且提供它源码的相关路径
# 你可以定义多个库，然后cmake为你构建他们
# gradle会自动打包共享库到你的apk中
add_library( # Sets the name of the library.
        native-lib

        # Sets the library as a shared library.
        SHARED

        # 导入所有源文件
        native-lib.cpp
        ${lame}
        ${lame2}
        )

#所有指定的预构建库，并将路径定义为变量，用于后面引入
find_library( # Sets the name of the path variable.
        log-lib
        # Specifies the name of the NDK library that
        # you want CMake to locate.
        log)

#指令cmake需要链接到的库
target_link_libraries(
        #指定目标library
        native-lib
        #链接目标库到log库
        ${log-lib})
```

编写MP3转换的c++实现，

在cpp/lamemp3文件夹下载新建pcm2pm3.cpp和pcm2pm3.h文件，这里贴出实现部分，完成代码参考项目源码

```
//初始化lame
int pcm2mp3::init(const char *pcm_path, const char *mp3_path, int sample_rate, int channel,
                  int bitRate) {
    int result=-1;
    pcm_file=fopen(pcm_path,"rb");
    if(pcm_file!= NULL){
        mp3_file=fopen(mp3_path,"wb");
        if (mp3_file!= NULL){
            //开始初始化
            lame_client=lame_init();
            lame_set_in_samplerate(lame_client,sample_rate);
            lame_set_out_samplerate(lame_client,sample_rate);
            lame_set_num_channels(lame_client,channel);
            lame_set_brate(lame_client,bitRate);
            lame_set_quality(lame_client,2);
            lame_init_params(lame_client);
            result=0;
        }
    }
    return result;
}
//pcm文件转mp3文件
void pcm2mp3::pcm_to_mp3() {
    int bufferSize = 1024 * 256;
    short *buffer = new short[bufferSize / 2];
    short *leftBuffer = new short[bufferSize / 4];
    short *rightBuffer = new short[bufferSize / 4];
    unsigned char* mp3_buffer = new unsigned char[bufferSize];
    size_t readBufferSize = 0;
    while ((readBufferSize = fread(buffer, 2, bufferSize / 2, pcm_file)) > 0) {
        for (int i = 0; i < readBufferSize; i++) {
            if (i % 2 == 0) {
                leftBuffer[i / 2] = buffer[i];
            } else {
                rightBuffer[i / 2] = buffer[i];
            }
        }
        size_t wroteSize = lame_encode_buffer(lame_client, (short int *) leftBuffer, (short int *) rightBuffer, (int)(readBufferSize / 2), mp3_buffer, bufferSize);
        fwrite(mp3_buffer, 1, wroteSize, mp3_file);
    }

    delete [] buffer;
    delete [] leftBuffer;
    delete [] rightBuffer;
    delete [] mp3_buffer;


}
```

接下来就是编写jni调用c++方法，定义了三个native方法，一个获取版本号用于测试lame加载是否成功，pcmTomp3进行文件转换，destroy进行相关对象的释放

```
public class LameJni {
    static {
        System.loadLibrary("native-lib");
    }
    public native String getVersion();
    //初始化lame
    public native int pcmTomp3(String pcmPath,String mp3Path,int sampleRate, int channel,  int bitRate);
    public native void destroy();

}
```

然后是对native方法的实现，这里只贴出转换的实现，关于jni的命名规则以及类型转换规则，这里不多赘述，查看jni相关资料即可

```
pcm2mp3 *mp3_encoder;
extern "C"
JNIEXPORT int JNICALL
Java_com_david_sampling_util_LameJni_pcmTomp3(JNIEnv *env, jclass clazz, jstring pcm_path,
                                          jstring mp3_path, jint sample_rate, jint channel,
                                          jint bit_rate) {
    int result=-1;
    const char *pcm_path_=env->GetStringUTFChars(pcm_path,0);
    const char *mp3_path_=env->GetStringUTFChars(mp3_path,0);
    mp3_encoder=new pcm2mp3();
    mp3_encoder->init(pcm_path_,mp3_path_,sample_rate,channel,bit_rate);
    mp3_encoder->pcm_to_mp3();
    env->ReleaseStringUTFChars(pcm_path,pcm_path_);
    env->ReleaseStringUTFChars(mp3_path,mp3_path_);
    return result;
}
```

然后java层通过jni调用即可实现mp3文件的编码

```
//注：本例经过尝试，发现如果采样率不缩小，那么声音会变快，原因还未找到
lameJni.pcmTomp3(pcmFile.getAbsolutePath(),mp3File.getAbsolutePath(),mSampleRate/2,mChannel==AudioFormat.CHANNEL_IN_MONO?1:2,128);
```

以上，即可将pcm转换为mp3，关于pcm的获取，可以参考上一篇：https://blog.csdn.net/s591628545/article/details/104525958

项目地址：https://github.com/kingdavidsun/AudioSampling.git

————————————————

版权声明：本文为CSDN博主「羽过天惊」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。

原文链接：https://blog.csdn.net/s591628545/article/details/104526112