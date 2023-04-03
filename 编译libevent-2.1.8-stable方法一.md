编译libevent-2.1.8-stable需要先在Android NDK上设置交叉编译环境，然后使用交叉编译工具链进行编译。

以下是编译libevent-2.1.8-stable的步骤：

1. 下载libevent-2.1.8-stable源代码并解压缩。
2. 打开终端，进入libevent-2.1.8-stable目录。
3. 设置交叉编译环境，指定NDK路径和架构，例如：

```
bashCopy codeexport NDK_ROOT=/path/to/ndk
export ARCH=arm
export TOOLCHAIN=$NDK_ROOT/toolchains/llvm/prebuilt/linux-x86_64
export SYSROOT=$NDK_ROOT/toolchains/llvm/prebuilt/linux-x86_64/sysroot
export CROSS_PREFIX=$TOOLCHAIN/bin/$ARCH-linux-androideabi-
export CC=$CROSS_PREFIX$clang
export CXX=$CROSS_PREFIX$clang++
```

其中，NDK_ROOT需要替换为实际的NDK路径，ARCH需要根据目标设备架构设置。

1. 运行configure脚本生成Makefile文件：

```
bashCopy code
./configure --host=$ARCH-linux-androideabi --prefix=$PWD/build
```

其中，--host选项需要根据目标设备架构设置，--prefix选项用于指定编译输出目录。

1. 使用make命令编译libevent库：

```
goCopy code
make -j8
```

其中，-j选项用于指定并行编译的线程数，可以根据系统CPU核数设置。

1. 使用make命令安装libevent库到指定目录：

```
goCopy code
make install
```

安装完成后，编译生成的库文件会存放在build/lib目录下。

注意：编译过程中可能会遇到各种错误，需要根据实际情况解决。同时，由于不同的NDK版本和架构可能存在差异，以上步骤仅供参考，具体操作还需根据实际情况调整。