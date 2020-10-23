# Android SDK 接入

**修改记录**

| 修订号   | 修改描述       | 修改日期       |
| ----- | -------- |  ---------- |
| 1.0.0 | 初稿完成       | 2020-10-22 |

本文为Android客户端接入本SDK的使用教程，只涉及SDK的使用方法，默认读者已经熟悉IDE的基本使用方法（本文以AndroidStudio为例），以及具有相应的编程知识基础等。

### 准备阶段

- 需要先将游戏内的可购买商品列表发送给发行商，物品属性包含物品id、物品价格、物品在游戏中对应的点数、物品描述等
- 发行商向开发商提供`appId`、`appSecret`、`sdkUrl`等配置信息

### 一、游戏配置

#### 1.1 依赖包引入

**直接导入**

导入`pinyouloginsdk-xxx.aar`、`pinyoupaysdk-xxx.aar`、`pinyouutilsdk.aar`并引入该aar包

**Gradle导入**

编辑`build.gradle`文件，增加google的资源库

```gradle
allprojects {
    repositories {
        google()
    }
}

dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    implementation 'androidx.appcompat:appcompat:1.2.0'
    implementation 'androidx.constraintlayout:constraintlayout:2.0.2'
    
    implementation 'com.google.android.gms:play-services-auth:18.1.0'
    implementation 'com.facebook.android:facebook-login:4.42.0'
    implementation 'com.google.code.gson:gson:2.8.6'
    implementation 'com.android.billingclient:billing:2.1.0'
    implementation 'com.squareup.okhttp3:okhttp:3.12.3'
    implementation 'com.squareup.okio:okio:1.15.0'
}
```

**NDK导入**

将libs目录下的arm64-v8a、armeabi、armeabi-v7a、x86目录复制到游戏工程的libs目录中；其中NDK包依赖，以Android Studio为例，在`build.gradle`中加入依赖。

```gradle
defaultConfig {
    multiDexEnabled true
        ndk {
            abiFilters "arm64-v8a","armeabi","armebi-v7a","x86"
        }
    }

    sourceSets {
        main {
            jniLibs.srcDirs = ['libs']
        }
    }
}
```

#### 1.2 AndroidManifest.xml配置

**配置权限**

```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="com.android.vending.BILLING" />
```

**参数配置**

配置默认的SDK应用ID和SDK接口服务器地址

```xml
<meta-data android:name="PINYOU_APPID" android:value="{appId}" />
<meta-data android:name="PINYOU_SDK_URL" android:value="{sdkUrl}" />
<meta-data android:name="com.facebook.sdk.ApplicationId" android:value="{facebook_app_id}" />
<meta-data android:name="com.google.android.gms.version" android:value="@integer/google_play_services_version"/>
```

**增加Activity界面的声明**

```xml
<activity android:name="com.pinyou.loginsdk.third.WebActivity" android:theme="@android:style/Theme.Translucent.NoTitleBar.Fullscreen"/>
<activity android:configChanges="keyboard|keyboardHidden|orientation|screenLayout|screenSize" android:label="@string/app_name" android:name="com.facebook.FacebookActivity" android:theme="@style/com_facebook_activity_theme"/>
<activity android:exported="true" android:name="com.facebook.CustomTabActivity">
    <intent-filter>
        <action android:name="android.intent.action.VIEW"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <category android:name="android.intent.category.BROWSABLE"/>
        <data android:scheme="{fb_login_protocol_scheme}"/>
    </intent-filter>
</activity>
<activity 
    android:excludeFromRecents="true" 
    android:exported="false" 
    android:name="com.google.android.gms.auth.api.signin.internal.SignInHubActivity" 
    android:theme="@android:style/Theme.Translucent.NoTitleBar"/>
<activity android:name="com.facebook.CustomTabMainActivity"/>
<activity 
    android:exported="false" 
    android:name="com.google.android.gms.common.api.GoogleApiActivity" 
    android:theme="@android:style/Theme.Translucent.NoTitleBar"/>
```

**增加Service的声明**

```xml
<service 
    android:exported="true" 
    android:name="com.google.android.gms.auth.api.signin.RevocationBoundService" 
    android:permission="com.google.android.gms.auth.api.signin.permission.REVOCATION_NOTIFICATION"/>
```

**增加Provider的声明**

```xml
<provider android:authorities="com.facebook.app.FacebookContentProvider{FB_APP_ID}" android:exported="true" android:name="com.facebook.FacebookContentProvider"/>
<provider android:authorities="{pkgname}.FacebookInitProvider" android:exported="false" android:name="com.facebook.internal.FacebookInitProvider"/>
```

