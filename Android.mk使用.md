# Android.mk使用

编译C/C++

```
//LOCAL_PATH指定当前工作目录，必须作为Android.mk的第一条语句。my-dir的定义位于build/core/definitions.mk
LOCAL_PATH := $(call my-dir)

//清空除LOCAL_PATH之外的模块描述变量。CLEAR_VARS的定义在build/core/config.mk
include $(CLEAR_VARS)

//定义编译生成的模块名称。注意，一个Android.mk中可能编译多个模块，每个模块都要分别命名
LOCAL_MODULE := helloworld

//user: 指该模块只在user版本下才编译
//eng: 指该模块只在eng版本下才编译
//tests: 指该模块只在tests版本下才编译
//optional:指该模块在所有版本下都编译
LOCAL_MODULE_TAGS ：=optional

//添加系统库
LOCAL_SHARED_LIBRARIES := liblog

//添加第三方库
LOCAL_LDFLAGS := $(LOCAL_PATH)/lib/libalbert.so
LOCAL_LDFLAGS := $(LOCAL_PATH)/lib/libalbert.a

//指定目标文件生成路径，绝对路径相对路径皆可。
LOCAL_MODULE_PATH := $(LOCAL_PATH)/bin

//指定引入头文件的目录
LOCAL_C_INCLUDES := $(LOCAL_PATH)/inc

//指定源文件。
LOCAL_SRC_FILES := src/helloworld.c

//指定生成的文件格式。
//BUILD_EXECUTABLE：可执行文件
//BUILD_SHARED_LIBRARY：动态库
//BUILD_STATIC_LIBRARY：静态库
//BUILD_STATIC_JAVA_LIBRARY：静态jar包
//BUILD_JAVA_LIBRARY：共享jar包
//BUILD_PACKAGE：APK
include $(BUILD_EXECUTABLE)
```

多个源文件

```
LOCAL_SRC_FILES := src/main.c \
	           	   src/albert.c
```

build/core/definitions.mk 中定义了all-c-files-under 函数，使用此函数即可自动添加指定目录下的所有.c文件。

```
//$(call all-java-files-under, src)：获取src目录下的所有 Java 文件。
//$(call all-c-files-under, src)：获取src目录下的所有 C 语言文件。
//$(call all-Iaidl-files-under, src) ：获取src目录下的所有 AIDL 文件。
//$(call all-makefiles-under, folder)：获取src目录下的所有 Make 文件。

LOCAL_SRC_FILES := $(call all-c-files-under)
```

编译APK

```
include $(CLEAR_VARS)
//引用jar库的别名
LOCAL_STATIC_JAVA_LIBRARIES := scanerjar
//编译生成的apk名称
LOCAL_PACKAGE_NAME := TOPUSDKServiceMk
//指定是否启用混淆，默认开启
LOCAL_PROGUARD_ENABLED := disabled
//指定混淆配置文件
LOCAL_PROGUARD_FLAG_FILES := proguard.cfg
//决定了其编译后的在ROM中的安装位置，如果不设置或者设置为false，安装位置为system/app；如果设置为true，安装位置为system/priv-app。
LOCAL_PRIVILEGED_MODULE := true
//同时编译32位和64位，LOCAL_MULTILIB := 32 只编译32位，LOCAL_MULTILIB := 64 只编译64位
LOCAL_MULTILIB := both
//指定资源文件目录
LOCAL_RESOURCE_DIR := $(LOCAL_PATH)/res
//使用系统签名
//testkey：普通APK，默认情况下使用
//platform：该APK完成一些系统的核心功能。经过对系统中存在的文件夹的访问测试，这种方式编译出来的APK所在进程的UID为system。
//shared：该APK需要和home/contacts进程共享数据。
//media：该APK是media/download系统中的一环。
LOCAL_CERTIFICATE := platform
include $(BUILD_PACKAGE)

//预编译
include $(CLEAR_VARS)
//指定预编译Java库别名，别名:jar文件的完整路径，这里的别名就是 LOCAL_STATIC_JAVA_LIBRARIES 所取的名字。
LOCAL_PREBUILT_STATIC_JAVA_LIBRARIES :=scanerjar:libs/libscaner.jar
include $(BUILD_MULTI_PREBUILT)
```

多个 编译目标

```
LOCAL_PATH := $(call my-dir)

# MODULE 1，编译生成libalbert.a
include $(CLEAR_VARS)
LOCAL_MODULE := libalbert
LOCAL_SRC_FILES := src/albert.c
include $(BUILD_STATIC_LIBRARY)

# MODULE 2，编译生成libalbert.so
include $(CLEAR_VARS)
LOCAL_MODULE := libalbert
LOCAL_MODULE_PATH := $(LOCAL_PATH)/lib
LOCAL_SRC_FILES := src/albert.c
include $(BUILD_SHARED_LIBRARY)
```

参考

[Android.mk官方文档](https://developer.android.google.cn/ndk/guides/android_mk.html)

————————————————

版权声明：本文为CSDN博主「Hufft」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。

原文链接：https://blog.csdn.net/stone_gentle/article/details/128735170