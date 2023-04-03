如果你使用ndk-build进行编译，可以按照以下步骤进行：

1. 下载libevent-2.1.8-stable源代码并解压缩。
2. 在libevent-2.1.8-stable目录下创建Android.mk文件，内容如下：

```
makefileCopy codeLOCAL_PATH := $(call my-dir)

include $(CLEAR_VARS)

LOCAL_MODULE := libevent
LOCAL_SRC_FILES := <source_files>
LOCAL_C_INCLUDES := <include_paths>
LOCAL_LDLIBS := -llog
LOCAL_CFLAGS := -DANDROID

include $(BUILD_SHARED_LIBRARY)
```

其中，LOCAL_MODULE指定编译生成的库文件名，LOCAL_SRC_FILES指定源代码文件列表，LOCAL_C_INCLUDES指定头文件路径，LOCAL_LDLIBS指定链接的系统库，LOCAL_CFLAGS指定编译选项。

1. 在libevent-2.1.8-stable目录下创建Application.mk文件，内容如下：

```
goCopy codeAPP_ABI := <target_architecture>
APP_PLATFORM := android-<min_sdk_version>
```

其中，APP_ABI指定目标设备架构，APP_PLATFORM指定最小支持的Android SDK版本。

1. 打开终端，进入libevent-2.1.8-stable目录。
2. 设置NDK路径：

```
javascriptCopy code
export NDK_ROOT=/path/to/ndk
```

其中，NDK_ROOT需要替换为实际的NDK路径。

1. 运行ndk-build命令进行编译：

```
bashCopy code
$NDK_ROOT/ndk-build
```

编译完成后，会在libevent-2.1.8-stable/libs/<target_architecture>/目录下生成编译生成的库文件。

注意：编译过程中可能会遇到各种错误，需要根据实际情况解决。同时，由于不同的NDK版本和架构可能存在差异，以上步骤仅供参考，具体操作还需根据实际情况调整。