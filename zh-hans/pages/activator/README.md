# 配网

配网业务包提供涂鸦 App 配网业务逻辑及 UI 界面。业务功能涵盖了目前涂鸦智能的 Wi-Fi 设备、ZigBee 设备、蓝牙设备、支持二维码扫码的设备（例如 GPRS & NB-IoT 设备）等不同类型的设备配网前置操作引导和具体入网激活实现。



## 接入组件

在工程的 `Podfile` 文件中添加商城业务包组件，并执行 `pod update` 命令

```ruby
source "https://github.com/TuyaInc/TYPublicSpecs.git"
source 'https://cdn.cocoapods.org/'

target 'your_target_name' do
  # 添加配网业务包
  pod 'TuyaSmartActivatorBizBundle'
end
```



**注意**

Wi-Fi  设备配网过程需要获取手机当前连接 Wi-Fi  名称，需要项目开启地位权限来获取 Wi-Fi  的名称，在 info.plist 中添加如下权限声明，创建 `CLLocationManager` 示例，并调用 `requestWhenInUseAuthorization` 方法。

```
NSLocationAlwaysAndWhenInUseUsageDescription
NSLocationAlwaysUsageDescription
NSLocationWhenInUseUsageDescription
```

二维码扫码功能需要系统相机权限，需要在 info.plist 中添加以下声明

```
NSCameraUsageDescription
```



## 自定义配置

### 1、蓝牙配网功能

配网业务包支持 Wi-Fi  、蓝牙等类型的设备配网，其中蓝牙配网为可选项，如当前 App 不需要蓝牙配网功能 ，只需要将自定义 `ty_custom_config.json` 中的 `needBle` 属性设置为 false 即可。

如果需要蓝牙配网功能，首先需要在项目的 info.Plist 文件中添加蓝牙权限的声明，设置 `ty_custom_config.json` 中的 `needBle` 属性设置为 true，然后需要在项目中添加以下依赖：

**系统权限**

```
NSBluetoothAlwaysUsageDescription
NSBluetoothPeripheralUsageDescription
```

**配置项**

```json
{
    "config": {
        "appId": 123,    
        "tyAppKey": "xxxxxxxxxxxx", 
        "appScheme": "tuyaSmart",
        "hotspotPrefixs": ["SmartLife"], 
        "needBle": true //设置为 true 则表示支持蓝牙设备的配网
    },
   "colors":{
        "themeColor": "#FF5A28", 
    }
}
```

**依赖**

```ruby
  pod 'TYBLEInterfaceImpl'
  pod 'TYBLEMeshInterfaceImpl'
  pod 'TuyaSmartBLEKit'
  pod 'TuyaSmartBLEMeshKit'
```

### 2、设备热点名称设置

涂鸦设备热点前缀默认为 `SmartLife` , 若当前设备的热点前缀名称已修改，则需要在  `ty_custom_config.json` 文件中设置 `hotspotPrefixs` 属性，设置当前设备的热点前缀。

```json
{
    "config": {
        "appId": 123,    
        "tyAppKey": "xxxxxxxxxxxx", 
        "appScheme": "tuyaSmart",
        "hotspotPrefixs": ["SL"], // 修改支持的设备热点前缀为 SL
        "needBle": true
    },
   "colors":{
        "themeColor": "#FF5A28", 
    }
}
```



## 服务协议

### 提供服务

配网业务包实现 `TYActivatorProtocol` 协议以提供服务，在 `TYModuleServices` 组件中查看 `TYActivatorProtocol.h` 协议文件内容为：

```objc

#ifndef TYActivatorProtocol_h
#define TYActivatorProtocol_h

typedef NS_ENUM(NSUInteger, TYActivatorCompletionNode) {
    TYActivatorCompletionNodeNormal
};

@class TuyaSmartHome;

@protocol TYActivatorProtocol <NSObject>

/**
 * Start config
 * Goto device config list view 
 */
- (void)gotoCategoryViewController;


/**
 *  Obtain device information after each device connection
 *  @param node completion node, default TYActivatorCompletionNodeNormal
 *  @param custionJump default false, set true for process not need to jump to de device panel
 */
- (void)activatorCompletion:(TYActivatorCompletionNode)node customJump:(BOOL)customJump completionBlock:(void (^)(NSArray * _Nullable deviceList))callback;


@end
#endif /* TYActivatorProtocol_h */

```



### 依赖服务

配网业务包正常运行需要依赖  `TYSmartHomeDataProtocol` 这个协议提供的协议方法，调用业务包之前需要实现以下协议

#### TYSmartHomeDataProtocol

提供配网所需当前家庭信息

```objc
/**
 获取当前的家庭，当前没有家庭的时候，返回nil。
 
 @return TuyaSmartHome
 */
- (TuyaSmartHome *)getCurrentHome;
```



## 使用指南

### 注意事项

1、任何接口调用之前，务必确认用户已登录

2、调用配网业务包逻辑前，要先实现 `TYSmartHomeDataProtocol` 中的协议方法`getCurrentHome`

Objective-C 示例

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



Swfit 示例

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



### 进入配网

Objective-C 示例

```objc
#import <TuyaSmartBizCore/TuyaSmartBizCore.h>
#import <TYModuleServices/TYActivatorProtocol.h>


- (void)gotoDeviceConfig {
    id<TYActivatorProtocol> impl = [[TuyaSmartBizCore sharedInstance] serviceOfProtocol:@protocol(TYActivatorProtocol)];
    [impl gotoCategoryViewController];
  
    // 获取配网结果
    [impl activatorCompletion:TYActivatorCompletionNodeNormal customJump:NO completionBlock:^(NSArray * _Nullable deviceList) {
        NSLog(@"deviceList: %@",deviceList);
    }];
}
```



Swfit 示例

```swift
let impl = TuyaSmartBizCore.sharedInstance().service(of: TYActivatorProtocol.self) as? TYActivatorProtocol
impl?.gotoCategoryViewController()

impl?.activatorCompletion(TYActivatorCompletionNodeNormal, customJump: false, completionBlock: { (evIdList:[Any]?) in
            print(devIdList ?? [])
        })
```





