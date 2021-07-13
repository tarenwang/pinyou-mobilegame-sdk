# H5 SDK 接入

**修改记录**

| 修订号   | 修改描述       | 修改日期       |
| ----- | -------- |  ---------- |
| 1.0.0 | 初稿完成       | 2021-07-12 |

本文为H5手游客户端接入本SDK的使用教程，只涉及SDK的使用方法，默认读者已经熟悉IDE的基本使用方法（建议使用VSCode），以及具有相应的编程知识基础等。

### 准备阶段

- 需要先将游戏内的可购买商品列表发送给发行商，物品属性包含物品id、物品价格、物品在游戏中对应的点数、物品描述等
- 发行商向开发商提供`appId`、`appSecret`、`sdkDomain`等配置信息

### 一、游戏html页面的head配置

#### 1.1 常规网页SEO配置

```html
<title>游戏名称</title>
<meta name="keywords" content="游戏关键词">
<meta name="description" content="游戏描述">
```

网页中常用SEO相关的参数配置，参数内容可以由运营和推广人员提供。

#### 1.2 favicon图标

请将游戏的图标`favicon.ico`（尺寸128x128像素）的文件放入H5手游主页面所在的域名根目录下

#### 1.3 iOS设备将网页加入桌面的配置

```html
<!-- ios主界面 -->
<meta name="application-name" content="游戏名称">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
<meta name="apple-mobile-web-app-title" content="游戏名称">
<link rel="apple-touch-icon" href="尺寸180像素的ICON(PNG格式)地址">
<link rel="apple-touch-icon-precomposed" sizes="57x57" href="尺寸57像素的ICON(PNG格式)地址">
<link rel="apple-touch-icon-precomposed" sizes="72x72" href="尺寸72像素的ICON(PNG格式)地址">
<link rel="apple-touch-icon-precomposed" sizes="114x114" href="尺寸114像素的ICON(PNG格式)地址">
<link rel="apple-touch-icon-precomposed" sizes="144x144" href="尺寸144像素的ICON(PNG格式)地址">
<link rel="apple-touch-startup-image" href="尺寸2048x2732像素的加载图像(PNG或JPG格式)地址" media="(device-width: 1024px) and (device-height: 1366px) and (-webkit-device-pixel-ratio: 2) and (orientation: portrait)">
<link rel="apple-touch-startup-image" href="尺寸1668x2388像素的加载图像(PNG或JPG格式)地址" media="(device-width: 834px) and (device-height: 1194px) and (-webkit-device-pixel-ratio: 2) and (orientation: portrait)">
<link rel="apple-touch-startup-image" href="尺寸1536x2048像素的加载图像(PNG或JPG格式)地址" media="(device-width: 768px) and (device-height: 1024px) and (-webkit-device-pixel-ratio: 2) and (orientation: portrait)">
<link rel="apple-touch-startup-image" href="尺寸1668x2224像素的加载图像(PNG或JPG格式)地址" media="(device-width: 834px) and (device-height: 1112px) and (-webkit-device-pixel-ratio: 2) and (orientation: portrait)">
<link rel="apple-touch-startup-image" href="尺寸1620x2160像素的加载图像(PNG或JPG格式)地址" media="(device-width: 810px) and (device-height: 1080px) and (-webkit-device-pixel-ratio: 2) and (orientation: portrait)">
<link rel="apple-touch-startup-image" href="尺寸1284x2778像素的加载图像(PNG或JPG格式)地址" media="(device-width: 428px) and (device-height: 926px) and (-webkit-device-pixel-ratio: 3) and (orientation: portrait)">
<link rel="apple-touch-startup-image" href="尺寸1170x2532像素的加载图像(PNG或JPG格式)地址" media="(device-width: 390px) and (device-height: 844px) and (-webkit-device-pixel-ratio: 3) and (orientation: portrait)">
<link rel="apple-touch-startup-image" href="尺寸1125x2436像素的加载图像(PNG或JPG格式)地址" media="(device-width: 375px) and (device-height: 812px) and (-webkit-device-pixel-ratio: 3) and (orientation: portrait)">
<link rel="apple-touch-startup-image" href="尺寸1242x2688像素的加载图像(PNG或JPG格式)地址" media="(device-width: 414px) and (device-height: 896px) and (-webkit-device-pixel-ratio: 3) and (orientation: portrait)">
<link rel="apple-touch-startup-image" href="尺寸828x1792像素的加载图像(PNG或JPG格式)地址" media="(device-width: 414px) and (device-height: 896px) and (-webkit-device-pixel-ratio: 2) and (orientation: portrait)">
<link rel="apple-touch-startup-image" href="尺寸1242-2208像素的加载图像(PNG或JPG格式)地址" media="(device-width: 414px) and (device-height: 736px) and (-webkit-device-pixel-ratio: 3) and (orientation: portrait)">
<link rel="apple-touch-startup-image" href="尺寸750x1334像素的加载图像(PNG或JPG格式)地址" media="(device-width: 375px) and (device-height: 667px) and (-webkit-device-pixel-ratio: 2) and (orientation: portrait)">
<link rel="apple-touch-startup-image" href="尺寸640x1136像素的加载图像(PNG或JPG格式)地址" media="(device-width: 320px) and (device-height: 568px) and (-webkit-device-pixel-ratio: 2) and (orientation: portrait)">
<!-- other -->
<meta name="format-detection" content="telephone=no">
<meta name="theme-color" content="#009BAA">
<meta name="msapplication-TileColor" content="#009BAA">
```

