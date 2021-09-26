
# ADB通过WIFI连接Android设备 #

ADB支持USB连接模式和TCPIP链接模式。我们可以用TCPIP模式通过WIFI无线连接ADB。设置非常简单。

**第一步**

确保电脑和Android设备连接在同一个WIFI网络环境。

**第二步**

用USB线连接Android设备。连接上之后你的电脑就会检查到设备并且ADB将会以USB模式启动。可以通过adb devices命令检查连接上的设备，用adb usb命令确认adb是运行在usb模式下面。

```
$ adb devices
List of devices attached
602001510014ba1f7155    device

$ adb usb
restarting in USB mode
```

**第三步**

用adb tcpip模式重启adb

```
$ adb tcpip 5555
restarting in TCP mode port: 5555
```

**第四步**

查看Android设备的IP地址，这里有三种方式查看Android设备IP。

设置－关于手机－状态信息－ip地址中查看

设置－WLAN-点击当前链接上的Wi-Fi查看IP

通过ADB命令查看设备IP地址：`adb shell netcfg`

**第五步**

知道设备IP地址之后，就可以用adb connect命令通过IP和端口号连接ADB了。

```
$ adb connect 192.168.101.200:5555
connected to 192.168.101.200:5555

$ adb devices
List of devices attached
602001510014ba1f7155    device
192.168.101.200:5555    device
```

看一下连接上的设备，usb连接和wifi连接都存在

拔掉USB线，你会发现设备仍然是连接上的，如果没有连接上，用刚才的命令重现尝试一下。

总结

采用wifi连接ADB和uiautomotor结合起来可以用来在usb线的状态下跑测试脚本，对于测试人员来说也是非常有帮助的。

常用命令

    adb connect 192.168.101.200
    adb devices
    adb disconnect 192.168.101.200


作者：如水至清

链接：https://www.jianshu.com/p/37901e3f7667

来源：简书

著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
