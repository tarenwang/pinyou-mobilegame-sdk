# Android SDK 接入

**修改记录**

| 修订号   | 修改描述       | 修改日期       |
| ----- | -------- |  ---------- |
| 1.0.0 | 初稿完成       | 2020-10-22 |
| 1.1.0 | 增加了用户协议和隐私政策窗口      | 2020-10-26 |
| 1.1.1 | 增加了获取提审状态的接口      | 2020-10-26 |
| 1.2.0 | 增加Application创建时调用的接口      | 2020-10-27 |
| 1.3.0 | 增加第3种登录方式   | 2020-10-28 |
| 1.4.0 | 增加游客绑定帐号接口   | 2020-10-28 |
| 1.4.1 | SDK初始化返回状态增加onDeny   | 2020-10-28 |
| 1.5.0 | 增加报错解决方案   | 2020-10-29 |
| 1.5.1 | meta-data里增加server_client_id   | 2020-11-09 |
| 1.5.2 | 支付接口里对金额小数作说明   | 2020-11-12 |
| 1.5.3 | 支付接口sign签名的算法里去掉extra   | 2020-11-16 |
| 1.6.0 | 增加获取所有内购商品列表接口   | 2020-11-24 |
| 1.7.0 | 增加退出游戏接口   | 2020-11-25 |
| 1.7.1 | 修改获取所有内购商品列表接口，使用回调方式  | 2020-11-30 |
| 1.7.2 | 修改google结算库的版本  | 2020-12-11 |
| 1.7.3 | 修改游客绑定账户方法的回调  | 2020-12-16 |
| 1.7.4 | 增加MOL支付Activity | 2020-12-29 |
| 1.7.5 | 删除build.gradle中的"armebi-v7a","x86" | 2021-01-22 |
| 1.8.0 | 增加appsflyer的三方广告归因功能  | 2021-01-29 |
| 1.8.1 | 修改NDK支持的CPU架构  | 2021-02-04 |

本文为Android客户端接入本SDK的使用教程，只涉及SDK的使用方法，默认读者已经熟悉IDE的基本使用方法（本文以AndroidStudio为例），以及具有相应的编程知识基础等。

### 准备阶段

- 需要先将游戏内的可购买商品列表发送给发行商，物品属性包含物品id、物品价格、物品在游戏中对应的点数、物品描述等
- 发行商向开发商提供`appId`、`appSecret`、`sdkUrl`等配置信息

### 一、游戏配置

#### 1.1 依赖包引入

**直接导入**

导入`pinyouloginsdk-xxx.aar`、`pinyoupaysdk-xxx.aar`、`pinyouutilsdk.aar`并引入

**Gradle导入**

编辑`build.gradle`文件，增加google的资源库

```gradle
allprojects {
    repositories {
        google()
        jcenter()
        mavenCentral()
    }
}

dependencies {
    ...
    
    implementation 'com.google.android.gms:play-services-auth:18.1.0'
    implementation 'com.facebook.android:facebook-login:[5,6)'
    implementation 'com.facebook.android:facebook-share:[5,6)'
    implementation 'com.google.code.gson:gson:2.8.6'
    implementation 'com.android.billingclient:billing:3.0.1'
    implementation 'com.squareup.okhttp3:okhttp:3.12.3'
    implementation 'com.squareup.okio:okio:1.15.0'
    implementation 'com.appsflyer:af-android-sdk:6.0.0'
    implementation 'com.android.installreferrer:installreferrer:1.1'
}
```

**NDK导入**

将libs目录下的arm64-v8a、armeabi-v7a目录复制到游戏工程的libs目录中；其中NDK包依赖，以Android Studio为例，在`build.gradle`中加入依赖。

```gradle
defaultConfig {
    multiDexEnabled true
        ndk {
            abiFilters "arm64-v8a","armeabi-v7a"
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
<uses-permission android:name="com.android.vending.CHECK_LICENSE" />
<uses-permission android:name="com.android.vending.BILLING" />
```

**参数配置**

配置默认的SDK应用ID和SDK接口服务器地址

