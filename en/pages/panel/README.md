# Device Control BizBundle

## Features Overview

- Tuya Smart iOS Device Control BizBundle is the core container of Tuya Smart Device Control Panel, based on the Tuya Smart iOS Home SDK, it provides the interface package for loading and controlling the device control panel to speed up the application development process. It mainly includes the following functions:
  - Load Device Panel (Supported hardware device type: WIFI、Zigbee、Mesh、BLE)
  - Device Panel Control (Supported Device and Group Control, Unsupported Group Manager)
  - Device Alarm



## Integrate

Add the `TuyaSmartPanelBizBundle` in the project's `Podfile` file and execute the` pod update` command

```ruby
source "https://github.com/TuyaInc/TuyaPublicSpecs.git"
source 'https://cdn.cocoapods.org/'

target 'your_target_name' do
  # TuyaSmart SDK
  pod "TuyaSmartHomeKit"
  # Add Device Control BizBundle
  pod 'TuyaSmartPanelBizBundle', 'xxx'
  # If you need the sweeper, please rely on the relevant plug-in of the sweeper
  # pod 'TuyaRNApi/Sweeper'
end
```

**Note**

The Device Control BizBundle encapsulates a series of RN interfaces for the panel to call, which will involve some of Apple's privacy rights statements.

- If the connected device panel is related to the use of photo albums (for example: cloud albums), you need to add the following permission statement in the project's `info.plist`:

```
NSPhotoLibraryAddUsageDescription
```

- If the connected device panel is related to using the camera (for example: cloud album), you need to add the following permission statement in the project's `info.plist`:

```
NSCameraUsageDescription
```

- If the connected device panel is related to the use of location information, you need to add the following permission statement in the project's `info.plist`:

```
NSLocationWhenInUseUsageDescription
```

- If the connected device panel uses a microphone (for example: a music lamp panel), you need to add the following permission statement in the project's `info.plist`:

```
NSMicrophoneUsageDescription
```

- If the connected device panel is Bluetooth related, you need to add the following permission statement in the project's `info.plist`:

```
NSBluetoothAlwaysUsageDescription
NSBluetoothPeripheralUsageDescription
```



## Service Protocol

### Provide Service

BizBundle  provide services of  `TYPanelProtocol.h ` in `TYModuleServices` component as follows:

```objc
@protocol TYPanelProtocol <NSObject>
NS_ASSUME_NONNULL_BEGIN

// clear panel cache
- (void)cleanPanelCache;

/**
 * jump to device control panel，by push
 *
 * @param device        DeviceModel
 * @param group         GroupModel
 * @param initialProps  Custom initialization parameters will be set into the initialProps of the RN application with'extraInfo' as the key
 * @param contextProps  Custom panel context will be set into Panel Context with'extraInfo' as key
 * @param completion Result callback
 */
- (void)gotoPanelViewControllerWithDevice:(TuyaSmartDeviceModel *)device
                                    group:(nullable TuyaSmartGroupModel *)group
                             initialProps:(nullable NSDictionary *)initialProps
                             contextProps:(nullable NSDictionary *)contextProps
                               completion:(void(^ _Nullable)(NSError * _Nullable error))completion;

/**
 * jump to device control panel，by present
 *
 * @param device        DeviceModel
 * @param group         GroupModel
 * @param initialProps  Custom initialization parameters will be set into the initialProps of the RN application with'extraInfo' as the key
 * @param contextProps  Custom panel context will be set into Panel Context with'extraInfo' as key
 * @param completion Result callback
 */
- (void)presentPanelViewControllerWithDevice:(TuyaSmartDeviceModel *)device
                                       group:(nullable TuyaSmartGroupModel *)group
                                initialProps:(nullable NSDictionary *)initialProps
                                contextProps:(nullable NSDictionary *)contextProps
                                  completion:(void (^ _Nullable)(NSError * _Nullable error))completion;

// RN Version For BizBundle
- (NSString *_Nonnull)rnVersionForApp;

NS_ASSUME_NONNULL_END
@end
```



### Dependent Services

