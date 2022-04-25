# 使用Android NDK 编译BoringSSL

1、下载boringssl(https://boringssl.googlesource.com/boringssl/)

2、下载NDK及环境变量配置（[NDK 下载  |  Android NDK  |  Android Developers (google.cn)](https://developer.android.google.cn/ndk/downloads/)）

NDK下载好后，配置一下NDK的环境变量，我是在/etc/profile 进行配置的。

配置如下：

export NDK_HOME=/home/android-ndk-r23b

export PATH=$NDK_HOME:$PATH

配置完后，执行source /etc/profile, 输入echo $NDK_HOME 确认是否配置成功。

如果输入echo $NDK_HOME出现配置的路径，则表示成功。

NDK配置好后，测试一下clang工具是否好用（之前我就碰到NDK下的clang不好用，出现了电脑卡死的问题，因为没有error提示，所以尝试了好久才解决了问题。）

测试方法：

以android-ndk-r23b-linux.zip为例，解压出来的文件夹名为：android-ndk-r23b

进入到下述目录：android-ndk-r23b/toolchains/llvm/prebuilt/linux-x86_64/bin

写一个任意的helloworld.cpp ,然后执行：clang++  helloworld.cpp

如果出现的clang++: command not found，这表示clang++ 工具是不好用的。

如果成功编译了，表示clang++ 工具是可用的。

这里需要注意一下：Docker环境下使用clang++  helloworld.cpp 会有问题，因为我的目标已经达成，所以没有继续去调查Docker下为什么会出问题。建议大家避免使用Docker环境来干这事。

3、 cmake 安装（[Download | CMake](https://cmake.org/download/)）

测试cmake安装成功方法：到任意目录 输入:cmake --version 

如果出现cmake version xxxxx，表示安装成功。

（印象中有依赖re2c, 如果出现错误提示，记得下载安装一下）

4、安装ninja  （[Releases · ninja-build/ninja · GitHub](https://github.com/ninja-build/ninja/releases)）

测试ninja安装成功方法：到任意目录输入:ninja --version,

如果出现版本号，比如：1.10.2，表示安装成功。

上述都准备好后，在boringssl的源码目录下，创建一个build文件，

然后再在build文件下创建一个build.sh脚本，脚本内容如下 ：

```
#!/bin/sh

cmake -DANDROID_ABI=arm64-v8a    #我是64位机器android，所以选了这个
-DANDROID_NATIVE_API_LEVEL=23  \
-DANDROID_NDK=$NDK_HOME \ 
-DCMAKE_TOOLCHAIN_FILE=$NDK_HOME/build/cmake/android.toolchain.cmake  \
-DCMAKE_BUILD_TYPE=Release  \
-DANDROID_PLATFORM=android-23 \ 
-DBUILD_SHARED_LIBS=1   \  #这个是为了编译动态库的，取消这个生成的就是静态库
-GNinja ..

cmake --build .   
```

#注意 cmake --build .后面需要加一个 点。

执行这个./build.sh 脚本，会在build 目录下生成的ssl和crypto 目录下生成两个动态库libssl.so和libcrypto.so，头文件就是boringssl根目录下的include。

到此成功完成。

————————————————

版权声明：本文为CSDN博主「IM12」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。

原文链接：https://blog.csdn.net/Iam_a_freshman/article/details/121564304