```xml
<meta-data android:name="PINYOU_APPID" android:value="{appId}" />
<meta-data android:name="PINYOU_SDK_URL" android:value="{sdkUrl}" />
<meta-data android:name="com.facebook.sdk.ApplicationId" android:value="@string/facebook_app_id" />
<meta-data android:name="server_client_id" android:value="@string/server_client_id" />
<meta-data android:name="PY_ADM_TYPE" android:value="{PY_ADM_TYPE}" />
<meta-data android:name="PY_AF_DEV_KEY" android:value="{PY_AF_DEV_KEY}" />
```

**增加Activity界面的声明**

```xml
<activity
    android:name="com.pinyou.loginsdk.third.WebActivity"
    android:theme="@android:style/Theme.Translucent.NoTitleBar.Fullscreen" />
<activity
    android:name="com.facebook.FacebookActivity" android:configChanges= "keyboard|keyboardHidden|screenLayout|screenSize|orientation" 
    android:label="@string/app_name" />
<activity
    android:name="com.facebook.CustomTabActivity" 
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="@string/fb_login_protocol_scheme" />
    </intent-filter>
</activity>
<activity
    android:name="com.pinyou.paysdk.payment.PayWebActivity"
    android:theme="@android:style/Theme.Translucent.NoTitleBar.Fullscreen"></activity>
```

**增加Provider的声明**

```xml
<provider
    android:authorities="com.facebook.app.FacebookContentProvider{FB_APP_ID}" 
    android:name="com.facebook.FacebookContentProvider" 
    android:exported="true"/>
```

**增加Receiver的声明**

```xml
<receiver android:name="com.appsflyer.SingleInstallBroadcastReceiver" android:exported="true">
	<intent-filter>
		<action android:name="com.android.vending.INSTALL_REFERRER" />
	</intent-filter>
</receiver>
```

参数`{FB_APP_ID}`是游戏发行商或者SDK方提供的。

#### 1.3 strings.xml配置

打开您的`/app/res/values/strings.xml`文件，增加Google和Facebook相关的配置信息

```xml
<string name="facebook_app_id">{FB_APP_ID}</string>
<string name="fb_login_protocol_scheme">fb{FB_APP_ID}</string>
<string name="server_client_id">{GOOGLE_CLIENT_ID}</string>
<string name="google_license_key">{GOOGLE_LICENSE_KEY}</string>
```

参数`{FB_APP_ID}`、`{GOOGLE_CLIENT_ID}`和`{GOOGLE_LICENSE_KEY}`是游戏发行商或者SDK方提供的。

**其他注意事项**

- 项目minSdkVersion必须设置为<span style="color:red">23</span>
- 项目targetSdkVersion必须设置为<span style="color:red">29</span>
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

#### 2.1 Application初始化(必接)

在Android应用创建的时候，必须调用此接口。

引入SDK类：

```java
import com.pinyou.loginsdk.PYLoginSDK;
import com.pinyou.utilsdk.AppsFlyer.PinyouAFApplication;
```

插入如下代码：

```java
public class MyApplication extends PinyouAFApplication {
    @Override
    public void onCreate() {
        super.onCreate();
        // 应用初始化方法调用
        PYLoginSDK.getInstance().applicationInit(this);
    }
}
```

#### 2.2 SDK初始化(必接)

引入SDK类：

```java
import com.pinyou.loginsdk.PYLoginSDK;
```

