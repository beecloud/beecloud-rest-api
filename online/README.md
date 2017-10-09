## BeeCloud REST API (Version 2)

## 简介

BeeCloud REST API文档的官方GitHub地址是 [https://github.com/beecloud/beecloud-rest-api](https://github.com/beecloud/beecloud-rest-api)

BeeCloud REST API提供支付，退款，订单查询，退款查询等支付相关的功能，所有第三方支付（如支付宝，微信支付，银联支付等）都统一在本API中.

#### 在线模拟
BeeCloud提供在线模拟REST调用，方便开发者调试：[REST API测试](https://beecloud.cn/rest/)


## 1. API请求地址
#### https://api.beecloud.cn

## 2. 支付

此接口为支付流程的第一步，主要功能在于生成订单，获取必要的参数信息，来进行下一步的支付流程。对于不同的渠道和支付方式，接口的返回值与后续的操作都不尽相同（例如微信App支付需要调用微信支付SDK的接口，支付宝网页支付需要跳转到获取的一段HTML网址等），请根据每一个channel的详细描述分别处理.

#### URL:   */2/rest/bill*
#### Method: *POST*
#### 请求参数格式: *JSON: Map*

#### 请求参数详情:
- 以下为公共参数：

参数名 | 类型 | 含义 | 描述 | 示例 | 是否必填
----  | ---- | ---- | ---- | ---- | ----
app_id | String | BeeCloud平台的AppID | App在BeeCloud平台的唯一标识 | 0950c062-5e41-44e3-8f52-f89d8cf2b6eb | 是
timestamp | Long | 签名生成时间 | 时间戳，毫秒数 | 1435890533866 | 是
app_sign | String | 加密签名 | 算法: md5(app\_id+timestamp+app\_secret)，32位16进制格式,不区分大小写 | b927899dda6f9a04afc57f21ddf69d69 | 是
channel| String | 渠道类型 | 根据不同场景选择不同的支付方式 | WX\_APP、WX\_WAP、WX\_NATIVE、WX\_JSAPI、WX\_MINI、ALI\_APP、ALI\_WEB、ALI\_QRCODE、ALI\_WAP、UN\_APP、UN\_WEB、UN\_WAP、APPLE、APPLE\_TEST、PAYPAL\_SANDBOX、PAYPAL\_LIVE、JD\_WAP、JD\_WEB、YEE\_WAP、YEE\_WEB、YEE\_NOBANKCARD、KUAIQIAN\_WAP、KUAIQIAN\_WEB、BD\_APP、BD\_WEB、BD\_WAP、BC\_GATEWAY、BC\_EXPRESS、BC\_APP、BC\_NATIVE、BC\_WX\_WAP、BC\_WX\_JSAPI、BC\_ALI\_QRCODE、BC\_ALI\_APP、BC\_ALI\_WAP、BC\_ALI\_WEB、BC\_WX\_APP、BC\_ALI\_JSAPI、BC\_WX\_MINI（详见附注）| 是
total_fee | Integer | 订单总金额 | 必须是正整数，单位为分 | 1 | 是
bill_no | String | 商户订单号 | 8到32位数字和/或字母组合，请自行确保在商户系统中唯一，同一订单号不可重复提交，否则会造成订单重复 | 201506101035040000001 | 是
title| String | 订单标题 | UTF8编码格式，32个字节内，最长支持16个汉字 | 白开水 | 是
optional | Map | 附加数据 | 用户自定义的参数，将会在webhook通知中原样返回，该字段主要用于商户携带订单的自定义数据 | {"key1":"value1","key2":"value2",...} | 否
analysis | Map | 分析数据 | 用于统计分析的数据，将会在控制台的统计分析报表中展示，**<mark>用户自愿上传</mark>** | 包括以下基本字段：`os_name(系统名称，如"iOS"，"Android")` `os_version(系统版本，如"5.1")` `model(手机型号，如"iPhone 6")` `app_name(应用名称)` `app_version(应用版本号)` `device_id(设备ID)` `category(类别，用户可自定义，如游戏分发渠道，门店ID等)` `browser_name(浏览器名称)` `browser_version(浏览器版本)` | 否
return_url | String | 同步返回页面| 支付渠道处理完请求后,当前页面自动跳转到商户网站里指定页面的http路径，**<mark>中间请勿有#,?等字符</mark>** | http://beecloud.cn/returnUrl.jsp | 当channel参数为 ALI\_WEB 或 ALI\_QRCODE 或  UN\_WEB 或 JD\_WAP 或 JD\_WEB时为必填, BC\_GATEWAY为京东渠道时，必填
notify_url | String | 商户自定义回调地址 | 商户可通过此参数设定回调地址，此地址会覆盖用户在控制台设置的回调地址。**<mark>必须以`http://`或`https://`开头</mark>** | http://beecloud.cn/notifyUrl.jsp
bill_timeout | Integer | 订单失效时间 | 必须为非零正整数，单位为秒，建议最短失效时间间隔必须<mark>大于</mark>360秒 | 360 | 否, **<mark>京东(JD)不支持该参数。</mark>** 
card_no| String | 银行卡号 | 银行卡号 | 8888888888888888 | 否，当渠道为BC_EXPRESS时，才为选填，其他渠道不要
limit_credit | Boolean | 禁用信用卡 | 禁用信用卡 | true/false | 否
bank | String | 银行名字 | 银行名字 | 中国银行 | BC\_GATEWAY为京东渠道时，必填。其他渠道不填写
card\_type| String | 卡类型 | 卡类型 | 卡类型， 1和2. 1代表信用卡；2代表借记卡 | BC\_GATEWAY为京东渠道时，必填。其他渠道不填写
store_id | String | 门店号| 门店号 | 卡门店号 | 选填
buyer\_id | String | 购买者id| 购买者id | 购买者id | 选填
seller\_id | String | 商家id| 商家id | 商家id | 选填
coupon\_id | String | 卡券id | 营销活动时使用 | 卡券id | 选填

> 注：channel的参数值含义：  
WX\_APP: 微信手机原生APP支付  
WX\_WAP: 微信移动网页支付  
WX\_NATIVE: 微信公众号二维码支付  
WX\_JSAPI: 微信公众号支付  
WX\_MINI: 微信小程序支付  
ALI\_APP: 支付宝手机原生APP支付  
ALI\_WEB: 支付宝PC网页支付  
ALI\_QRCODE: 支付宝内嵌二维码支付   
ALI\_WAP: 支付宝移动网页支付  
UN\_APP: 银联手机原生APP支付  
UN\_WEB: 银联PC网页支付  
UN\_WAP: 银联移动网页支付  
JD\_WAP: 京东移动网页支付   
JD\_WEB: 京东PC网页支付  
YEE\_WAP: 易宝移动网页支付  
YEE\_WEB: 易宝PC网页支付  
YEE\_NOBANKCARDB: 易宝充值卡支付  
KUAIQIAN\_WAP: 快钱移动网页支付  
KUAIQIAN\_WEB: 快钱PC网页支付  
PAYPAL\_LIVE: PAYPAL生产环境支付   
PAYPAL\_SANDBOX: PAYPAL沙箱环境支付   
BD\_APP: 百度手机原生APP支付   
BD\_WAP: 百度移动网页支付   
BD\_WEB: 百度PC网页支付   
BC\_GATEWAY: BC版网关支付  
BC\_EXPRESS: BC版快捷支付  
BC\_APP: BC版手机APP支付  
BC\_NATIVE: BC版微信二维码支付  
BC\_WX\_WAP: BC版微信手机WAP支付  
BC\_WX\_JSAPI: BC版微信公众号支付  
BC\_ALI\_QRCODE: BC版支付宝线下扫码支付  
BC\_ALI\_APP: BC版支付宝手机原生APP支付  
BC\_ALI\_WAP: BC版支付宝移动网页支付  
BC\_ALI\_WEB: BC版支付宝PC网页支付  
BC\_WX\_APP: BC版微信手机原生APP支付  
BC\_ALI\_JSAPI: BC版支付宝服务窗支付  
BC\_WX\_MINI: BC版微信小程序支付  

- 以下是`微信移动网页支付(WX_WAP)及BC版微信手机WAP支付(BC_WX_WAP)`的**<mark>必填</mark>**参数

参数名 | 类型 | 含义 | 例子
---- | ---- | ---- | ----
analysis| Map | 自定义参数 | {"ip":"58.110.23.35","key\_b":"value\_b"}

- 以下是`微信公众号支付(WX_JSAPI、BC_WX_JSAPI)`的**<mark>必填</mark>**参数

参数名 | 类型 | 含义 | 例子
---- | ---- | ---- | ----
openid| String | 用户相对于微信公众号的唯一id | 0950c062-5e41-44e3-8f52-f89d8cf2b6eb

- 以下是`支付宝网页支付(ALI_WEB)`的**<mark>选填</mark>**参数

参数名 | 类型 | 含义 | 例子
---- | ---- | ---- | ----
show_url| String | 商品展示地址以http://开头 | http://beecloud.cn

- 以下是`支付宝移动网页支付(ALI_WAP)`的**<mark>选填</mark>**参数

参数名 | 类型 | 含义 | 例子
---- | ---- | ---- | ----
use_app| Bool | 是否尝试掉起支付宝APP原生支付 | true

- 以下是`支付宝内嵌二维码支付(ALI_QRCODE)`的**<mark>必填</mark>**参数

参数名 | 类型 | 含义 | 例子
---- | ---- | ---- | ----
qr\_pay\_mode| String | 二维码类型 | 0,1,3

> 注： 二维码类型含义   
0： 订单码-简约前置模式, 对应 iframe 宽度不能小于 600px, 高度不能小于 300px   
1： 订单码-前置模式, 对应 iframe 宽度不能小于 300px, 高度不能小于 600px  
3： 订单码-迷你前置模式, 对应 iframe 宽度不能小于 75px, 高度不能小于 75px  

- 以下是`易宝移动网页支付(YEE_WAP)`的**<mark>必填</mark>**参数

参数名 | 类型 | 含义 
---- | ---- | ----
identity_id | String | 50位以内数字和/或字母组合，易宝移动网页（一键）支付用户唯一标识符，用于绑定用户一键支付的银行卡信息

- 以下是`微信小程序支付(WX_MINI)`的**<mark>必填</mark>**参数

参数名 | 类型 | 含义 | 例子
---- | ---- | ---- | ----
openid| String | 用户相对于微信公众号的唯一id | 0950c062-5e41-44e3-8f52-f89d8cf2b6eb 

- 以下是`易宝点卡支付(YEE_NOBANKCARD)`的**<mark>必填</mark>**参数

参数名 | 类型 | 含义 
---- | ---- | ----
cardno | String | 点卡卡号，每种卡的要求不一样，例如易宝支持的QQ币卡号是9位的，江苏省内部的QQ币卡号是15位，易宝不支付
cardpwd | String | 点卡密码，简称卡密
frqid | String | 点卡类型编码，骏网一卡通(JUNNET),盛大卡(SNDACARD),神州行(SZX),征途卡(ZHENGTU),Q币卡(QQCARD),联通卡(UNICOM),久游卡(JIUYOU),易充卡(YICHONGCARD),网易卡(NETEASE),完美卡(WANMEI),搜狐卡(SOHU),电信卡(TELECOM),纵游一卡通(ZONGYOU),天下一卡通(TIANXIA),天宏一卡通(TIANHONG),32 一卡通(THIRTYTWOCARD)
> 注： total_fee(订单金额)必须和充值卡面额相同，否则会造成**<mark>金额丢失(渠道方决定)</mark>**

- 以下是`花呗支付(ALI_APP、ALI_WEB、ALI_QRCODE、ALI_WAP（老版不支持）、ALI_OFFLINE_QRCODE、ALI_SCAN)`的**<mark>必填</mark>**参数

参数名 | 类型 | 含义 | 例子
---- | ---- | ---- | ----
optional | Map | 自定义参数 | {"hb\_fq\_num":"3","hb\_fq\_seller\_percent":"100"}  hb\_fq\_num仅支持传入3、6、12的字符串，hb\_fq\_seller\_percent仅支持传入100、0的字符串

#### 返回类型: *JSON: Map*
#### 返回参数:

- **公共返回参数**

参数名 | 类型 | 含义 
---- | ---- | ----
result_code | Integer | 返回码，0为正常
result_msg  | String | 返回信息， OK为正常
err_detail  | String | 具体错误信息
id  | String | 成功发起支付后返回支付表记录唯一标识

- **公共返回参数取值列表及其含义**

result_code | result_msg             | 含义
----        | ----      		       | ----
0           | OK                     | 调用成功
1           | APP\_INVALID           | 根据app\_id找不到对应的APP或者app\_sign不正确
2           | PAY\_FACTOR_NOT\_SET   | 支付要素在后台没有设置
3           | CHANNEL\_INVALID       | channel参数不合法
4           | MISS\_PARAM            | 缺少必填参数
5           | PARAM\_INVALID         | 参数不合法
6           | CERT\_FILE\_ERROR      | 证书错误
7           | CHANNEL\_ERROR         | 渠道内部错误
14          | RUNTIME\_ERROR         | 运行时错误

> **当result_code不为0时，如需详细信息，请查看err\_detail字段**

**<mark>以下字段在result_code为0时有返回</mark>**

- WX_APP

参数名 | 类型 | 含义 
---- | ---- | ----
app_id | String | 微信应用APPID
partner_id | String | 微信支付商户号
package  | String | 微信支付打包参数
nonce_str  | String | 随机字符串
timestamp | String | 当前时间戳，单位是毫秒，13位
pay_sign  | String | 签名值
prepay_id  | String | 微信预支付id

- WX\_NATIVE、BC\_NATIVE  BC\_ALI\_QRCODE

参数名 | 类型 | 含义 
---- | ---- | ----
code_url | String | 二维码地址

- WX\_JSAPI、BC\_WX\_JSAPI

参数名 | 类型 | 含义 
---- | ---- | ----
app_id | String | 微信应用APPID
package  | String | 微信支付打包参数
nonce_str  | String | 随机字符串
timestamp | String | 当前时间戳，单位是毫秒，13位
pay_sign  | String | 签名
sign_type  | String | 签名类型，固定为MD5

- BC\_WX\_WAP

参数名 | 类型 | 含义 
---- | ---- | ----
url  | String | BC版微信WAP的跳转url

- ALI_APP

参数名 | 类型 | 含义 
---- | ---- | ----
order\_string | String | 支付宝签名串

- ALI\_WEB，ALI\_WAP

参数名 | 类型 | 含义 
---- | ---- | ----
html | String | 支付宝跳转form，是一段HTML代码，自动提交
url  | String | 支付宝跳转url

- ALI\_OFFLINE\_QRCODE

参数名 | 类型 | 含义 
---- | ---- | ----
qr_code | String | 二维码码串,可以用二维码生成工具根据该码串值生成对应的二维码

- ALI_QRCODE

参数名 | 类型 | 含义 
---- | ---- | ----
html | String | 支付宝跳转form，是一段HTML代码，自动提交
url  | String | 支付宝内嵌二维码地址，是一个URL

- UN_APP

参数名 | 类型 | 含义 
---- | ---- | ----
tn | String | 银联支付ticket number

- UN\_WEB、UN\_WAP、JD\_WAP、JD\_WEB、KUAIQIAN\_WAP、KUAIQIAN\_WEB

参数名 | 类型 | 含义 
---- | ---- | ----
html | String | 支付页自动提交form表单内容

- YEE\_WAP、YEE\_WEB、BD\_WAP、BD\_WEB

参数名 | 类型 | 含义 
---- | ---- | ----
url | String | 支付页跳转地址

- BD\_APP

参数名 | 类型 | 含义 
---- | ---- | ----
orderInfo | String | 百度支付order info
  

## 3. 退款

退款接口仅支持对已经支付成功的订单进行退款，且目前对于同一笔订单，仅能退款成功一次（对于同一个退款请求，如果第一次退款申请被驳回，仍可以进行二次退款申请）。 退款金额refund\_fee必须小于或者等于原始支付订单的实付金额，如果是小于，则表示部分退款.

退款接口包含预退款功能，当need_approval字段的值为true时，该接口开启预退款功能，预退款仅创建退款记录，并不真正发起退款，需后续调用审核接口，或者通过BeeCloud控制台的预退款界面，审核同意或者否决，才真正发起退款或者拒绝预退款。

#### URL: */2/rest/refund*
#### Method: *POST*

#### 请求参数格式: *JSON: Map*
#### 请求参数详情:

- 参数列表

参数名 | 类型 | 含义   | 描述 | 例子 | 是否必填 |
---- | ---- | ---- | ---- | ---- | ----
app_id | String | BeeCloud应用APPID | BeeCloud的唯一标识 | 0950c062\-5e41\-44e3\-8f52\-f89d8cf2b6eb | 是 
timestamp | Long | 签名生成时间 | 时间戳，毫秒数 | 1435890533866 | 是
app_sign | String | 加密签名 | 算法: md5(app\_id+timestamp+master\_secret)，32位16进制格式,不区分大小写 | b927899dda6f9a04afc57f21ddf69d69 | 是
channel| String | 渠道类型 | 根据不同渠道选不同的值 | WX ALI UN KUAIQIAN BD JD YEE | 否
refund_no | String | 商户退款单号 | 格式为:退款日期(8位) + 流水号(3~24 位)。请自行确保在商户系统中唯一，且退款日期必须是发起退款的当天日期,同一退款单号不可重复提交，否则会造成退款单重复。流水号可以接受数字或英文字符，建议使用数字，但不可接受“000” | 201506101035040000001 | 是
bill_no | String | 商户订单号 | 发起支付时填写的订单号 | 201506101035040000001 | 是 
refund_fee | Integer | 退款金额 | 必须为正整数，单位为分，必须小于或等于对应的已支付订单的实付金额 | 1 | 是
notify_url | String | 商户自定义退款回调地址 | 商户可通过此参数设定退款回调地址，此地址会覆盖用户在控制台设置的退款回调地址。**<mark>必须以`http://`或`https://`开头</mark>** | http://beecloud.cn/notifyUrl.jsp | 否
optional | Map | 附加数据 | 用户自定义的参数，将会在webhook通知中原样返回，该字段主要用于商户携带订单的自定义数据 | {"key1":"value1","key2":"value2",...} | 否
need_approval| Bool | 是否为预退款 | 预退款need_approval值传true,直接退款不传此参数或者传false | true | 否
refund_account | Integer | 退款资金来源 | 1:可用余额退款 0:未结算资金退款（默认使用未结算资金退款） | 1 | 否

#### 返回类型: *JSON: Map*
#### 返回参数:

- 公共返回参数

参数名 | 类型 | 含义 
---- | ---- | ----
result\_code | Integer | 返回码，0为正常
result\_msg  | String | 返回信息，OK为正常
err\_detail  | String | 具体错误信息
id  | String | 成功发起预退款或者直接退款后返回退款表记录唯一标识

- 返回码和返回信息取值及含义参见支付公共返回参数部分, 以下是退款所特有的

result\_code | result\_msg                | 含义
----        | ----      			       | ----
8           | NO\_SUCH_BILL             | 没有该订单
9           | BILL\_UNSUCCESS            | 该订单没有支付成功
10          | REFUND\_EXCEED\_TIME       | 已超过可退款时间
11          | ALREADY\_REFUNDING         | 该订单已有正在处理中的退款
12          | REFUND\_AMOUNT\_TOO\_LARGE | 提交的退款金额超出可退额度
13          | NO\_SUCH\_REFUND           | 没有该退款记录

**当channel为`ALI`，并且不是预退款时，以下字段在result_code为0时有返回**
 
参数名 | 类型 | 含义 
---- | ---- | ----
url | String | 支付宝退款地址，需用户在支付宝平台上手动输入支付密码处理

  
## 4. 预退款批量审核

批量审核接口仅支持预退款，批量审核分为批量驳回和批量同意。

#### URL: */2/rest/refund*
#### Method: *PUT*

#### 请求参数格式: *JSON: Map*
#### 请求参数详情:

- 参数列表

参数名 | 类型 | 含义   | 描述 | 例子 | 是否必填 |
---- | ---- | ---- | ---- | ---- | ----
app_id | String | BeeCloud应用APPID | BeeCloud的唯一标识 | 0950c062\-5e41\-44e3\-8f52\-f89d8cf2b6eb | 是 
timestamp | Long | 签名生成时间 | 时间戳，毫秒数 | 1435890533866 | 是
app_sign | String | 加密签名 | 算法: md5(app\_id+timestamp+master\_secret)，32位16进制格式,不区分大小写 | b927899dda6f9a04afc57f21ddf69d69 | 是
channel| String | 渠道类型 | 根据不同渠道选不同的值 | WX ALI UN KUAIQIAN BD JD YEE | 是
ids | List<String> | 退款记录id列表 | 批量审核的退款记录的唯一标识符集合 | ["d9690a6e-ae99-44b7-9904-bd9d43fcc21b", "6f263aa6-111d-4c95-b51e-001b3f7e6ddf", "  7d1f69e4-3ff2-4b7d-b764-eb018620e00d"] | 是
agree| Bool | 同意或者驳回 | 批量驳回传false，批量同意传true | true | 是
deny_reason| String | 驳回理由 | 批量驳回原因 | "审核不通过" | 否

#### 返回类型: *JSON: Map*
#### 返回参数:

- 公共返回参数

参数名 | 类型 | 含义 
---- | ---- | ----
result\_code | Integer | 返回码，0为正常
result\_msg  | String | 返回信息，OK为正常
err\_detail  | String | 具体错误信息

- 返回码和返回信息取值及含义参见支付公共返回参数部分

**当agree为true时，以下字段在result_code为0时有返回**
 
参数名 | 类型 | 含义 
---- | ---- | ----
result_map | Map<String, String>; |批量同意单笔结果集合，key:单笔记录id; value:此笔记录结果。 当退款处理成功时，value值为"OK"；当退款处理失败时， value值为具体的错误信息。


**当channel为`ALI`时，以下字段在agree为true时有返回**
 
参数名 | 类型 | 含义 
---- | ---- | ----
url | String | 支付宝退款地址，需用户在支付宝平台上手动输入支付密码处理
  
  
## 5. 订单查询

#### URL:   */2/rest/bills*
#### Method: *GET*

#### 请求参数类型: *JSON, 以para=**{}**的方式请求*

示例: para={"key\_a":1,"key\_b":"value\_b"}, 需要对para=后面的部分做URL encode.

#### 请求参数详情:
参数名 | 类型 | 含义 | 描述 | 例子 | 是否必填
----  | ---- | ---- | ---- | ---- | ----
app_id | String | BeeCloud应用APPID | BeeCloud的唯一标识 | 0950c062-5e41-44e3-8f52-f89d8cf2b6eb | 是
timestamp | Long | 签名生成时间 | 时间戳，毫秒数 | 1435890533866 | 是
app_sign | String | 加密签名 | 算法: md5(app\_id+timestamp+app\_secret)，32位16进制格式,不区分大小写 | b927899dda6f9a04afc57f21ddf69d69 | 是
channel| String | 渠道类型 | 根据不同场景选择不同的支付方式 | WX、WX\_APP、WX\_NATIVE、WX\_JSAPI、ALI、ALI\_APP、ALI\_WEB、ALI\_QRCODE、ALI_WAP、UN、UN\_APP、UN\_WEB、UN\_WAP、PAYPAL、PAYPAL\_SANDBOX、PAYPAL\_LIVE、JD_WAP、JD_WEB、YEE_WAP、YEE_WEB、KUAIQIAN_WAP、KUAIQIAN_WEB、JD、YEE、KUAIQIAN、BD、BD\_APP、BD\_WEB、BD\_WAP、BC、BC\_GATEWAY、BC\_EXPRESS、BC\_APP、BC\_NATIVE、BC\_WX\_WAP、BC\_WX\_JSAPI、BC\_ALI\_QRCODE(详见附注）| 否
bill_no | String | 商户订单号 | 发起支付时填写的订单号 | 201506101035040000001 | 否
buyer_id | String | 购买者id | 发起支付时填写的购买者id |demo@beecloud.cn | 否
seller_id | String | 商家id | 发起支付时填写的商家id |123456789 | 否
store_id | String | 门店号 | 发起支付时填写的门店号 |123456789 | 否
operator_id | String | 商户的操作员编号 | OP001 | 否
spay_result | Bool | 订单是否成功 | 标识订单是否支付成功 | true | 否
refund_result | Bool | 订单是否已退款 | 标识订单是否已退款 | true | 否
need_detail | Bool | 是否需要返回渠道详细信息 | 决定是否需要返回渠道的回调信息，true为需要 | true | 否
start_time | Long | 起始时间 | 毫秒时间戳, 13位 | 1435890530000 | 否
end_time | Long | 结束时间 | 毫秒时间戳, 13位   | 1435890540000 | 否
skip | Integer| 查询起始位置 | 默认为0. 设置为10表示忽略满足条件的前10条数据| 0 | 否
limit| Integer | 查询的条数 | 默认为10，最大为50. 设置为10表示只返回满足条件的10条数据 | 10 | 否

> 注：  
1. bill\_no, start\_time, end\_time等查询条件互相为**<mark>且</mark>**关系  
2. start\_time, end\_time指的是订单生成的时间，而不是订单支付的时间   


#### 返回类型: *JSON: Map*
#### 返回参数:

- 返回参数

参数名 | 类型 | 含义 
---- | ---- | ----
result\_code | Integer| 返回码，0为正常
result\_msg  | String | 返回信息， OK为正常
err\_detail  | String | 具体错误信息
bills | List<Map> | 订单列表

- 返回码和返回信息取值及含义参见支付公共返回参数部分  

- bills说明，每个Map的key\-value

参数名         | 类型          | 含义 
----          | ----         | ----
id      | String       | 订单记录的唯一标识，可用于查询单笔记录
bill\_no      | String       | 订单号
bill\_fee | Integer         | 订单金额，单位为分
total\_fee    | Integer         | 实付金额，单位为分
discount | Integer         | 优惠金额，单位为分
coupon\_id | String | 卡券ID，没有用到返回null
trade\_no    | String         | 渠道交易号， 当支付成功时有值
channel       | String       | 渠道类型 WX、ALI、UN、JD、YEE、KUAIQIAN、PAYPAL、BD
sub_channel         | String       | 子渠道类型 WX_APP、WX_NATIVE、WX_JSAPI、WX_SCAN、ALI_APP、ALI_SCAN、ALI_WEB、ALI_QRCODE、ALI_OFFLINE_QRCODE、ALI_WAP、UN_APP、UN_WEB、UN_WAP、PAYPAL_SANDBOX、PAYPAL_LIVE、JD_WAP、JD_WEB、YEE_WAP、YEE_WEB、YEE_NOBANKCARD、KUAIQIAN_WAP、KUAIQIAN_WEB、BD_APP、BD_WEB、BD_WAP(详见 2. 支付 附注）
title         | String       | 订单标题
buyer_id         | String       | 购买者id
seller_id         | String       | 商家id
store_id         | String       | 门店号
operator_id | String | 商户的操作员编号
spay\_result  | Bool         | 订单是否成功
create_time | Long         | 订单创建时间, 毫秒时间戳, 13位
optional | String | 附加数据,用户自定义的参数，将会在webhook通知中原样返回，该字段是JSON格式的字符串 {"key1":"value1","key2":"value2",...}
message_detail | String         | 渠道详细信息， 当need_detail传入true时返回
revert_result  | Bool         | 订单是否已经撤销
refund_result  | Bool         | 订单是否已经退款
  
  
## 6. 订单总数查询

#### URL:   */2/rest/bills/count*
#### Method: *GET*

#### 请求参数类型: *JSON, 以para=**{}**的方式请求*

示例: para={"key\_a":1,"key\_b":"value\_b"}, 需要对para=后面的部分做URL encode.

#### 请求参数详情:
参数名 | 类型 | 含义 | 描述 | 例子 | 是否必填
----  | ---- | ---- | ---- | ---- | ----
app_id | String | BeeCloud应用APPID | BeeCloud的唯一标识 | 0950c062-5e41-44e3-8f52-f89d8cf2b6eb | 是
timestamp | Long | 签名生成时间 | 时间戳，毫秒数 | 1435890533866 | 是
app_sign | String | 加密签名 | 算法: md5(app\_id+timestamp+app\_secret)，32位16进制格式,不区分大小写 | b927899dda6f9a04afc57f21ddf69d69 | 是
channel| String | 渠道类型 | 根据不同场景选择不同的支付方式 | WX、WX\_APP、WX\_NATIVE、WX\_JSAPI、ALI、ALI\_APP、ALI\_WEB、ALI\_QRCODE、ALI_WAP、UN、UN\_APP、UN\_WEB、UN\_WAP、PAYPAL、PAYPAL\_SANDBOX、PAYPAL\_LIVE、JD_WAP、JD_WEB、YEE_WAP、YEE_WEB、KUAIQIAN_WAP、KUAIQIAN_WEB、JD、YEE、KUAIQIAN、BD、BD\_APP、BD\_WEB、BD\_WAP、BC、BC\_GATEWAY、BC\_EXPRESS、BC\_APP、BC\_NATIVE、BC\_WX\_WAP、BC\_WX\_JSAPI、BC\_ALI\_QRCODE(详见附注）| 否
bill_no | String | 商户订单号 | 发起支付时填写的订单号 | 201506101035040000001 | 否
spay_result | Bool | 订单是否成功 | 标识订单是否支付成功 | true | 否
start_time | Long | 起始时间 | 毫秒时间戳, 13位 | 1435890530000 | 否
end_time | Long | 结束时间 | 毫秒时间戳, 13位   | 1435890540000 | 否

> 注：  
1. bill\_no, start\_time, end\_time等查询条件互相为**<mark>且</mark>**关系  
2. start\_time, end\_time指的是订单生成的时间，而不是订单支付的时间   


#### 返回类型: *JSON: Map*
#### 返回参数:

- 返回参数

参数名 | 类型 | 含义 
---- | ---- | ----
result\_code | Integer| 返回码，0为正常
result\_msg  | String | 返回信息， OK为正常
err\_detail  | String | 具体错误信息
count | Integer | 查询订单结果数量

- 返回码和返回信息取值及含义参见支付公共返回参数部分
  
  
## 7. 退款查询

#### URL:   */2/rest/refunds*
#### Method: GET

#### 请求参数类型: JSON，以para=**{}**的方式请求

示例: para={"key\_a":1,"key\_b":"value\_b"}, 需要对para=后面的部分做URL encode.

#### 请求参数详情:

参数名 | 类型 | 含义 | 描述 | 例子 | 是否必填
----  | ---- | ---- | ---- | ---- | ----
app_id | String | BeeCloud应用APPID | BeeCloud的唯一标识 | 0950c062-5e41-44e3-8f52-f89d8cf2b6eb | 是
timestamp | Long | 签名生成时间 | 时间戳，毫秒数 | 1435890533866 | 是
app_sign | String | 加密签名 | 算法: md5(app\_id+timestamp+app\_secret)，不区分大小写 | b927899dda6f9a04afc57f21ddf69d69 | 是
channel| String | 渠道类型 | 根据不同场景选择不同的支付方式 | WX、WX\_NATIVE、WX\_JSAPI、ALI、ALI\_APP、ALI\_WEB、ALI\_QRCODE、ALI_WAP、UN、UN\_APP、UN\_WEB、UN\_WAP、JD_WAP、JD_WEB、YEE_WAP、YEE_WEB、KUAIQIAN_WAP、KUAIQIAN_WEB、JD、YEE、KUAIQIAN、BD、BD\_APP、BD\_WEB、BD\_WAP(详见2.支付附注）| 否
bill_no | String | 商户订单号 | 发起支付时填写的订单号 | 201506101035040000001 | 否
refund_no | String | 商户退款单号 | 发起退款时填写的退款单号 | 201506101035040000001 | 否
start_time | Long | 起始时间 | 毫秒时间戳, 13位 | 1435890530000 | 否
end_time | Long | 结束时间 | 毫秒时间戳, 13位   | 1435890540000 | 否
need_approval | Bool | 需要审核 | 标识退款记录是否为预退款   | true | 否
need_detail | Bool | 是否需要返回渠道详细信息 | 决定是否需要返回渠道的回调信息，true为需要 | true | 否
skip | Integer | 查询起始位置 | 默认为0. 设置为10，表示忽略满足条件的前10条数据| 0 | 否
limit| Integer | 查询的条数 | 默认为10，最大为50. 设置为10，表示只查询满足条件的10条数据 | 10 | 否

> 注：  
1. bill\_no, refund\_no, start\_time, end\_time等查询条件互相为**<mark>且</mark>**关系.   
2. start\_time, end\_time指的是订单生成的时间，而不是订单支付的时间.   


#### 返回类型: *JSON, Map*
#### 返回详情:

- 返回参数

参数名 | 类型 | 含义 
---- | ---- | ----
result\_code | Integer| 返回码，0为正常
result\_msg  | String | 返回信息， OK为正常
err\_detail  | String | 具体错误信息
refunds | List<Map> | 退款列表

- 返回码和返回信息取值及含义参见支付公共返回参数部分

- refunds说明，每个Map的key\-value

参数名      | 类型         | 含义 
----       | ----        | ----
id      | String       | 退款记录的唯一标识，可用于查询单笔记录
bill\_no    | String      | 订单号
refund\_no  | String      | 退款号
total\_fee  | Integer      | 实付金额，单位为分
refund\_fee | Integer      | 退款金额，单位为分
title         | String       | 订单标题
channel    | String      | 渠道类型 WX、ALI、UN、JD、YEE、KUAIQIAN、BD
sub\_channel    | String      | 子渠道类型 WX\_NATIVE、WX\_JSAPI、WX\_APP、ALI\_APP、ALI\_WEB、ALI\_QRCODE、UN\_APP、UN\_WEB、UN\_WAP、JD_WAP、JD_WEB、YEE_WAP、YEE_WEB、KUAIQIAN_WAP、KUAIQIAN_WEB、BD\_APP、BD\_WEB、BD\_WAP(详见 1. 支付 附注）
finish     | bool        | 退款是否完成
result     | bool        | 退款是否成功
optional | String | 附加数据,用户自定义的参数，将会在webhook通知中原样返回，该字段是JSON格式的字符串 {"key1":"value1","key2":"value2",...}
message\_detail | String         | 渠道详细信息， 当need_detail传入true时返回
create\_time | Long       | 退款创建时间, 毫秒时间戳, 13位
  
  
## 8. 退款总数查询

#### URL:   */2/rest/refunds/count*
#### Method: GET

#### 请求参数类型: JSON，以para=**{}**的方式请求

示例: para={"key\_a":1,"key\_b":"value\_b"}, 需要对para=后面的部分做URL encode.

#### 请求参数详情:

参数名 | 类型 | 含义 | 描述 | 例子 | 是否必填
----  | ---- | ---- | ---- | ---- | ----
app_id | String | BeeCloud应用APPID | BeeCloud的唯一标识 | 0950c062-5e41-44e3-8f52-f89d8cf2b6eb | 是
timestamp | Long | 签名生成时间 | 时间戳，毫秒数 | 1435890533866 | 是
app_sign | String | 加密签名 | 算法: md5(app\_id+timestamp+app\_secret)，不区分大小写 | b927899dda6f9a04afc57f21ddf69d69 | 是
channel| String | 渠道类型 | 根据不同场景选择不同的支付方式 | WX、WX\_NATIVE、WX\_JSAPI、ALI、ALI\_APP、ALI\_WEB、ALI\_QRCODE、ALI_WAP、UN、UN\_APP、UN\_WEB、UN\_WAP、JD_WAP、JD_WEB、YEE_WAP、YEE_WEB、KUAIQIAN_WAP、KUAIQIAN_WEB、JD、YEE、KUAIQIAN、BD、BD\_APP、BD\_WEB、BD\_WAP(详见2.支付附注）| 否
bill_no | String | 商户订单号 | 发起支付时填写的订单号 | 201506101035040000001 | 否
refund_no | String | 商户退款单号 | 发起退款时填写的退款单号 | 201506101035040000001 | 否
start_time | Long | 起始时间 | 毫秒时间戳, 13位 | 1435890530000 | 否
end_time | Long | 结束时间 | 毫秒时间戳, 13位   | 1435890540000 | 否
need_approval | Bool | 需要审核 | 标识退款记录是否为预退款   | true | 否

> 注：  
1. bill\_no, refund\_no, start\_time, end\_time等查询条件互相为**<mark>且</mark>**关系.   
2. start\_time, end\_time指的是订单生成的时间，而不是订单支付的时间.   


#### 返回类型: *JSON, Map*
#### 返回详情:

- 返回参数

参数名 | 类型 | 含义 
---- | ---- | ----
result\_code | Integer| 返回码，0为正常
result\_msg  | String | 返回信息， OK为正常
err\_detail  | String | 具体错误信息
count | Integer | 查询退款结果数量

- 返回码和返回信息取值及含义参见支付公共返回参数部分
  
  
## 9. 退款状态更新

退款状态更新接口提供查询退款状态以更新退款状态的功能，用于对退款后不发送回调的渠道（WX、YEE、KUAIQIAN、BD）退款后的状态更新。

#### URL:   */2/rest/refund/status*
#### Method: GET

#### 请求参数类型:JSON，以para=**{}**的方式请求

示例: para={"key\_a":1,"key\_b":"value\_b"}, 需要对para=后面的部分做URL encode.

#### 请求参数详情:

参数名 | 类型 | 含义 | 描述 | 例子 | 是否必填
----  | ---- | ---- | ---- | ---- | ----
app_id | String | BeeCloud应用APPID | BeeCloud的唯一标识 | 0950c062-5e41-44e3-8f52-f89d8cf2b6eb | 是
timestamp | Long | 签名生成时间 | 时间戳，毫秒数 | 1435890533866 | 是
app_sign | String | 加密签名 | 算法: md5(app\_id+timestamp+app\_secret)，32位16进制格式，不区分大小写 | b927899dda6f9a04afc57f21ddf69d69 | 是
channel| String | 渠道类型 | 根据不同场景选择不同的支付方式 | 目前只支持WX、YEE、KUAIQIAN、BD | 是
refund_no | String | 商户退款单号 | 发起退款时填写的退款单号 | 201506101035040000001 | 是

#### 返回类型: *JSON, Map*
#### 返回详情:

- 返回参数

参数名 | 类型 | 含义 
---- | ---- | ----
result\_code | Integer | 返回码，0为正常
result\_msg  | String | 返回信息， OK为正常
err\_detail  | String | 具体错误信息
refund\_status | String | 退款状态

- 返回码和返回信息取值及含义参见支付公共返回参数部分
  
  
## 10. 退款订单查询(指定ID)

#### URL:   */2/rest/refund/{id}*
#### Method: *GET*
#### id : 退款订单唯一标识

#### 请求参数类型: *JSON, 以para=**{}**的方式请求*

示例: para={"key\_a":1,"key\_b":"value\_b"}, 需要对para=后面的部分做URL encode.

#### 请求参数详情:
参数名 | 类型 | 含义 | 描述 | 例子 | 是否必填
----  | ---- | ---- | ---- | ---- | ----
app_id | String | BeeCloud应用APPID | BeeCloud的唯一标识 | 0950c062-5e41-44e3-8f52-f89d8cf2b6eb | 是
timestamp | Long | 签名生成时间 | 时间戳，毫秒数 | 1435890533866 | 是
app_sign | String | 加密签名 | 算法: md5(app\_id+timestamp+app\_secret)，32位16进制格式,不区分大小写 | b927899dda6f9a04afc57f21ddf69d69 | 是


#### 返回类型: *JSON: Map*
#### 返回参数:

- 返回参数

参数名 | 类型 | 含义 
---- | ---- | ----
result\_code | Integer| 返回码，0为正常
result\_msg  | String | 返回信息， OK为正常
err\_detail  | String | 具体错误信息，有错误时，不会返回refund结果
refund | Map | 退款结果

- 返回码和返回信息取值及含义参见支付公共返回参数部分

- refund说明，每个Map的key\-value

参数名      | 类型         | 含义 
----       | ----        | ----
id      | String       | 退款记录的唯一标识，可用于查询单笔记录
bill\_no | String | 支付订单号
channel | String | WX、ALI、UN、JD、KUAIQIAN、BD、YEE
sub\_channel | String | WX_APP、WX_NATIVE、WX_JSAPI、ALI_APP、ALI_WEB、ALI_QRCODE、ALI_WAP、UN_APP、UN_WEB、UN_WAP、JD_WAP、JD_WEB、YEE_WAP、YEE_WEB、KUAIQIAN_WAP、KUAIQIAN_WEB、BD_APP、BD_WEB、BD_WAP (详见 2. 支付 附注）
finish | Bool | 退款是否完成
create_time | Long | 退款创建时间, 毫秒时间戳, 13位
optional | String | 附加数据,用户自定义的参数，将会在webhook通知中原样返回，该字段是JSON格式的字符串 {"key1":"value1","key2":"value2",...}
result | Bool| 退款是否成功
title | String | 商品标题
total_fee | Integer | 实付金额，单位为分
refund_fee | Integer | 退款金额，单位为分
refund_no | String | 退款单号
message\_detail | String         | 渠道详细信息
  
  
## 11. 支付订单查询(指定ID)

#### URL:   */2/rest/bill/{id}*
#### Method: *GET*
#### id : 支付订单唯一标识

#### 请求参数类型: *JSON, 以para=**{}**的方式请求*

示例: para={"key\_a":1,"key\_b":"value\_b"}, 需要对para=后面的部分做URL encode.

#### 请求参数详情:
参数名 | 类型 | 含义 | 描述 | 例子 | 是否必填
----  | ---- | ---- | ---- | ---- | ----
app_id | String | BeeCloud应用APPID | BeeCloud的唯一标识 | 0950c062-5e41-44e3-8f52-f89d8cf2b6eb | 是
timestamp | Long | 签名生成时间 | 时间戳，毫秒数 | 1435890533866 | 是
app_sign | String | 加密签名 | 算法: md5(app\_id+timestamp+app\_secret)，32位16进制格式,不区分大小写 | b927899dda6f9a04afc57f21ddf69d69 | 是


#### 返回类型: *JSON: Map*
#### 返回参数:

- 返回参数

参数名 | 类型 | 含义 
---- | ---- | ----
result\_code | Integer| 返回码，0为正常
result\_msg  | String | 返回信息， OK为正常
err\_detail  | String | 具体错误信息，有错误时，不会返回pay结果
pay | Map | 支付结果

- 返回码和返回信息取值及含义参见支付公共返回参数部分

- pay说明，每个Map的key\-value

参数名      | 类型         | 含义 
----       | ----        | ----
id      | String       | 订单记录的唯一标识，可用于查询单笔记录
bill\_no | String | 支付订单号
channel | String | WX、ALI、UN、JD、KUAIQIAN、YEE、BD、PAYPAL
sub\_channel | String | WX_APP、WX_NATIVE、WX_JSAPI、WX_SCAN、ALI_APP、ALI_SCAN、ALI_WEB、ALI_QRCODE、ALI_OFFLINE_QRCODE、ALI_WAP、UN_APP、UN_WEB、UN_WAP、PAYPAL_SANDBOX、PAYPAL_LIVE、JD_WAP、JD_WEB、YEE_WAP、YEE_WEB、YEE_NOBANKCARD、KUAIQIAN_WAP、KUAIQIAN_WEB、BD_APP、BD_WEB、BD_WAP (详见 2. 支付 附注）
trade\_no | String | 渠道返回的交易号，当支付成功时有值
create\_time | Long | 订单创建时间, 毫秒时间戳, 13位
optional | String | 附加数据,用户自定义的参数，将会在webhook通知中原样返回，该字段是JSON格式的字符串 {"key1":"value1","key2":"value2",...}
spay\_result | Bool| 订单是否成功
title | String | 商品标题
bill\_fee | Integer         | 订单金额，单位为分
total\_fee    | Integer         | 实付金额，单位为分
discount | Integer         | 优惠金额，单位为分
coupon\_id | String | 卡券ID，没有用到返回null
message\_detail | String         | 渠道详细信息
revert\_result  | Bool         | 订单是否已经撤销
refund\_result  | Bool         | 订单是否已经退款
  
  
## 12. 认证支付(需要开通)

认证支付分为三步。

- 1.发起支付，传入支付参数，获取token，支付记录的id，且收到短信验证码。api参考上面“2 支付”，注意:在optional中传入phone_no(手机号)，card_no（银行卡号），id_no（身份证号），customer_name（银行卡持有者姓名）等四个要素，或者传入token（第一次发起支付时返回的授权码）一个要素；channel为BC\_EXPRESS；buyer_id必填，用于标识一个用户，id必须唯一
- 2.确认支付，传入token,短信验证码，支付记录的id, 接口如下
- 3.接受webhook，更新订单状态，参考webhook文档

#### 确认支付接口
#### URL:   */2/rest/bill/confirm*
#### Method: *POST*
#### 请求参数格式: *JSON: Map*

#### 请求参数详情:
- 以下为公共参数：

参数名 | 类型 | 含义 | 描述 | 示例 | 是否必填
----  | ---- | ---- | ---- | ---- | ----
app_id | String | BeeCloud平台的AppID | App在BeeCloud平台的唯一标识 | 0950c062-5e41-44e3-8f52-f89d8cf2b6eb | 是
timestamp | Long | 签名生成时间 | 时间戳，毫秒数 | 1435890533866 | 是
app_sign | String | 加密签名 | 算法: md5(app\_id+timestamp+app\_secret)，32位16进制格式,不区分大小写 | b927899dda6f9a04afc57f21ddf69d69 | 是
token | String | 渠道返回的token | 渠道返回的token | 09999950c062f89d8cf2b6eb | 是
bc_bill_id | String | BeeCloud生成的唯一支付记录id | BeeCloud生成的唯一支付记录id | 0950c062-5e41-44e3-8f52-f89d8cf2b6eb | 是
verify_code | String | 短信验证码 | 短信验证码 | 666666 | 是



#### 返回类型: *JSON: Map*
#### 返回参数:

- 返回参数

参数名 | 类型 | 含义 
---- | ---- | ----
result\_code | Integer| 返回码，0为正常
result\_msg  | String | 返回信息， OK为正常
err\_detail  | String | 具体错误信息，有错误时，不会返回pay结果
