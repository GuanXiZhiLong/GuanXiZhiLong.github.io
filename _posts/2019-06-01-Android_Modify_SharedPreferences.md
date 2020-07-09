---
title: "通过修改存档破解Android游戏元气骑士解锁内购人物"
excerpt: "通过修改存档破解Android游戏元气骑士解锁内购人物"
last_modified_at: 2018-06-02T12:00:00-14:40
header:
tags:
- Android
toc: true
---

# 免Root修改Android应用的Shared Preferences
## 准备工作
1. 一台启用调试模式的Android手机,无需root
2. 一台电脑(本文使用Linux操作系统)
3. 必备软件(如:adb openssl tar dd star等)

## 前言
本文以破解游戏元气骑士为例进行演示，破解思路来源于[此处](https://resources.infosecinstitute.com/android-hacking-security-part-15-hacking-android-apps-using-backup-techniques/)

## 具体步骤
1. 本文所述教程是在Linux平台进行的,请保证你的电脑有tar, dd, openssl, star组件,其中star从[此处](http://sourceforge.net/projects/adbextractor/)下载
2. 首先需要通过adb以备份的形式将元气骑士存档数据导出,命令如下:

    `adb backup com.ChillyRoom.DungeonShooter -f backup.ab`

    需要在手机上授权备份操作,其中com.ChillyRoom.DungeonShooter是元气骑士的包名,backup.ab是导出的存档的文件名
3. 上一步导出的.ab文件是Android备份文件,其中前24字节是文件头,我们需要用dd把前24字节剔除,这样剩下的就是压缩后的tar归档文件,再使用openssl进行解压,可以通过以下命令完成:

    `dd if=backup.ab bs=24 skip=1 | openssl zlib -d > backup.tar`

4. 现在我们得到了一个打包后的存档文件,现在需要用tar把所有文件的打包顺序记录下来,修改数据之后的打包流程需要用到,使用如下命令:

    `tar -tf backup.tar > backup.list`

    保存的list文件名为backup.list

5. 现在就可以将tar文件解压了

    `tar -xf backup.tar`

    解压完成之后会有一个app目录,下一级是游戏的包名命名的目录,再往下就是分类的所有存档数据,如下所示(一些本次教程用不到的文件被隐藏):
```
apps
    └── com.ChillyRoom.DungeonShooter
        ├── db
        ├── ef
        ├── f
        ├── r
        └── sp-------->表示Shared Preferences
            ├── admob.xml
            ├── appsflyer-data.xml
            ├── avidly_ads_analysis_key_dic_.xml
            ├── avidly_ads_sdk.xml
            ├── cbPrefs.xml
            ├── com.ChillyRoom.DungeonShooter_preferences.xml
            ├── com.ChillyRoom.DungeonShooter.v2.playerprefs.xml-------->此文件是需要修改的文件
            ├── com.facebook.ads.FEATURE_CONFIG.xml
            ├── com.facebook.internal.preferences.APP_SETTINGS.xml
            ├── com.facebook.sdk.appEventPreferences.xml
            ├── com.facebook.sdk.attributionTracking.xml
            ├── com.google.android.gms.analytics.prefs.xml
            ├── com.google.android.gms.appid.xml
            ├── com.google.android.gms.measurement.prefs.xml
            ├── com.google.android.gms.signin.xml
            ├── com.vungle.sdk.xml
            ├── DEBUG_PREF.xml
            ├── FBAdPrefs.xml
            ├── ktplay.xml
            ├── SDKIDFA.xml
            └── WebViewChromiumPrefs.xml
```
6. 如上所示,我们打开`com.ChillyRoom.DungeonShooter.v2.playerprefs.xml`文件,根据各个变量的名字我们可以看出一些含意:
```
    <string name="c13_unlock">false</string> 变量名中的c为character角色的缩写,这个变量的意义是13号角色是否解锁,这样我们可以通过把指改为true来达到解锁角色的目的

    <int name="c12_skin3" value="-1" /> 同理,此处应该为12号角色的3号皮肤,对于值的分析我个人认为如果是2000这样的数值应该为需要花费宝石解锁,为0时是需要花费现金解锁,为-1时是尚未推出的皮肤,为1则为已解锁.这里请自行修改
    
    <int name="last_gems" value="2123" />
    <int name="gems" value="2441" /> 这里两个变量是宝石数量,可以自行修改想要的数值
    
    其他数据以及其他配置文件的数据请自行发挥,本文不作分析(留作业)
```
7. 在上一步修改完数据后接下来要做的就是将修改完成的数据导回去,需要用到star工具,下载链接上面已经提到过,命令如下:

    `star -c -v -f newbackup.tar -no-dirslash list=backup.list`

    用到了之前导出的列表文件,打包的新文件名为newbackup.tar

8. 现在需要用dd把之前剔除的Android备份文件的文件头取出来,使用如下命令:

    `dd if=backup.ab bs=24 count=1 of=newbackup.ab`

    把原备份文件backup.ab的文件头保存为newbackup.ab

9. 再把修改过的存档文件压缩,前面拼上文件头:

    `openssl zlib -in newbackup.tar >> newbackup.ab`

10. 大功告成,新的备份文件打包完成,最后一步,把修改过的备份文件newbackup.ab通过adb恢复回去:

    `adb restore newbackup.ab`

11. 这个时候就可以打开你的游戏查看修改的成果啦!Happy Hacking!
