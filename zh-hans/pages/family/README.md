# 家庭业务包

## 介绍

家庭业务包提供了家庭管理的能力，包括：
* 创建家庭
* 加入家庭
* 家庭设置

其中家庭设置包含的功能有：
* 设置家庭名称
* 房间管理
* 家庭成员管理

## 接入组件

在工程的 `Podfile` 文件中添加家庭业务包组件，并执行 `pod update` 命令

```ruby
source "https://github.com/TuyaInc/TuyaPublicSpecs.git"
source 'https://cdn.cocoapods.org/'

target 'your_target_name' do
  # TuyaSmart SDK
  pod "TuyaSmartHomeKit"
  # 添加 家庭业务包
  pod 'TuyaSmartFamilyBizBundle', '~> 3.22.0'
end
```

## 服务协议

### 提供服务

家庭业务包实现 `TYFamilyProtocol` 协议以提供服务，在 `TYModuleServices` 组件中查看 `TYFamilyProtocol.h` 协议文件内容为：

```objc
#import <Foundation/Foundation.h>
#import <UIKit/UIKit.h>

typedef void(^DoneActionBlock)(NSString *changedName);

/// Family Management Service
@protocol TYFamilyProtocol <NSObject>

@optional

/// jump to Family Management ViewController
- (void)gotoFamilyManagement;

@end
```

## 使用指南

### 注意事项

1. 使用任何接口之前，务必确认用户已登录

### 跳转到家庭管理页面

Objective-C 示例

```objc
#import <TYModuleServices/TYModuleServices.h>
#import <TuyaSmartBizCore/TuyaSmartBizCore.h>

id<TYFamilyProtocol> impl = [[TuyaSmartBizCore sharedInstance] serviceOfProtocol:@protocol(TYFamilyProtocol)];
if ([impl respondsToSelector:@selector(gotoFamilyManagement)]) {
    [impl gotoFamilyManagement];
}
```

Swift 示例

```swift
guard let impl = TuyaSmartBizCore.sharedInstance().service(of: TYFamilyProtocol.self) as? TYFamilyProtocol else {
    return
}
impl.gotoFamilyManagement?()
```

### 邀请家庭成员

#### 邀请
在 `家庭设置`页面有个 `添加成员` 功能，目前有两类邀请添加方式：
1. 通过涂鸦账号邀请，输入对方的账号信息即可邀请对方加入自己的家庭
2. 通过邀请码方式邀请对方加入家庭

#### 接受邀请
1. 对于邀请码方式，直接在`家庭管理`->`加入一个家庭`页面输入邀请码即可

2. 对于涂鸦账号邀请方式，对方再次进入`家庭管理`页面后会显示`待加入家庭`, 点击可以接受或者拒绝

若要实现实时弹出邀请弹窗的需求，可以通过设置 `TuyaSmartHomeManager` 的代理实现
```objc
- (void)homeManager:(TuyaSmartHomeManager *)manager didAddHome:(TuyaSmartHomeModel *)home;
```
通过 `TuyaSmartHomeModel` 的 `dealStatus`判断是否是新邀请家庭
```objc
typedef NS_ENUM(NSUInteger, TYHomeStatus) {
    TYHomeStatusPending = 1,      /**< 待加入 受邀者未决定是否加入对应家庭 Not deciding whether to join the home */
    TYHomeStatusAccept,           /**< 受邀者已同意加入对应家庭 The invitee has agreed to join the home */
    TYHomeStatusReject            /**< 受邀者已拒绝加入对应家庭 The invitee have refused to join the home */
};
```

实现邀请弹窗代码示例 Objective-C：
```objc
TuyaSmartHomeManager *manager = [TuyaSmartHomeManager new];
manager.delegate = self;

....
以下为当前控制器代理实现代码部分

- (void)homeManager:(TuyaSmartHomeManager *)manager didAddHome:(TuyaSmartHomeModel *)homeModel {
    if (homeModel.dealStatus <= TYHomeStatusPending && homeModel.name.length > 0) {
    ///弹出接受邀请弹窗代码
    
    ///接受邀请
    TuyaSmartHome *home = [TuyaSmartHome homeWithHomeId:homeModel.homeId];
    [home joinFamilyWithAccept:YES success:^(BOOL result) {} failure:^(NSError *error) {}];
    
    ///拒绝邀请
    [home joinFamilyWithAccept:NO success:^(BOOL result) {} failure:^(NSError *error) {}];
    }
}
```

实现邀请弹窗代码示例 Swift：
```Swift
let manager = TuyaSmartHomeManager()
manager.delegate = self

....
以下为当前控制器代理实现代码部分

func homeManager(_ manager: TuyaSmartHomeManager!, didAddHome homeModel: TuyaSmartHomeModel!) {
    if (homeModel.dealStatus.rawValue <= TYHomeStatus.pending.rawValue) && (homeModel.name.isEmpty == false) {
        ///弹出接受邀请弹窗代码
        
        ///接受邀请
        let home = TuyaSmartHome(homeId: homeModel.homeId)
        home?.joinFamily(withAccept: true, success: { (result) in
            
        }, failure: { (error) in
            
        })
        
        ///拒绝邀请
        home?.joinFamily(withAccept: false, success: { (result) in
            
        }, failure: { (error) in
            
        })
        
    }
}

```