# BeeCloud 企业打款 REST API

## 简介

1. 场景  
商户需要将资金从自己的账户/虚拟账户 转入他人的账户/虚拟账户，例如：  
代理商返点；用户提现；用户返点
2. 目前支持渠道及产品 （**持续增加中**）
 - BeeCloud企业打款  
 	打款到个人或者企业银行账户
 	
 - 微信企业打款(公众号)  
 	打款到个人微信零钱
 	
 - 微信红包  
   发红包给个人微信
   
 - 支付宝单笔打款
   
 - 批量（批量目前只支持支付宝）

## 使用前准备

1. 注册成为BeeCloud用户并通过企业认证
2. 选择您需要的支付渠道及产品， 自助开通或**通过BeeCloud代申请服务**
3. [帮助中心](https://help.beecloud.cn/) &ensp;&ensp; [错误码对照表](https://github.com/beecloud/beecloud-rest-api/tree/master/error_code)

## API请求地址
### https://api.beecloud.cn

## BeeCloud企业打款 - 打款到银行卡

### URL: */2/rest/bc_transfer/*
### METHOD: *POST*
### Content-type: *application/json*
### 请求参数详情

参数名 | 类型 | 含义 | 描述 | 例子 | 是否必填
----  | ---- | ---- | ---- | ---- | ----
app_id | String | BeeCloud平台的AppID | App在BeeCloud平台的唯一标识 | 0950c062-5e41-44e3-8f52-f89d8cf2b6eb | 是
timestamp | Long | 签名生成时间 | 时间戳，毫秒数 | 1435890533866 | 是
app_sign | String | 加密签名 | 算法: md5(app\_id + timestamp + **master\_secret**)，32位16进制格式,不区分大小写 | b927899dda6f9a04afc57f21ddf69d69 | 是
total_fee | Integer | 打款订单总金额 | 必须是正整数，单位为分 | 1 | 是
bill_no | String | 商户订单号 | 8到32位数字和/或字母组合，请自行确保在商户系统中唯一，同一订单号不可重复提交，否则会造成订单重复 | 201506101035040000001 | 是
title| String | 打款订单标题 | UTF8编码格式，32个字节内，最长支持16个汉字 | 白开水 | 是
trade_source | String | 交易源| UTF8编码格式，目前只能填写OUT_PC | OUT_PC | 是
bank\_fullname | String | 银行全名 | 银行全称 | 中国银行，而不能写成"中行",因为“中行”也是中信银行和中兴银行的缩写 | 是
card_type|String | 银行卡类型 | 区分借记卡和信用卡 | DE代表借记卡，CR代表信用卡，其他值为非法 | 是
account_type|String | 收款帐户类型 | 区分对公和对私 | 帐户类型，P代表私户，C代表公户，其他值为非法 | 是
account_no|String | 收款帐户号 | 收款方的银行卡号 | 6222691921993848888 | 是
account_name|String | 收款帐户名称 | 收款方的姓名或者单位名 | 黄晓明 | 是
mobile | String | 银行绑定的手机号 | 银行绑定的手机号，当需要手机收到银行入账信息时，该值必填，前提是该手机在银行有短信通知业务，否则收不到银行信息 | 13888888888 | 否
optional | Map | 附加数据 | 用户自定义的参数，将会在Webhook通知中原样返回，该字段主要用于商户携带订单的自定义数据 | {"key1":"value1","key2":"value2",...} | 否

### 返回结果 (JSON, Map)

参数名 | 类型 | 含义 
---- | ---- | ----
result_code | Integer | 返回码，0为正常
result_msg  | String | 返回信息， OK为正常
err_detail  | String | 具体错误信息
id| String| 打款记录唯一标识

注1: 错误码（错误详细信息 参考 **err_detail** 字段)

result_code | result_msg             | 含义
----           | ----     	        | ----
0              | OK                     | 调用成功
1              | APP\_INVALID           | 根据app\_id找不到对应的APP或者app\_sign不正确
4              | MISS\_PARAM            | 缺少必填参数
5              | PARAM\_INVALID         | 参数不合法
14             | RUNTIME\_ERROR         | 运行时错误
15             | NETWORK\_ERROR         | 网络异常错误

### BeeCloud企业打款 - 支持银行

### URL: */2/rest/bc_transfer/banks*
### METHOD: *GET*

#### 请求参数类型: *JSON, 以para=**{}**的方式请求*

示例: para={"key\_a":"value\_a"}, 需要对para=后面的部分做URL encode.

参数名 | 类型 | 含义 | 描述 | 例子 | 是否必填
----  | ---- | ---- | ---- | ---- | ----
type | String | 业务类型 | 业务类型 | P\_DE:对私借记卡,P\_CR:对私信用卡,C:对公账户 | 是

### 返回结果 (JSON, Map)

参数名 | 类型 | 含义 
---- | ---- | ----
result_code | Integer | 返回码，0为正常
result_msg  | String | 返回信息， OK为正常
err_detail  | String | 具体错误信息
size| Integer | 支持的银行个数
bank_list | list| list节点值为支持银行的全称

注1: 错误码（错误详细信息 参考 **err_detail** 字段)

result_code | result_msg             | 含义
----           | ----     	        | ----
0              | OK                     | 调用成功
4              | MISS\_PARAM            | 缺少必填参数
5              | PARAM\_INVALID         | 参数不合法

## 微信企业打款/微信红包/支付宝单笔打款

