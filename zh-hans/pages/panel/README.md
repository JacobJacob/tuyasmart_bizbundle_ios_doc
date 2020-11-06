# 设备控制业务包

## 功能介绍

设备控制业务包是涂鸦智能设备控制面板的核心容器，在涂鸦智能 iOS Home SDK 的基础上，提供了设备控制面板的加载和控制的接口封装，加速应用开发过程。主要包括以下功能：

- 面板加载（加载多种设备类型，支持：WIFI、Zigbee、Mesh、BLE）
- 设备控制（支持单设备和群组的控制，不支持群组管理）
- 设备定时



## 接入组件

在工程的 `Podfile` 文件中添加设备控制业务包组件，并执行 `pod update` 命令

```ruby
source "https://github.com/TuyaInc/TuyaPublicSpecs.git"
source 'https://cdn.cocoapods.org/'

target 'your_target_name' do
  # TuyaSmart SDK
  pod "TuyaSmartHomeKit"
  # 添加设备控制业务包
  pod 'TuyaSmartPanelBizBundle', 'xxx'
  # 若需要扫地机功能，请依赖扫地机相关插件
  # pod 'TuyaRNApi/Sweeper'
end
```

**注意**

设备控制业务包中封装了一系列 RN 接口供面板调用，其中会涉及到部分苹果隐私权限的声明。

- 如果接入的设备面板有使用相册相关的（例如：云相册），则需要在工程的 `info.plist` 中添加如下权限声明：

```
NSPhotoLibraryAddUsageDescription
```

- 如果接入的设备面板有使用照相机相关的（例如：云相册），则需要在工程的 `info.plist` 中添加如下权限声明：

```
NSCameraUsageDescription
```

- 如果接入的设备面板有使用位置信息相关的，则需要在工程的 `info.plist` 中添加如下权限声明：

```
NSLocationWhenInUseUsageDescription
```

- 如果接入的设备面板有使用到麦克风（例如：音乐灯面板），则需要在工程的 `info.plist` 中添加如下权限声明：

```
NSMicrophoneUsageDescription
```

- 如果接入的设备面板是蓝牙相关的，则需要在工程的 `info.plist` 中添加如下权限声明：

```
NSBluetoothAlwaysUsageDescription
NSBluetoothPeripheralUsageDescription
```



## 服务协议

### 提供服务

设备控制业务包实现 `TYPanelProtocol` 协议以提供服务，在 `TYModuleServices` 组件中查看 `TYPanelProtocol.h` 协议文件内容为：

```objc
@protocol TYPanelProtocol <NSObject>
NS_ASSUME_NONNULL_BEGIN

// 清除面板缓存
- (void)cleanPanelCache;

/**
 * 跳转面板，push 的方式
 *
 * @param device        设备模型
 * @param group         群组模型
 * @param initialProps  自定义初始化参数，会以 'extraInfo' 为 key 设置进 RN 应用的 initialProps 中
 * @param contextProps  自定义面板上下文，会以 'extraInfo' 为 key 设置进 Panel Context 中
 * @param completion 完成跳转后的结果回调
 */
- (void)gotoPanelViewControllerWithDevice:(TuyaSmartDeviceModel *)device
                                    group:(nullable TuyaSmartGroupModel *)group
                             initialProps:(nullable NSDictionary *)initialProps
                             contextProps:(nullable NSDictionary *)contextProps
                               completion:(void(^ _Nullable)(NSError * _Nullable error))completion;

/**
 * 跳转面板，present 的方式
 *
 * @param device 设备模型
 * @param group 群组模型
 * @param initialProps  自定义初始化参数，会以 'extraInfo' 为 key 设置进 RN 应用的 initialProps 中
 * @param contextProps  自定义面板上下文，会以 'extraInfo' 为 key 设置进 Panel Context 中
 * @param completion 完成跳转后的结果回调
 */
- (void)presentPanelViewControllerWithDevice:(TuyaSmartDeviceModel *)device
                                       group:(nullable TuyaSmartGroupModel *)group
                                initialProps:(nullable NSDictionary *)initialProps
                                contextProps:(nullable NSDictionary *)contextProps
                                  completion:(void (^ _Nullable)(NSError * _Nullable error))completion;

// RN版本号
- (NSString *_Nonnull)rnVersionForApp;

NS_ASSUME_NONNULL_END
@end
```



### 依赖服务

设备控制业务包主要功能为加载设备，针对不同设备会有不同的一些功能，为保证这些功能正常运行，会依赖如下几个协议： `TYSmartHomeDataProtocol`、 `TYDeviceDetailProtocol`、 `TYSettingsProtocol`、 `TYOTAGeneralProtocol`、 `TYGroupHandleProtocol`、 `TYRNCameraProtocol`、 `TYCameraProtocol`。

