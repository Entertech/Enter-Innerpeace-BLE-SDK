# Innerpeace BLE SDK
该SDK用于Android设备与易休蓝牙通信。

**注意：**

**1、回调是非UI线程，不能直接进行UI操作**

**2、建议监听回调里面不要有耗时操作**

## 集成
将 `enter-bledevice-v1.1.3.jar` 复制到工程，添加为Lib。

**build.gradle**，由于蓝牙底层依赖以下库，所以工程需要添加。
```gradle
 implement 'com.polidea.rxandroidble:rxandroidble:1.11.0''
```
**AndroidManifest.xml**

SDK需要以下权限，**Android 6.0以上高危权限需要动态申请，详情查阅官方文档**
```xml
    <uses-permission android:name="android.permission.BLUETOOTH" />
    <uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
```
## 使用

### 初始化

需要传入application context
```java
   bleDeviceManager = BleDeviceManager.getInstance(this)
```
### 设备连接

#### 连接附近信号最强的设备（未知设备mac地址）

**方法说明**

扫描并连接附近信号最强的设备，需传入用户id，如不需要用户绑定功能则传入固定值

**示例代码**

```kotlin
  bleDeviceManager.scanNearDeviceAndConnect(userId, fun(){
    Logger.d("扫描成功")
  }, fun(mac: String) {
    Logger.d("连接成功$mac")
  }){msg->
    Logger.d("连接失败")
  }
```

**参数说明**

| 参数                   | 类型            | 说明                                                         |
| ---------------------- | --------------- | ------------------------------------------------------------ |
| userId                 | Long            | 用户Id。连接的时候需要传入用户id，id不匹配的不能连接. 如果需求不需要绑定功能，代码中可写入固定id。 |
| scanSuccessCallBack    | () -> Unit      | 扫描成功回调                                                 |
| connectSuccessCallBack | (String) ->Unit | 连接成功回调                                                 |
| failedCallBack         | (String)->Unit  | 失败回调                                                     |

#### 根据指定mac连接（已知设备mac地址）

**方法说明**

连接指定mac地址设备，需要传入用户id、mac地址

**示例代码**

```kotlin
  bleDeviceManager.scanMacAndConnect(userId, mac, fun(mac: String) {
    Logger.d("连接成功$mac")
  }){msg->
    Logger.d("连接失败")
  }
```

**参数说明**

| 参数            | 类型           | 说明        |
| --------------- | -------------- | ----------- |
| userId          | Long           | 用户Id      |
| mac             | String         | 设备mac地址 |
| successCallBack | (String)->Unit | 成功回调    |
| failedCallBack  | (String)->Unit | 失败回调    |

### 设备断开

**方法说明**

断开与设备的连接

**示例代码**

```kotlin
bleDeviceManager.disConnect()
```

### 获取设备连接状态

**方法说明**

获取当前设备连接状态

**示例代码**

```kotlin
boolean isConnected = bleDeviceManager.isConnected()
```

**返回值说明**

| 参数        | 类型    | 说明                              |
| ----------- | ------- | --------------------------------- |
| isConnected | Boolean | 设备已连接为true，未连接为false。 |

### 设置监听接口

**监听接口生命周期需要管理，不需要监听了，请调用remove**

#### 添加原始脑波监听

**方法说明**

添加原始脑波监听，通过该监听可从硬件中获取原始脑波数据

**示例代码**

```kotlin
  var rawDataListener = fun(data:ByteArray){
        Logger.d(Arrays.toString(data))
  }
  bleDeviceManager.addRawDataListener(rawDataListener)
```

**参数说明**

| 参数            | 类型                | 说明         |
| --------------- | ------------------- | ------------ |
| rawDataListener | （ByteArray）->Unit | 原始脑波回调 |

> **原始脑波数据说明**
>
> 从脑波回调中返回的原始脑波数据是一个长度为17的字节数组，前两个字节为包编号，后15个字节为有效脑波数据
>
> **正常数据示例**
>
> [0, -94, 21, -36, 125, 21, -12, -75, 22, 8, 61, 22, 10, -72, 22, 15, -19]
>
> **异常数据示例（未检测到脑波数据）**
>
> [0, 101, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1]

#### 移除原始脑波监听

**方法说明**

如果不想受到脑波数据，移除监听即可

**示例代码**

```kotlin
bleDeviceManager.removeRawDataListener(rawDataListener)
```

**参数说明**

| 参数            | 类型                | 说明         |
| --------------- | ------------------- | ------------ |
| rawDataListener | （ByteArray）->Unit | 原始脑波回调 |

#### 添加佩戴信号监听

**方法说明**

添加该监听，可实时获取设备佩戴质量

**代码示例**

```kotlin
contactListener = fun(state: ContactState) {
   napManager.setTouchState(ContactState.GOOD == state);//将从硬件层获取的佩戴信号传入算法
}
bleDeviceManager.addContactListener(contactListener)

```

**参数说明**

| 参数            | 类型                   | 说明                                                         |
| --------------- | ---------------------- | ------------------------------------------------------------ |
| contactListener | （ContactState）->Unit | 佩戴信号回调。返回的ContactState为一个枚举类型，枚举值为ContactState.GOOD、ContactState.POOR、ContactState.BAD,分别表示佩戴信号质量：好、一般、差 |

#### 移除佩戴信号监听

**方法说明**

