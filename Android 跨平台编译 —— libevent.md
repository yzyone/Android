# [Android 跨平台编译 —— libevent](https://my.oschina.net/zzxzzg/blog/1623523)

此文大量引用参考 [https://www.dplord.com/2017/10/10/android-ndk-compile-libevent/](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.dplord.com%2F2017%2F10%2F10%2Fandroid-ndk-compile-libevent%2F)

# 前言

  前言都在 [Android 跨平台编译 —— BOOST](https://my.oschina.net/zzxzzg/blog/1621360)

  PS: 有一些基础知识介绍，还是挺重要的。

# 准备工作

  编译环境 mac os x

  ndk 版本 ndk 16rb

  libevent 版本 2.1.8

  编译工具 cmake

  假设我们已经在 android studio 上安装了 ndk 和 cmake。

  首先需要下载 libevent ，这里有个问题，如果直接从官网上下载 tar 包，解压之后是没有 CMakeLists.txt 文件的，所以无法使用 cmake 进行编译。所以我们需要从 github 上直接下载，地址 [https://github.com/libevent/libevent](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Flibevent%2Flibevent)

[  ](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Flibevent%2Flibevent)去 clone 下来，然后 checkout 到 release-2.1.8-stable 这个 tag 上。

  环境已经准备就绪。

# 编译

## 步骤 1

  首先打开根目录下的 CMakeLists.txt 文件，有几处改动

```
diff --git a/CMakeLists.txt b/CMakeLists.txt
index b4a34f3d..c32721d1 100644
--- a/CMakeLists.txt
+++ b/CMakeLists.txt
@@ -105,7 +105,7 @@ option(EVENT__BUILD_SHARED_LIBRARIES
     "Define if libevent should be built with shared libraries instead of archives" OFF)
 
 option(EVENT__DISABLE_DEBUG_MODE
-    "Define if libevent should build without support for a debug mode" OFF)
+    "Define if libevent should build without support for a debug mode" ON)
 
 option(EVENT__ENABLE_VERBOSE_DEBUG
     "Enables verbose debugging" OFF)
@@ -117,7 +117,7 @@ option(EVENT__DISABLE_THREAD_SUPPORT
     "Define if libevent should not be compiled with thread support" OFF)
 
 option(EVENT__DISABLE_OPENSSL
-    "Define if libevent should build without support for OpenSSL encrpytion" OFF)
+    "Define if libevent should build without support for OpenSSL encrpytion" ON)
 
 option(EVENT__DISABLE_BENCHMARK
     "Defines if libevent should build without the benchmark exectuables" OFF)
@@ -328,8 +328,8 @@ CHECK_FUNCTION_EXISTS_EX(strtoll EVENT__HAVE_STRTOLL)
 CHECK_FUNCTION_EXISTS_EX(vasprintf EVENT__HAVE_VASPRINTF)
 CHECK_FUNCTION_EXISTS_EX(sysctl EVENT__HAVE_SYSCTL)
 CHECK_FUNCTION_EXISTS_EX(accept4 EVENT__HAVE_ACCEPT4)
-CHECK_FUNCTION_EXISTS_EX(arc4random EVENT__HAVE_ARC4RANDOM)
-CHECK_FUNCTION_EXISTS_EX(arc4random_buf EVENT__HAVE_ARC4RANDOM_BUF)
+#CHECK_FUNCTION_EXISTS_EX(arc4random EVENT__HAVE_ARC4RANDOM)
+#CHECK_FUNCTION_EXISTS_EX(arc4random_buf EVENT__HAVE_ARC4RANDOM_BUF)
 CHECK_FUNCTION_EXISTS_EX(epoll_create1 EVENT__HAVE_EPOLL_CREATE1)
 CHECK_FUNCTION_EXISTS_EX(getegid EVENT__HAVE_GETEGID)
 CHECK_FUNCTION_EXISTS_EX(geteuid EVENT__HAVE_GETEUID)
@@ -492,7 +492,7 @@ CHECK_TYPE_SIZE("void *" EVENT__SIZEOF_VOID_P)
 #CHECK_FILE_OFFSET_BITS()
 #set(EVENT___FILE_OFFSET_BITS _FILE_OFFSET_BITS)
 
-include(CheckWaitpidSupportWNOWAIT)
+#include(CheckWaitpidSupportWNOWAIT)
 
 # Verify kqueue works with pipes.
 if (EVENT__HAVE_KQUEUE)
@@ -846,17 +846,17 @@ if (EVENT__BUILD_SHARED_LIBRARIES)
                                       ${CMAKE_THREAD_LIBS_INIT}
                                       ${LIB_PLATFORM})
 
-  set_target_properties(event
-        PROPERTIES SOVERSION
-            ${EVENT_ABI_LIBVERSION})
+#  set_target_properties(event
+#        PROPERTIES SOVERSION
+#            ${EVENT_ABI_LIBVERSION})
 
-  set_target_properties(event_core
-        PROPERTIES SOVERSION
-            ${EVENT_ABI_LIBVERSION})
+#  set_target_properties(event_core
+#        PROPERTIES SOVERSION
+#            ${EVENT_ABI_LIBVERSION})
 
-  set_target_properties(event_extra
-        PROPERTIES SOVERSION
-            ${EVENT_ABI_LIBVERSION})
+#  set_target_properties(event_extra
+#        PROPERTIES SOVERSION
+#            ${EVENT_ABI_LIBVERSION})
 
 else (EVENT__BUILD_SHARED_LIBRARIES)
     set(EVENT_EXTRA_FOR_TEST event_extra)
```

  首先我关闭了 debug 模式和 openssl 的支持（关于需要的设置因人而异，用户可以自己选择设置。）

  另外我关闭了 arc4random 和 arc4random_buf 的检测（关于这部分我会在下文说明）

  去掉了 CheckWaitpidSupportWNOWAIT

  另外去掉了编译出来的库文件的版本后缀。（比如最后库文件是 libevent.a 而不是 libevent.a.2.1.8）



## 步骤 2

  在根目录下创建 android.sh，内容如下

```bash
NDK_PATH=/Users/yxwang/Library/Android/sdk/ndk-bundle/ #换成你自己的ndk path
SHELL_FOLDER=$(cd "$(dirname "$0")";pwd)

BUILD_PATH=$SHELL_FOLDER/android_build/
echo $BUILD_PATH
if [ -x "$BUILD_PATH" ]; then
		rm -rf $BUILD_PATH
fi
mkdir $BUILD_PATH
mkdir $BUILD_PATH/out


for abi in armeabi armeabi-v7a arm64-v8a x86 x86_64
do

  #cmake      
  MakePath=./cmake/build-$abi
  echo $MakePath
	if [ -x "$MakePath" ]; then
		rm -rf $MakePath
	fi
	mkdir $MakePath
	
	OUTPUT_PATH=$BUILD_PATH/out/$abi/
	echo $OUTPUT_PATH
	if [ -x "$OUTPUT_PATH" ]; then
		rm -rf $OUTPUT_PATH
	fi
	mkdir $OUTPUT_PATH
	
	cd $MakePath
	
    # DCMAKE_INSTALL_PREFIX 最后install的路径 这里是 android_build/$abi
    # DCMAKE_TOOLCHAIN_FILE 这个的路劲在android studio中创建一个带有ndk的项目，编译一下，然后
    # 在.externalNativeBuild/cmake/***/cmake_build_command.txt中找到
    # stl 我们使用c++_static
	cmake -DCMAKE_TOOLCHAIN_FILE=$NDK_PATH/build/cmake/android.toolchain.cmake \
    -DANDROID_NDK=$NDK_PATH                      \
    -DCMAKE_BUILD_TYPE=Release                     \
    -DANDROID_ABI=$abi          \
    -DANDROID_NATIVE_API_LEVEL=16                  \
    -DANDROID_STL=c++_static \
    -DCMAKE_CXX_FLAGS=-frtti -fexceptions --std=c++1z \
    -DCMAKE_INSTALL_PREFIX=$OUTPUT_PATH \
    ../..
	
	make -j4
	make install
	
	cd ../..
	
done
```

## 步骤三

  运行上面的脚本 (chmod +x android.sh 添加执行权限才能运行。)

  发现在编译过程中会报出如下错误

```
            ^
In file included from /Users/yxwang/workspace/gittest/libevent/evutil_rand.c:134:
/Users/yxwang/workspace/gittest/libevent/./arc4random.c:498:1: error: static declaration of 'arc4random_buf' follows non-static
      declaration
arc4random_buf(void *buf_, size_t n)
^
/Users/yxwang/Library/Android/sdk/ndk-bundle/sysroot/usr/include/stdlib.h:124:6: note: previous declaration is here
void arc4random_buf(void* __buf, size_t __n);
     ^
1 error generated.
```

  什么意思呢，就是我 arc4random_buf 重复定义了，并且一处是 static 一处不是，这就出现了冲突，导致编译失败。我们分别来看下两处代码

```
// arc4random.c

ARC4RANDOM_EXPORT void
arc4random_buf(void *buf_, size_t n)
{
	.....
}

// $NDK_PATH/sysroot/usr/include/stdlib.h
void arc4random_buf(void* __buf, size_t __n);
```

  其中 ARC4RANDOM_EXPORT 在 evutil_rand.c 中有定义

```
#define ARC4RANDOM_EXPORT static
```

  查看 evutil_rand.c 文件，有一个非常关键的宏 EVENT__HAVE_ARC4RANDOM 表示运行的系统是否自带 ARC4RANDOM 功能。

  如果没有自带，那么 ARC4RANDOM_EXPORT 才会被定义为 static。这里他被定义为 static 了，也就是我们告诉了编译器我们没有自带 ARC4RANDOM。我们是如何实现的？就是通过

  -CHECK_FUNCTION_EXISTS_EX(arc4random EVENT__HAVE_ARC4RANDOM)
  -CHECK_FUNCTION_EXISTS_EX(arc4random_buf EVENT__HAVE_ARC4RANDOM_BUF)
  +#CHECK_FUNCTION_EXISTS_EX(arc4random EVENT__HAVE_ARC4RANDOM)
  +#CHECK_FUNCTION_EXISTS_EX(arc4random_buf EVENT__HAVE_ARC4RANDOM_BUF)

  关闭了 arc4random 检测。

  如果我们开启这个检测，EVENT__HAVE_ARC4RANDOM 就会被设置为 1 。

  我们可以通过查看编译过程中自动生成的 event-config.h 文件查看这个宏是否有被设置。

  （如果使用上面提供的脚本可以在 cmake/build-xxx/include/event2 文件夹中找到自动生成的文件）

  所以干嘛多次一举嘛！干脆开启这个检测好了，尝试编译，竟然通过了！！但是，有个并不算好的消息，一旦你打开检测，cmake 检测到 arc4random 存在，arc4random_buf 存在，就把宏打开了，表示你已经实现了 arc4random 相关的代码了，我就不去定义实现了。

  然后在运行使用的时候，比如调用到了 evutil_rand.c 中的这个方法

```
void
evutil_secure_rng_add_bytes(const char *buf, size_t n)
{
	arc4random_addrandom((unsigned char*)buf,
	    n>(size_t)INT_MAX ? INT_MAX : (int)n);
}
```

  进一步会调用 arc4random_addrandom，编译器就会报错，表示 arc4random_addrandom 这个函数不存在！

  虽然在 libevent 中这个函数式定义了但是它是现在 arc4random.c 文件中，只有 EVENT__HAVE_ARC4RANDOM 未设置的时候才会被 #include 到 evutil_rand.c 中。

  所以理论上来说 arc4random_addrandom 一旦你设置了 EVENT__HAVE_ARC4RANDOM，就意味着你需要去实现 arc4random_addrandom 方法，但是在 ndk 中，arc4random 是被裁剪了的，他只有如下三个方法

​                uint32_t arc4random(void);
​                uint32_t arc4random_uniform(uint32_t __upper_bound);
​                void arc4random_buf(void* __buf, size_t __n);

  我们可以在 stdlib.h 中看到声明。

  所以为了解决这个问题，做法是手动修改 stdlib.h 文件，将以上三个定义注释掉。（编译通过之后重新打开，保证我们没有修改 ndk，防止以后使用错误）其中 stdlib.h 文件在 ndk-bundle/sysroot/usr/include 中。

# 编译完成

  再次运行 android.sh 之后编译完成，输出文件在 android-build 文件夹中，各个 abi 都已经编译出来。