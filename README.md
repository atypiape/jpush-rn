# JPush-RN

极光推送 `JPush` SDK 的 `React Native` 封装，支持 `Android` 和 `iOS`，Fork 自 [jpush-react-native](https://github.com/jpush/jpush-react-native)。

----

为啥自己维护一份呢？因为官方 `jpush-react-native` 项目做得比较早，各方面更新比较慢。最初我跟着 `jpush-react-native` 文档配置，很多东西没生效，最后去看官方 Android [SDK 集成](https://docs.jiguang.cn/jpush/client/Android/android_guide)和 iOS [SDK 集成](https://docs.jiguang.cn/jpush/client/iOS/ios_guide_new)文档才恍然大悟。

如果有一点 Android 和 iOS 开发经验，建议也去看下官方的[客户端 SDK](https://docs.jiguang.cn/jpush/client/)文档，很多问题在里面都可以找到答案。

有疑问或者本项目存在问题，请在 [Issues](https://github.com/atypiape/jpush-rn/issues) 中反馈，非常感谢。

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

`Android` 使用 `mavenCentral` 自动集成 [JPush](https://mvnrepository.com/artifact/cn.jiguang.sdk/jpush) SDK，当前版本为 `5.2.2`。

### 2.2. iOS

`iOS` 使用 `Cocoapods` 自动导入 [JPush](https://cocoapods.org/pods/JPush) SDK，当前版本为 `5.2.0`。

## 3. 配置 (React Native 0.60 及以上版本)

*备注：如果你的项目是 `0.59` 及以下版本，请参考  [jpush-react-native](https://www.npmjs.com/package/jpush-react-native) 中的手动配置方式。*

### 3.1. Android

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
        <action android:name="cn.jpush.android.intent.SERVICE_MESSAGE" />
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
-keep class * extends cn.jpush.android.service.JPushMessageService { *; }

-dontwarn cn.jiguang.**
-keep class cn.jiguang.** { *; }
```

**重新编译**

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

在 `ios` 目录下执行以下命令：

```bash
cd ios
pod install
```
如果报错，可尝试执行命令：

```bash
pod deintegrate
```
