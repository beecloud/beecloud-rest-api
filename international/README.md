## BeeCloud RESTful API
## 境外支付 

## 1. API请求地址
#### https://api.beecloud.cn

## 2. 支付

此接口为支付流程的第一步，主要功能在于生成订单，获取必要的参数信息，来进行下一步的支付流程. 对于不同的渠道和支付方式，接口的返回值与后续的操作（例如微信App支付需要调用微信支付SDK的接口，支付宝网页支付需要跳转到获取的一段HTML网址等）都不尽相同，请根据每一个channel的详细描述分别处理.

#### URL:   */1/rest/international/bill*
#### Method: *POST*
#### 请求参数格式: *JSON: Map*

#### 请求参数详情:
- 以下为公共参数：

参数名 | 类型 | 含义 | 描述 | 例子 | 是否必填
----  | ---- | ---- | ---- | ---- | ----
app_id | String | BeeCloud平台的AppID | App在BeeCloud平台的唯一标识 | 0950c062-5e41-44e3-8f52-f89d8cf2b6eb | 是
timestamp | Long | 签名生成时间 | 时间戳，毫秒数 | 1435890533866 | 是
app_sign | String | 加密签名 | 算法: md5(app\_id+timestamp+app\_secret)，32位16进制格式,不区分大小写 | b927899dda6f9a04afc57f21ddf69d69 | 是
channel| String | 渠道类型 | 根据不同场景选择不同的支付方式 | PAYPAL_PAYPAL, PAYPAL_CREDITCARD, PAYPAL_SAVED_CREDITCARD(详见附注）| 是
total_fee | Integer | 订单总金额 | 单位分 | 1 | 是
currency | String | 三位货币种类代码 | 见附录 | USD | 是
bill_no | String | 商户订单号 | 8到32位数字和/或字母组合，请自行确保在商户系统中唯一，同一订单号不可重复提交，否则会造成订单重复 | 201506101035040000001 | 是
title| String | 订单标题 | UTF8编码格式，32个字节内，最长支持16个汉字 | 白开水 | 是
credit\_card_info | Map | 信用卡信息 | 信用卡信息 | {"card\_number":"420243123344","expire\_month":07,"expire\_year":2020,"cvv":"204","first\_name:"jim","last\_name":"Green", "card\_type":"visa"} | 当channel 为PAYPAL_CREDITCARD必填
credit\_card_id| String | 信用卡id | 当使用PAYPAL_CREDITCARD支付完成后会返回一个credit\_card_id | CARD\_ADMJ324234DJLKJS | 当channel为PAYPAL_SAVED_CREDITCARD时为必填
return_url | String | 同步返回页面| 支付渠道处理完请求后,当前页面自动跳转到商户网站里指定页面的http路径不包含?及& | http://baidu.com | 当channel参数为PAYPAL_PAYPAL时为必填



- 以下是`credit_card_info`的参数

参数名 | 类型 | 含义 | 例子
---- | ---- | ---- | ----
card\_number| String | 卡号 | 420243123344
expire\_month| int | 过期时间中的月 | 12
expire\_year| int | 过期时间中的年 | 2020
cvv| int | 信用卡的三位cvv码 | 123
first\_name | String | 用户名字 | Jim
last\_name | String | 用户的姓 | Green
card\_type | String | 卡类别 visa/mastercard/discover/amex | visa

- 以下是`currency`参数 的对照表

名称 | 缩写 
---- | ---- | 
Australian dollar|	AUD  
Brazilian real**	|BRL  
Canadian dollar|	CAD  
Czech koruna|	CZK    
Danish krone|	DKK  
Euro|	EUR  
Hong Kong dollar|	HKD  
Hungarian forint|	HUF  
Israeli new shekel|	ILS  
Japanese yen|	JPY  
Malaysian ringgit|	MYR  
Mexican peso|	MXN  
New Taiwan dollar|	TWD  
New Zealand dollar|	NZD  
Norwegian krone|	NOK  
Philippine peso|	PHP  
Polish złoty|	PLN  
Pound sterling|	GBP  
Singapore dollar|	SGD  
Swedish krona	|SEK  
Swiss franc|	CHF  
Thai baht	|THB  
Turkish lira|	TRY  
United States dollar|	USD  

- 以下是`channel`参数 的对照表

channel | 含义 
---- | ---- | 
PAYPAL_PAYPAL|	跳转到paypal使用paypal内支付  
PAYPAL_CREDITCARD	|直接使用信用卡支付（paypal渠道） 
PAYPAL_SAVED_CREDITCARD|	使用存储的信用卡id支付（信用卡信息存储在PAYPAL)  

- return_url说明  
   当channel为PAYPAL\_PAYPAL时， 用户支付完成后页面会跳转到return_url， 并带上以下参数(应只做展示用)

参数 | 含义 
---- | ---- | 
bill_no|	下单时传输的订单号  
result	|  订单状态 approved/cancle..，只有approved为支付成功
  

#### 返回类型: *JSON: Map*
#### 返回参数:

- **公共返回参数**

参数名 | 类型 | 含义 
---- | ---- | ----
result_code | Integer | 返回码，0为正常
result_msg  | String | 返回信息， OK为正常
err_detail  | String | 具体错误信息
url |String| 当channel 为PAYPAL_PAYPAL时返回，跳转支付的url
credit_card_id | String| 当channel为PAYPAL_CREDITCARD时返回， 信用卡id
id| String| 订单id

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
14          | RUNTIME\_ERROR          | 运行时错误


## 3. 退款 （开发中）

退款接口仅支持对已经支付成功的订单经行退款，且目前对于同一笔订单，仅能退款成功一次（对于同一个退款请求，如果第一次退款申请被驳回，仍可以进行二次退款申请）. 退款金额refund\_fee必须小于或者等于原始支付订单的total\_fee，如果是小于，则表示部分退款.

#### URL: */1/rest/international/refund*
#### Method: *POST*

#### 请求参数格式: *JSON: Map*
#### 请求参数详情:

- 参数列表

参数名 | 类型 | 含义   | 描述 | 例子 | 是否必填 |
---- | ---- | ---- | ---- | ---- | ----
app_id | String | BeeCloud应用APPID | BeeCloud的唯一标识 | 0950c062\-5e41\-44e3\-8f52\-f89d8cf2b6eb | 是 
timestamp | Long | 签名生成时间 | 时间戳，毫秒数 | 1435890533866 | 是
app_sign | String | 加密签名 | 算法: md5(app\_id+timestamp+app\_secret)，32位16进制格式,不区分大小写 | b927899dda6f9a04afc57f21ddf69d69 | 是
channel| String | 渠道类型 | 根据不同渠道选不同的值 | WX ALI UN | 否
refund_no | String | 商户退款单号 | 格式为:退款日期(8位) + 流水号(3~24 位)。请自行确保在商户系统中唯一，且退款日期必须是发起退款的当天日期,同一退款单号不可重复提交，否则会造成退款单重复。流水号可以接受数字或英文字符，建议使用数字，但不可接受“000” | 201506101035040000001 | 是
bill_no | String | 商户订单号 | 发起支付时填写的订单号 | 201506101035040000001 | 是 
refund_fee | Integer | 退款金额 | 必须为正整数，单位为分，必须小于或等于对应的已支付订单的total_fee | 1 | 是
optional | Map | 附加数据 | 用户自定义的参数，将会在webhook通知中原样返回，该字段主要用于商户携带订单的自定义数据 | {"key1":"value1","key2":"value2",...} | 否

#### 返回类型: *JSON: Map*
#### 返回参数:

- 公共返回参数

参数名 | 类型 | 含义 
---- | ---- | ----
result\_code | Integer | 返回码，0为正常
result\_msg  | String | 返回信息，OK为正常
err\_detail  | String | 具体错误信息

- 公共返回参数取值及含义参见支付公共返回参数部分, 以下是退款所特有的

result\_code | result\_msg                | 含义
----        | ----      			       | ----
8           | NO\_SUCH_BILL             | 没有该订单
9           | BILL\_UNSUCCESS            | 该订单没有支付成功
10          | REFUND\_EXCEED\_TIME       | 已超过可退款时间
11          | ALREADY\_REFUNDING         | 该订单已有正在处理中的退款
12          | REFUND\_AMOUNT\_TOO\_LARGE | 提交的退款金额超出可退额度
13          | NO\_SUCH\_REFUND           | 没有该退款记录

**当channel为`ALI_APP`、`ALI_WEB`、`ALI_QRCODE`时，以下字段在result_code为0时有返回**
 
参数名 | 类型 | 含义 
---- | ---- | ----
url | String | 支付宝退款地址，需用户在支付宝平台上手动输入支付密码处理


</br>
## 4. 订单查询

#### URL:   */1/rest/bills*
#### Method: *GET*

#### 请求参数类型: *JSON, 以para=**{}**的方式请求*

示例: para={"key\_a":1,"key\_b":"value\_b"}, 需要对para=后面的部分做URL encode.

#### 请求参数详情:
参数名 | 类型 | 含义 | 描述 | 例子 | 是否必填
----  | ---- | ---- | ---- | ---- | ----
app_id | String | BeeCloud应用APPID | BeeCloud的唯一标识 | 0950c062-5e41-44e3-8f52-f89d8cf2b6eb | 是
timestamp | Long | 签名生成时间 | 时间戳，毫秒数 | 1435890533866 | 是
app_sign | String | 加密签名 | 算法: md5(app\_id+timestamp+app\_secret)，32位16进制格式,不区分大小写 | b927899dda6f9a04afc57f21ddf69d69 | 是
channel| String | 渠道类型 | 根据不同场景选择不同的支付方式 | WX、WX\_APP、WX\_NATIVE、WX\_JSAPI、ALI、ALI\_APP、ALI\_WEB、ALI\_QRCODE、ALI\_OFFLINE\_QRCODE、ALI_WAP、UN、UN\_APP、UN\_WEB、PAYPAL、JD_WAP、JD_WEB、YEE_WAP、YEE_WEB、KUAIQIAN_WAP、KUAIQIAN_WEB、JD、YEE、KUAIQIAN(详见附注）| 否
bill_no | String | 商户订单号 | 发起支付时填写的订单号 | 201506101035040000001 | 否
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
count | Integer | 查询订单结果数量
bills | List<Map> | 订单列表

> 公共返回参数取值及含义参见支付公共返回参数部分  

- bills说明，每个Map的key\-value

参数名         | 类型          | 含义 
----          | ----         | ----
bill\_no      | String       | 订单号
total\_fee    | Integer         | 订单金额，单位为分
channel       | String       | WX\_NATIVE、WX\_JSAPI、WX\_APP、ALI\_APP、ALI\_WEB、ALI\_QRCODE、ALI\_OFFLINE_QRCODE、ALI_WAP、UN\_APP、UN\_WEB、PAYPAL、JD_WAP、JD_WEB、YEE_WAP、YEE_WEB、KUAIQIAN_WAP、KUAIQIAN_WEB(详见 1. 支付 附注）
title         | String       | 订单标题
spay\_result  | Bool         | 订单是否成功
created\_time | Long         | 订单创建时间, 毫秒时间戳, 13位


## 5. 退款查询

#### URL:   */1/rest/refunds*
#### Method: GET

#### 请求参数类型: JSON，以para=**{}**的方式请求

示例: para={"key\_a":1,"key\_b":"value\_b"}, 需要对para=后面的部分做URL encode.

#### 请求参数详情:

参数名 | 类型 | 含义 | 描述 | 例子 | 是否必填
----  | ---- | ---- | ---- | ---- | ----
app_id | String | BeeCloud应用APPID | BeeCloud的唯一标识 | 0950c062-5e41-44e3-8f52-f89d8cf2b6eb | 是
timestamp | Long | 签名生成时间 | 时间戳，毫秒数 | 1435890533866 | 是
app_sign | String | 加密签名 | 算法: md5(app\_id+timestamp+app\_secret)，不区分大小写 | b927899dda6f9a04afc57f21ddf69d69 | 是
channel| String | 渠道类型 | 根据不同场景选择不同的支付方式 | WX、WX\_NATIVE、WX\_JSAPI、ALI、ALI\_APP、ALI\_WEB、ALI\_QRCODE、ALI\_OFFLINE_QRCODE、ALI_WAP、UN、UN\_APP、UN\_WEB、PAYPAL、JD_WAP、JD_WEB、YEE_WAP、YEE_WEB、KUAIQIAN_WAP、KUAIQIAN_WEB、JD、YEE、KUAIQIAN(详见1.支付附注）| 否
bill_no | String | 商户订单号 | 发起支付时填写的订单号 | 201506101035040000001 | 否
refund_no | String | 商户退款单号 | 发起退款时填写的退款单号 | 201506101035040000001 | 否
start_time | Long | 起始时间 | 毫秒时间戳, 13位 | 1435890530000 | 否
end_time | Long | 结束时间 | 毫秒时间戳, 13位   | 1435890540000 | 否
skip | Integer | 查询起始位置 | 默认为0. 设置为10，表示忽略满足条件的前10条数据| 0 | 否
limit| Integer | 查询的条数 | 默认为10，最大为50. 设置为10，表示只查询满足条件的10条数据 | 10 | 否

> 注：  
1. bill\_no, refund\_no, start\_time, end\_time等查询条件互相为**<mark>且</mark>**关系.   
2. start\_time, end\_time指的是订单生成的时间，而不是订单支付的时间.   


#### 返回类型: *JSON, Map*
#### 返回详情:

- 公共返回参数

参数名 | 类型 | 含义 
---- | ---- | ----
result\_code | Integer| 返回码，0为正常
result\_msg  | String | 返回信息， OK为正常
err\_detail  | String | 具体错误信息
count | Integer | 查询退款结果数量
refunds | List<Map> | 退款列表

> 公共返回参数取值及含义参见支付公共返回参数部分

- refunds说明，每个Map的key\-value

参数名      | 类型         | 含义 
----       | ----        | ----
bill\_no    | String      | 订单号
refund\_no  | String      | 退款号
total\_fee  | Integer      | 订单金额，单位为分
refund\_fee | Integer      | 退款金额，单位为分
title         | String       | 订单标题
channel    | String      | WX\_NATIVE、WX\_JSAPI、WX\_APP、ALI\_APP、ALI\_WEB、ALI\_QRCODE、ALI\_OFFLINE_QRCODE、UN\_APP、UN\_WEB、PAYPAL、JD_WAP、JD_WEB、YEE_WAP、YEE_WEB、KUAIQIAN_WAP、KUAIQIAN_WEB(详见 1. 支付 附注）
finish     | bool        | 退款是否完成
result     | bool        | 退款是否成功
created\_time | Long       | 退款创建时间, 毫秒时间戳, 13位