#### TYSmartHomeDataProtocol

提供加载设备面板所需的当前家庭信息，**必须实现**

```objc
/**
 获取当前的家庭，当前没有家庭的时候，返回nil。
 
 @return TuyaSmartHome
 */
- (TuyaSmartHome *)getCurrentHome;
```

####  TYDeviceDetailProtocol

设备面板界面右上角点击跳转事件

```objc
/**
 导航栏右边按钮点击事件，跳转到设备详情页

 @param device 设备
 @param group 群组，若有就传
 */
- (void)gotoDeviceDetailDetailViewControllerWithDevice:(TuyaSmartDeviceModel *)device group:(TuyaSmartGroupModel *)group;
```

#### TYSettingsProtocol

面板内触发下发 dp 控制设备时，提供可选开关供 App 发出音效。实现如下方法返回是否开启音效：

```objc
/**
 * 检查是否开启音效
 */
- (BOOL)soundEnabled;
```

#### TYOTAGeneralProtocol

进入设备面板时，提供检查设备固件更新的事件。实现如下方法用于检查固件升级：

```objc
/**
 检查设备固件更新，如果有更新会显示展示出固件更新提示
 
 @param deviceModel 需要检查固件升级的设备
 @param isManual 是否手动检测升级
  @param theme 主题色
 YES: 手动检测升级，检测时弹出loading框。当有固件新版本时(检测升级、强制升级、提醒升级)，显示OTA VC。
 NO: 自动检测升级, 检测时不弹出loading框。当有强制升级时、提醒升级时，弹出固件升级提示，点确定后显示OTA VC。
 */
- (void)checkFirmwareUpgrade:(TuyaSmartDeviceModel *)deviceModel isManual:(BOOL)isManual theme:(TYOTAControllerTheme)theme;
```

#### TYGroupHandleProtocol

BLE Mesh 品类的设备，面板内有需要跳转到 Mesh 群组界面的事件，若需要跳转 mesh 群组，实现如下方法：

```objc
/**
跳转本地 mesh 群组

@available 1.0.0
@param params 
@param success 
@param failure 
*/
- (void)impl_jumpToMeshLocalGroup:(NSDictionary*)params success:(RCTResponseSenderBlock)success failure:(RCTResponseErrorBlock)failure ;
```

#### TYRNCameraProtocol

加载摄像头设备 RN 面板的跳转事件，若同时接入摄像头面板业务包 `TuyaSmartCameraRNPanelBizBundle`，则不需要实现；

```objc
/**
 获取摄像头RN面板
 @param devId 摄像头设备的devId
 */
- (UIViewController *)cameraRNPanelViewControllerWithDeviceId:(NSString *)devId;
```

#### TYCameraProtocol

加载摄像头设备原生面板的跳转事件，若同时接入摄像头面板业务包 `TuyaSmartCameraPanelBizBundle`，则不需要实现；

```objc
/**
 获取摄像头Native面板
 @param devId 摄像头设备的devId
 @param uiName 摄像头设备的uiName，不同的uiName对应不同版本的面板
 */
- (UIViewController *)viewControllerWithDeviceId:(NSString *)devId uiName:(NSString *)uiName;
```



## 使用指南

### 注意事项

1、任何接口调用之前，务必确认用户已登录

2、调用设备控制业务包逻辑前，要先实现 `TYSmartHomeDataProtocol` 中的协议方法`getCurrentHome`

Objective-C 

```objc
#import <TuyaSmartBizCore/TuyaSmartBizCore.h>
#import <TYModuleServices/TYSmartHomeDataProtocol.h>

- (void)initCurrentHome {
    // 注册要实现的协议
    [[TuyaSmartBizCore sharedInstance] registerService:@protocol(TYSmartHomeDataProtocol) withInstance:self];
}

// 实现对应的协议方法
- (TuyaSmartHome *)getCurrentHome {
    TuyaSmartHome *home = [TuyaSmartHome homeWithHomeId:@"当前家庭id"];
    return home;
}
```

Swift

```swift
import TuyaSmartDeviceKit

class TYActivatorTest: NSObject,TYSmartHomeDataProtocol{
  	
    func test() {
        TuyaSmartBizCore.sharedInstance().registerService(TYSmartHomeDataProtocol.self, withInstance: self)
    }
    
    func getCurrentHome() -> TuyaSmartHome! {
        let home = TuyaSmartHome.init(homeId: 111)
        return home
    }
    
}
```

### 清除面板缓存

Objective-C 

```objc
id<TYPanelProtocol> impl = [[TuyaSmartBizCore sharedInstance] serviceOfProtocol:@protocol(TYPanelProtocol)];
[impl cleanPanelCache];
```