The main function of the device control biz bundle is to load the device, and there will be different functions for different devices. To ensure the normal operation of these functions, it will rely on the following protocols： `TYSmartHomeDataProtocol`、 `TYDeviceDetailProtocol`、 `TYSettingsProtocol`、 `TYOTAGeneralProtocol`、 `TYGroupHandleProtocol`、 `TYRNCameraProtocol`、 `TYCameraProtocol`。

#### TYSmartHomeDataProtocol

Provide the current family information required to load the device panel, **must be implemented**

```objc
/**
 Get the current family. If there is no family, return nil.
 
 @return TuyaSmartHome
 */
- (TuyaSmartHome *)getCurrentHome;
```

####  TYDeviceDetailProtocol

Click the jump event in the Panel Toolbar Right Menu of the device panel interface

```objc
/**
 Click the button click event on the right side of the navigation bar to jump to the device details page

 @param device DeviceModel
 @param group GroupModel, maybe nil.
 */
- (void)gotoDeviceDetailDetailViewControllerWithDevice:(TuyaSmartDeviceModel *)device group:(TuyaSmartGroupModel *)group;
```

#### TYSettingsProtocol

When triggering the release of dp control devices in the panel, an optional switch is provided for the app to emit sound effects. Implement the following method to return whether to enable sound effects:

```objc
/**
 * Check if audio is turned on
 */
- (BOOL)soundEnabled;
```

#### TYOTAGeneralProtocol

When entering the device panel, provide an event to check the device firmware update. Implement the following method to check the firmware upgrade:

```objc
/**
 Check the device firmware update, if there is an update, it will display the firmware update prompt
 
 @param deviceModel DeviceModel
 @param isManual Whether to manually detect the upgrade
 @param theme theme color
 */
- (void)checkFirmwareUpgrade:(TuyaSmartDeviceModel *)deviceModel isManual:(BOOL)isManual theme:(TYOTAControllerTheme)theme;
```

#### TYGroupHandleProtocol

For BLE Mesh devices, there are events in the panel that need to jump to the Mesh group interface. If you need to jump to the mesh group, implement the following methods:

```objc
/**
Jump to local mesh group

@available 1.0.0
@param params 
@param success 
@param failure 
*/
- (void)impl_jumpToMeshLocalGroup:(NSDictionary*)params success:(RCTResponseSenderBlock)success failure:(RCTResponseErrorBlock)failure ;
```

#### TYRNCameraProtocol

Load the jump event of the RN panel of the camera device, if you access the camera panel service package `TuyaSmartCameraRNPanelBizBundle` at the same time, it does not need to be implemented;

```objc
/**
 Get camera RN panel
 @param devId The devId of the camera device
 */
- (UIViewController *)cameraRNPanelViewControllerWithDeviceId:(NSString *)devId;
```

#### TYCameraProtocol

Load the jump event of the native panel of the camera device. If the camera panel business package `TuyaSmartCameraPanelBizBundle` is connected at the same time, it does not need to be implemented;

```objc
/**
	Get Camera Native Panel
 @param devId The devId of the camera device
 @param uiName The uiName of the camera device, different uiName corresponds to different versions of the panel
 */
- (UIViewController *)viewControllerWithDeviceId:(NSString *)devId uiName:(NSString *)uiName;
```



## Guidance

### Attention

1、Make sure that the user is logged in before using any interface

2、Before using bizbundle，must  implement the protocol method `getCurrentHome` in `TYSmartHomeDataProtocol`

Objective-C 

