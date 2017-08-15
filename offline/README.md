## BeeCloud 线下 REST API (Version 2)

## 简介

BeeCloud REST API文档的官方GitHub地址是 [https://github.com/beecloud/beecloud-rest-api](https://github.com/beecloud/beecloud-rest-api)

## Part1. 文档

### 1. API请求地址
#### https://api.beecloud.cn

### 2. 支付

此接口为支付流程的第一步，主要功能在于生成订单，获取必要的参数信息，来进行下一步的支付流程. 对于不同的渠道和支付方式，接口的返回值与后续的操作（例如微信App支付需要调用微信支付SDK的接口，支付宝网页支付需要跳转到获取的一段HTML网址等）都不尽相同，请根据每一个channel的详细描述分别处理.

#### URL:   */2/rest/offline/bill*
#### Method: *POST*
#### 请求参数格式: *JSON: Map*

#### 请求参数详情:
- 以下为公共参数：

参数名 | 类型 | 含义 | 描述 | 例子 | 是否必填
----  | ---- | ---- | ---- | ---- | ----
app_id | String | BeeCloud平台的AppID | App在BeeCloud平台的唯一标识 | 0950c062-5e41-44e3-8f52-f89d8cf2b6eb | 是
timestamp | Long | 签名生成时间 | 时间戳，毫秒数 | 1435890533866 | 是
app_sign | String | 加密签名 | 算法: md5(app\_id+timestamp+app\_secret)，32位16进制格式,不区分大小写 | b927899dda6f9a04afc57f21ddf69d69 | 是
channel| String | 渠道类型 | 根据不同场景选择不同的支付方式 | WX\_NATIVE、WX_SCAN、ALI\_OFFLINE\_QRCODE、ALI_SCAN、SCAN、BC\_ALI\_SCAN,BC\_WX\_SCAN(详见附注）| 是
total_fee | Integer | 订单总金额 | 必须是正整数，单位为分 | 1 | 是
bill_no | String | 商户订单号 | 8到32位数字和/或字母组合，请自行确保在商户系统中唯一，同一订单号不可重复提交，否则会造成订单重复 | 201506101035040000001 | 是
title| String | 订单标题 | UTF8编码格式，32个字节内，最长支持16个汉字 | 白开水 | 是
auth_code | String | 用户授权码| 当商户用扫码枪扫用户的条形码时得到的字符串 | 23891113455872 | 当channel参数为BC\_ALI\_SCAN,BC\_WX\_SCAN,WX_SCAN或ALI_SCAN 时为必填
notify_url | String | 商户自定义回调地址 | 商户可通过此参数设定回调地址，此地址会覆盖用户在控制台设置的回调地址。**<mark>必须以`http://`或`https://`开头</mark>** | http://beecloud.cn/notifyUrl.jsp
optional | Map | 附加数据 | 用户自定义的参数，将会在Webhook通知中原样返回，该字段主要用于商户携带订单的自定义数据 | {"key1":"value1","key2":"value2",...} | 否
analysis | Map | 分析数据 | 用于统计分析的数据，将会在控制台的统计分析报表中展示，**<mark>用户自愿上传</mark>** | 包括以下基本字段：`os_name(系统名称，如"iOS"，"Android")` `os_version(系统版本，如"5.1")` `model(手机型号，如"iPhone 6")` `app_name(应用名称)` `app_version(应用版本号)` `device_id(设备ID)` `category(类别，用户可自定义，如游戏分发渠道，门店ID等)` `browser_name(浏览器名称)` `browser_version(浏览器版本)` | 否
store_id | string | 门店号| 门店号 | 卡门店号 | 选填
coupon\_id | string | 卡券id | 营销活动时使用 | 卡券id | 选填

> 注1：channel的参数值含义：  
WX\_NATIVE: 微信二维码支付   
ALI\_OFFLINE\_QRCODE: 支付宝二维码支付  
WX\_SCAN: 微信条形码支付  
ALI\_SCAN: 支付宝条形码支付  
SCAN: 支付宝、微信统一条形码支付
BC\_ALI\_SCAN: BC版支付宝条形码支付  
BC\_WX\_SCAN: BC版支付宝条形码支付


- 以下是支付宝条码(ALI_SCAN)的选填参数：

参数名 | 类型 | 含义 | 描述 | 例子 | 是否必填
----  | ---- | ---- | ---- | ---- | ---
terminal_id | string | 机具终端编号 | 商户机具终端编号 最长32位 | NJ\_T\_001 | 若机具商接入，terminal\_id(机具终端编号)必填，store\_id(商户门店编号)选填
store\_id | string | 商户门店编号 | 商户门店编号 最长32位 | NJ\_001 | 若系统商接入，store\_id（商户的门店编号）必填，terminal\_id(机具终端编号)选填

#### 返回类型: *JSON: Map*
#### 返回参数:

- **公共返回参数**

参数名 | 类型 | 含义 
---- | ---- | ----
result_code | Integer | 返回码，0为正常
result_msg  | String | 返回信息， OK为正常
err_detail  | String | 具体错误信息

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

- ALI\_SCAN | WX\_SCAN | BC\_ALI\_SCAN|BC\_WX\_SCAN

参数名 | 类型 | 含义 
---- | ---- | ----
pay_result | boolean | 是否支付成功，false代表在等待用户输入密码，需查询

- WX\_NATIVE | ALI\_OFFLINE_QRCODE

参数名 | 类型 | 含义 
---- | ---- | ----
code_url | String | 二维码地址

- SCAN

参数名 | 类型 | 含义 
---- | ---- | ----
channel_type | String | 具体条形码支付类型，ALI\_SCAN或者WX\_SCAN

</br>

### 3. 订单状态查询

#### URL:   */2/rest/offline/bill/status*
#### Method: POST

#### 请求参数类型:JSON

#### 请求参数详情:

参数名 | 类型 | 含义 | 描述 | 例子 | 是否必填
----  | ---- | ---- | ---- | ---- | ----
app_id | String | BeeCloud应用APPID | BeeCloud的唯一标识 | 0950c062-5e41-44e3-8f52-f89d8cf2b6eb | 是
timestamp | Long | 签名生成时间 | 时间戳，毫秒数 | 1435890533866 | 是
app_sign | String | 加密签名 | 算法: md5(app\_id+timestamp+app_key)，32位16进制格式，不区分大小写 | b927899dda6f9a04afc57f21ddf69d69 | 是
bill_no | String | 订单号 | 要查询的订单号 | -|是
channel| String | 渠道类型 | 根据不同场景选择不同的支付方式 | WX\_NATIVE、WX\_SCAN、ALI\_OFFLINE\_QRCODE、ALI\_SCAN、BC\_ALI\_SCAN、BC\_WX\_SCAN(详见支付附注） | 否

#### 返回类型: *JSON, Map*
#### 返回详情:

- 公共返回参数

参数名 | 类型 | 含义 
---- | ---- | ----
result\_code | Integer | 返回码，0为正常
result\_msg  | String | 返回信息， OK为正常
err\_detail  | String | 具体错误信息
pay_result | Bool | 订单是否成功

> 公共返回参数取值及含义参见支付公共返回参数部分

### 4. 撤销订单

#### URL:   */2/rest/offline/bill/{bill_no}*
#### Method: POST JSON
#### bill_no : 订单号

#### 请求参数类型:JSON

#### 请求参数详情:

参数名 | 类型 | 含义 | 描述 | 例子 | 是否必填
----  | ---- | ---- | ---- | ---- | ----
app_id | String | BeeCloud应用APPID | BeeCloud的唯一标识 | 0950c062-5e41-44e3-8f52-f89d8cf2b6eb | 是
timestamp | Long | 签名生成时间 | 时间戳，毫秒数 | 1435890533866 | 是
app_sign | String | 加密签名 | 算法: md5(app\_id+timestamp+app\_secret)，32位16进制格式，不区分大小写 | b927899dda6f9a04afc57f21ddf69d69 | 是
channel| String | 渠道类型 | 根据不同场景选择不同的支付方式 |WX_SCAN、ALI\_OFFLINE\_QRCODE、ALI_SCAN(详见支付附注） | 否
method | String | 订单修改的方法 | REVERT代表取消订单 | REVERT | 是

#### 返回类型: *JSON, Map*
#### 返回详情:

- 公共返回参数

参数名 | 类型 | 含义 
---- | ---- | ----
result\_code | Integer | 返回码，0为正常
result\_msg  | String | 返回信息， OK为正常
err\_detail  | String | 具体错误信息
revert_status | bool | 订单是否取消	

> 公共返回参数取值及含义参见支付公共返回参数部分


### 5. 订单查询

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
channel| String | 渠道类型 | 根据不同场景选择不同的支付方式 | WX、WX\_APP、WX\_NATIVE、WX\_JSAPI、ALI、ALI\_APP、ALI\_WEB、ALI\_QRCODE、ALI\_WAP、UN、UN\_APP、UN\_WEB、PAYPAL、PAYPAL\_SANDBOX、PAYPAL\_LIVE、JD\_WAP、JD\_WEB、YEE\_WAP、YEE\_WEB、KUAIQIAN\_WAP、KUAIQIAN\_WEB、JD、YEE、KUAIQIAN、BD、BD\_APP、BD\_WEB、BD\_WAP、BC\_ALI\_SCAN,BC\_WX\_SCAN(详见附注）| 否
bill_no | String | 商户订单号 | 发起支付时填写的订单号 | 201506101035040000001 | 否
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

- 公共返回参数

参数名 | 类型 | 含义 
---- | ---- | ----
result\_code | Integer| 返回码，0为正常
result\_msg  | String | 返回信息， OK为正常
err\_detail  | String | 具体错误信息
bills | List<Map> | 订单列表

> 公共返回参数取值及含义参见支付公共返回参数部分  

- bills说明，每个Map的key\-value

参数名         | 类型          | 含义 
----          | ----         | ----
id      | String       | 订单记录的唯一标识，可用于查询单笔记录
bill\_no      | String       | 订单号
bill\_fee | Integer         | 订单金额，单位为分
total\_fee    | Integer         | 实付金额，单位为分
discount | Integer         | 优惠券金额，单位为分
coupon\_id | String | 卡券ID，没有用到返回null
trade\_no    | String         | 渠道交易号， 当支付成功时有值
channel       | String       | 渠道类型 WX、ALI、UN、JD、YEE、KUAIQIAN、PAYPAL、BD
sub_channel         | String       | 子渠道类型 WX\_APP、WX\_NATIVE、WX\_JSAPI、WX\_SCAN、ALI\_APP、ALI\_SCAN、ALI\_WEB、ALI\_QRCODE、ALI\_OFFLINE\_QRCODE、ALI\_WAP、UN\_APP、UN\_WEB、PAYPAL\_SANDBOX、PAYPAL\_LIVE、JD\_WAP、JD\_WEB、YEE\_WAP、YEE\_WEB、YEE\_NOBANKCARD、KUAIQIAN\_WAP、KUAIQIAN\_WEB、BD\_APP、BD\_WEB、BD\_WAP
title         | String       | 订单标题
spay\_result  | Bool         | 订单是否成功
create\_time | Long         | 订单创建时间, 毫秒时间戳, 13位
optional | String | 附加数据,用户自定义的参数，将会在webhook通知中原样返回，该字段是JSON格式的字符串 {"key1":"value1","key2":"value2",...}
message_detail | String         | 渠道详细信息， 当need\_detail传入true时返回
revert_result  | Bool         | 订单是否已经撤销
refund_result  | Bool         | 订单是否已经退款


### 6.  退款

退款接口仅支持对已经支付成功的订单进行退款，且目前对于同一笔订单，仅能退款成功一次（对于同一个退款请求，如果第一次退款申请被驳回，仍可以进行二次退款申请）。 退款金额refund\_fee必须小于或者等于原始支付订单的total\_fee，如果是小于，则表示部分退款.


#### URL: */2/rest/offline/refund*
#### Method: *POST*

#### 请求参数格式: *JSON: Map*
#### 请求参数详情:

- 参数列表

参数名 | 类型 | 含义   | 描述 | 例子 | 是否必填 |
---- | ---- | ---- | ---- | ---- | ----
app_id | String | BeeCloud应用APPID | BeeCloud的唯一标识 | 0950c062\-5e41\-44e3\-8f52\-f89d8cf2b6eb | 是 
timestamp | Long | 签名生成时间 | 时间戳，毫秒数 | 1435890533866 | 是
app_sign | String | 加密签名 | 算法: md5(app\_id+timestamp+master\_secret)，32位16进制格式,不区分大小写 | b927899dda6f9a04afc57f21ddf69d69 | 是
channel| String | 渠道类型 | 根据不同渠道选不同的值 | 暂时仅支持ALI、WX| 否
refund_no | String | 商户退款单号 | 格式为:退款日期(8位) + 流水号(3~24 位)。请自行确保在商户系统中唯一，且退款日期必须是发起退款的当天日期,同一退款单号不可重复提交，否则会造成退款单重复。流水号可以接受数字或英文字符，建议使用数字，但不可接受“000” | 201506101035040000001 | 是
bill_no | String | 商户订单号 | 发起支付时填写的订单号 | 201506101035040000001 | 是 
refund_fee | Integer | 退款金额 | 必须为正整数，单位为分，必须小于或等于对应的已支付订单的total_fee | 1 | 是
refund_reason | String | 退款的原因说明 | 正常退款 | 否
optional | Map | 附加数据 | 用户自定义的参数，将会在webhook通知中原样返回，该字段主要用于商户携带订单的自定义数据 | {"key1":"value1","key2":"value2",...} | 否
operator_id | String | 商户的操作员编号 | OP001 | 否
store_id | String | 商户的门店编号 | NJ_S_001 | 否
refund_account | Integer | 退款资金来源 | 1:可用余额退款 0:未结算资金退款（默认使用未结算资金退款） | 1 | 否

#### 返回类型: *JSON: Map*
#### 返回参数:

- 公共返回参数

参数名 | 类型 | 含义 
---- | ---- | ----
result\_code | Integer | 返回码，0为正常
result\_msg  | String | 返回信息，OK为正常
err\_detail  | String | 具体错误信息
id  | String | 成功发起退款后返回退款表记录唯一标识
refund_result | Bool | 退款是否成功

> 公共返回参数取值及含义参见支付公共返回参数部分, 以下是退款所特有的

result\_code | result\_msg                | 含义
----        | ----      			       | ----
8           | NO\_SUCH_BILL             | 没有该订单
9           | BILL\_UNSUCCESS            | 该订单没有支付成功
10          | REFUND\_EXCEED\_TIME       | 已超过可退款时间
11          | ALREADY\_REFUNDING         | 该订单已有正在处理中的退款
12          | REFUND\_AMOUNT\_TOO\_LARGE | 提交的退款金额超出可退额度
13          | NO\_SUCH\_REFUND           | 没有该退款记录



## Part2. 应用场景简介

### 1.条形码支付
####第一步： 商户（例如超市）中， 用户结账时使用支付宝或者微信的条形码支付，向收银员展示条形码；  
####第二步： 商户收银员使用扫码枪扫描用户的条形码获得授权码auth_code，并连同订单信息 使用下单接口；  
####第三步： 下单接口可能有三种返回状态：  
1. resul\_code != 0, 代表支付发起失败，请参考err_detail字段  
2. resul\_code == 0 && pay\_result = True （对应用户的此次支付无需输入密码情况）代表支付成功。 **<mark>可以结束此次支付**</mark>  
3. resul\_code == 0 && pay\_result = False （对应用户需要输入密码确认，代表用户还在支付过程。） **<mark>需进行第四步</mark>**  
   
####第四步： 如果下单返回是3状态， 则需要循环调用订单状态查询接口， 直至返回结果是支付成功则**<mark>可以结束此次支付**</mark>；或者超过时间或者重试次数 调用 撤销订单接口


### 2.二维码支付
####第一步： 商户（例如超市）中， 用户结账时使用支付宝或者微信的二维码支付；  
####第二步： 商户收银员整理订单信息 使用下单接口；
####第三步： 利用下单接口返回的 **<mark>qr_code</mark>** 生成二维码图片供用户扫描支付
####第四步： 循环调用订单状态查询接口直至返回支付成功结果**<mark>可以结束此次支付**； 或者超时，超出重试次数， 调用 撤销订单接口
