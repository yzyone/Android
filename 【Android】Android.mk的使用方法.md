# 【Android】Android.mk的使用方法



## 1 Android.mk的作用

Android.mk是Android源码中的编译机制。常用功能如下：

C/C++

- 可执行文件
- 动态库
- 静态库

Java

- jar包
- APK

## 2 Android.mk的基本用法

为了便于理解，此处不罗列各种变量。接下来通过具体的功能示例，结合具体代码解析。

### 2.1 生成可执行文件

**2.1.1 生成到默认目录**

本小节demo目录结构如下：

```

.
├── Android.mk
└── src
    └── helloworld.c
```

helloworld.c如下：

```
#include <stdio.h>

int main() {
	printf("Hello world!\n");
	return 0;
}
```

Android.mk如下：

```
LOCAL_PATH := $(call my-dir)

include $(CLEAR_VARS)

LOCAL_MODULE := helloworld
LOCAL_SRC_FILES := src/helloworld.c

include $(BUILD_EXECUTABLE)
```

进入目录，mm -j编译，可以看到生成out/target/product/Hi3751V560/system/bin/helloworld。

**2.1.2 生成到指定目录**

本小节demo目录结构如下：

```
.
├── Android.mk
├── bin
└── src
    └── helloworld.c
```

helloworld.c如下：

```
#include <stdio.h>

int main() {
	printf("Hello world!\n");
	return 0;
}
```

Android.mk如下：

```
LOCAL_PATH := $(call my-dir)

include $(CLEAR_VARS)

LOCAL_MODULE := helloworld
LOCAL_MODULE_PATH := $(LOCAL_PATH)/bin
LOCAL_SRC_FILES := src/helloworld.c

include $(BUILD_EXECUTABLE)
```

进入目录，mm -j编译，可以看到生成bin/helloworld。

**2.1.3 相关变量、函数说明**

**2.1.3.1 LOCAL_PATH $(call my-dir)**

LOCAL_PATH指定当前工作目录，必须作为Android.mk的第一条语句。

my-dir的定义位于build/core/definitions.mk，摘录如下：

```
###########################################################
## Retrieve the directory of the current makefile
## Must be called before including any other makefile!!
###########################################################

# Figure out where we are.
define my-dir
$(strip \
  $(eval LOCAL_MODULE_MAKEFILE := $$(lastword $$(MAKEFILE_LIST))) \
  $(if $(filter $(BUILD_SYSTEM)/% $(OUT_DIR)/%,$(LOCAL_MODULE_MAKEFILE)), \
    $(error my-dir must be called before including any other makefile.) \
   , \
    $(patsubst %/,%,$(dir $(LOCAL_MODULE_MAKEFILE))) \
   ) \
 )
endef
```

**2.1.3.2 include $(CLEAR_VARS)**

include $(CLEAR_VARS)的作用是清空除LOCAL_PATH之外的模块描述变量。

CLEAR_VARS的定义在build/core/config.mk中。摘录如下：

	CLEAR_VARS:= $(BUILD_SYSTEM)/clear_vars.mk

BUILD_SYSTEM的定义则是在build/core/main.mk中。摘录如下：

	BUILD_SYSTEM := $(TOPDIR)build/make/core

由BUILD_SYSTEM和CLEAR_VARS可知，include $(CLEAR_VARS)的具体实现在build/make/core/clear_vars.mk中。其篇幅较长，摘录如下：

```
###########################################################
## Clear out values of all variables used by rule templates.
###########################################################

# '',true
LOCAL_32_BIT_ONLY:=
LOCAL_AAPT2_ONLY:=
LOCAL_AAPT_FLAGS:=
LOCAL_AAPT_INCLUDE_ALL_RESOURCES:=
LOCAL_AAPT_NAMESPACES:=
LOCAL_ADDITIONAL_CERTIFICATES:=
LOCAL_ADDITIONAL_DEPENDENCIES:=
LOCAL_ADDITIONAL_HTML_DIR:=
LOCAL_ADDITIONAL_JAVA_DIR:=
LOCAL_AIDL_INCLUDES:=
```