Swift

```swift
let impl = TuyaSmartBizCore.sharedInstance().service(of: TYPanelProtocol.self) as? TYPanelProtocol
impl?.cleanPanelCache()
```

### 打开面板

Objective-C 

```objc
id<TYPanelProtocol> impl = [[TuyaSmartBizCore sharedInstance] serviceOfProtocol:@protocol(TYPanelProtocol)];
// 方式一: 默认 Push 方式跳转
[impl gotoPanelViewControllerWithDevice:deviceModel group:nil initialProps:nil contextProps:nil completion:^(NSError * _Nullable error) {
    if (error) {
        NSLog(@"Load error: %@", error);
    }
}];
// 方式二: 使用 Present 方式跳转
[impl presentPanelViewControllerWithDevice:deviceModel group:nil initialProps:nil contextProps:nil completion:^(NSError * _Nullable error) {
    if (error) {
        NSLog(@"Load error: %@", error);
    }
}];
// 方式三: 获取面板视图控制器，自行跳转
[impl getPanelViewControllerWithDeviceModel:device groupModel:group initialProps:nil contextProps:nil completionHandler:^(__kindof UIViewController * _Nullable panelViewController, NSError * _Nullable error) {

}];
```

Swift

```swift
let impl = TuyaSmartBizCore.sharedInstance().service(of: TYPanelProtocol.self) as? TYPanelProtocol
// 方式一: 默认 Push 方式跳转
impl?.gotoPanelViewController(withDevice: deviceModel!, group: nil, initialProps: nil, contextProps: nil, completion: { (error) in
    if let e = error {
        print("\(e)")
    }
})
// 方式二: 使用 Present 方式跳转
impl?.presentPanelViewController(withDevice: deviceModel!, group: nil, initialProps: nil, contextProps: nil, completion: { (error) in
    if let e = error {
        print("\(e)")
    }
})
// 方式三: 获取面板视图控制器，自行跳转
let impl = TuyaSmartBizCore.sharedInstance().service(of: TYPanelProtocol.self) as? TYPanelProtocol
impl?.getPanelViewController(with: deviceModel!, groupModel: nil, initialProps: nil, contextProps: nil, completionHandler: { (panelViewController, error) in
		if let e = error {
        print("\(e)")
    }
})
```

## 常见问题

**1. 为什么调用 Push 方式跳转面板之后，其他视图控制器的导航栏样式变了？**

设备控制业务包本质是提供一个视图控制器来加载 React-Native 渲染的界面，所以该视图控制器内部将导航栏进行了相关的隐藏处理，主要是改变了透明度。因此在使用 push 方式进入面板控制器之后，pop 回自身控制器时，需要将样式还原（主要原因是 iOS 提供的 push 和 pop 动画是使用 NavigationController 来管理的，其中使用的 NavigationBar 是唯一的；在 NavigationController 的 Stack 存储结构下，每当 Stack 中的 ViewController 修改了导航栏，势必会影响其他 ViewController 展示的效果）。以下建议两种样式还原方式：

方式一：

```objective-c
// 1. 导入头文件
#import <TYNavigationController/TYNavigationTopBarProtocol.h>
// 2. 显示导航栏
self.ty_topBarHidden = NO;
self.ty_topBarAlpha = 1.0;
// 3. 还原导航栏样式(例如：修改背景色)
self.ty_topBarBackgroundImage = <#Image#>;
// Or
self.ty_topBarBackgroundColor = <#Color#>;
```

方式二：

```objective-c
// 1. 导入头文件
#import <TYNavigationController/TYNavigationCallbackProtocol.h>
// 2. 实现 TYNavigationCallbackProtocol 提供的代理方法
- (void)ty_naviTransitioning:(id<UIViewControllerTransitionCoordinatorContext>)context {
    // 3. 显示导航栏
    [self.navigationController setNavigationBarHidden:false animated:true];
    // 4. 还原导航栏样式(例如：修改背景色)
    [self.navigationController.navigationBar setBarTintColor:<#Color#>];
  	// Or
    [self.navigationController.navigationBar setBackgroundImage:<#Image#> forBarMetrics:UIBarMetricsDefault];
}
```

**2. 依赖插件的时候，`pod update` 报错？**

工程在使用 cocoapods 集成业务包时，由于 pod 的逻辑，Podfile 中没有指明每个 Pod 库的版本时，默认会拉取最新的正式版本号（x.y.z，其中 x、y、z 均为数字）。若需要依赖设备控制业务包的扩展功能时，即需要依赖相关插件，由于插件的版本号属于 pre-release，因此，需要在 Podfile 中指明设备控制业务包的所在基线的最新日期标明的版本号（[点击查看业务包版本号](../versions.md)）。