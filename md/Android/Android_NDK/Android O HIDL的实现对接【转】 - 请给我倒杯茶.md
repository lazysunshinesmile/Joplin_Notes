Android O HIDL的实现对接【转】 - 请给我倒杯茶

Android O HIDL的实现对接  
1. HIDL的定义  
1.1. 关于Android更新  
2. HIDL处于系统哪个部位及怎么通信的  
2.1. Android 系统架构包含以下组件  
2.2. HAL的类型  
3. HIDL的实现  
4. HIDL版本维护  
5. 实例对接HIDL完整过程  
5.1. 新功能接口的添加  
5.2. 修改HIDL及HAL层文件  
5.2.1. HIDL文件修改  
5.2.2. HAL文件生成  
5.2.2. HAL层与HIDL的实现对接  
5.3. HAL层与framework层对接  
5.4. 应用层接口的添加  
5.5. 验证功能  
Android O HIDL的实现对接  
Google对于HIDL的详细说明，以及语法解析链接如下(ps: 需要翻墙才可以打开)  
https://source.android.com/devices/architecture/hidl/  
以下是个人学习整理的资料，以及实做的一个例子

1. HIDL的定义  
HIDL 读作 hide-l，全称是 Hardware Interface Definition Language。它在 Android Project Treble 中被起草，在 Android 8.0 中被全面使用。其诞生目的是，框架可以在无需重新构建 HAL 的情况下进行替换。HAL将由供应商或SOC 制造商构建，放置在设备的 /vendor 分区中，这样一来，框架就可以在其自己的分区中通过 OTA 进行替换，而无需重新编译 HAL

1.1. 关于Android更新  
利用新的供应商接口，Project Treble 将供应商实现（由芯片制造商编写的设备专属底层软件）与 Android 操作系统框架分离开来。Android 7.x 及更早版本中没有正式的供应商接口，因此设备制造商必须更新大量 Android 代码才能将设备更新到新版 Android 系统。

 图 1. Treble 推出前的 Android 更新环境

Treble 提供了一个稳定的新供应商接口，供设备制造商访问 Android 代码中特定于硬件的部分，这样一来，设备制造商只需更新 Android 操作系统框架，即可跳过芯片制造商直接提供新的 Android 版本：

 图 2. Treble 推出后的 Android 更新环境

2. HIDL处于系统哪个部位及怎么通信的  
2.1. Android 系统架构包含以下组件

2.2. HAL的类型  
为了更好地实现模块化，Android 8.0 对 Android 操作系统底层进行了重新架构。作为此变化的一部分，运行 Android 8.0 的设备必须支持绑定式或直通式 HAL：  
• 绑定式 HAL。以 HAL 接口定义语言 (HIDL) 表示的 HAL。这些 HAL 取代了早期 Android 版本中使用的传统 HAL 和旧版 HAL。在绑定式 HAL 中，Android 框架和 HAL 之间通过 Binder 进程间通信 (IPC) 调用进行通信。所有在推出时即搭载了 Android 8.0 或后续版本的设备都必须只支持绑定式 HAL。  
• 直通式 HAL。以 HIDL 封装的传统 HAL 或旧版 HAL。这些 HAL 封装了现有的 HAL，可在绑定模式和 Same-Process（直通）模式下使用。升级到 Android 8.0 的设备可以使用直通式 HAL。

3. HIDL的实现  
Android O 对 Android 操作系统的架构重新进行了设计，以在独立于设备的 Android 平台与特定于设备和供应商的代码之间定义清晰的接口。Android 已经以 HAL 接口的形式（在 hardware/libhardware 中定义为 C 标头）定义了许多此类接口。HIDL 将这些 HAL 接口替换为稳定的带版本接口，它们可以是采用 C++（如下所述）或 Java 的客户端和服务器端 HIDL 接口。

HIDL 接口具有客户端和服务器实现：  
• HIDL 接口的客户端实现是指通过在该接口上调用方法来使用该接口的代码  
• 服务器实现是指 HIDL 接口的实现，它可接收来自客户端的调用并返回结果（如有必要）

在从 libhardware HAL 转换为 HIDL HAL 的过程中，HAL 实现成为服务器，而调用 HAL 的进程则成为客户端。默认实现可提供直通和绑定式 HAL，并可能会随着时间而发生变化，如下所示：

4. HIDL版本维护  
HIDL 要求每个使用 HIDL 编写的接口均必须带有版本编号。HAL 接口一经发布便会被冻结，如果要做任何进一步的更改，都只能在接口的新版本中进行。虽然无法对指定的已发布接口进行修改，但可通过其他接口对其进行扩展。