移除该监听，则不会受到佩戴信号

**代码示例**

```kotlin
bleDeviceManager.removeContactListener(contactListener)
```

**参数说明**

| 参数            | 类型                   | 说明         |
| --------------- | ---------------------- | ------------ |
| contactListener | （ContactState）->Unit | 佩戴信号回调 |

#### 添加电量监听

**方法说明**

添加电量监听，添加后会每隔30秒回调一次

**代码示例**

```kotlin
var batteryListener = fun(byte: Byte) {
   Logger.d("battery = $byte")
}
bleDeviceManager.addBatteryListener(batteryListener)
```

**参数说明**

| 参数            | 类型            | 说明     |
| --------------- | --------------- | -------- |
| batteryListener | （Byte）-> Unit | 电量回调 |

#### 移除电量监听

**方法说明**

移除后，将不会收到电量回调

**代码示例**

```kotlin
bleDeviceManager.removeBatteryListener(batteryListener)
```

**参数说明**

| 参数            | 类型            | 说明     |
| --------------- | --------------- | -------- |
| batteryListener | （Byte）-> Unit | 电量回调 |

### 采集与停止

#### 开始采集

**方法说明**

开始采集，调用这个接口开始采集脑波数据

**示例代码**

```kotlin
bleDeviceManager.startCollection()
```

#### 停止采集

**方法说明**

停止采集，调用该方法停止采集脑波数据

**示例代码**

```kotlin
bleDeviceManager.stopCollection()
```

### DFU（设备固件升级）

支持DFU（Device Firmware Update），即设备远程固件升级。由于本SDK底层依赖Android-DFU-Library库，因此如有需要DFU，需在你的build.gradle文件中添加如下依赖：

```groovy
compile 'no.nordicsemi.android:dfu:1.7.0'
```

#### DFU的使用

首先需要定义一个服务并继承DfuBaseService，如下：

```
public class DfuService extends DfuBaseService {
	@Override
	protected Class<? extends Activity> getNotificationTarget() {
 	   return NotificationActivity.class;//当点击固件升级中的通知栏时，会打开
                                        //NotificationActivity这个Activity
	}

	@Override
	protected boolean isDebug() {
 	   //该方法表示是否打印更多的调试日志
	    return BuildConfig.DEBUG;//BuildConfig.DEBUG为false，如果要打印更多的日志则直接返回true
	}
}
```

NotificationActivity类代码如下：

```
public class NotificationActivity extends Activity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        if (isTaskRoot()) {
            // 这里声明你要跳转的Activity
            final Intent intent = new Intent(this, MyActivity.class);
            intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
            intent.putExtras(getIntent().getExtras());
            startActivity(intent);
        }
        finish();
    }
}
```

> *注意：别忘了在AndroidManifest.xml文件中注册以上service和Activity*

接下来需要创建一个远程升级的监听，用来监听升级过程中各个过程：

```kotlin
private val mDfuProgressListener = object : DfuProgressListenerAdapter() {

     //设备正在连接
        override fun onDeviceConnecting(deviceAddress: String?) {
        }
       //设备已连接
        override fun onDeviceConnected(deviceAddress: String?) {
        }
       //正准备开始升级
        override fun onDfuProcessStarting(deviceAddress: String?) {
        }
       //设备开始升级
        override fun onDfuProcessStarting(deviceAddress: String?) {
        }

        override fun onEnablingDfuMode(deviceAddress: String?) {
        }

          //固件验证
        override fun onFirmwareValidating(deviceAddress: String?) {
        }
      //设备正在断开
        override fun onDeviceDisconnecting(deviceAddress: String?) {
        }
      //升级完成
        override fun onDfuCompleted(deviceAddress: String?) {
        }
      //由于意外原因，升级流产，升级失败
        override fun onDfuAborted(deviceAddress: String?) {
        }
      //升级中进度回调
        override fun onProgressChanged(deviceAddress: String?, percent: Int, speed: Float, avgSpeed: Float, currentPart: Int, partsTotal: Int) {
        }
      //升级失败
        override fun onError(deviceAddress: String?, error: Int, errorType: Int, message: String?) {
        }
    }
```

#### 注册监听

要在相应的生命周期中注册及取消注册该监听

**代码示例**

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
   super.onCreate(savedInstanceState)
   setContentView(R.layout.activity_device_update_tip)
   DfuServiceListenerHelper.registerProgressListener(this, mDfuProgressListener)
}
override fun onDestroy() {
   super.onDestroy()
   DfuServiceListenerHelper.unregisterProgressListener(this, mDfuProgressListener)
}
```

#### 开始升级

```kotlin
val starter = DfuServiceInitiator(deviceMac) //传入设备mac地址
          .setDeviceName(deviceName) //传入设备名称
           .setkeepBond(true) //是否保持设备绑定
//调用此方法使Nordic nrf52832进入bootloader模式
starter.setUnsafeExperimentalButtonlessServiceInSecureDfuEnabled(true)
//设置文件路径
starter.setZip(FileUtil.getFirmwareDir() + "/firmware.zip")
//开始升级
starter.start(this, DfuService::class.java)
```

对于Android 8.0及以上的系统，如果你想让DFU服务能够在通知栏中展示进度条，你需要创建一个通知频道。最简单的方式就是调用以下方法：

```kotlin
DfuServiceInitiator.createDfuNotificationChannel(context);
```
