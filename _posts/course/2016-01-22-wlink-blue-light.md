---
layout: post_page
title: “Blue light使用--页面预览”
---

### Blue light使用--页面预览

#### 蓝牙灯泡智能控制

```
管理类WLBlueToothManager：读取和发送指令（UUID 包括 serviceUUID、writeCharacterUUID、readCharacterUUID）

一、搜索外设（scanForPeripherals）

前提是必须中心准备好（isCentralReady）才开始检索设备，直到搜索到才进行下一步（此处添加一个KVO进行观察，一旦centerReady（change[NSKeyValueChangeNewKey]）

二、正式开始搜索任务（beginScanPeripheralTask）

在此处进行过滤操作（checkVaildWithPeripheral），如果是特定标识产品，则连接设备，发现服务。同时设定超时限定，如超过有效时间则调用

搜索设备超时（discoverPeripheralTimeOut）［其中搜索到了设备，但没有一个连接的（没有connected）则不做处理］，否则发出未搜索到设备的通知，显示离线体验模式。
三、连接外设，发现服务（discoverServiceWithPeripheral）
调用（discoverServicesWithCompletion 和 discoverCharacteristicsWithCompletion）

四、界面
1. 灯组页面WLLightList
在viewDidLoad方法中搜索外设（scanForPeripherals），将搜索连接且符合特定标识的设备加入一个默认灯组中，同时将灯组添加入数组，并记录当前选中的那一个蓝牙灯泡。（注意：这里要对蓝牙灯泡和离线模式下的进行区分）
注册connectPeripheral: 、 disconnectPeripheral:、discoverPeripheralTimeout:）等相应通知

二、主界面MainViewController（6个子控制器，自定义tabbar等）
1.颜色界面：有照明模式（连接进入即开灯），冷暖调节（色温），灯泡颜色控制等，使用单例发送指令，如：

// 关闭照明模式，切换至彩灯模式
[[WLBlueToothManager shareManager] setupColorWithPeripheral:nil color:color brightnessRate:self.brightnessSlider.value];
2.场景界面默认设定了5个场景（collection view实现），用户自定义场景（后期又要求取消此功能）

3.设置界面包括设备命名和定时开关灯操作（TapGesture手势，同时判断点击所属类型）

4.灯组界面即是用户自定义将蓝牙灯泡设定成组同时控制，功能与单个控制一致（包括冷暖，亮度，开关，颜色，定时，命名）以及添加、修改、删除功能。

5.律动界面，主要使用了第三方EZAudio和ZLHistogramAudioPlot实现声波采集配合矩状图实现律动效果，并根据相应算法实现灯泡颜色变幻。

6.软件信息界面，就是与软件与公司介绍相关

同时还使用了SVProgresHUD进行相应提示，Masonry进行UI布局

五、代码：

//  WLBlueToothManager.h
// 搜索到设备的通知
FOUNDATION_EXTERN NSString *const kWLDiscoverPeripheralNotification;
FOUNDATION_EXTERN NSString *const kWLDiscoverPeripheralNotificationKey;
// 搜索设备超时的通知
FOUNDATION_EXTERN NSString *const kWLDiscoverPeripheralTimeoutNotification;
// 发现设备block回调
typedef void(^WLBluetoothManagerDiscoverPeripheralsCallback)(NSArray<WLPeripheral *>* peripherals);

@interface WLBlueToothManager : NSObject
/**
*  灯组的数组,可以添加多个灯组
*/
@property (nonatomic, strong) WLBuleGroup *selectedGroup;

/**
*  当前连接成功，并且选择控制的设备
*/
@property (nonatomic, strong, readonly) WLPeripheral *selectPeripheral;
/**
*  选取的下标
*/
@property (nonatomic, assign) NSInteger selectIndex;

/**
*  蓝牙灯泡管理类单例
*/
+ (instancetype)shareManager;

/**
*  搜索蓝牙设备
*/
- (void)scanForPeripherals;

/**
*  停止搜索蓝牙设备
*/
- (void)stopForPeripherals;

/**
*  开灯
*
*  @param peripheral 蓝牙设备（外设）
*/
- (void)turnOnTheBlueWithPeripheral:(WLPeripheral *)peripheral;

/**
*  关灯
*
*  @param peripheral 蓝牙设备（外设）
*/
- (void)turnOffTheBlueWithPeripheral:(WLPeripheral *)peripheral;

/**
*  设置灯泡的冷暖光
*
*  @param warm       暖光比例
*  @param brightness 亮度比例
*  @param peripheral 要设置的设备
*/
- (void)setupWarmAndColdWithColdRate:(float)warm andBrightnessRate:(float)brightness andPeripheral:(WLPeripheral *)peripheral;

/**
*  设置灯泡颜色
*
*  @param peripheral 设备
*  @param color      颜色
*  @param brightness 亮度
*/
- (void)setupColorWithPeripheral:(WLPeripheral *)peripheral color:(UIColor *)color brightnessRate:(float)brightness;

/**
*  设置场景颜色
*
*  @param redValue   红色值
*  @param greenValue 绿色值
*  @param blueValue  蓝色值
*  @param peripheral 设备
*/
- (void)setupSceneWithRed:(NSInteger)redValue green:(NSInteger)greenValue blue:(NSInteger)blueValue andPeripheral:(WLPeripheral *)peripheral;

/**
*  设置定时开灯
*
*  @param seconds    秒数
*  @param peripheral 设备
*/
- (void)setupTimingOpenBlueToothLightWithSeconds:(NSInteger)seconds andPeripheral:(WLPeripheral *)peripheral;

/**
*  设置定时关灯
*
*  @param seconds    秒数
*  @param peripheral 设备
*/
- (void)setupTimingOffBlueToothLightWithSeconds:(NSInteger)seconds andPeripheral:(WLPeripheral *)peripheral;
```

#### 搜索页面－过滤非特定标识蓝牙设备
![搜索页面－过滤非特定标识蓝牙设备](http://upload-images.jianshu.io/upload_images/1634375-5919bba06f7340f3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 颜色控制界面，控制灯泡颜色，亮度，冷暖
![颜色控制界面，控制灯泡颜色，亮度，冷暖](http://upload-images.jianshu.io/upload_images/1634375-633e43881e72f80e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 亮度与颜色明暗度控制
![亮度与颜色明暗度控制](http://upload-images.jianshu.io/upload_images/1634375-ea4a6b3f5b70b1f8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 设备命名－设定自动开启与关闭功能
![设备命名－设定自动开启与关闭功能](http://upload-images.jianshu.io/upload_images/1634375-7996b7b615bd726a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 灯组模式－同时对多个蓝牙灯泡组合控制开关，颜色等
![灯组模式－同时对多个蓝牙灯泡组合控制开关，颜色等](http://upload-images.jianshu.io/upload_images/1634375-799061915e35717a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 灯组自定义命名与灯泡组合选择
![灯组自定义命名与灯泡组合选择](http://upload-images.jianshu.io/upload_images/1634375-8081b5d49690fec8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 声音律动－根据外界声音进行律动，变幻灯泡颜色
![声音律动－根据外界声音进行律动，变幻灯泡颜色](http://upload-images.jianshu.io/upload_images/1634375-08208ed86662c9a9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 关于－产品版本介绍
![关于－产品版本介绍](http://upload-images.jianshu.io/upload_images/1634375-c3f46da5185e0890.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 软件信息－公司介绍
![软件信息－公司介绍](http://upload-images.jianshu.io/upload_images/1634375-4e8de3f784545975.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 更多详情需下载app，购买特定产品方可体验。



