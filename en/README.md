# Tuya iOS BizBundle

## Overview

Tuya iOS BizBundle is a vertical business bundle that contains logic modules and UI pages. It's designed to provide customers with the ability to quickly access the Tuya business module based on the Tuya Smart Home SDK.

Tuya iOS BizBundle currently offered includes:
- H5 Mall
- Device Control Panel
- IPC Device Control Panel
- Device Activation

## Architecture

Tuya iOS BizBundle is opened in a service-oriented manner, and all function access is provided in the form of `Protocol`.

![Architecture](./pages/images/architecture.png)

There are two ways to use the protocol: **get service** and **provide service**

### Get Service

Obtain the instance which implement the service protocol provided by the BizBundle through `BizCore`, then call its service method to achieve the business purpose

```mermaid
classDiagram
    Protocol <|.. BizBundle : 业务包实现服务协议
    Protocol .. BizCore
    BizCore <.. YourClass : 获取实现服务协议的实例
    class Protocol{
        +doSomeThing()
    }
    class BizCore{
        +serviceOfProtocol() id~Protocol~
    }
    class BizBundle{
    }
    class YourClass{
        +id~Protocol~ serviceImpl
    }
```

### Provide Service

Some BizBundle depend on services that are not implemented, At this time, you can provide your own class to implement the corresponding service, and register it to `BizCore` to improve the BizBundle functions

```mermaid
classDiagram
    Protocol <|.. YourClass : 实现服务协议
    BizCore <.. YourClass : 注册你的服务实现类或实例
    Protocol .. BizCore
    BizCore <.. BizBundle
    class BizBundle{
    }
    class Protocol{
        +doSomeThing()
    }
    class BizCore{
        +registerService(Protocol, Instance)
        +registerService(Protocol, Class)
        +registerRouteWithHandler(Block)
    }
    class YourClass{
        +doSomeThing()
    }
```