### URL: */2/rest/transfer/*
### METHOD: *POST*
### Content-type: *application/json*
### 请求参数详情

参数名 | 类型 | 含义 | 描述 | 例子 | 是否必填
----  | ---- | ---- | ---- | ---- | ----
app_id | String | BeeCloud平台的AppID | App在BeeCloud平台的唯一标识 | 0950c062-5e41-44e3-8f52-f89d8cf2b6eb | 是
timestamp | Long | 签名生成时间 | 时间戳，毫秒数 | 1435890533866 | 是
app_sign | String | 加密签名 | 算法: md5(app\_id+timestamp+**master\_secret**)，32位16进制格式,不区分大小写 | b927899dda6f9a04afc57f21ddf69d69 | 是
channel| String | 渠道类型 | 根据不同场景选择不同的支付方式 | WX_REDPACK, WX\_TRANSFER, ALI\_TRANSFER(详见附注）| 是
transfer_no | String | 打款单号 | 支付宝为11-32位数字字母组合， 微信企业打款为8-32位数字字母组合，微信红包为10位数字 | udjfiienx2334/8372839123 | 是
total_fee | Int | 打款金额 | 此次打款的金额,单位分,正整数(微信红包1.00-200元，微信打款>=1元) | 100 | 是
desc | String | 打款说明 | 此次打款的说明 | 赔偿 | 是
channel_user\_id | String | 用户id | 支付渠道方内收款人的标示, 微信为openid, 支付宝为支付宝账户 | someone@126.com |是
channel_user\_name | String | 用户名| 支付渠道内收款人账户名， 支付宝必填 | 支付宝某人 | 否
redpack_info | Map | 红包信息 | 微信红包的详细描述，详见附注, 微信红包必填 | - | 否
account_name|String|打款方账号名称|打款方账号名全称，支付宝必填|苏州比可网络科技有限公司|否

注1: channel的参数值含义      
WX\_REDPACK: 微信红包  
WX\_TRANSFER: 微信企业打款  
ALI\_TRANSFER: 支付宝企业打款 

注2: redpack_info 参数列表

参数名 | 类型 | 含义 | 例子
---- | ---- | ---- | ----
send_name| String | 红包发送者名称 32位 | BeeCloud
wishing | String | 红包祝福语 128 位| BeeCloud祝福开发者工作顺利!
act_name | String | 红包活动名称 32位 | BeeCloud开发者红包轰动

### 返回结果 (JSON, Map)
参数名 | 类型 | 含义 
---- | ---- | ----
result_code | Integer | 返回码，0为正常
result_msg  | String | 返回信息， OK为正常
err_detail  | String | 具体错误信息
url | String | 支付宝需要跳转到支付宝链接输入支付密码确认

注1: 错误码（错误详细信息 参考 **err_detail**字段)


## 批量打款 - 支付宝

### URL: */2/rest/transfers/*
### METHOD: *POST*
### Content-type: *application/json*
### 请求参数详情

参数名 | 类型 | 含义 | 描述 | 例子 | 是否必填
----  | ---- | ---- | ---- | ---- | ----
app_id | String | BeeCloud平台的AppID | App在BeeCloud平台的唯一标识 | 0950c062-5e41-44e3-8f52-f89d8cf2b6eb | 是
timestamp | Long | 签名生成时间 | 时间戳，毫秒数 | 1435890533866 | 是
app_sign | String | 加密签名 | 算法: md5(app\_id+timestamp+**master\_secret**)，32位16进制格式,不区分大小写 | b927899dda6f9a04afc57f21ddf69d69 | 是
channel| String | 渠道类型 | 根据不同场景选择不同的支付方式 | 目前只支持ALI| 是
batch_no | String | 打款单号 | 支付宝为11-32位数字字母组合 | 2015110525023331aa | 是
account_name | String | 付款方名称| 付款账号账户全称 | 苏州xx科技有限公司 | 是
transfer_data | List<Map> | 付款详细 | 包含每一笔的具体信息(参见注释) |  -- | 是

注2: 支付宝transfer_data每一个Map的参数列表

参数名 | 类型 | 含义 | 例子
 ---- | ---- | ---- | ----
transfer_id| String | 打款流水号 | 0315006
receiver_account | String | 收款方账户| someone@126.com
receiver_name | String | 收款方账号姓名 | 支付宝某人
transfer_fee | Int | 打款金额，单位为分 | 10
transfer_note | String | 打款备注 | 赔款

### 返回结果 (JSON, Map)

参数名 | 类型 | 含义 
---- | ---- | ----
result_code | Integer | 返回码，0为正常
result_msg  | String | 返回信息， OK为正常
err_detail  | String | 具体错误信息
url | String | 支付宝需要跳转到支付宝链接输入支付密码确认

result_code | result_msg          | 含义
----        | ----                   | ----
0           | OK                     | 微信企业打款成功 / 微信红包发送成功。支付宝返回result\_code 0 需要跳转到url 地址的页面去输入支付宝支付密码
1           | APP\_INVALID           | 根据app\_id找不到对应的APP或者app\_sign不正确
2           | PAY\_FACTOR\_NOT\_SET  | 支付要素在后台没有设置
3           | CHANNEL\_INVALID       | channel参数不合法
4           | MISS\_PARAM            | 缺少必填参数
5           | PARAM\_INVALID         | 参数不合法
6           | CERT\_FILE\_ERROR      | 证书错误
7           | CHANNEL\_ERROR         | 渠道内部错误
14          | RUNTIME\_ERROR         | 运行时错误
 
