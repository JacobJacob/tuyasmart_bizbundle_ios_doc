# Network Configuration BizBundle

Device Activation BizBundle supports  device configuration logic and UI interface of Tuya App. Business functions cover different types of devices such as Wi-Fi devices, ZigBee devices, Bluetooth devices, and QR code scanning devices (such as GPRS & NB-IoT devices) that currently support Tuya Smart.  BizBundle provide devices network operation guidance and specific network access process implementation



## Access

Add the bizbundle in  the `Podfile` file, then execute pod update command

```ruby
source "https://github.com/TuyaInc/TYPublicSpecs.git"
source 'https://cdn.cocoapods.org/'

target 'your_target_name' do
  # add device config bizbundle
  pod 'TuyaSmartActivatorBizBundle'
end
```



**Note**

The Wi-Fi device Activation process needs to obtain the  ssid of the current Wi-Fi , and the project needs to add the following permission statement in info.plist. Then create a CLLocationManager object, and call the requestWhenInUseAuthorization method.

```
NSLocationAlwaysAndWhenInUseUsageDescription
NSLocationAlwaysUsageDescription
NSLocationWhenInUseUsageDescription
```



The QR code scanning function requires system camera permissions, and the following permission statement needs to be added to info.plist.

```
NSCameraUsageDescription
```



## Custom Config

### 1. Bluetooth Function

BizBundle supports Wi-Fi, Bluetooth and other types of devices to connect to the network. Set `needBle` false，when not needed. 

If you need the Bluetooth distribution function, you need to add the Bluetooth permission declaration in the project's info.Plist file, set the `needBle`  property in `ty_custom_config.json`  to true, and then add the following dependencies to the project:

**Permission statement**

```
NSBluetoothAlwaysUsageDescription
NSBluetoothPeripheralUsageDescription
```

**Configuration**

```json
{
    "config": {
        "appId": 123,    
        "tyAppKey": "xxxxxxxxxxxx", 
        "appScheme": "tuyaSmart",
        "hotspotPrefixs": ["SmartLife"], 
        "needBle": true // true ,need bluetooth
    },
   "colors":{
        "themeColor": "#FF5A28", 
    }
}
```

**Dependencies**

```ruby
  pod 'TYBLEInterfaceImpl'
  pod 'TYBLEMeshInterfaceImpl'
  pod 'TuyaSmartBLEKit'
  pod 'TuyaSmartBLEMeshKit'
```



### 2. Custom Device Hotspot Name Prefix

Tuya Device hotspot name prefix defaults to `SmartLife`.  Can custom hotspot name prefix as your device's through setting `hotspotPrefixs` in `ty_custom_config.json`.

```json
{
    "config": {
        "appId": 123,    
        "tyAppKey": "xxxxxxxxxxxx", 
        "appScheme": "tuyaSmart",
        "hotspotPrefixs": ["SL"], //  device hotspots name prefix 
        "needBle": true
    },
   "colors":{
        "themeColor": "#FF5A28", 
    }
}

```



## Service Protocol

### Provide Service

BizBundle  provide services of  `TYActivatorProtocol.h ` in `TYModuleServices` component as follows:

```objc
#ifndef TYActivatorProtocol_h
#define TYActivatorProtocol_h

@class TuyaSmartHome;

@protocol TYActivatorProtocol <NSObject>

/**
 * Start config
 * Goto device config list view 
 */
- (void)gotoCategoryViewController;

/**
 *  Obtain device information after each device connection
 */
- (void)getActivatedDeviceListWithCompletion:(void (^)(NSArray *_Nullable devIdList))completion;

@end
#endif /* TYActivatorProtocol_h */

```



### Dependent Services

BizBundle running depends on the protocol method provided by TYSmartHomeDataProtocol. Before calling the BizBundle method, the following protocol needs to be implemented

#### TYSmartHomeDataProtocol

Set current home infomation  by implementing the following method.

```objc
/**
 Get current home info 
 
 @return TuyaSmartHome
 */
- (TuyaSmartHome *)getCurrentHome;
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
    TuyaSmartHome *home = [TuyaSmartHome homeWithHomeId:@"current home id"];
    return home;
}
```



Swfit 

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



### Start Config

Objective-C 

```objc
#import <TuyaSmartBizCore/TuyaSmartBizCore.h>
#import <TYModuleServices/TYActivatorProtocol.h>


- (void)gotoDeviceConfig {
    id<TYActivatorProtocol> impl = [[TuyaSmartBizCore sharedInstance] serviceOfProtocol:@protocol(TYActivatorProtocol)];
    [impl gotoCategoryViewController];
  
    // get result
    [impl activatorCompletion:TYActivatorCompletionNodeNormal customJump:NO completionBlock:^(NSArray * _Nullable deviceList) {
        NSLog(@"deviceList: %@",deviceList);
    }];
}
```



Swfit 

```swift
let homeImpl = TuyaSmartBizCore.sharedInstance().service(of: TYActivatorProtocol.self) as? TYActivatorProtocol
homeImpl?.goActivatorRootView()

impl?.activatorCompletion(TYActivatorCompletionNodeNormal, customJump: false, completionBlock: { (evIdList:[Any]?) in
            print(devIdList ?? [])
        })
```


