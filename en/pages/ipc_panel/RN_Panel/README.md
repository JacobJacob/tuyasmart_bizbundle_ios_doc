# TuyaSmart iOS IPC ReactNative biz Bundle

## Functional Overview

TuyaSmart iOS IPC ReactNatice biz Bundle is a series of panel SDK related to camera functions developed based on [Tuya Smart Camera SDK](<https://tuyainc.github.io/tuyasmart_camera_ios_sdk_doc/>). It mainly includes the following functions:

-Preview panel, playback panel, cloud storage panel, message center panel, album panel, settings panel.

## Integrate

Add the following code to the ```Podfile``` file:

```ruby
source "https://github.com/TuyaInc/TuyaPublicSpecs.git"
source 'https://cdn.cocoapods.org/'

target 'your_target_name' do
  # add cameraRNPanelBizBundle
  pod 'TuyaSmartCameraRNPanelBizBundle'
end
```

Then execute the ``pod update`` command in the project root directory to integrate third-party libraries.

Please refer to the use of CocoaPods: [CocoaPods Guides](https://guides.cocoapods.org/)

**Note**

The BizBundle encapsulates a series of RN interfaces for the panel to call, which will involve some of Apple's privacy rights statements.

- If the connected device panel is related to the use of photo albums (for example: albums), you need to add the following permission statement in the project's `info.plist`:

```
NSPhotoLibraryAddUsageDescription
```

- If the connected device panel uses a microphone (for example: camera talk), you need to add the following permission statement in the project's `info.plist`:

```
NSMicrophoneUsageDescription
```

## Service Protocol

### Provide Service

BizBundle  provide services of  `TYRNCameraProtocol.h ` in `TYModuleServices` component as follows:

```objc
#import <UIKit/UIKit.h>

@protocol TYRNCameraProtocol <NSObject>

/**
 obtain camera RN Panel
 @param devId deviceModel.devId
 */
- (UIViewController *)cameraRNPanelViewControllerWithDeviceId:(NSString *)devId;

@end
```

### Dependent Services

The main function of the biz bundle is to load the device, and there will be different functions for different devices. To ensure the normal operation of these functions, it will rely on the following protocols： `TYSmartHomeDataProtocol` 、 `TYOTAGeneralProtocol`。

#### TYSmartHomeDataProtocol

Provide the current family information required to load the device panel, **must be implemented**

```objc
/**
 Get the current family. If there is no family, return nil.
 
 @return TuyaSmartHome
 */
- (TuyaSmartHome *)getCurrentHome;
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

## Guidance

### Attention

1. Make sure that the user is logged in before using any interface
2. This interface is only fit for camera device, which device.deviceModel.category is 'sp'.
3. Before using bizbundle，must  implement the protocol method `getCurrentHome` in `TYSmartHomeDataProtocol`

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
    TuyaSmartHome *home = [TuyaSmartHome homeWithHomeId:@"current home id"];
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

### 

### Obtain Preview Panel (UIViewController)

Camera react-native preview panel, including real-time video preview, sharpness switch, sound switch control, screenshot, recording, intercom and other functions, motion detection, PTZ direction control, favorite point addition / deletion, cruise control, etc.

Objc

```objc
id<TYRNCameraProtocol> impl = [[TuyaSmartBizCore sharedInstance] serviceOfProtocol:@protocol(TYRNCameraProtocol)];
UIViewController *vc = [impl cameraRNPanelViewControllerWithDeviceId:self.deviceModel.devId];
[self.navigationController pushViewController:vc animated:YES];
```

Swift

```objc
let impl = TuyaSmartBizCore.sharedInstance().service(of: TYRNCameraProtocol.self) as? TYRNCameraProtocol
impl?.cameraRNPanelViewControllerWithDeviceId(withDeviceId: deviceModel.devId!) 
```