```objc
#import <TuyaSmartBizCore/TuyaSmartBizCore.h>
#import <TYModuleServices/TYSmartHomeDataProtocol.h>

- (void)initCurrentHome {
    // register service
    [[TuyaSmartBizCore sharedInstance] registerService:@protocol(TYSmartHomeDataProtocol) withInstance:self];
}

// implementation
- (TuyaSmartHome *)getCurrentHome {
    TuyaSmartHome *home = [TuyaSmartHome homeWithHomeId:@"current_home_id"];
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

### Clear Panel Cache

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

### Open Panel

Objective-C 

```objc
id<TYPanelProtocol> impl = [[TuyaSmartBizCore sharedInstance] serviceOfProtocol:@protocol(TYPanelProtocol)];
// Method 1: Jump by default Push method
[impl gotoPanelViewControllerWithDevice:deviceModel group:nil initialProps:nil contextProps:nil completion:^(NSError * _Nullable error) {
    if (error) {
        NSLog(@"Load error: %@", error);
    }
}];
// Method 2: Jump using Present
[impl presentPanelViewControllerWithDevice:deviceModel group:nil initialProps:nil contextProps:nil completion:^(NSError * _Nullable error) {
    if (error) {
        NSLog(@"Load error: %@", error);
    }
}];
// Method 3: Get the panel view controller and jump by itself
[impl getPanelViewControllerWithDeviceModel:device groupModel:group initialProps:nil contextProps:nil completionHandler:^(__kindof UIViewController * _Nullable panelViewController, NSError * _Nullable error) {

}];
```

Swift

```swift
let impl = TuyaSmartBizCore.sharedInstance().service(of: TYPanelProtocol.self) as? TYPanelProtocol
// Method 1: Jump by default Push method
impl?.gotoPanelViewController(withDevice: deviceModel!, group: nil, initialProps: nil, contextProps: nil, completion: { (error) in
    if let e = error {
        print("\(e)")
    }
})
// Method 2: Jump using Present
impl?.presentPanelViewController(withDevice: deviceModel!, group: nil, initialProps: nil, contextProps: nil, completion: { (error) in
    if let e = error {
        print("\(e)")
    }
})
// Method 3: Get the panel view controller and jump by itself
let impl = TuyaSmartBizCore.sharedInstance().service(of: TYPanelProtocol.self) as? TYPanelProtocol
impl?.getPanelViewController(with: deviceModel!, groupModel: nil, initialProps: nil, contextProps: nil, completionHandler: { (panelViewController, error) in
		if let e = error {
        print("\(e)")
    }
})
```

## FAQ

**1. Why does the navigation bar style of other view controllers change after calling the Push method to jump to the panel?**

The essence of the device control business package is to provide a view controller to load the interface rendered by React-Native, so the navigation bar is hidden inside the view controller, mainly to change the transparency. Therefore, after using the push method to enter the panel controller, when pop back to its own controller, you need to restore the style (the main reason is that the push and pop animations provided by iOS are managed by NavigationController, and the NavigationBar used is the only one; in NavigationController Under the Stack storage structure, whenever the ViewController in the Stack modifies the navigation bar, it will inevitably affect the display effect of other ViewControllers). Two ways to restore styles are suggested below:

Case 1.

```objective-c
// 1. Import header file
#import <TYNavigationController/TYNavigationTopBarProtocol.h>
// 2. Show navigation bar
self.ty_topBarHidden = NO;
self.ty_topBarAlpha = 1.0;
// 3. Restore the navigation bar style (for example: modify the background color)
self.ty_topBarBackgroundImage = <#Image#>;
// Or
self.ty_topBarBackgroundColor = <#Color#>;
```

Case 2.

```objective-c
// 1. Import header file
#import <TYNavigationController/TYNavigationCallbackProtocol.h>
// 2. Implement the proxy method provided by TYNavigationCallbackProtocol
- (void)ty_naviTransitioning:(id<UIViewControllerTransitionCoordinatorContext>)context {
    // 3. Show navigation bar
    [self.navigationController setNavigationBarHidden:false animated:true];
    // 4. Restore the navigation bar style (for example: modify the background color)
    [self.navigationController.navigationBar setBarTintColor:<#Color#>];
  	// Or
    [self.navigationController.navigationBar setBackgroundImage:<#Image#> forBarMetrics:UIBarMetricsDefault];
}
```

**2. When relying on plugins, `pod update` reports an error?**

When the project uses cocoapods to integrate the business package, due to the logic of the pod, when the version of each Pod library is not specified in the Podfile, the latest official version number (x.y.z, where x, y, and z are all numbers) will be pulled by default. If you need to rely on the extended functions of the device control service package, you need to rely on the relevant plug-in. Since the version number of the plug-in is pre-release, you need to specify the version number marked with the latest date of the baseline of the device control service package in the Podfile ( [Click to view the business package version number](../versions.md)).