不难看出，其功能主要是清除环境变量。

**2.1.3.3 LOCAL_MODULE**

定义编译生成的模块名称。注意，一个Android.mk中可能编译多个模块，每个模块都要分别命名。详见2.3小节。

**2.1.3.4 LOCAL_MODULE_PATH**

指定目标文件生成路径，绝对路径相对路径皆可。对于本小节的demo而言，以下两种写法等价：

```
LOCAL_MODULE_PATH := $(LOCAL_PATH)/bin
LOCAL_MODULE_PATH := ./bin
```

**2.1.3.5 LOCAL_SRC_FILES**

指定源文件。对于多个源文件的情况，详见2.3小节。

**2.1.3.6 include $(BUILD_EXECUTABLE)**

指定生成的文件格式。

BUILD_EXECUTABLE：可执行文件
BUILD_SHARED_LIBRARY：动态库
BUILD_STATIC_LIBRARY：静态库
BUILD_STATIC_JAVA_LIBRARY：静态jar包
BUILD_JAVA_LIBRARY：共享jar包
BUILD_PACKAGE：APK

此外，还有预编译的功能。详见2. 小节

### 2.2 多源文件编译

**2.2.1 手动将源文件添加到Android.mk**

本小节demo目录结构如下：

```
.
├── Android.mk
├── bin
└── src
    ├── albert.c
    └── main.c
```

albert.c内容如下：

```
#include <stdio.h>

int albert() {
	printf("This albert function!\n");
	return 0;
}
```

main.c内容如下：

```
#include <stdio.h>

extern int albert();

int main() {
	albert();
	return 0;
}
```

Android.mk内容如下：

```
LOCAL_PATH := $(call my-dir)

include $(CLEAR_VARS)

LOCAL_MODULE := albert
LOCAL_MODULE_PATH := $(LOCAL_PATH)/bin
LOCAL_SRC_FILES := src/main.c \
	           src/albert.c

include $(BUILD_EXECUTABLE)
```

**2.2.2 使用NDK提供的函数宏**

build/core/definitions.mk中定义了all-c-files-under函数。使用此函数，即可自动添加指定目录下的所有.c文件。同理，对于Java文件可以使用all-java-files-under函数，此处不再赘述。

为了后续深入理解，我们向下追溯代码。all-c-files-under函数相关内容摘录如下：

```
###########################################################
## Find all of the c files under the named directories.
## Meant to be used like:
##    SRC_FILES := $(call all-c-files-under,src tests)
###########################################################

define all-c-files-under
$(call all-named-files-under,*.c,$(1))
endef

###########################################################
## Find all of the files under the named directories with
## the specified name.
## Meant to be used like:
##    SRC_FILES := $(call all-named-files-under,*.h,src tests)
###########################################################

define all-named-files-under
$(call find-files-in-subdirs,$(LOCAL_PATH),"$(1)",$(2))
endef
```

本小节demo示例其余部分与2.2.1一致，仅Android.mk略有不同：

```
LOCAL_PATH := $(call my-dir)

include $(CLEAR_VARS)

LOCAL_MODULE := albert
LOCAL_MODULE_PATH := $(LOCAL_PATH)/bin
LOCAL_SRC_FILES := $(call all-c-files-under)

include $(BUILD_EXECUTABLE)
```

### 2.3 引入头文件

在2.2小节中，main.c使用extern引入albert()函数，这是不大正统的写法。更为规范的方法是使用头文件来引入函数。在Android.mk中，可以使用LOCAL_C_INCLUDES指定头文件的目录。

注意，使用LOCAL_C_INCLUDES引入的头文件，可以用<>。

本小节demo目录结构如下：

```
.
├── Android.mk
├── bin
├── inc
│   └── albert.h
└── src
    ├── albert.c
    └── main.c
```

albert.c内容如下：