**增加Receiver的声明**

```xml
<receiver android:exported="false" android:name="com.facebook.CurrentAccessTokenExpirationBroadcastReceiver">
    <intent-filter>
        <action android:name="com.facebook.sdk.ACTION_CURRENT_ACCESS_TOKEN_CHANGED"/>
    </intent-filter>
</receiver>
```

#### 1.3 strings.xml配置

增加Google和Facebook相关的配置信息

```xml
<string name="facebook_app_id">{FB_APP_ID}</string>
<string name="fb_login_protocol_scheme">fb{FB_APP_ID}</string>
<string name="server_client_id">{GOOGLE_CLIENT_ID}</string>
```

**其他注意事项**

- 项目targetSdkVersion必须设置为<span style="color:red">27</span>
- Unity3D游戏工程关闭自动权限申请功能

```xml
<application>
   <meta-data android:name="unityplayer.SkipPermissionsDialog" android:value="true" />
</application>
```

- <span style="color:red">删除隐私敏感权限</span>：`android.permission.READ_PHONE_STATE`（GooglePlay禁止获取imei）
- 文件操作请在<span style="color:red">应用私有目录</span>中进行，避免在SD卡公共目录中进行文件读写操作（可以避免申请外部存储权限）

#### 1.4 资源配置

配置游戏各像素icon图标，分别在**drawable-hdpi**、**drawable-ldpi**、**drawable-mdpi**、**drawable-xhdpi**、**drawable-xxhdpi**、**drawable-xxxhdpi**下配置对应像素为**72\*72**、**36\*36**、**48\*48**、**96\*96**、**144\*144**、**192\*192**的游戏icon，注意：icon像素必须匹配，drawable中不添加游戏icon

### 二、游戏调用平台

注：以下接口暂只提供直java层，具体java层与游戏层数据交互由接入方自行实现

#### 2.1 初始化(必接)

```java
PYLoginSDK.getInstance().init(activity, new InitCallbak() {
    @Override
    public void onSuccess() {
        // 初始化成功
    }

    @Override
    public void onFail() {
        // 初始化失败
    }
});
```

参数是游戏Activity对象（注意：必须游戏界面未启动前调用该接口，如不能保证可放在游戏MainActivity的`onCreate`生命周期中执行）

#### 2.2 切换SDK显示语言(必接)

此方法适用于多地区多语言发行的手游。研发商可以在游戏的任何地方使用此方法，SDK会根据选择的语言进行切换。

```java
PYLoginSDK.getInstance().switchLanguage("en", SwitchLanguageCallback() {
    @Override
    public void onSuccess() {
        // 切换语言成功
    }

    @Override
    public void onFail() {
        // 切换语言失败
    }
});
```

