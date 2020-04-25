# 框架接入



## 业务包内容介绍

| 组件                        | 内容说明                                         | 注意 |
| --------------------------- | ------------------------------------------------ | ---- |
| TuyaSmartBizCore            | 业务基础处理库，用于启动业务包以及自定义一些功能 | 必选 |
| TYModuleServices            | 业务包模块实现的协议                             | 必选 |
| TuyaSmartPanelSDK           | 提供了设备控制面板的加载和控制的接口封装         | 可选 |
| TuyaCameraPanelSDK          | 提供了一系列摄像机面板相关功能                   | 可选 |
| TuyaSmartMallBizBundle      | 提供商城功能                                     | 可选 |
| TuyaSmartActivatorBizBundle | 提供设备配网业务功能                             | 可选 |



`TuyaSmartBizCore` 和 `TYModuleServices` 是使用业务包必须要依赖的基础库。

###  `TuyaSmartBizCore` 介绍

提供启动业务包以及自定义配置功能

#### 主题色及自定义配置功能

该部分的配置，需要客户按照如下方式生成一份名为 ty_custom_config.json 的文件放入工程目录下

```json
{
    "config": {
        "appId": 123,    
        "tyAppKey": "xxxxxxxxxxxx", 
        "appScheme": "tuyaSmart",
        "hotspotPrefixs": "SmartLife",
        "needBle": true,
        "scanDeviceClose": false,
    },
   "colors":{
        "themeColor": "#FF5A28", 
    }
}
```



**参数介绍**

| 参数            | 说明                         |
| --------------- | ---------------------------- |
| appId           | 应用 Id                      |
| tyAppKey        | 涂鸦开发者平台生成的 App Key |
| appScheme       | App Scheme                   |
| hotspotPrefixs  | 配网设备热点前缀             |
| needBle         | 是否需要支持蓝牙设备配网     |
| scanDeviceClose | 是否需要关闭自动发现配网     |
| themeColor      | UI 主题色设置                |



#### 接口介绍

业务包基础库，提供业务包调用的入口方法，同时也提供了客户需要实现协议服务的注册方法，具体提供的方法如下图：

```objc
/**
 * Get the instance which implement the special service protocol
 * eg:
 *  id<TYMallProtocol> impl = [[TuyaSmartBizCore sharedInstance] serviceOfProtocol:@protocol(TYMallProtocol)];
 *  [impl xxx]; // your staff...
 *
 * @param serviceProtocol service protocol
 * @return instance
 */
- (id)serviceOfProtocol:(Protocol *)service;

/**
 * Register a instance for service which can not be served by BizBundle itself
 * Each service can only register one instance or class at a time, whichever is the last
 *
 * @param service   service protocol
 * @param instance  instance which conform to the service protocol, strong reference
 *                  unregister if nil
 */
- (void)registerService:(Protocol *)service withInstance:(id)instance;

/**
 * Register a class for service which can not be served by BizBundle itself
 * Each service can only register one instance or class at a time, whichever is the last
 *
 *
 * @param service   service protocol
 * @param cls       class which conform to the service protocol, [cls new] to get instance
 *                  unregister if nil
 */
- (void)registerService:(Protocol *)service withClass:(Class)cls;

/**
 * Register route handler for route which can not be handled by BizBundle itself
 * @param handler   block to handle route
 *                  @param url  the route url
 *                  @param raw  the route raw data
 *                  @return true if route can be handled, otherwise return false
 */
- (void)registerRouteWithHandler:(BOOL(^)(NSString *url, NSDictionary *raw))handler;
```



### `TYModuleServices`

提供各个业务包实现的服务协议



## 快速集成

### 使用 CocoaPods 集成

在 `Podfile` 文件中添加以下内容完成业务包核心库添加：

```ruby
source "https://github.com/TuyaInc/TYPublicSpecs.git"
source 'https://cdn.cocoapods.org/'

platform :ios, '9.0'

target 'your_target_name' do
   pod "TuyaSmartBizCore"
   pod "TYModuleServices"
   # 若需要相关业务包功能，请添加相关业务库
   pod "TuyaSmartXXXBizBundle"
end
```



然后在项目根目录下执行 `pod update` 命令，集成第三方库。

CocoaPods 的使用请参考：[CocoaPods Guide](https://guides.cocoapods.org)



### 初始化业务包

1. 打开项目设置，Target => General，修改```Bundle Identifier```为涂鸦开发者平台上注册的 App 对应的 iOS 包名。

2. 将上面准备工作中下载的安全图片导入到工程根目录，重命名为```t_s.bmp```，并加入「项目设置 => Target => Build Phases => Copy Bundle Resources」中。

3. 在项目的```PrefixHeader.pch```文件添加以下内容（Swift 项目可以添加在```xxx_Bridging-Header.h```桥接文件中）：

   ```objc
   #import <TuyaSmartBizCore/TuyaSmartBizCore.h>
   ```

4. 打开`AppDelegate.m`文件，在`[AppDelegate application:didFinishLaunchingWithOptions:]`方法中，使用在涂鸦开发者平台上，App 对应的 `App Key`，`App Secret` 初始化SDK：

   ObjC

   ```objc
   [[TuyaSmartSDK sharedInstance] startWithAppKey:<#your_app_key#> secretKey:<#your_secret_key#>];
   ```

   Swift

   ```swift
   TuyaSmartSDK.sharedInstance()?.start(withAppKey: <#your_app_key#>, secretKey: <#your_secret_key#>)
   ```

5. 填写业务包初始化需要的配置

   ```json
   {
       "config": {
           "appId": 123,    
           "tyAppKey": "xxxxxxxxxxxx", 
           "appScheme": "tuyaSmart",
           "hotspotPrefixs": "SmartLife",
           "needBle": true,
           "scanDeviceClose": false,
       },
      "colors":{
           "themeColor": "#FF5A28", 
       }
   }
   ```

   生成名为 ty_custom_config.json 的 json 文件，放到工程目录下。

至此，涂鸦业务包准备工作已经完成，可以开始业务包的使用，具体业务包的集成使用请到业务包导航目录下寻找相应的说明。

### Debug 模式

在开发的过程中可以开启 Debug 模式，打印一些日志用于分析问题。

ObjC

```objc
#ifdef DEBUG
    [[TuyaSmartSDK sharedInstance] setDebugMode:YES];
#else
#endif
```

Swift

```swift
#if DEBUG
   TuyaSmartSDK.sharedInstance()?.debugMode = true
#else
#endif

```















