
# adb shell input keyevent 控制按键输入的数值 #

adb shell的功能很强大，可以使用很多功能，今天我们说下通过控制按键输入：`adb shell input keyevent xx` ，具体数值xx如下


    KEYCODE_CALL 进入拨号盘 5
    KEYCODE_ENDCALL 挂机键 6
    KEYCODE_HOME 按键Home 3
    KEYCODE_MENU 菜单键 82
    KEYCODE_BACK 返回键 4
    KEYCODE_SEARCH 搜索键 84
    KEYCODE_CAMERA 拍照键 27
    KEYCODE_FOCUS 拍照对焦键 80
    KEYCODE_POWER 电源键 26
    KEYCODE_NOTIFICATION 通知键 83
    KEYCODE_MUTE 话筒静音键 91
    KEYCODE_VOLUME_MUTE 扬声器静音键 164
    KEYCODE_VOLUME_UP 音量增加键 24
    KEYCODE_VOLUME_DOWN 音量减小键 25


控制键


    KEYCODE_ENTER 回车键 66
    KEYCODE_ESCAPE ESC键 111
    KEYCODE_DPAD_CENTER 导航键 确定键 23
    KEYCODE_DPAD_UP 导航键 向上 19
    KEYCODE_DPAD_DOWN 导航键 向下 20
    KEYCODE_DPAD_LEFT 导航键 向左 21
    KEYCODE_DPAD_RIGHT 导航键 向右 22
    KEYCODE_MOVE_HOME 光标移动到开始键 122
    KEYCODE_MOVE_END 光标移动到末尾键 123
    KEYCODE_PAGE_UP 向上翻页键 92
    KEYCODE_PAGE_DOWN 向下翻页键 93
    KEYCODE_DEL 退格键 67
    KEYCODE_FORWARD_DEL 删除键 112
    KEYCODE_INSERT 插入键 124
    KEYCODE_TAB Tab键 61
    KEYCODE_NUM_LOCK 小键盘锁 143
    KEYCODE_CAPS_LOCK 大写锁定键 115
    KEYCODE_BREAK Break/Pause键 121
    KEYCODE_SCROLL_LOCK 滚动锁定键 116
    KEYCODE_ZOOM_IN 放大键 168
    KEYCODE_ZOOM_OUT 缩小键 169


利用命令“`adb shell input keyevent <键值>`”可以实现自动化。例如“adb shell input keyevent 3”就可以按下Home键。]

    执行返回：adb shell input keyevent 4
    执行灭屏亮屏：adb shell input keyevent 26
    执行解锁屏幕：adb shell input keyevent 82

总结

以上所述是小编给大家介绍的adb shell input keyevent 控制按键输入的数值,希望对大家有所帮助，如果大家有任何疑问请给我留言，小编会及时回复大家的。在此也非常感谢大家对脚本之家网站的支持！

如果你觉得本文对你有帮助，欢迎转载，烦请注明出处，谢谢！