所有的语言`language`请参考附录中的[显示语言](#显示语言)

#### 2.3 登录(必接)

SDK提供给研发商两种接入登录的方式：
1. SDK提供登录弹出界面，包含帐号、密码登录和各种三方登录
2. 不使用SDK的登录界面，游戏自己实现各种三方登录的按钮

研发商根据需要选择一种方式接入。

```java
// 使用SDK登录界面
PYLoginSDK.getInstance().login(new LoginCallback() {
    @Override
    public void onSuccess(UserInfo user) {
        // 登录成功
        // user.accountId   帐号唯一标识
        // user.token       登陆令牌
        // user.userName    帐号名
        // user.nickName    昵称
        // user.avatarUrl   用户头像URL
        // user.loginType   帐号类型(facebook,google,apple,guest)
        // user.firstLogin  是否首次登录
    }

    @Override
    public void onCancel() {
        // 登录取消
    }

    @Override
    void void onFail(int code, String msg) {
        // 登录错误
    }
});

// 不使用SDK登录界面，游戏自己实现Google、Facebook、还有游客登录的按钮；当按钮被点击时，分别调用以下接口
// Google举例
PYLoginSDK.getInstance().loginWithThirdParty("google", new LoginCallback() {
    @Override
    public void onSuccess(UserInfo user) {
        // 登录成功
        // user.accountId   帐号唯一标识
        // user.token       登陆令牌
        // user.userName    帐号名
        // user.nickName    昵称
        // user.avatarUrl   用户头像URL
        // user.loginType   帐号类型(facebook,google,apple,guest)
        // user.firstLogin  是否首次登录
    }

    @Override
    public void onCancel() {
        // 登录取消
    }

    @Override
    public void void onFail(int code, String msg) {
        // 登录错误
    }
});
```

所有的帐号类型`accountType`请参考附录中的[三方帐号类型](#三方帐号类型)

#### 2.4 是否显示三方登录按钮(选接)

当研发商选择游戏自己实现Google、Facebook等三方登录按钮的情况下，必须先调用此接口，如果返回`true`，才能在游戏中显示相关按钮；如果研发商选择使用SDK的登录界面，此接口不用接入。

```java
// Google举例
if (PYLoginSDK.getInstance().showThirdPartyLoginButton("google")) {
    // 游戏中显示Google登录按钮
} else {
    // 游戏中隐藏Google登录按钮
}
```

所有的帐号类型`accountType`请参考附录中的[三方帐号类型](#三方帐号类型)

#### 2.5 支付(必接)

本接口中sign签名请 [参考签名规则](server-api-overview.md#签名规则)，**参与签名计算的参数包含appId、accountId、token、productId、roleId、serverId、amount、currency、extra**。为了保障支付的安全性，签名计算请在游戏服务端中进行，保证`appSecret`不被非法破解获取。

```java
PayParam param = new PayParam();
param.SetAppId("SDK平台分配的游戏ID");
param.SetAccountId("SDK平台账号ID");
param.SetToken("SDK平台账号Token");
param.SetProductId("商品ID");
param.SetProductName("商品名称");
param.SetRoleId("玩家的游戏角色ID");
param.SetRoleName("玩家的游戏角色名");
param.SetServerId("游戏服务器ID");
param.SetServerName("游戏服务器名");
param.SetAmount(1.00f);
param.SetCurrency("USD"); // 所有的货币类型`currency`请参考附录中的货币类型
param.SetExtra("支付成功时原样返回至游戏服务器的额外参数；可以是游戏生成的订单号");
param.SetNotifyUrl("内购商品支付完成的通知URL"); // 如果以SDK后台设置的通知URL为主，则此参数可以不传
param.SetSign("签名");
// 进行支付
PYPaySDK.getInstance().pay(param, new PayCallback() {
    @Override
    public void onSuccess() {
        // 支付成功
        // 注意：SDK返回成功后，不能马上发送物品给玩家，要等到游戏服务端收到“内购商品支付完成通知”后才能发送
    }

    @Override
    public void onFail(int code,String msg) {
        // 支付失败
    }
});
```

#### 2.6 注销登录(选接)

```java
PYLoginSDK.getInstance().logout(new LogoutCallback() {
    @Override
    public void onSuccess() {
        // 注销成功
    }

    @Override
    public void onFail(int code, String msg) {
        // 注销失败
    }
});
```

#### 2.7 分享(选接)

```java
Map<String, Object> shareParams = new HashMap<?>();
shareParams.put("roleName", "test1");
shareParams.put("roleLevel", 10);
// 进行分享
PYLoginSDK.getInstance().share("SDK分配的分享ID", shareParams, new ShareCallback() {
    @Override
    public void onSuccess() {
        // 分享成功
    }

    @Override
    public void onCancel() {
        // 取消分享
    }

    @Override
    public void onError(int code, String msg) {
        // 分享失败
    }
});
```

#### 2.9 Activity生命周期(必接)

游戏主窗体中直接重写一下父类方法：

```java
public void onActivityResult(int requestCode, int resultCode, Intent data) {
    PYLoginSDK.getInstance().onActivityResult(requestCode, resultCode, data);
    super.onActivityResult(requestCode, resultCode, data);
}

public void onDestroy() {
    PYLoginSDK.getInstance().onDestroy();
    super.onDestroy();
}
```

### 三、打包注意事项

在build.gradle文件中加入：

```groovy
packagingOptions {
    exclude 'META-INF/LICENSE'
    exclude 'META-INF/NOTICE'
    exclude 'META-INF/DEPENDENCIES'
    exclude 'META-INF/MANIFEST.MF'
}

android {
    lintOptions {
        abortOnError false
    }
}
```

### 附录

#### 显示语言

|language|说明|
|---|---|
|en|英文|
|chs|简体中文|
|cht|繁体中文|
|th|泰语|

#### 三方帐号类型

|accountType|说明|
|---|---|
|google|Google帐号|
|facebook|Facebook帐号|
|guest|游客帐号|

#### 货币类型

|currency|说明|
|---|---|
|USD|美元|
|GC|游戏代币|
|CNY|人民币|