请根据以上配置中的尺寸，来自定义H5手游在iOS系统中加入到主桌面后，在桌面上展现的icon图标；以及从主桌面打开H5手游，默认加载的展示图。

#### 1.4 PWA相关配置

需要在head中导入PWA使用的manifest配置。导入方式如下：

```html
<!-- pwa -->
<link rel="manifest" href="./manifest.json?v=v1">
```

其中`manifest.json`的地址和名称可以自定义，`manifest.json`文件中的内容如下：

```json
{
    "name":"游戏名称",
    "short_name":"游戏名称",
    "theme_color":"#009BAA",
    "icons":[
        {
            "src":"尺寸152像素的ICON(PNG格式)地址",
            "sizes":"152x152",
            "type":"image/png"
        },
        {
            "src":"尺寸180像素的ICON(PNG格式)地址",
            "sizes":"180x180",
            "type":"image/png"
        },
        {
            "src":"尺寸192像素的ICON(PNG格式)地址",
            "sizes":"192x192",
            "type":"image/png"
        },
        {
            "src":"尺寸512像素的ICON(PNG格式)地址",
            "sizes":"512x512",
            "type":"image/png"
        },
        {
            "src":"尺寸192像素的ICON(PNG格式)地址",
            "sizes":"192x192",
            "type":"image/png",
            "purpose":"maskable"
        },
        {
            "src":"尺寸512像素的ICON(PNG格式)地址",
            "sizes":"512x512",
            "type":"image/png",
            "purpose":"maskable"
        }
    ],
    "start_url":"游戏主页面URL",
    "display":"standalone",
    "background_color":"#009BAA",
    "orientation":"portrait"
}
```

请将其中的`游戏名称`设置成自己的游戏名称，替换里面各个尺寸图标，并配置H5游戏的主页面地址`start_url`，设置游戏的方向`orientation`(竖版请使用**portrait**，横板请使用**landscape**)。

#### 1.5 社交分享配置

```html
<!-- twitter分享 -->
<meta name="twitter:url" content="游戏主页面URL"/>
<meta name="twitter:title" content="游戏名称"/>
<meta name="twitter:description" content="游戏描述"/>
<meta name="twitter:site" content="游戏官网URL">
<meta name="twitter:card" content="summary_large_image"/>
<meta name="twitter:image" content="游戏分享截图地址"/>
<!-- facebook分享 -->
<meta property="og:url" content="游戏主页面URL"/>
<meta property="og:title" content="游戏名称"/>
<meta property="og:description" content="游戏描述"/>
<meta property="og:image" content="游戏分享截图地址"/>
<meta property="og:type" content="website"/>
```

### 二、SDK的引入与使用

#### 2.1 依赖js库引入

在H5手游主html页面中导入品游SDK。注意，引入依赖库后才能进行以下接口的调用。

**直接导入**

```html
<script src="pysdk.min.js"></script>
```

**动态创建script块导入(推荐使用)**

```html
<script>
(function() {
    var js = document.createElement("script");
    js.src = "pysdk.min.js?t=" + (new Date().getTime());
    var s = document.getElementsByTagName("script")[0]; 
    s.parentNode.insertBefore(js, s);
})();
</script>
```

#### 2.2 监听SDK所有事件回调(必接)

