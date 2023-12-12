# JPush-React-Native

极光推送 `JPush` SDK 的 `React Native` 封装，支持 `Android` 和 `iOS`，Fork 自 [jpush-react-native](https://github.com/jpush/jpush-react-native)。

## ChangeLog

1. 从 `jpush-react-native 2.7.5` 开始，重新支持 TypeScript；
2. 由于 `jcore-react-native 1.6.0` 存在编译问题，从 `jcore-react-native 1.7.0` 开始，还是需要在 `AndroidManifest.xml` 中添加配置代码，具体参考 配置-2.1 Android


## 1. 安装

**NPM:**

```bash
npm i jpush-rn
```

**YARN:**

```bash
yarn add jpush-rn
```

* 注意：如果项目里没有 `jcore-rn`，需要安装：

**NPM:**

  ```bash
  npm i jcore-rn
  ```
**YARN:**

  ```bash
  yarn add jcore-rn
  ```

## 2. 配置

### 2.1 Android

在 `AndroidManifest.xml` 中添加配置代码：

```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
```

#### 2.1.1 手动配置 (React Native 0.60 以下版本)

安装完成后连接原生库，进入到根目录执行：

```bash
react-native link
```

或

```bash
react-native link jcore-rn
react-native link jpush-rn
```

* build.gradle

```groovy
android {
      defaultConfig {
          applicationId "yourApplicationId"           //在此替换你的应用包名
          ...
          manifestPlaceholders = [
                  JPUSH_APPKEY: "yourAppKey",         //在此替换你的APPKey
                  JPUSH_CHANNEL: "yourChannel"        //在此替换你的channel
          ]
      }
  }
```

```groovy
dependencies {
      ...
      implementation project(':jpush-rn')  // 添加 jpush 依赖
      implementation project(':jcore-rn')  // 添加 jcore 依赖
  }
```

* setting.gradle

  ```groovy
  include ':jpush-rn'
  project(':jpush-rn').projectDir = new File(rootProject.projectDir, '../node_modules/jpush-rn/android')
  include ':jcore-rn'
  project(':jcore-rn').projectDir = new File(rootProject.projectDir, '../node_modules/jcore-rn/android')
  ```

* AndroidManifest.xml

  ```groovy
  <meta-data
  	android:name="JPUSH_CHANNEL"
  	android:value="${JPUSH_CHANNEL}" />
  <meta-data
  	android:name="JPUSH_APPKEY"
  	android:value="${JPUSH_APPKEY}" />    
  ```

### 2.2 iOS

注意：您需要打开 `ios` 目录下的 `.xcworkspace` 文件修改您的包名

### 2.2.1 pod

```bash
pod install
```

* 注意：如果项目里使用 `pod` 安装过，请先执行命令：

```bash
pod deintegrate
```

### 2.2.2 手动方式 (React Native 0.60 以下版本)

* Libraries

```bash
Add Files to "your project name"
node_modules/jcore-rn/ios/RCTJCoreModule.xcodeproj
node_modules/jpush-rn/ios/RCTJPushModule.xcodeproj
```

* Capabilities

```bash
Push Notification --- ON
```

* Build Settings

```bash
All --- Search Paths --- Header Search Paths --- +
$(SRCROOT)/../node_modules/jcore-rn/ios/RCTJCoreModule/
$(SRCROOT)/../node_modules/jpush-rn/ios/RCTJPushModule/
```

* Build Phases

```bash
libz.tbd
libresolv.tbd
UserNotifications.framework
libRCTJCoreModule.a
libRCTJPushModule.a
```

## 3. 引用

### 3.1 Android

参考：[MainApplication.java](https://github.com/jpush/jpush-rn/tree/master/example/android/app/src/main/java/com/example/MainApplication.java)

### 3.2 iOS

参考：[AppDelegate.m](https://github.com/jpush/jpush-rn/tree/master/example/ios/example/AppDelegate.m) 

### 3.3 js

参考：[App.js](https://github.com/jpush/jpush-rn/blob/dev/example/App.js) 

## 4. API

详见：[index.js](https://github.com/jpush/jpush-rn/blob/master/index.js)

## 5.  其他

* 集成前务必将example工程跑通
* 如有紧急需求请前往[极光社区](https://community.jiguang.cn/c/question)
* 上报问题还麻烦先调用JPush.setLoggerEnable(true}，拿到debug日志

 

