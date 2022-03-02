---
cover: /img/flutter.jpeg
---


最近发现一个坑：
睡觉的时候苹果手机自动进行更新。导致设备已升级为15.3版本

而Xcode支持的设备包。只支持到15.1。因此 flutter devices 无法读取到真机设备。


解决方法：


第一步：
mac在访达中找到xcode 右键显示包内容
一直进入到目录：
/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/DeviceSupport

发现缺少15.3版本的包

第二步：
https://github.com/filsv/iPhoneOSDeviceSupport
去下载对应版本的包

第三步
把包解压后拷贝到上面路径。

第四步
重启电脑

第五步
重启手机