5. 实例对接HIDL完整过程  
因为要google规定，已经发的hal版本是不能再更改的，除非再update成新的版本，而在porting的平台上，几乎都没有遵守这个规定，只是在原先的基础版本上去update而已，把要修改的文件进行重新继承整理，没更新版本。但这样却也可以过google的测试。。。  
而对于新加的可以修改p2p group密码的接口，这个是一个新的接口，所以，也没有重新生成新的版本去update。下面让我们一起来看下这个hidl新接口的添加过程

5.1. 新功能接口的添加  
因为这个是添加了一个新的接口功能，所以在wpa_supplicant中添加了可以修改p2p group密码的接口，新加接口记为  
p2p_set_passphrase()
```shell
# wpa_cli -ip2p0 -p /data/misc/wifi/sockets/  
> SET p2p_pwd “12345678”  
<3>P2P-GROUP-STARTED p2p-p2p0-1 GO ssid="DIRECT-ac-Android_e8q7" freq=2472 passphrase="12345678" go_dev_addr=01:bc:43:cf:04:44 [PERSISTENT]  
<3>AP-STA-CONNECTED 4a:d5:dd:5a:74:14 p2p_dev_addr=4a:d5:dd:5a:74:14”  
````
5.2. 修改HIDL及HAL层文件  
5.2.1. HIDL文件修改  
`wpa_supplicant_8/wpa_supplicant/hidl/1.0`    
因为修改的是p2p相关部分，所以，可以直接查看p2p_iface.cpp，按其中的接口setSsidPostfix()进行类似添加即可，修改密码接口名为setPasswdPostfix()

5.2.2. HAL文件生成  
`hardware/interfaces/wifi/supplicant/1.0`  
如果此时把HIDL中的接口setPasswdPostfix()添加到HAL中的ISupplicantP2pIface.hal中，当build code时，系统会提示frozen的相关错误，说明这个文件的接口是不能添加，以及修改的。如果要修改，只能第三方厂家自己重新实现一套HAL接口。  
而第三方Icecream，可以在vendor/Icecream/hardware/interfaces下面去创建自己对应的目录实现HAL层的对接。

所以，对于第三方来说，我们得重新创建目录了，如

`vendor/Icecream/hardware/interfaces/wifi/supplicant/1.0`
同时把要修改的HAL文件进行修改，因为我们只用到了ISupplicantP2pIface.hal文件，而我们还是要用到之前hardware/interfaces/wifi/supplicant/1.0目录中的其他文件，如此，我们不用直接使用ISupplicantP2pIface.hal，而是要对这个文件进行继承的方式进行，创建对应的IIcecreamSupplicantP2pIface.hal来继承ISupplicantP2pIface.hal，如下:
```shell
package Icecream.hardware.wifi.supplicant@1.0;

import android.hardware.wifi.supplicant@1.0::ISupplicantP2pIface;

/**  
 \* Interface exposed by the supplicant for each P2P mode network  
 \* interface (e.g p2p0) it controls.  
 */

interface IIcecreamSupplicantP2pIface  
    extends android.hardware.wifi.supplicant@1.0::ISupplicantP2pIface {  
  /**  
   \* Set the postfix to be used for P2P Passwd's.  
   *  
   \* @param postfix String to be appended to Passwd.  
   \* @return status Status of the operation.  
   *         Possible status codes:  
   *         |SupplicantStatusCode.SUCCESS|,  
   *         |SupplicantStatusCode.FAILURE_ARGS_INVALID|,  
   *         |SupplicantStatusCode.FAILURE_UNKNOWN|,  
   *         |SupplicantStatusCode.FAILURE_IFACE_INVALID|  
   */  
  setPasswdPostfix(vec<uint8_t> postfix) generates (SupplicantStatus status);  
};  
```
如上，我们已经把Hal文件创建好了，那接下来就得创建Android.mk跟Android.bp文件了。  
而此时，我们只要先source、lunch一下后，再执行一下  
vendor/Icecream/hardware/interfaces/update-makefiles.sh即可以在生成对应的Android.mk跟Android.db了。  
update-makefiles.sh内容如下：
```shell
#!/bin/bash

source system/tools/hidl/update-makefiles-helper.sh

do_makefiles_update \  
    "Icecream.hardware:vendor/Icecream/hardware/interfaces/" \  
    "android.hardware:hardware/interfaces" \  
    "android.hidl:system/libhidl/transport"  