```html
<script>
// 监听所有SDK回调事件
window.addEventListener(PY_CONST.EVENT_NAME, function(e) {
    switch (e.action) {
        case PY_EVENT_ACTION.ACTION_INIT_SUCCESS: // 初始化成功
            console.log("PySDK.init初始化成功, code=" + e.code + ", msg=" + e.msg);
            break;
        case PY_EVENT_ACTION.ACTION_INIT_FAIL: // 初始化失败
            console.log("PySDK.init初始化失败, code=" + e.code + ", msg=" + e.msg);
            break;
        case PY_EVENT_ACTION.ACTION_LOGIN_SUCCESS: // 登录成功
            var user = e.data;
            var user_info = 'accountId=' + user.accountId + "<br />"
                + 'token=' + user.token + "<br />"
                + 'userName=' + user.userName + "<br />"
                + 'nickname=' + user.nickname + "<br />"
                + 'avatarUrl=' + user.avatarUrl + "<br />"
                + 'firstLogin=' + user.firstLogin + "<br />"
                + 'type=' + user.loginType;
            loginType = user.loginType;
            console.log("PySDK.login登录成功, code=" + e.code + ", msg=" + e.msg);
            console.log("帐号信息如下：<br />" + user_info);
            break;
        case PY_EVENT_ACTION.ACTION_LOGIN_FAIL: // 登录失败
            console.log("PySDK.login登录失败, code=" + e.code + ", msg=" + e.msg);
            break;
        case PY_EVENT_ACTION.ACTION_LOGIN_CANCEL: // 登录取消
            console.log("PySDK.login登录取消, code=" + e.code + ", msg=" + e.msg);
            break;
        case PY_EVENT_ACTION.ACTION_LOGOUT_SUCCESS: // 注销成功
            console.log("PySDK.logout注销成功, code=" + e.code + ", msg=" + e.msg);
            break;
        case PY_EVENT_ACTION.ACTION_LOGOUT_FAIL: // 注销失败
            console.log("PySDK.logout注销失败, code=" + e.code + ", msg=" + e.msg);
            break;
        case PY_EVENT_ACTION.ACTION_PAY_SUCCESS: // 支付成功
            // 因为Mycard支付技术原因，此回调无法执行，游戏发货请参考服务端的游戏发货接口
            console.log("PySDK.pay支付成功, code=" + e.code + ", msg=" + e.msg);
            console.log("订单号, orderId=" + e.data);
            break;
        case PY_EVENT_ACTION.ACTION_PAY_FAIL: // 支付失败
            // 因为Mycard支付技术原因，此回调无法执行
            showInfo("PySDK.pay支付失败, code=" + e.code + ", msg=" + e.msg);
            break;
        case PY_EVENT_ACTION.ACTION_PAY_CANCEL: // 支付取消
            // 因为Mycard支付技术原因，此回调无法执行
            console.log("PySDK.pay支付取消, code=" + e.code + ", msg=" + e.msg);
            break;
        case PY_EVENT_ACTION.ACTION_BIND_SUCCESS: // 绑定账号成功
            var user = e.data;
            var user_info = 'accountId=' + user.accountId + "<br />"
                + 'token=' + user.token + "<br />"
                + 'userName=' + user.userName + "<br />"
                + 'nickname=' + user.nickname + "<br />"
                + 'avatarUrl=' + user.avatarUrl + "<br />"
                + 'firstLogin=' + user.firstLogin + "<br />"
                + 'type=' + user.loginType;
            console.log("PySDK.bind游客绑定成功, code=" + e.code + ", msg=" + e.msg);
            console.log("帐号信息如下：<br />" + user_info);
            break;
        case PY_EVENT_ACTION.ACTION_BIND_CANCEL: // 绑定账号取消
            console.log("PySDK.bind游客绑定取消, code=" + e.code + ", msg=" + e.msg);
            break;
        case PY_EVENT_ACTION.ACTION_BIND_FAIL: // 绑定账号错误
            console.log("PySDK.bind游客绑定错误, code=" + e.code + ", msg=" + e.msg);
            break;
        default:
            console.log("未知回调事件, action=" + e.action);
            break;
    }
});
</script>
```

#### 2.3 初始化(必接)

