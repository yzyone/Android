# Tutorial: Compile OpenSSL 1.1.1 for Android application #

OpenSSL is a software library for applications that secure communications over computer networks against eavesdropping or need to identify the party at the other end. It is widely used by Internet servers, including the majority of HTTPS websites.

OpenSSL is also used in some Android applications that require cryptography functions. Usually, Android developers use the Native Development Kit (NDK) to compile the source code into a native library and package it into your APK.

Why upgrade OpenSSL to 1.1.1

OpenSSL version 1.1.1 series are released in September 2018, along with that, the Long Term Support (LTS) version is also updated. Here is the brief from the OpenSSL website:

- Version 1.1.1 series are the latest LTS, supported until 11th September 2023
- Version 1.0.2 series (released January 2015) will continue to be supported until 31st December 2019
- Version 1.1.0 series (released August 2016) will continue to be supported until 11th September 2019
- The 0.9.8, 1.0.0 and 1.0.1 versions are now out of support and should not be used.

In other words, if you started using OpenSSL in your project several years ago and didn’t actively update the library, you probably run into the situation that the library you used will go out of support sometimes in 2019. Versions that out of support won’t receive security fixes anymore.

All users of 1.0.2 and 1.1.0 are encouraged to upgrade to 1.1.1 as soon as possible.

This article will provide instructions for building the latest OpenSSL library version 1.1.1c for Android devices.

What do we need

In the previous version of OpenSSL, there are quite a lot of configurations to write to compile the source code into a native library. In the latest OpenSSL version 1.1.1, the build process has never been as easy as today. Here are some tools we need:

- A build machine (MacOs in this article)
- NDK version r20
- OpenSSL 1.1.1c

Step by step

1. Download and prepare

Download and unzip the NDK package into a directory: `https://developer.android.com/ndk/downloads/index.html`

Download and unzip the OpenSSL 1.1.1c into the same directory: `https://www.openssl.org/source/`

Set variables ANDROID_NDK_HOME and OPENSSL_DIR.

```
# Set directory
SCRIPTPATH=`realpath .`
export ANDROID_NDK_HOME=$SCRIPTPATH/android-ndk-r20
OPENSSL_DIR=$SCRIPTPATH/openssl-1.1.1c
```

2. Find the toolchain for your build machine

If using r19 or newer, the NDK’s default toolchains are standalone toolchains, the toolchains installed by default with the NDK may be used in place. All we need to do is to find and tell OpenSSL the corresponding toolchain for your build machine.

make_standalone_toolchain.py script is no longer needed for interfacing with arbitrary build systems

	toolchains_path=$(python toolchains_path.py --ndk ${ANDROID_NDK_HOME})

Take away script for toolchains_path.py

```
#!/usr/bin/env python
"""
    Get the toolchains path
"""
import argparse
import atexit
import inspect
import os
import shutil
import stat
import sys
import textwrap

def get_host_tag_or_die():
    """Return the host tag for this platform. Die if not supported."""
    if sys.platform.startswith('linux'):
        return 'linux-x86_64'
    elif sys.platform == 'darwin':
        return 'darwin-x86_64'
    elif sys.platform == 'win32' or sys.platform == 'cygwin':
        host_tag = 'windows-x86_64'
        if not os.path.exists(os.path.join(NDK_DIR, 'prebuilt', host_tag)):
            host_tag = 'windows'
        return host_tag
    sys.exit('Unsupported platform: ' + sys.platform)


def get_toolchain_path_or_die(ndk, host_tag):
    """Return the toolchain path or die."""
    toolchain_path = os.path.join(ndk, 'toolchains/llvm/prebuilt',
                                  host_tag)
    if not os.path.exists(toolchain_path):
        sys.exit('Could not find toolchain: {}'.format(toolchain_path))
    return toolchain_path

def main():
    """Program entry point."""
    parser = argparse.ArgumentParser(description='Optional app description')
    parser.add_argument('--ndk', required=True,
                    help='The NDK Home directory')
    args = parser.parse_args()

    host_tag = get_host_tag_or_die()
    toolchain_path = get_toolchain_path_or_die(args.ndk, host_tag)
    print toolchain_path

if __name__ == '__main__':
    main()
```

3. Configure the OpenSSL environment

Reference can be found from NOTES.ANDROID in OPENSSL_DIR.

```
# Set compiler clang, instead of gcc by default
CC=clang
# Add toolchains bin directory to PATH
PATH=$toolchains_path/bin:$PATH
# Set the Android API levels
ANDROID_API=21
# Set the target architecture
# Can be android-arm, android-arm64, android-x86, android-x86 etc
architecture=android-arm
```

4. Create the make file

```
# Create the make file
cd ${OPENSSL_DIR}
./Configure ${architecture} -D__ANDROID_API__=$ANDROID_API
```

5. Build

```
# Build
make
```

6. Copy the outputs

The outputs we need are the includes files, and the .so and .a files.

```
# Copy the outputs
OUTPUT_INCLUDE=$SCRIPTPATH/output/include
OUTPUT_LIB=$SCRIPTPATH/output/lib/${architecture}
mkdir -p $OUTPUT_INCLUDE
mkdir -p $OUTPUT_LIB
cp -RL include/openssl $OUTPUT_INCLUDE
cp libcrypto.so $OUTPUT_LIB
cp libcrypto.a $OUTPUT_LIB
cp libssl.so $OUTPUT_LIB
cp libssl.a $OUTPUT_LIB
```

Take away

Finally, compiling the OpenSSL library for Android application has never been as convenient as today. If you have been working with the legacy version of the OpenSSL library, and you need to upgrade to the latest version, this article can help you in compiling the source code into a native library.

We come out with the following script to compile the latest OpenSSL library version 1.1.1c for Android devices.

Enjoy working with OpenSSL and NDK.

```
#!/bin/bash
set -e
set -x

# Set directory
SCRIPTPATH=`realpath .`
export ANDROID_NDK_HOME=$SCRIPTPATH/android-ndk-r20
OPENSSL_DIR=$SCRIPTPATH/openssl-1.1.1c

# Find the toolchain for your build machine
toolchains_path=$(python toolchains_path.py --ndk ${ANDROID_NDK_HOME})

# Configure the OpenSSL environment, refer to NOTES.ANDROID in OPENSSL_DIR
# Set compiler clang, instead of gcc by default
CC=clang

# Add toolchains bin directory to PATH
PATH=$toolchains_path/bin:$PATH

# Set the Android API levels
ANDROID_API=21

# Set the target architecture
# Can be android-arm, android-arm64, android-x86, android-x86 etc
architecture=android-arm

# Create the make file
cd ${OPENSSL_DIR}
./Configure ${architecture} -D__ANDROID_API__=$ANDROID_API

# Build
make

# Copy the outputs
OUTPUT_INCLUDE=$SCRIPTPATH/output/include
OUTPUT_LIB=$SCRIPTPATH/output/lib/${architecture}
mkdir -p $OUTPUT_INCLUDE
mkdir -p $OUTPUT_LIB
cp -RL include/openssl $OUTPUT_INCLUDE
cp libcrypto.so $OUTPUT_LIB
cp libcrypto.a $OUTPUT_LIB
cp libssl.so $OUTPUT_LIB
cp libssl.a $OUTPUT_LIB
```