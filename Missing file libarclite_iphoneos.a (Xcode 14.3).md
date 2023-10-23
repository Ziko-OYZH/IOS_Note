# Missing file libarclite_iphoneos.a (Xcode 14.3)

```
File not found: /Applications/Xcode-beta.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/lib/arc/libarclite_iphoneos
```

**报错原因：**libarclite_iphoneos文件缺失

在升级Xcode14.3后，由于CocoaPods引入第三方库



推荐解决方案：

```shell
cd /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/lib/
sudo mkdir arc
cd  arc
sudo git clone https://github.com/kamyarelyasi/Libarclite-Files.git
```



或访问

```
https://github.com/kamyarelyasi/Libarclite-Files
```

手动下载文件并拽入arc文件夹下