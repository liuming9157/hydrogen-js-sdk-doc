# 触发器

## 概览
### 触发器配置卡片

![触发器配置](/images/dashboard/trigger-config.jpg)

此面板配置一些触发器的基本信息，或者进行查阅触发器触发日志等操作。

触发类型目前有两种：

- [数据表](#数据表)
- [微信支付回调](#微信支付回调)

### 条件卡片

![触发器条件](/images/dashboard/trigger-condition.jpg)

对应上面的触发类型，设置不同触发类型的参数。

### 动作卡片

![触发器动作](/images/dashboard/trigger-action.jpg)

当触发器的条件被满足，将会执行触发器中的动作。

目前有四种动作类型可选，分别如下：

- 邮件
- 微信模板消息
- webhook
- 数据表操作

不同的触发类型对应可选的动作类型不同，每个动作类型又有可选模板变量的区别，总结如下：

| 动作类型    | 模板变量 |触发类型：数据表  | 触发类型：微信支付回调 |
| ---------- | :----- | :------------ | :------------------ |
| 邮件        | ✔️     | ✔️             | ✔️ |
| 微信模板消息 | ✔️     | ✔️             | ✔️ |
| webhook    | ✘      | ✔️             | ✔️ |
| 数据表操作   | ✔️     | ✔️             | ✔️ |


## 触发类型

### 数据表
触发条件：当指定的数据表发生改变（增、删、改），满足触发器设置的条件时。

使用场景：新用户注册成功，向新用户发送一封欢迎邮件。

下面是一些参数说明：

| 事件类型 | 说明 |
| :-------|:--- |
| create  | 数据行被创建时触发 |
| update  | 数据行被更新时触发 |
| delete  | 数据行被删除时触发 |

| 满足条件 | 说明 |
| :----- |:---- |
| 任一    | OR  满足任一条件 |
| 所有    | AND 同时满足所有条件 |

目前触发条件支持的数据类型及其对应的操作符如下：

| 数据类型 | 操作符                            |
| :------ | :------------------------------- |
| array   | contains, isempty |
| boolean | =, !=, isempty |
| date    | =, !=, >, >=, <, <=, isempty, range |
| integer | =, !=, >, >=, <, <=, isempty, range |
| number  | =, !=, >, >=, <, <=, isempty, range |
| string  | regex, =, !=, isempty |


### 微信支付回调
触发条件：当用户微信支付成功时。

使用场景：用户微信支付成功，微信模板消息提醒用户订单详情。

## 动作

### 邮件
执行结果：向指定邮件地址发送一封邮件。

>**info**
>注：收件人和邮件标题也可以输入模板变量

### 微信模板消息
执行结果：向指定用户发送一条微信模板消息。

使用前要在[微信小程序后台][1]添加消息模板。当在微信小程序后台增加、删除模板后，请在微信模板消息动作编辑卡片点击**更新模板**按钮，用以更新知晓云保存的模板缓存。

微信模板消息需要配合小程序来触发，无法单独在后台触发。具体触发方法为：在小程序页面中添加 form 组件，在提交表单的回调中取得 formId，调用 [BaaS.wxReportTicket](../js-sdk/template-message.md) 保存 formId，保存成功后，当触发器被触发后，这时用户就可以在手机收到通知。
注意这里 form 组件需要添加 **report-submit** 属性，否则在回调事件对象中无法获取 formId。

### WebHook
执行结果：向指定 URL 发送一个 POST 请求。

执行动作时，服务器发送请求参数如下：

#### 请求 Headers
| 请求头部                      | 说明 |
| ---------------------------- | --- |
| X-Hydrogen-Webhook-Action-Id | 本次触发器触发动作的 uuid，webhook 重试时此 id 保持不变 |
| X-Hydrogen-Trigger-Event     | on_create, on_update, on_delete |

#### 请求 Body
**请求 Body 为一个 JSON Web Tokens 的文本，需要开发者自己去验证并解码，可以在这里[在线调试 jwt](https://jwt.io/)**

示例：
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJuYW1lIjoiMTIzIn0.FGhYH5IF-PkNV8b4SNh-1WKwV8h-Gj8JYVlXmUdCGs8
```

若触发类型为数据表，则请求 Body 解码后为数据行内容，示例如下：
```json
{
  _read_perm: [ 'user:*' ],
  name: '1',
  created_at: 1517970094,
  updated_at: 1517970094,
  created_by: 35674431,
  _write_perm: [ 'user:35674431' ],
  id: '5a7a62aefff1d610ab2ddb10'
}
```
数据行内容根据 event 类型有所差别，具体差异如下

| event 类型 | 请求 Body |
| --------- | ----------|
| on_create | 创建后的数据行信息 |
| on_update | 更新后的数据行信息 |
| on_delete | 被删除的数据行的原信息 |

若触发类型为微信支付回调，则请求 Body 解码后为微信支付结果。示例如下：
```json
{ trade_no: '5a7a695008443e1a69e1a06a',
  transaction_no: '5a7a695008443e1a69e1a06a',
  merchandise_description: 'test',
  merchandise_snapshot: {},
  merchandise_schema_id: 1,
  merchandise_record_id: '5a7a695008443e1a69e1a06a',
  status: 'success',
  refund_status: null,
  ip_address: '127.0.0.1',
  created_by_id: 123,
  paid_at: 1517970094 }
```

> **info**
> 注：webhook 不支持模板变量


### 数据表操作
执行结果：批量修改指定的数据表的数据行。
数据表操作动作的一些参数说明如下：

| 操作 | 被触发时动作说明 |
| :-- | :------------ |
| 创建 | 创建一行数据 |
| 更新 | 修改符合条件的数据行 |
| 删除 | 删除符合条件的数据行 |

当操作为**更新**或者**删除**时，需要配置查询条件，筛选出指定的数据行。数据表查询条件相关文档请[参考这里](schema.md)

> **info**
>注：此处的查询条件中也可以输入模板变量

目前**赋值操作**支持的数据类型及其对应的操作符如下：

| 数据类型 | 操作符 |
| :------ | :---------------------------- |
| array   | =, append, append_unique, remove |
| boolean | = |
| date    | = |
| integer | =, inc_by |
| number  | =, inc_by |
| string  | = |
| geojson | = |

>**info**
>注：append, append_unique, remove, inc_by 为原子操作符，相关文档请[参考这里](../js-sdk/schema/update-record.md)

## 模板变量的使用
部分动作中支持插入变量，您可以点击动作底部的"**可选变量**"查看该动作中支持添加的所有变量。

如需插入变量，请按照`{{变量名}}`的形式插入到邮件文本中

    例：您购买的产品{{product}}已经发货，请注意查收。

如需插入微信用户信息，请输入`{{created_by.***}}`。

    例：尊敬的{{created_by.nickname}}，您购买的产品{{product}}已经发货，请注意查收。

对于 date 类型的变量，可以自定义输出的格式，格式为 `{{created_at | date:"format"}}`，其中 format 为输出的格式，例如需要 2017-09-20 16:05:14 这样的输出格式，变量的格式为 `{{created_at | date:"Y-m-d  H:i:s"}}`，具体 format 的意义可参考「[date 格式参数说明][2]」

## 实战教程
[触发器实战教程请移步这里](../support/practice/trigger.md)

  [1]: https://mp.weixin.qq.com/
  [2]: http://support.minapp.com/hc/kb/article/1085622/?from=draft