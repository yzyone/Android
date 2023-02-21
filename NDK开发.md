# NDK开发

FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':app:mergeDebugNativeLibs'.
> A failure occurred while executing com.android.build.gradle.internal.tasks.MergeJavaResWorkAction
> 2 files found with path 'lib/arm64-v8a/libswscale.so'.
> If you are using jniLibs and CMake IMPORTED targets, see
> https://developer.android.com/r/tools/jniLibs-vs-imported-targets

 

原因：

jniLibs和CMAKE混用，导致生成了两个so，系统不知道使用哪个so

方案一：在defaultConfig中加入packagingOptions选项，指定优先选择哪个

externalNativeBuild {
    cmake {
        cppFlags ''
    }
}

packagingOptions {
    pickFirst 'lib/arm64-v8a/libswscale.so'
    pickFirst 'lib/arm64-v8a/libavcodec.so'
    pickFirst 'lib/lib/arm64-v8a/libavutil.so'
    pickFirst 'lib/arm64-v8a/libavutil.so'
    pickFirst 'lib/arm64-v8a/libavformat.so'
    pickFirst 'lib/arm64-v8a/libavfilter.so'
    pickFirst 'lib/arm64-v8a/libswresample.so'
    pickFirst 'lib/arm64-v8a/libavdevice.so'
}


方案二：将jniLibs修改名称，如：jni，并在CMakeLists.txt中修改引用路径

报错：/bin/bash^M: bad interpreter: No such file or directory

原因：windows上编辑的文件在Linux上运行，格式不对

1.sed -i "s/\r//" filename 或sed -i "s/^M//" filename，直接将回车符替换为空字符串。

2.vim filename，编辑文件，执行“: set ff=unix”，将文件设置为unix格式，然后执行“:wq”，保存退出。

————————————————

版权声明：本文为CSDN博主「zhangzhuo1024」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。

原文链接：https://blog.csdn.net/zhangzhuo1024/article/details/117604945