```html
<script>
var config = {
    'sdkDomain' : '{SDK_DOMAIN}',
    'appId' : '{APP_ID}',
    'cpsId' : '',
    'versionCode' : '1.0.0',
    'versionName' : '12',
    'debug' : true,
    'apkDownloadUrl' : '',
};

// 初始化
PySDK.init(config);
</script>
```

**config参数说明：**

|参数名|是否必填|说明|
|---|---|---|
|sdkDomain|是|SDK服务端域名|
|appId|是|SDK分配的游戏唯一ID|
|cpsId|是|用来区分投放来源，可以为空|
|versionCode|是|版本号|
|versionName|是|版本build号|
|debug|是|调试模式，是否显示![](assets/vconsole.png)|
|apkDownloadUrl|否|当玩家使用android手机浏览器玩游戏时，界面弹出提示推荐玩家下载apk|

#### 2.4 登录(必接)

调用此接口会弹出SDK登录界面。注意，此方法必须在SDK初始化成功`PY_EVENT_ACTION.ACTION_INIT_SUCCESS`之后才能调用。

```html
<script>
// 登录
PySDK.login();
</script>
```

#### 2.5 游客帐号绑定固定帐号(选接)

当用户登录成功后`PY_EVENT_ACTION.ACTION_LOGIN_SUCCESS`，返回的用户信息是游客帐号时(`user.loginType="guest"`)，游戏中可以提供绑定帐号的菜单或者按钮，引导玩家进行帐号绑定工作。

```html
<script>
// 游客绑定
PySDK.bind();
</script>
```

#### 2.6 获取提审状态(选接)

当游戏在各应用市场进行提审过程的时候，游戏客户端中需要展现特定的状态。比如登录方式，是否可以分享，内购商品的显示切换等。**注意：此接口必须在初始化成功之后才能使用**

```html
<script>
// 游客绑定
if (PySDK.getReviewStatus()) {
    // 游戏中显示提审状态
} else {
    // 游戏中显示正式状态
}
</script>
```

#### 2.7 支付(必接)

本接口中sign签名请 [参考签名规则](server-api-overview.md#签名规则)，**参与签名计算的参数包含appId、accountId、token、productId、roleId、serverId、amount、currency**。为了保障支付的安全性，签名计算请在游戏服务端中进行，保证`appSecret`不被非法破解获取。

```html
<script>
var paymentInfo = {
    "appId" : "SDK平台分配的游戏ID",
    "accountId" : "SDK平台账号ID",
    "token" : "SDK平台账号Token",
    "productId" : "商品ID",
    "productName" : "商品名称",
    "roleId" : "玩家的游戏角色ID",
    "roleName" : "玩家的游戏角色名",
    "serverId" : "游戏服务器ID",
    "serverName" : "游戏服务器名",
    "amount" : 33.00, // 金额，必须带2位小数
    "currency" : "TWD", // 所有的货币类型`currency`请参考附录中的货币类型
    "extra" : "支付成功时原样返回至游戏服务器的额外参数；可以是游戏生成的订单号",
    "notifyUrl" : "内购商品支付完成的通知URL", // 如果以SDK后台设置的通知URL为主，则此参数可以不传
    "sign" : "签名",
};
PySDK.pay(paymentInfo);
</script>
```

#### 2.8 注销登录(选接)

```html
<script>
// 注销登录
PySDK.logout();
</script>
```

#### 2.9 打开用户协议和隐私政策窗口(选接)

当游戏客户端里需要显示的加入用户协议和隐私政策的时候，点击相应链接的时候需要调用此方法。**注意：此接口必须在初始化成功之后才能使用**

```html
<script>
// 打开用户协议窗口
PySDK.openUserAgreement();

// 打开隐私政策窗口
PySDK.openPrivacyPolicy();
</script>
```

#### 2.10 获取游戏内购商品列表(选接)

当游戏中需要根据玩家所在国家/地区使用的货币来自定义显示游戏中的内购使用货币和价格时，需要使用该方法。**注意：此接口必须在初始化成功之后才能使用**

```html
<script>
var products = PySDK.getAllInAppProducts();
for (var p in products) {
    console.log(p.title); //商品名称
    console.log(p.productId); //SDK创建订单使用的商品id
    console.log(p.sdkCurrency); //SDK创建订单使用的货币
    console.log(p.sdkPrice); //SDK创建订单使用的金额，浮点型
    console.log(p.useCurrency); //玩家购买使用的货币
    console.log(p.usePrice); //玩家购买需要花费的金额，字符串类型
}
</script>
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