```
#include <stdio.h>

int albert() {
	printf("This albert function!\n");
	return 0;
}
```

main.c内容如下：

```
#include <stdio.h>
#include <albert.h>

int main() {
	albert();
	return 0;
}
```

Android.mk内容如下：

```
LOCAL_PATH := $(call my-dir)

include $(CLEAR_VARS)

LOCAL_MODULE := albert
LOCAL_MODULE_PATH := $(LOCAL_PATH)/bin

LOCAL_C_INCLUDES := $(LOCAL_PATH)/inc
LOCAL_SRC_FILES := $(call all-c-files-under)

include $(BUILD_EXECUTABLE)
```

### 2.4 编译生成动态库

和编译可执行文件相比，编译动态库仅需做两处改动：

将BUILD_EXECUTABLE改为BUILD_SHARED_LIBRARY；
LOCAL_MODULE要写成libxxx的格式
本小节demo目录结构如下：

```
.
├── Android.mk
├── lib
└── src
    └── albert.c
```

albert.c内容如下：

```
#include <stdio.h>

int albert() {
	printf("This albert function!\n");
	return 0;
}
```

Android.mk内容如下：

```
LOCAL_PATH := $(call my-dir)

include $(CLEAR_VARS)

LOCAL_MODULE := libalbert
LOCAL_MODULE_PATH := $(LOCAL_PATH)/lib
LOCAL_SRC_FILES := src/albert.c

include $(BUILD_SHARED_LIBRARY)
```

### 2.5 编译生成静态库

和编译动态库相比，编译静态库只需将BUILD_SHARED_LIBRARY改为BUILD_STATIC_LIBRARY。

注意，在编译静态库Android.mk中，即使使用LOCAL_MODULE_PATH指定输出路径，也不会生效。我们将在接下来的demo中看到这个现象。

本小节demo目录结构如下：

```
.
├── Android.mk
├── lib
└── src
    └── albert.c
```

albert.c内容如下：

```
#include <stdio.h>

int albert() {
	printf("This albert function!\n");
	return 0;
}
```

Android.mk内容如下：

```
LOCAL_PATH := $(call my-dir)

include $(CLEAR_VARS)

LOCAL_MODULE := libalbert
LOCAL_MODULE_PATH := $(LOCAL_PATH)/lib
LOCAL_SRC_FILES := src/albert.c

include $(BUILD_STATIC_LIBRARY)
```

执行mm -j编译，可以看到生成out/target/product/Hi3751V560/obj/STATIC_LIBRARIES/libalbert_intermediates/libalbert.a。

### 2.6 在一个Android.mk中编译生成多个目标文件

一个Android.mk中可以编译生成多个目标文件，仅需在一个MODULE结束之后，使用include $(CLEAR_VARS)清空模块描述变量，再写一个新的MODULE即可。

接下来，我们使用一个Android.mk，一次生成libalbert.a和libalbert.so。

本小节demo目录结构如下：

```
.
├── Android.mk
├── lib
└── src
    └── albert.c
```

albert.c内容如下：

```
#include <stdio.h>

int albert() {
	printf("This albert function!\n");
	return 0;
}
```

Android.mk内容如下：

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

编译后，生成两个文件：out/target/product/Hi3751V560/obj/STATIC_LIBRARIES/libalbert_intermediates/libalbert.a、lib/libalbert.so。

### 2.7 添加系统库

使用LOCAL_SHARED_LIBRARIES，可以将系统库添加到Android.mk中。

下面以ALOGE为例说明。

本小节demo目录结构如下：

```
.
├── Android.mk
├── bin
└── src
    └── main.c
```

main.c内容如下：

```
#include <stdio.h>
#include <utils/Log.h>

#ifdef LOG_TAG
#undef LOG_TAG
#define LOG_TAG "MAIN"
#endif

int main() {
	ALOGE("main function");
	return 0;
}
```

Android.mk内容如下：

