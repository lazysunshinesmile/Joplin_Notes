AIDL 实际使用问题

# 内部类的使用
1. aidl文件
`P2pConfiguration.aidl`
	```
	package com.gs.settings.entity;

	parcelable P2pConfiguration;
	parcelable P2pConfiguration.WpsInfo;
	```

2. java文件
	**注意必须是静态内部类**
	``` java
	package android.net.wifi.p2p;


	import android.os.Parcel;
	import android.os.Parcelable;

	/**
	 * @en P2pConfiguration
	 * @zh P2pConfiguration
	 * @device am3x00
	 * @crestronApi
	 */
	public class P2pConfiguration implements Parcelable {

		protected P2pConfiguration(Parcel in) {
			mDeviceAddress = in.readString();
			mGroupOwnerIntent = in.readInt();
		}

		public static final Creator<P2pConfiguration> CREATOR = new Creator<P2pConfiguration>() {
			@Override
			public P2pConfiguration createFromParcel(Parcel in) {
				return new P2pConfiguration(in);
			}

			@Override
			public P2pConfiguration[] newArray(int size) {
				return new P2pConfiguration[size];
			}
		};

		@Override
		public int describeContents() {
			return 0;
		}

		@Override
		public void writeToParcel(Parcel dest, int flags) {
			dest.writeString(mDeviceAddress);
			dest.writeInt(mGroupOwnerIntent);
		}

		/**
		 * @en P2pConfiguration
		 * @zh P2pConfiguration
		 * @device am3x00
		 * @crestronApi
		 */
		public static class  WpsInfo implements Parcelable  {


			/**
			 * @en Push button configuration
			 * @zh Push button configuration
			 * @device am3x00
			 * @crestronApi
			 */
			public static final int PBC     = 0;

			/**
			 * @en Display pin method configuration - pin is generated and displayed on device
			 * @zh Display pin method configuration - pin is generated and displayed on device
			 * @device am3x00
			 * @crestronApi
			 */
			public static final int DISPLAY = 1;

			/**
			 * @en Keypad pin method configuration - pin is entered on device
			 * @zh Keypad pin method configuration - pin is entered on device
			 * @device am3x00
			 * @crestronApi
			 */
			public static final int KEYPAD  = 2;

			/**
			 * @en Label pin method configuration - pin is labelled on device
			 * @zh Label pin method configuration - pin is labelled on device
			 * @device am3x00
			 * @crestronApi
			 */
			public static final int LABEL   = 3;

			/**
			 * @en setup
			 * @zh setup
			 * @device am3x00
			 * @crestronApi
			 */
			public int setup;


			/**
			 * @en Passed with pin method configuration
			 * @zh Passed with pin method configuration
			 * @device am3x00
			 * @crestronApi
			 */
			public String pin;

			public WpsInfo() {
			}

			protected WpsInfo(Parcel in) {
				setup = in.readInt();
				pin = in.readString();
			}

			public static final Creator<WpsInfo> CREATOR = new Creator<WpsInfo>() {
				@Override
				public WpsInfo createFromParcel(Parcel in) {
					return new WpsInfo(in);
				}

				@Override
				public WpsInfo[] newArray(int size) {
					return new WpsInfo[size];
				}
			};

			@Override
			public int describeContents() {
				return 0;
			}

			@Override
			public void writeToParcel(Parcel dest, int flags) {
				dest.writeInt(setup);
				dest.writeString(pin);
			}
		}

		/**
		 * @en Device Address
		 * @zh Device Address;
		 * @device am3x00
		 * @crestronApi
		 */
		public String mDeviceAddress;

		/**
		 * @en Group Owner Intent
		 * @zh Group Owner Intent
		 * @device am3x00
		 * @crestronApi
		 */
		public int mGroupOwnerIntent = -1;

		/**
		 * @en WpsInfo
		 * @zh WpsInfo
		 * @device am3x00
		 * @crestronApi
		 */
		public WpsInfo mWpsInfo = new WpsInfo();
	}
	```

# bp文件编写，不用写parcelable的类
1. `bp文件`
	```bp
	srcs: ["java/**/*.java"] + [
        "aidl/com/gs/settings/ISettingPhoneManager.aidl",
        "aidl/com/gs/settings/listener/ISettingPhoneListener.aidl",
        "aidl/com/gs/settings/IWifiConfigManager.aidl",
        "aidl/com/gs/settings/ISecuritySettingsManager.aidl",
        "aidl/com/gs/settings/IDisplaySettingsManager.aidl",
        "aidl/com/gs/settings/receiver/WifiApEventsReceiver.aidl",
        "aidl/com/gs/settings/receiver/WifiDirectEventsReceiver.aidl",
        "aidl/com/gs/settings/entity/ApConfiguration.aidl",
        "aidl/com/gs/settings/entity/P2pConfiguration.aidl"
    ],
	```
	还有一些aidl并没有写入，因为这些aidl是实现了Parcelable，已经存在与同名包下，不需要系统帮助实现。编译的时候会先去同名包下面查看是否有这个类，有就不自主创建，没有再去创建。
2. `WifiApEventsReceiver.aidl`
	```
	package com.gs.settings.receiver;
	import com.gs.settings.entity.StaDevice;

	interface WifiApEventsReceiver {
		 void onWifiApStarted();
		 void onWifiAPStopped();
		 void onWifiApClientLists(out List<StaDevice> staDevices);
	}
	```

# in、out、inout 
参考[你真的理解AIDL中的in，out，inout么](../../Android/Android_NDK/你真的理解AIDL中的in，out，inout么.md)

# 类型
AIDL中支持的数据类型有：	 	 
 	 	 
|支持类型|	需要import?|	备注|
|--|--|--|
|Java基本类型	|	不需要import	 |
|String		  |	不需要import	||
|CharSequence |不需要import| 
|List         |不需要import|List接收方必须是ArrayList，List内的元素必须是AIDL支持的类型；
|Map			|不需要import|	Map接收方必须是HashMap，Map内的元素必须是AIDL支持的类型；
|其他AIDL定义的AIDL接口|	需要import|	传递的是引用（有待考究，应该跟in，out有关的）|
|实现Parcelable的类|	需要import|	值传递|


# AIDL传递接口给服务器
`IListener.aidl`
```aidl
interface IListener {
     void onStateChange(boolean enable);
}
```
`Listener.java`
```java
public abstract class Listener extends IListener.Stub {
	//abstract 一定是public
	public abstract void onStateChange(boolean enable);
}
```