``` 
update-makefiles.sh内容的写法，可以参考hardware/interfaces/update-makefiles.sh，再添加自己平台对应的hal目录路径即可

在执行了update-makefiles.sh后，你可以看到自己的目录会多出了Android.bp、Android.mk
```shell
vendor/Icecream/hardware/interfaces$ tree wifi  
wifi  
├── Android.bp  
└── supplicant  
    └── 1.0  
        ├── Android.bp  
        ├── Android.mk  
        └── IIcecreamSupplicantP2pIface.hal  
``` 
对于以上的Android.mk中，这个要对应其父类  
hardware/interfaces/wifi/supplicant/1.0/Android.mk的写法，注意把父类中的对应LIBRARIES在这个子类中都加上去，避免后面要build code error，要再重新检查一下，还会有让你莫名其妙的错误，让你无从下手，这点要注意了。  
所以，在Android.mk中，还要在LOCAL_JAVA_LIBRARIES与LOCAL_STATIC_JAVA_LIBRARIES中添加父类依赖的  
android.hidl.base-V1.0-java这个lib进去

这时，HAL目前就已经生成了，我们可以直接在这个目录mm，让其在out目录下生成对应的java跟cpp文件，以借framework中的java调用，以及hidl层中cpp调用

5.2.2. HAL层与HIDL的实现对接  
以上第二步已经把HAL层生成，以及对应文件也生成了，接下来就得进行把hal层跟hidl层对接起来了。  
这时在hidl层中，原生在使用ISupplicantP2pIface这个类的以及其对应命名空间的，得全部改成IIcecreamSupplicantP2pIface的来继承了，这时会出现很多error，所以，接下来，得针对这些error一个一个解决，最后让hidl可以build过

5.3. HAL层与framework层对接  
经过上面5.1、5.2，已经把HIDL的实现跟hal层对接起来了，接下来把hal层跟framework对接起来，并向上层应用提供接口调用
```shell
frameworks/opt/net/wifi/service/java/com/android/server/wifi/p2p  
frameworks/base/wifi/java/android/net/wifi/p2p  
``` 
同5.2中的(3)一样，把SupplicantP2pIfaceHal.java中原先对应的ISupplicantP2pIface类全部改成IIcecreamSupplicantP2pIface，再仿照setSsidPostfix()接口实现对应要添加的接口setPasswdPostfix()，然后一层一层往上加

`SupplicantP2pIfaceHal.java-->WifiP2pNative.java-->WifiP2pServiceImpl.java-->WifiP2pManager.aidl-->WifiP2pManager.java`

5.4. 应用层接口的添加  
在TvSetting中去添加对应的接口，对于这个问题，目前是加在点击wifi direct中的搜索时，就会去设置group的密码，这样，当连接时，group就可以直接使用设置过的密码进行连接了
```shell
 diff --git a/Settings/src/com/android/tv/settings/connectivity/p2p/WifiP2pSettings.java b/Settings/src/com/android/tv/settings/connectivity/p2p/WifiP2pSettings.java  
index 2488272..91d486d 100644  
--- a/Settings/src/com/android/tv/settings/connectivity/p2p/WifiP2pSettings.java  
+++ b/Settings/src/com/android/tv/settings/connectivity/p2p/WifiP2pSettings.java  
@@ -620,6 +620,9 @@ public class WifiP2pSettings extends SettingsPreferenceFragment  
                 }  
             });  
         }  
+  
+        Log.d(TAG, "startSearch set p2p passwd");  
+        mWifiP2pManager.setP2pPasswd("12348765");  
     }

     private void updateDevicePref() {  
``` 
5.5. 验证功能  
1）wifi打开后  
2）在命令行执行以下操作，用于观察建立group时的passwd是否是正确的  
```
# wpa_cli -ip2p0 -p /data/misc/wifi/sockets/
```shell
3）进入wifi Direct中，点击搜索  
4）用手机进行Wifi Direct直连，或者手机进行Miracast连接。  
5）观察刚刚串口中的打印信息passphrase，是否跟你自己设定的正确  
<3>P2P-GROUP-STARTED p2p-p2p0-2 GO ssid=”DIRECT-ac-Android_e8q7” freq=2472 passphrase=”12348765” go_dev_addr=01:bc:43:cf:04:44 [PERSISTENT]  
<3>AP-STA-CONNECTED 4a:d5:dd:5a:74:14 p2p_dev_addr=4a:d5:dd:5a:74:14”

以上的passphrase=”12348765”，跟在TvSetting中添加的代码是一样，说明已经设定成功了

以上的HIDL对接流程，对以后再进行相关方面的接口添加，减少以后相关porting新接口的操作，或要再重新从1.0 –>1.1当参考用