```
LOCAL_PATH := $(call my-dir)

include $(CLEAR_VARS)

LOCAL_MODULE := aloge
LOCAL_MODULE_PATH := $(LOCAL_PATH)/bin

LOCAL_SHARED_LIBRARIES := liblog
LOCAL_SRC_FILES := src/main.c

include $(BUILD_EXECUTABLE)
```

### 2.8 添加第三方库

使用以下语句，可以添加第三方库：

```
LOCAL_LDFLAGS := $(LOCAL_PATH)/lib/libalbert.so
LOCAL_LDFLAGS := $(LOCAL_PATH)/lib/libalbert.a
```

本小节demo目录结构如下（libalbert.so是2.4小节编译生成的）：

```
.
|____lib
| |____libalbert.so
|____src
| |____main.c
|____inc
| |____albert.h
|____bin
|____Android.mk
```

albert.h内容如下：

```
#ifndef _ALBERT_H_
#define _ALBERT_H_

int albert();

#endif
```

main.c内容如下：

```
#include <stdio.h>
#include <albert.h>

int main() {
	albert();
	return 0;
}
```

Android.mk内容如下：

```
LOCAL_PATH := $(call my-dir)

include $(CLEAR_VARS)

LOCAL_MODULE := test
LOCAL_MODULE_PATH := $(LOCAL_PATH)/bin

LOCAL_LDFLAGS := $(LOCAL_PATH)/lib/libalbert.so
LOCAL_C_INCLUDES := $(LOCAL_PATH)/inc
LOCAL_SRC_FILES := src/main.c

include $(BUILD_EXECUTABLE)
```

使用2.5小节生成的静态库，也可以用这种方式引入，仅需将Android.mk中的名称改为libalbert.a即可。

### 2.9 编译jar包

**2.9.1 编译生成静态jar包**

静态jar包：使用.class文件打包生成的jar文件，可以在任何Java虚拟机运行。

使用以下指令，即可编译生成静态jar包（和静态库类似，即使指定LOCAL_MODULE_PATH，也是生成到out目录下）：

	include $(BUILD_STATIC_JAVA_LIBRARY)    # 编译生成静态jar包

本小节demo目录结构如下：

```
.
|____lib
|____src
| |____com
| | |____albert
| | | |____www
| | | | |____Albert.java
|____Android.mk
```

Albert.java内容如下：

```
package com.albert.www;

public class Albert {
	public static void albert() {
		System.out.println("This is albert function!");
	}
}
```

Android.mk内容如下：

```
LOCAL_PATH := $(call my-dir)

include $(CLEAR_VARS)

LOCAL_MODULE := com.albert.www
LOCAL_MODULE_PATH := $(LOCAL_PATH)/lib

LOCAL_SRC_FILES := $(call all-subdir-java-files)

include $(BUILD_STATIC_JAVA_LIBRARY)           # 编译生成静态jar包
```

**2.9.2 编译生成动态jar包**

动态jar包：在静态jar包基础之上使用.dex打包生成的jar文件，.dex是Android系统使用的文件格式。

使用以下指令，即可编译生成动态jar包：

    include $(BUILD_JAVA_LIBRARY)    # 编译生成动态jar包

本小节demo目录结构如下：

```
.
|____lib
|____src
| |____com
| | |____albert
| | | |____www
| | | | |____Albert.java
|____Android.mk
```

Albert.java内容如下：

```
package com.albert.www;

public class Albert {
	public static void albert() {
		System.out.println("This is albert function!");
	}
}
```

Android.mk内容如下：

```
LOCAL_PATH := $(call my-dir)

include $(CLEAR_VARS)

LOCAL_MODULE := com.albert.www
LOCAL_MODULE_PATH := $(LOCAL_PATH)/lib

LOCAL_SRC_FILES := $(call all-subdir-java-files)

include $(BUILD_JAVA_LIBRARY)    # 编译生成动态jar包
```

未完待续

————————————————

版权声明：本文为CSDN博主「Albert-陌尘」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。

原文链接：https://blog.csdn.net/murongmochen/article/details/120210812