# Android tcpdump TCP 抓包 #

## 常用抓取命令 ##

```
adb shell rm /sdcard/capture.pcap
adb shell  /data/local/tcpdump -i any -p -s 0 -w /sdcard/capture.pcap

adb pull /sdcard/capture.pcap capture.pcap
```

## 安装 ##

**使用准备**

- 设备需要root权限
- tcpdump 二进制文件
- wireshark 分析工具

https://www.wireshark.org/

tcpdump for android 说明

http://www.androidtcpdump.com/

**安装tcpdump到设备**

adb shell, su获得root权限

tcpdump 需要在命令行运行目录中存在

```
adb push tcpdump /data/local/tcpdump
adb shell chmod 6755 /data/local/tcpdump
```

**使用 tcpdump**

```
cd /data/local
./tcpdump -i any -p -s 0 -w /sdcard/capture.pcap
```

拉取抓获的tcp/udp包

	adb pull /sdcard/capture.pcap

用wireshark打开capture.pcap即可分析log

tcpdump 参数说明

```
        # "-i any": listen on any network interface
　　# "-p": disable promiscuous mode (doesn't work anyway)
　　# "-s 0": capture the entire packet
　　# "-w": write packets to a file (rather than printing to stdout)
　　... do whatever you want to capture, then ^C to stop it ...
```

## 错误处理 ##

**Android5.0系统下用tcpdump抓包失败**

在Android5.0系统下用tcpdump抓包失败，但是在5.0之前的系统上可以正常抓包

	error: only position independent executables (PIE) are supported.

这是由于PIE安全机制所引起的，从Android4.1开始引入该机制

PIE机制它会随机分配程序的内存地址从而令攻击者更难发现程序的溢出漏洞

PIE机制详细介绍 https://en.wikipedia.org/wiki/Position-independent_code

作者：木猫尾巴

链接：https://www.jianshu.com/p/ca6cdc825ad3

来源：简书

著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。