插入如下代码：

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

    @Override
    public void onDeny() {
        // 用户没有授权，不能使用游戏
        // 游戏停止加载资源、停止加载进入登录界面
    }
});
```

参数是游戏Activity对象（注意：必须游戏界面未启动前调用该接口，如不能保证可放在游戏MainActivity的`onCreate`生命周期中执行）

#### 2.3 切换SDK显示语言(必接)

此方法适用于多地区多语言发行的手游。研发商可以在游戏的任何地方使用此方法，SDK会根据选择的语言进行切换。

引入SDK类：

```java
import com.pinyou.loginsdk.PYLoginSDK;
```

插入如下代码：

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

#### 2.4 登录(必接)

SDK提供给研发商三种接入登录的方式：
1. SDK提供登录弹出界面，包含帐号、密码登录和各种三方登录
2. 不使用SDK的登录界面，游戏自己实现各种三方登录的按钮
3. SDK提供登录弹出界面，只包含三方登录

研发商根据需要选择一种方式接入。

引入SDK类：

```java
import com.pinyou.loginsdk.PYLoginSDK;
```

插入如下代码：

```java
// 使用SDK登录界面
PYLoginSDK.getInstance().login(new LoginCallback() {
    @Override
    public void onSuccess(UserInfo user) {
        // 登录成功
        // user.accountId   帐号唯一标识
        // user.token       登录令牌
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
    public void onFail(int code, String msg) {
        // 登录错误
    }
});

// 不使用SDK登录界面，游戏自己实现Google、Facebook、还有游客登录的按钮；当按钮被点击时，分别调用以下接口
// Google举例
PYLoginSDK.getInstance().loginWithThirdParty("google_playgame", new LoginCallback() {
    @Override
    public void onSuccess(UserInfo user) {
        // 登录成功
        // user.accountId   帐号唯一标识
        // user.token       登录令牌
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
    public void onFail(int code, String msg) {
        // 登录错误
    }
});

// 使用SDK登录界面，只包含三方登录
PYLoginSDK.getInstance().loginWithAllThirdParties(new LoginCallback() {
    @Override
    public void onSuccess(UserInfo user) {
        // 登录成功
        // user.accountId   帐号唯一标识
        // user.token       登录令牌
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
    public void onFail(int code, String msg) {
        // 登录错误
    }
});
```

所有的帐号类型`accountType`请参考附录中的[三方帐号类型](#三方帐号类型)

#### 2.5 是否显示三方登录按钮(选接)

当研发商选择游戏自己实现Google、Facebook等三方登录按钮的情况下，必须先调用此接口，如果返回`true`，才能在游戏中显示相关按钮；如果研发商选择使用SDK的登录界面，此接口不用接入。**注意：此接口必须在初始化成功之后才能使用**

引入SDK类：

```java
import com.pinyou.loginsdk.PYLoginSDK;
```

插入如下代码：

```java
// Google举例
if (PYLoginSDK.getInstance().showThirdPartyLoginButton("google_playgame")) {
    // 游戏中显示Google登录按钮
} else {
    // 游戏中隐藏Google登录按钮
}
```

所有的帐号类型`accountType`请参考附录中的[三方帐号类型](#三方帐号类型)

#### 2.6 游客帐号绑定固定帐号(必接)

当用户使用游客(`user.loginType="guest"`)方式进行登录后，游戏中应该显示的提供绑定帐号的菜单或者按钮，引导玩家进行帐号绑定工作。

引入SDK类：

```java
import com.pinyou.loginsdk.PYLoginSDK;
```

(选接) **假如游戏里设置了游客绑定账户功能，可以使用如下接口，否则，无需使用：**

```java
PYLoginSDK.getInstance().guestBinding();
```

(必接) 同时，绑定回调单独设置（越早越好，请在调用SDK初始化后执行调用）

```java
PYLoginSDK.getInstance().setBindCallback(new BindCallback() {
    @Override
    public void onSuccess(UserInfo userInfo) {
        // 绑定成功
        // user.accountId   帐号唯一标识
        // user.token       登录令牌
        // user.userName    帐号名
        // user.nickName    昵称
        // user.avatarUrl   用户头像URL
        // user.loginType   帐号类型(facebook,google,apple,guest)
        // user.firstLogin  是否首次登录
    }

    @Override
    public void onCancel() {
        // 绑定取消
    }

    @Override
    public void onFail(int code, String msg) {
        // 绑定错误
    }
});
```

#### 2.7 获取提审状态(选接)

当游戏在各应用市场进行提审过程的时候，游戏客户端中需要展现特定的状态。比如登录方式，是否可以分享，内购商品的显示切换等。**注意：此接口必须在初始化成功之后才能使用**

引入SDK类：

```java
import com.pinyou.loginsdk.PYLoginSDK;
```

插入如下代码：

```java
if (PYLoginSDK.getInstance().getReviewStatus()) {
    // 游戏中显示提审状态
} else {
    // 游戏中显示正式状态
}
```

#### 2.8 支付(必接)

本接口中sign签名请 [参考签名规则](server-api-overview.md#签名规则)，**参与签名计算的参数包含appId、accountId、token、productId、roleId、serverId、amount、currency**。为了保障支付的安全性，签名计算请在游戏服务端中进行，保证`appSecret`不被非法破解获取。

引入SDK类：

```java
import com.pinyou.paysdk.PYPaySDK;
```

插入如下代码：

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
param.SetAmount(1.00f); // 必须带2位小数。注意，签名里不包含f
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

#### 2.9 注销登录(选接)

引入SDK类：

```java
import com.pinyou.loginsdk.PYLoginSDK;
```

插入如下代码：

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

#### 2.10 分享(选接)

引入SDK类：

```java
import com.pinyou.loginsdk.PYLoginSDK;
```

插入如下代码：

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

#### 2.11 Activity生命周期(必接)

游戏主窗体中直接重写一下父类方法。

引入SDK类：

```java
import com.pinyou.loginsdk.PYLoginSDK;
```

插入如下代码：

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

#### 2.12 打开用户协议和隐私政策窗口(选接)

当游戏客户端里需要显示的加入用户协议和隐私政策的时候，点击相应链接的时候需要调用此方法。

引入SDK类：

```java
import com.pinyou.loginsdk.PYLoginSDK;
```

插入如下代码：

```java
// 打开用户协议窗口
PYLoginSDK.getInstance().openFullScreenWindow(PYLoginSDKOpenWindowType.USER_AGREEMENT);

// 打开隐私政策窗口
PYLoginSDK.getInstance().openFullScreenWindow(PYLoginSDKOpenWindowType.PRIVACY_POLICY);
```

#### 2.13 获取游戏内购商品列表(选接)

当游戏中需要根据玩家所在国家/地区使用的货币来自定义显示游戏中的内购使用货币和价格时，需要使用该方法。

**注意：该方法必须在调用初始化方法后才能使用**

引入SDK类：

```java
import com.pinyou.paysdk.PYPaySDK;
```

插入如下代码：

```java
PYPaySDK.getInstance().getAllInAppProducts(new InAppProductCallback() {
    @Override
    public void onSuccess(List<InAppProduct> products) {
        // 其中InAppProduct类的属性说明
        // title: 商品名称
        // productId: SDK创建订单使用的商品id
        // sdkCurrency: SDK创建订单使用的货币
        // sdkPrice: SDK创建订单使用的金额，浮点型
        // useCurrency: 玩家购买使用的货币
        // usePrice = "$564.59"; 玩家购买需要花费的金额，字符串类型
    }

    @Override
    public void onFail(int code, String msg) {
        // 获取错误
    }
});
```

#### 2.14 退出游戏(选接)

当玩家安卓手机上点击回退按钮的时候，需要弹出窗口让玩家确认是否退出游戏。请在`onKeyDown`默认方法里进行调用。

**如果游戏自带退出游戏确认窗口，可以不调用本接口**

引入SDK类：

```java
import com.pinyou.loginsdk.PYLoginSDK;
```

插入如下代码：

```java
@Override
public boolean onKeyDown(int keyCode, KeyEvent event) {
    if (keyCode == KeyEvent.KEYCODE_BACK && event.getRepeatCount() == 0) {
        // 如果游戏有自己的退出游戏确认窗口，则以下应使用游戏的逻辑，不需要使用以下exitGame方法
        PYLoginSDK.getInstance().exitGame();
        return true;
    }

    return super.onKeyDown(keyCode, event);
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
|google_playgame|Google帐号|
|facebook_android|Facebook帐号|
|guest|游客帐号|
|pinyou|品游帐号|

#### 货币类型

|currency|说明|
|---|---|
|USD|美元|
|GC|游戏代币|
|CNY|人民币|
|TWD|台湾币|

### 注意

#### 报错解决方案

1. 假如游戏客户端引擎使用的是Unity，打包的时候出现SDK包中的`google-service`与Unity自带的`google-service`版本冲突下，会出现报错`Execution failed for task ':checkReleaseDuplicateClasses'...Duplicate class android.support.v4.app.INotificationSideChannel found in modules classes.jar (androidx.core:core:1.0.0) and classes.jar (com.android.support:support-compat:27.0.2)...`；请将`build.gradle`文件中的`implementation 'com.google.android.gms:play-services-auth:18.1.0'`改成`implementation 'com.google.android.gms:play-services-auth:15.0.0'`
