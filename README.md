# JPush-RN

极光推送 `JPush` SDK 的 `React Native` 封装，支持 `Android` 和 `iOS`，Fork 自 [jpush-react-native](https://github.com/jpush/jpush-react-native)。

----

为啥自己维护一份呢？因为官方 `jpush-react-native` 项目做得比较早，各方面更新比较慢。最初我跟着 `jpush-react-native` 文档配置，很多东西没生效，最后去看官方 [Android SDK 集成](https://docs.jiguang.cn/jpush/client/Android/android_guide)和 [iOS SDK 集成](https://docs.jiguang.cn/jpush/client/iOS/ios_guide_new)文档才恍然大悟。我把这部分的配置说明写在本文档下方，希望对你有帮助。

如果有一点 Android 和 iOS 开发经验，建议也去看下官方的[客户端 SDK](https://docs.jiguang.cn/jpush/client/)文档，很多问题在里面都可以找到答案。

有疑问或者本项目存在问题，请在 [Issues](https://github.com/atypiape/jpush-rn/issues) 中反馈，非常感谢。

**备注：** 
- 对 Android 14 (API 34) 做了适配。
- 修复 iOS 报错："JPushModule.setDebugMode(): Error while converting JavaScript argument 0 to Objective C type BOOL. Objective C type BOOL is unsupported."

## 1. 安装

依赖到 `JCore` SDK，所以 [jcore-rn](https://www.npmjs.com/package/jcore-rn) 也需要安装。

**NPM:**

```bash
npm i jcore-rn jpush-rn
```

**YARN:**

```bash
yarn add jcore-rn jpush-rn
```

## 2. SDK 版本

### 2.1. Android

`Android` 使用 `mavenCentral` 自动集成 [JPush](https://mvnrepository.com/artifact/cn.jiguang.sdk/jpush) SDK，当前版本为 `5.6.0`。

### 2.2. iOS

`iOS` 使用 `Cocoapods` 自动导入 [JPush](https://cocoapods.org/pods/JPush) SDK，当前版本为 `5.5.0`。

## 3. 配置

### 3.1 Android

#### 3.1.1 链接静态库

React Native 0.60 及以上版本是自动链接的，无需理会。如果你的项目是 `0.59` 及以下版本，请参考 [jpush-react-native](https://www.npmjs.com/package/jpush-react-native) 中的手动配置方式。

#### 3.1.2 添加 JPush 配置

* 修改 `android/build.gradle`，添加以下内容：

```groovy
android {
    defaultConfig {
        applicationId "yourApplicationId"   // 在此替换你的应用包名
        ...
        manifestPlaceholders = [
            JPUSH_APPKEY: "yourAppKey",     // 在此替换你的 APPKey
            JPUSH_CHANNEL: "yourChannel"    // 在此替换你的 channel
        ]
    }
}
```

* 修改 `android/app/src/main/AndroidManifest.xml`，添加以下内容：

```xml
<meta-data
  android:name="JPUSH_CHANNEL"
  android:value="${JPUSH_CHANNEL}" />
<meta-data
  android:name="JPUSH_APPKEY"
  android:value="${JPUSH_APPKEY}" />
    
<!-- Required since 5.2.0, 用于接收应用内消息等 -->
<service
    android:name="cn.jiguang.plugins.push.receiver.JPushModuleReceiver"
    android:enabled="true"
    android:exported="false" >
    <intent-filter>
        <action android:name="cn.jpush.android.intent.RECEIVER_MESSAGE" />
        <category android:name="${applicationId}" />
    </intent-filter>
</service>
```

* 修改 `android/app/proguard-rules.pro`，添加以下内容：

```groovy
-dontoptimize
-dontpreverify

-dontwarn cn.jpush.**
-keep class cn.jpush.** { *; }
-keep class * extends cn.jpush.android.service.JPushMessageReceiver { *; }

-dontwarn cn.jiguang.**
-keep class cn.jiguang.** { *; }
```

#### 3.1.3 重新编译项目

正常情况下，执行 `react-native run-android` 编译运行即可。

如果报错，可以尝试清除缓存，再重新编译：

```bash
# 清除 Android 编译缓存
cd android
./gradlew clean
rm -rf .gradle
cd ..

# 删除 node_modules 目录，重新安装依赖
rm -rf node_modules
npm install # 或者 yarn install
```

### 3.2 iOS

#### 3.2.1 链接静态库

React Native 0.60 及以上版本，只需执行下面的命令即可。如果你的项目是 `0.59` 及以下版本，请参考 [jpush-react-native](https://www.npmjs.com/package/jpush-react-native) 中的手动配置方式。

```bash
cd ios
pod install
```

如果报错，可尝试执行命令：

```bash
pod deintegrate
```

#### 3.2.2 添加 JPush 配置

**引入头文件**

将以下代码添加到 `AppDelegate.m` 引用头文件的位置

```objc
// 引入 jpush-rn 模块头文件
#import <RCTJPushModule.h>
#import "JPUSHService.h"
// iOS10 注册 APNs 所需头文件
#ifdef NSFoundationVersionNumber_iOS_9_x_Max
#import <UserNotifications/UserNotifications.h>
#endif
// 如果需要使用 idfa 功能，引入所需要的头文件（可选）
// #import <AdSupport/AdSupport.h>
```
**添加初始化 APNs 代码**

将以下代码添加到 `AppDelegate.m` 文件的 `-(BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions` 中：

```objc
// Required, 初始化 APNs
JPUSHRegisterEntity *entity = [[JPUSHRegisterEntity alloc] init];
if (@available(iOS 12.0, *)) {
  entity.types = JPAuthorizationOptionAlert | JPAuthorizationOptionBadge |
                 JPAuthorizationOptionSound |
                 JPAuthorizationOptionProvidesAppNotificationSettings;
}
[JPUSHService
    registerForRemoteNotificationConfig:entity
                               delegate:(id<JPUSHRegisterDelegate>)self];
```

**注册 APNs 成功回调方法，并上报 DeviceToken**

将以下代码添加到 `AppDelegate.m` 文件中：

```objc
- (void)application:(UIApplication *)application
didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken {

  /// Required - 注册 DeviceToken
  [JPUSHService registerDeviceToken:deviceToken];
}
```

**添加处理 APNs 通知回调方法**

将以下代码添加到 `AppDelegate.m` 文件中：

```objc
// iOS 12 Support
- (void)jpushNotificationCenter:(UNUserNotificationCenter *)center
    openSettingsForNotification:(UNNotification *)notification {
  if (notification && [notification.request.trigger
                          isKindOfClass:[UNPushNotificationTrigger class]]) {
    // 从通知界面直接进入应用
  } else {
    // 从通知设置界面进入应用
  }
}

- (void)application:(UIApplication *)application
    didReceiveRemoteNotification:(NSDictionary *)userInfo
          fetchCompletionHandler:
              (void (^)(UIBackgroundFetchResult))completionHandler {
  // Required, iOS 7 Support
  [JPUSHService handleRemoteNotification:userInfo];
  // 传递给 React Native
  [[NSNotificationCenter defaultCenter]
      postNotificationName:J_APNS_NOTIFICATION_ARRIVED_EVENT
                    object:userInfo];
  completionHandler(UIBackgroundFetchResultNewData);
}

- (void)application:(UIApplication *)application
    didReceiveRemoteNotification:(NSDictionary *)userInfo {

  // Required, For systems with less than or equal to iOS 6
  [JPUSHService handleRemoteNotification:userInfo];
}

// 自定义消息
- (void)networkDidReceiveMessage:(NSNotification *)notification {
  NSDictionary *userInfo = [notification userInfo];
  [[NSNotificationCenter defaultCenter]
      postNotificationName:J_CUSTOM_NOTIFICATION_EVENT
                    object:userInfo];
}
```

**关于应用内消息**

[官方文档](https://docs.jiguang.cn/jpush/client/iOS/ios_sdk#%E5%BA%94%E7%94%A8%E5%86%85%E6%B6%88%E6%81%AF)中有提到，应用内消息默认不展示，可通过[获取接口](https://docs.jiguang.cn/jpush/client/iOS/ios_api#%E8%8E%B7%E5%8F%96%E8%87%AA%E5%AE%9A%E4%B9%89%E6%B6%88%E6%81%AF%E6%8E%A8%E9%80%81%E5%86%85%E5%AE%B9)自行编码处理。

## 4. 点击通知，打开 URL/DeepLink

如果想点击通知打开 URL 或 DeepLink，可以参考 `React Native` 官方文档的 [APIs - Linking](https://reactnative.dev/docs/linking) 部分。
