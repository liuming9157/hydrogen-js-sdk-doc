# 知晓云 JS SDK

## 简介

知晓云 JS SDK 是方便开发者结合知晓云产品，进行小程序开发的工具包，通过该 SDK，你可以在小程序中操作存储在知晓云中的数据表、内容库、媒体文件，及方便地调用微信登录，微信支付，发送模板消息等功能。

## 引入 SDK 并初始化

{% tabs sdkfile="js 文件版", sdkplugin="小程序插件版" %}

{% content "sdkfile" %}

#### 下载并导入 SDK

a. [下载最新版 SDK 到本地](./download-sdk.md)

b. 将下载解压后得到的 SDK js 文件放在小程序项目目录中

c. 在 `app.js` 中引入 SDK js 文件，注意，请使用下载的 SDK 对应的版本替换下面代码片段里的 `sdk-v<version>`

```js
// app.js
App({
  onLaunch() {
    // require SDK
    require('./sdk-v<version>')
  }
})
```

#### 初始化 SDK

通过初始化 SDK ，知晓云服务可以验证当前的小程序是否是有效合法的，只有通过验证的小程序才能使用 SDK 提供的全部功能。

在[知晓云后台 - 我的应用](https://cloud.minapp.com/admin/profile/)页面获取要接入知晓云服务的小程序 ClientID, 按照如下方式进行 SDK 初始化:

```js
// app.js
App({
  onLaunch() {
    require('./sdk-v<version>')

    // 初始化 SDK
    let clientID = '知晓云管理后台获取到的 ClientID'
    wx.BaaS.init(clientID)
  }
})
```

#### 完成服务器域名配置

在[这里](/newbies/README.md#小程序第三方授权以及服务器域名配置)可查看详细的配置方法。

{% content "sdkplugin" %}

使用知晓云 SDK 小程序插件，相对于引入 SDK js 文件方式，省去了下载 SDK 文件和配置服务器域名操作，同时可以更加方便的切换不同版本的 SDK。

#### 添加知晓云 SDK 小程序插件

在小程序的管理后台【设置】-【第三方服务】-【插件管理】中添加插件，根据知晓云 SDK 小程序插件的 AppID 「**wxc6b86e382a1e3294**」搜索到我们的插件，添加后等待审核，审核通过即可使用。

#### 在项目配置中声明使用知晓云 SDK 小程序插件

在项目的配置文件 `app.json` 中对插件进行引入声明，目前知晓云 SDK 插件最新版本为 `0.1.0`。

```json
"plugins": {
  "sdkPlugin": {
    "version": "0.1.0",
    "provider": "wxc6b86e382a1e3294"
  }
}
```

#### 初始化 SDK

通过初始化 SDK ，知晓云服务可以验证当前的小程序是否是有效合法的，只有通过验证的小程序才能使用 SDK 提供的全部功能。

在[知晓云后台 - 我的应用](https://cloud.minapp.com/admin/profile/)页面获取要接入知晓云服务的小程序 ClientID, 按照如下方式进行 SDK 初始化:

```js
//app.js
App({
  onLaunch: function() {
    wx.BaaS = requirePlugin('sdkPlugin')
    //让插件帮助完成登录、支付等功能
    wx.BaaS.wxExtend(wx.login, wx.getUserInfo, wx.requestPayment)

    let clientID = '知晓云管理后台获取到的 ClientID'
    wx.BaaS.init(clientID)
  }
})
```

{% endtabs %}


## 使用 SDK

成功初始化 SDK 后，即可使用 SDK 完成数据操作，内容操作等功能了。如下，在控制台创建一张表（参考[控制台操作-数据表](../dashboard/schema.md) 一节），获取其 tableID ，并插入一条数据。

```js
let tableID = 10
let Product = new wx.BaaS.TableObject(tableID)
let product = Product.create()

let apple = {
  name: 'apple',
  price: 1,
  desc: ['good'],
  amount: 0
}

product.set(apple).save().then(res => {
  console.log('成功插入数据：', res)
}, err => {
  // err
})
```


更多内容，可查看以下功能模块介绍：

* [登入登出](./signin-signout.md)
* [用户](./user.md)
* [数据表](./schema/README.md)
* [内容库](./content/README.md)
* [文件](./file/README.md)
* [支付](./payment/README.md)
* [模板消息](./template-message.md)
* [微信加密数据解密](./wechat-decrypt.md)
