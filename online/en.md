#BeeCloud REST API

## Abstract

This document’s official Github - [https://github.com/beecloud/beecloud-rest-api](https://github.com/beecloud/beecloud-rest-api)


The BeeCloud REST API provides a shortcut to start an online Third Party Payment, such as Alipay、Wechat、UnionPay etc. With BeeCloud‘s API, you can start、refund and query your payment without suffering from coding for each Third Party Payment provider.

### Online Api Testing

We also provide an online webpage for understanding our API - [tesing our API](https://beecloud.cn/rest/#/api/bill)

## 1. API URL
#### https://api.beecloud.cn

## 2.Bill - Start payment 
Use this API to fetch parameters for staring each Third Party's payment. It veries in code（javacript api or html form submit）with each Third Party to jump to their pages. But do not worry, we also provide demos for that.

#### PATH:   */1/rest/bill*
#### HTTP Method: *POST*
#### HTTP Content-type: *application/json*

#### Common Parameters：

KEY | TYPE | MEANING | DESCRIPTION | E.G | Required
----  | ---- | ---- | ---- | ---- | ----
app_id | String | APP ID in BeeCloud | BeeCloud's distinct ID| 0950c062-5e41-44e3-8f52-f89d8cf2b6eb | YES
timestamp | Long | timestamp for signature | in milliseconds | 1435890533866 | YES
app_sign | String | signature for security | algorithm: md5(app\_id+timestamp+app\_secret)，32 hexadecimal, lowercased | b927899dda6f9a04afc57f21ddf69d69 | YES
channel| String | type of Third Party Payment | choose what you need| WX\_APP、WX\_NATIVE、WX\_JSAPI、ALI\_APP、ALI\_WEB、ALI\_QRCODE、ALI\_OFFLINE\_QRCODE、ALI\_WAP、UN\_APP、UN\_WEB、PAYPAL\_SANDBOX、PAYPAL\_LIVE、JD\_WAP、JD\_WEB、YEE\_WAP、YEE\_WEB、YEE\_NOBANKCARD、KUAIQIAN\_WAP、KUAIQIAN\_WEB、BD\_APP、BD\_WEB、BD\_WAP(See Note）| YES
total_fee | Integer | total fee of payment | must be whole number，in fen(0.01RMB) | 1 | YES
bill_no | String | the distinct ID for payment | combinations of 8-32 numbers and / or letter; Error will occur if repeated | 201506101035040000001 | YES
title| String | title for this payment | UTF8 Encoding，within 32 ASCII(within 16 Chinese character)| 白开水 | YES
optional | Map | user defined data | defined by you，and returned in [webhook](https://github.com/beecloud/beecloud-webhook) with same key-"optional" | {"key1":"value1","key2":"value2",...} | NO
return_url | String | the webpage customer will see when payment succeed | After payment being paid, webpage of the Third Party's payment will jump to this url automatically | http://beecloud.cn/returnUrl.jsp | required if channel in ALI\_WEB or ALI\_QRCODE or UN\_WEB or JD\_WAP or JD\_WEB
bill_timeout | Integer | payment's timeout time | must be whole number，in seconds，nice to be greater than 30 | 121 | NO, **<mark>JD and KUAIQIAN do not support this param</mark>** 

>NOTE: Channel description:  

>WX\_APP: start Wechat(its APP)'s payment in your APP  
WX\_NATIVE: gernerate an QR-code for wechat(its APP)'s payment  
WX\_JSAPI: start Wechat(its APP)'s payment from its webkit by using javascript  
ALI\_APP: start Alipay(its APP)'s payment in your APP    
ALI\_WEB: start Alipay's payment in your webpage for PC     
ALI\_QRCODE: gernerate an QR-code for Apipay(its APP)'s payment  
ALI\_OFFLINE_QRCODE: gernerate an QR-code for Apipay(its APP)'s payment with certain hardware  
ALI\_WAP: start Alipay's payment in your webpage for mobile webrowser   
UN\_APP: start Unionpay(its plugin)'s payment in your APP   
UN\_WEB: start Unionpay's payment in your webpage for PC     
JD\_WAP: start JingDong‘s payment in your webpage for mobile webrowser     
JD\_WEB: start JingDong‘s payment in your webpage for PC   
YEE\_WAP: start Yeepay‘s payment in your webpage for mobile webrowser    
YEE\_WEB: start Yeepay‘s payment in your webpage PC    
YEE\_NOBANKCARDB: start Yeepay‘s payment in your webpage PC with special recharge credit cards （not cards from bank）  
KUAIQIAN\_WAP: start Kuaiqian‘s payment in your webpage for mobile webrowser    
KUAIQIAN\_WEB: start Kuaiqian‘s payment in your webpage for PC 
PAYPAL\_LIVE: PAYPAL in product    
PAYPAL\_SANDBOX: PAYPAL in sandbox  
BD\_APP: start Baidu(its plugin)'s payment in your APP     
BD\_WAP: start Baidu's payment in your webpage for mobile webrowser   
BD\_WEB: start Baidu's payment in your webpage for PC 

#### Optional Params

- when use channel "WX_JSAPI", params below are **<mark>required</mark>**:

KEY | TYPE | DESCRIPTION | E.G
---- | ---- | ---- | ----
openid| String | customer id for your Wechat’s Official Accounts | 0950c062-5e41-44e3-8f52-f89d8cf2b6eb

- when use channel "ALI_WEB", you **can** use params below:

KEY | TYPE | DESCRIPTION | E.G
---- | ---- | ---- | ----
show_url| String | the link for displaying your product embed in payment-webpage| http://beecloud.cn

- when use channel "ALI_QRCODE", params below are **<mark>required</mark>**:

KEY | TYPE | DESCRIPTION | E.G
---- | ---- | ---- | ----
qr\_pay\_mode| String | QRCODE type | 0,1,3

>NOTE: QRCODE-type  
0： simple front-mode, width  of iframe  must be no less than 600px and height no less than 300px   
1： front-mode, width  of iframe  must be no less than 300px and height no less than 600px    
3： mini-mode, both width and height of iframe must be no less than 75px

- when use channel "YEE_NOBANKCARD",  params below are **<mark>required</mark>**:
- 
KEY | TYPE | DESCRIPTION | E.G
---- | ---- | ---- | ----
cardno | String | id of credit card，it's rule and length varies in differenc area of China
cardpwd | String | the password for payment
frqid | String | id of card‘s type：骏网一卡通(JUNNET),盛大卡(SNDACARD),神州行(SZX),征途卡(ZHENGTU),Q币卡(QQCARD),联通卡(UNICOM),久游卡(JIUYOU),易充卡(YICHONGCARD),网易卡(NETEASE),完美卡(WANMEI),搜狐卡(SOHU),电信卡(TELECOM),纵游一卡通(ZONGYOU),天下一卡通(TIANXIA),天宏一卡通(TIANHONG),32 一卡通(THIRTYTWOCARD)

> NOTE：Since this card can be only used once, total_fee must equal to the credit of card. Otherwise,**<mark>remaining credit will be lost</mark>**

#### Content-type of Response : *application/json*

#### Parameters of Response：

- **Common** Parameters

KEY | TYPE | DESCRIPTION
---- | ---- | ----
result_code | Integer | 0 - OK
result_msg  | String | "OK" means what it means
err_detail  | String | text of error's detail 
id  | String | if OK, the distinct ID for your payment in BeeCloud

- **result_code and result_msg you may meet**

result_code | result_msg             | Description
----        | ----                   | ----
0           | OK                     | success
1           | APP\_INVALID           | App is not created properly in BeeCloud, or app\_sign is incorrect
2           | PAY\_FACTOR_NOT\_SET   | Config data for third party payment channels not set in BeeCloud
3           | CHANNEL\_INVALID       | Channel not valid
4           | MISS\_PARAM            | Required parameter missing 
5           | PARAM\_INVALID         | Parameter not valid
6           | CERT\_FILE\_ERROR      | error of certification for third party payment channels
7           | CHANNEL\_ERROR         | Error from third party payment channels
14          | RUNTIME\_ERROR         | Runtime error

> **err\_detail would be useful when shit happens**

> when your get 0 for result_code, you can receive additional params as below


- when you use channel "WX_APP", additional params:

KEY | TYPE | DESCRIPTION
---- | ---- | ----
app_id | String | your APPID for wechat
partner_id | String | ID for wechat official account
package  | String | param for packaging wechat's payment
nonce_str  | String | random string for signature
timestamp | String | current timestamp in millisecond，should consist of 13 number
pay_sign  | String | signature
prepay_id  | String | ID for prepayment in wechat

- when you use channel "WX\_NATIVE", additional params:

KEY | TYPE | DESCRIPTION
---- | ---- | ----
code_url | String | msg, usually a link for QRCODE

- when you use channel "WX\_JSAPI", additional params:

KEY | TYPE | DESCRIPTION
---- | ---- | ----
app_id | String | your APPID for wechat
package  | String | param for packaging wechat's payment
nonce_str  | String | random string for signature
timestamp | String | current timestamp in millisecond，should consist of 13 number
pay_sign  | String | signature
sign_type  | String | signature's algorithm, should be MD5

- when you use channel "ALI\_APP", additional params:

KEY | TYPE | DESCRIPTION
---- | ---- | ----
order\_string | String | string to be signatured

- when you use channel "ALI\_WEB", additional params:

KEY | TYPE | DESCRIPTION
---- | ---- | ----
html | String | html string - submit a form to Alipay's payment gateway
url  | String | Alipay's url to payment gateway, containing data in uri

- when you use channel "ALI\_OFFLINE\_QRCODE", additional params:

KEY | TYPE | DESCRIPTION
---- | ---- | ----
qr_code | String | msg, usually a link for QRCODE

- when you use channel "ALI\_QRCODE", additional params:

KEY | TYPE | DESCRIPTION
---- | ---- | ----
html | String | html string - submit a form to Alipay's payment gateway
url  | String | a link for QRCODE

- when you use channel "UN\_APP", additional params:

KEY | TYPE | DESCRIPTION
---- | ---- | ----
tn | String | ticket number for Unionpay

- when you use channel "UN\_WEB、JD\_WAP、JD\_WEB、KUAIQIAN\_WAP、KUAIQIAN\_WEB", additional params:

KEY | TYPE | DESCRIPTION
---- | ---- | ----
html | String | html string - submit a form to channel's payment gateway

- when you use channel "YEE\_WAP、YEE\_WEB、BD\_WAP、BD\_WEB", additional params:

KEY | TYPE | DESCRIPTION
---- | ---- | ----
url | String | channel's url to payment gateway, containing data in uri

- when you use channel "BD\_APP", additional params:


KEY | TYPE | DESCRIPTION
---- | ---- | ----
orderInfo | String | order info for Baidu

<br>

## 3. Refund - Start refunding or prerefunding 

Only payment that has been paid can successfully call this API, and  each payment can be refunded only once. Furthermore, fee of refunding must within the "total_fee" of each payment.

#### PATH: */1/rest/refund*
#### HTTP Method: *POST*
#### HTTP Content-type: *application/json*

#### Common Parameters:

KEY | TYPE | MEANING | DESCRIPTION | E.G | Required
----  | ---- | ---- | ---- | ---- | ----
app_id | String | APP ID in BeeCloud | BeeCloud's distinct ID| 0950c062-5e41-44e3-8f52-f89d8cf2b6eb | YES
timestamp | Long | timestamp for signature | in milliseconds | 1435890533866 | YES
app_sign | String | signature for security | algorithm: md5(app\_id+timestamp+app\_secret)，32 hexadecimal, lowercased | b927899dda6f9a04afc57f21ddf69d69 | YES
refund_no | String | ID for refunding | format:date time(8characters) + your unique id(3~24characters). It must be unique in your refunding system, and date should be the day refunding. Characters are limited to numbers and modern English alphabet except “000” | 201506101035040000001 | YES
bill_no | String | the payment id | the bill_no used when start this payment with bill API | 201506101035040000001 | YES
refund_fee | Integer | refunding fee | should be whole number in fen(0.01RMB) and no greater than total_fee of payment | 1 | 是
channel| String | type of Third Party Payment | choose what you need | WX ALI UN KUAIQIAN BD JD YEE | NO
optional | Map | user defined data | defined by you，and returned in [webhook](https://github.com/beecloud/beecloud-webhook) with same key-"optional" | {"key1":"value1","key2":"value2",...} | NO
need_approval| Bool | flag for prerefunding | if true , means starting prerefunding instead of refunding | true | NO

#### Content-type of Response : *application/json*

#### Parameters of Response :

- **Common** Parameters

KEY | TYPE | DESCRIPTION
---- | ---- | ----
result_code | Integer | 0 - OK
result_msg  | String | "OK" means what it means
err_detail  | String | text of error's detail 
id  | String | if OK, the distinct ID for your refund/prerefund in BeeCloud

- **result_code and result_msg you may meet**

result\_code | result\_msg                | 含义
----        | ----      			       | ----
0			  | OK                        | success
8           | NO\_SUCH_BILL             | no matched payment for bill_no
9           | BILL\_UNSUCCESS            | payment is not paid
10          | REFUND\_EXCEED\_TIME       | timeout 
11          | ALREADY\_REFUNDING         | already been refuding
12          | REFUND\_AMOUNT\_TOO\_LARGE | refund_fee is greater than total_fee of this payment
13          | NO\_SUCH\_REFUND           | no matched refunding|prerefunding data

- **when you use channel `ALI_APP` or `ALI_WEB` or`ALI_QRCODE` and result_code equals to 0, additional params:**

KEY | TYPE | DESCRIPTION
---- | ---- | ----
url | String | Alipay's url to login and confirm this refunding

<br>

## 4. Approve - Approve prerefunding
Your can only approve or disapprove prerefundment created by refund API.

#### PATH:   */1/rest/approve*
#### HTTP Method: *POST*
#### HTTP Content-type: *application/json*

#### Common Parameters：

KEY | TYPE | MEANING | DESCRIPTION | E.G | Required
----  | ---- | ---- | ---- | ---- | ----
app_id | String | APP ID in BeeCloud | BeeCloud's distinct ID| 0950c062-5e41-44e3-8f52-f89d8cf2b6eb | YES
timestamp | Long | timestamp for signature | in milliseconds | 1435890533866 | YES
app_sign | String | signature for security | algorithm: md5(app\_id+timestamp+app\_secret)，32 hexadecimal, lowercased | b927899dda6f9a04afc57f21ddf69d69 | YES
ids | List<String> | an list of refund id | ids returned by refund API | ["d9690a6e-ae99-44b7-9904-bd9d43fcc21b", "6f263aa6-111d-4c95-b51e-001b3f7e6ddf", "  7d1f69e4-3ff2-4b7d-b764-eb018620e00d"] | YES
agree| Bool | agree to refund | true meaning approve, and false the opposite  | true | YES
channel| String | type of Third Party Payment | choose what you need | WX ALI UN KUAIQIAN BD JD YEE | NO
deny_reason| String | reason for disapprove | reason for disapprove | "NO REFUND" | NO

#### Content-type of Response : *application/json*

#### Parameters of Response :

- **Common** Parameters

KEY | TYPE | DESCRIPTION
---- | ---- | ----
result_code | Integer | 0 - OK
result_msg  | String | "OK" means what it means
err_detail  | String | text of error's detail 


- **when you set true for agree, additional params:**

KEY | TYPE | DESCRIPTION
---- | ---- | ----
result_map | Map&lt;String, Map&lt;String, Object&gt;&gt; | each prerefund's result (same as common parameter)

<br>

## 5. Bills - Query payment's status

#### PATH:   */1/rest/bills*
#### HTTP Method: *GET*

>NOTE: use uri format: param = JSON String, params below is in this JSON. And JSON string should be urlencoded.

#### Common Parameters：

KEY | TYPE | MEANING | DESCRIPTION | E.G | Required
----  | ---- | ---- | ---- | ---- | ----
app_id | String | APP ID in BeeCloud | BeeCloud's distinct ID| 0950c062-5e41-44e3-8f52-f89d8cf2b6eb | YES
timestamp | Long | timestamp for signature | in milliseconds | 1435890533866 | YES
app_sign | String | signature for security | algorithm: md5(app\_id+timestamp+app\_secret)，32 hexadecimal, lowercased | b927899dda6f9a04afc57f21ddf69d69 | YES
bill_no | String | the payment id | the bill_no used when start this payment with bill API | 201506101035040000001 | NO
channel| String | type of Third Party Payment | choose what you need| WX、WX\_APP、WX\_NATIVE、WX\_JSAPI、ALI、ALI\_APP、ALI\_WEB、ALI\_QRCODE、ALI\_OFFLINE\_QRCODE、ALI\_WAP、UN、UN\_APP、UN\_WEB、PAYPAL、PAYPAL\_SANDBOX、PAYPAL\_LIVE、JD、JD\_WAP、JD\_WEB、YEE、YEE\_WAP、YEE\_WEB、YEE\_NOBANKCARD、KUAIQIAN、KUAIQIAN\_WAP、KUAIQIAN\_WEB、BD、BD\_APP、BD\_WEB、BD\_WAP| NO
start_time | Long | condition start timestamp | in millseconds, 13 nubmer | 1435890530000 | no
end_time | Long | condition end timestamp  | in millseconds, 13 nubmer | 1435890540000 | NO
skip | Integer| result skip count | default to 0| 0 | NO
limit| Integer | max number of result to return  | default to 10 and within 50| 10 | NO

>NOTE:
1. bill\_no, start\_time, end\_time has **<mark>AND</mark>** effect for query 
2. start\_time, end\_time are condition for timestamp on payment's creation instead of timestamp it got paid

#### Content-type of Response : *application/json*

#### Parameters of Response :

- **Common** Parameters

KEY | TYPE | DESCRIPTION
---- | ---- | ----
result_code | Integer | 0 - OK
result_msg  | String | "OK" means what it means
err_detail  | String | text of error's detail 
count | Integer | the amount of payment matched query
bills | List<map> | the list of payment matched query

- Parameters of each payment in bills

KEY | TYPE | DESCRIPTION
bill\_no      | String       |  please see the Bill API
total\_fee    | Integer         | payment's fee in fen (0.01RMB)
channel       | String       | WX\_NATIVE、WX\_JSAPI、WX\_APP、ALI\_APP、ALI\_WEB、ALI\_QRCODE、ALI\_OFFLINE_QRCODE、ALI_WAP、UN\_APP、UN\_WEB、JD_WAP、JD_WEB、YEE_WAP、YEE_WEB、KUAIQIAN_WAP、KUAIQIAN_WEB、PAYPAL\_SANDBOX、PAYPAL\_LIVE、BD\_APP、BD\_WAP、BD\_WEB
title         | String       | 
spay\_result  | Bool         | true if paid successfully
created\_time | Long         | timestamp on payment's creation


##6. Refunds - Query refunding's status

#### PATH:   */1/rest/bills*
#### HTTP Method: *GET*

>NOTE: use uri format: param = JSON String, params below is in this JSON. And JSON string should be urlencoded.

#### Common Parameters：

KEY | TYPE | MEANING | DESCRIPTION | E.G | Required
----  | ---- | ---- | ---- | ---- | ----
app_id | String | APP ID in BeeCloud | BeeCloud's distinct ID| 0950c062-5e41-44e3-8f52-f89d8cf2b6eb | YES
timestamp | Long | timestamp for signature | in milliseconds | 1435890533866 | YES
app_sign | String | signature for security | algorithm: md5(app\_id+timestamp+app\_secret)，32 hexadecimal, lowercased | b927899dda6f9a04afc57f21ddf69d69 | YES
channel| String | type of Third Party Payment | choose what you need| WX、WX\_APP、WX\_NATIVE、WX\_JSAPI、ALI、ALI\_APP、ALI\_WEB、ALI\_QRCODE、ALI\_OFFLINE\_QRCODE、ALI\_WAP、UN、UN\_APP、UN\_WEB、PAYPAL、PAYPAL\_SANDBOX、PAYPAL\_LIVE、JD、JD\_WAP、JD\_WEB、YEE、YEE\_WAP、YEE\_WEB、YEE\_NOBANKCARD、KUAIQIAN、KUAIQIAN\_WAP、KUAIQIAN\_WEB、BD、BD\_APP、BD\_WEB、BD\_WAP| NO
bill_no | String | the payment id | the bill_no used when start this payment with bill API | 201506101035040000001 | NO
refund_no | String | ID for refunding | format:date time(8characters) + your unique id(3~24characters). It must be unique in your refunding system, and date should be the day refunding. Characters are limited to numbers and modern English alphabet except “000” | 201506101035040000001 | NO
start_time | Long | condition start timestamp | in millseconds, 13 nubmer | 1435890530000 | NO
end_time | Long | condition end timestamp  | in millseconds, 13 nubmer | 1435890540000 | NO
skip | Integer| result skip count | default to 0| 0 | NO
limit| Integer | max number of result to return  | default to 10 and within 50| 10 | NO

>NOTE:
1. bill\_no, refund\_no,start\_time, end\_time has **<mark>AND</mark>** effect for query 
2. start\_time, end\_time are condition for timestamp on payment's creation instead of timestamp it got paid

#### Content-type of Response : *application/json*

#### Parameters of Response :

- **Common** Parameters

KEY | TYPE | DESCRIPTION
---- | ---- | ----
result_code | Integer | 0 - OK
result_msg  | String | "OK" means what it means
err_detail  | String | text of error's detail 
count | Integer | the amount of refundment matched query
refunds | List<map> | the list of refundment matched query

- Parameters of each payment in refunds

KEY | TYPE | DESCRIPTION
bill\_no      | String       |  please see the Bill API
refund\_no      | String       |  please see the Refund API
total\_fee    | Integer         | payment's fee in fen (0.01RMB)
refund\_fee | Integer      | refunding's fee in fen (0.01RMB)
channel       | String       | WX\_NATIVE、WX\_JSAPI、WX\_APP、ALI\_APP、ALI\_WEB、ALI\_QRCODE、ALI\_OFFLINE_QRCODE、ALI_WAP、UN\_APP、UN\_WEB、JD_WAP、JD_WEB、YEE_WAP、YEE_WEB、KUAIQIAN_WAP、KUAIQIAN_WEB、PAYPAL\_SANDBOX、PAYPAL\_LIVE、BD\_APP、BD\_WAP、BD\_WEB
title         | String       | 
finish  | Bool         | true if refunding finished
result  | Bool         | true if refunding succeed
created\_time | Long         | timestamp on refunding's creation


##7. refund status - update refunding's status

#### PATH:   */1/rest/refund/status*
#### HTTP Method: *GET*
>NOTE: use uri format: param = JSON String, params below is in this JSON. And JSON string should be urlencoded.

#### Common Parameters：

KEY | TYPE | MEANING | DESCRIPTION | E.G | Required
----  | ---- | ---- | ---- | ---- | ----
app_id | String | APP ID in BeeCloud | BeeCloud's distinct ID| 0950c062-5e41-44e3-8f52-f89d8cf2b6eb | YES
timestamp | Long | timestamp for signature | in milliseconds | 1435890533866 | YES
app_sign | String | signature for security | algorithm: md5(app\_id+timestamp+app\_secret)，32 hexadecimal, lowercased | b927899dda6f9a04afc57f21ddf69d69 | YES
channel| String | type of Third Party Payment | choose what you need| WX、YEE、KUAIQIAN、BD | YES
refund\_no      | String       |  please see the Refund API | -- | 201506101035040000001 | YES

#### Content-type of Response : *application/json*

#### Parameters of Response :

- **Common** Parameters

KEY | TYPE | DESCRIPTION
---- | ---- | ----
result_code | Integer | 0 - OK
result_msg  | String | "OK" means what it means
err_detail  | String | text of error's detail 
refund_status | List<map> | the status of refunding


##8. transfes - excute batch number of payment from your Alipay account

#### PATH:   */1/rest/transfers*
#### HTTP Method: *POST*
#### HTTP Content-type: *application/json*

#### Common Parameters:

KEY | TYPE | MEANING | DESCRIPTION | E.G | Required
----  | ---- | ---- | ---- | ---- | ----
app_id | String | APP ID in BeeCloud | BeeCloud's distinct ID| 0950c062-5e41-44e3-8f52-f89d8cf2b6eb | YES
timestamp | Long | timestamp for signature | in milliseconds | 1435890533866 | YES
app_sign | String | signature for security | algorithm: md5(app\_id+timestamp+app\_secret)，32 hexadecimal, lowercased | b927899dda6f9a04afc57f21ddf69d69 | YES
channel| String | type of Third Party Payment | choose what you need| only "ALI" | YES
batch_no | String | id for this batch operation | in 11-32 number or English alphabet | 201506101035040000001 | YES
account_name | String | Alipay account that should pay | -- | 毛毛 | YES
transfer_data | List<Map> | batch's payment data | total number of payments should be no greater than 1000 | YES


- Parameters of each payment in tranfer_data

KEY | TYPE | DESCRIPTION
----  | ---- | ---- 
transfer_id | String | ID for each payment, number or English alphabet within 32 charactes| 1507290001a
receiver_account | String | receipion | someone@126.com
receiver_name | String | Alipay account who get paid | 某某人
transfer_fee | int | payment‘s fee in fen(0.01RMB) | 100
transfer_note | String | NOTE | 打赏


#### Content-type of Response : *application/json*

#### Parameters of Response :

- **Common** Parameters

KEY | TYPE | DESCRIPTION
---- | ---- | ----
result_code | Integer | 0 - OK
result_msg  | String | "OK" means what it means
err_detail  | String | text of error's detail 
url | String | the Alipay's url to login and confirm

## 9. Query refund by ID


#### PATH:   */1/rest/refund/{id}*
#### HTTP Method: *GET*
>NOTE: use uri format: param = JSON String, params below is in this JSON. And JSON string should be urlencoded.

>{id} was returned in refund API.

#### Common Parameters：

KEY | TYPE | MEANING | DESCRIPTION | E.G | Required
----  | ---- | ---- | ---- | ---- | ----
app_id | String | APP ID in BeeCloud | BeeCloud's distinct ID| 0950c062-5e41-44e3-8f52-f89d8cf2b6eb | YES
timestamp | Long | timestamp for signature | in milliseconds | 1435890533866 | YES
app_sign | String | signature for security | algorithm: md5(app\_id+timestamp+app\_secret)，32 hexadecimal, lowercased | b927899dda6f9a04afc57f21ddf69d69 | YES

#### Content-type of Response : *application/json*
#### Parameters of Response :

- **Common** Parameters

KEY | TYPE | DESCRIPTION
---- | ---- | ----
result_code | Integer | 0 - OK
result_msg  | String | "OK" means what it means
err_detail  | String | text of error's detail 
refund | Map | the refund item

- Parameters of refund

KEY | TYPE | DESCRIPTION
---- | ---- | ----
bill\_no | String | see in bill API
channel | String | WX、ALI、UN、JD、KUAIQIAN、BD、YEE、PAYPAL
sub\_channel | String | WX\_NATIVE、WX\_JSAPI、WX\_APP、ALI\_APP、ALI\_WEB、ALI\_QRCODE、ALI\_OFFLINE_QRCODE、ALI_WAP、UN\_APP、UN\_WEB、YEE\_WEB、YEE\_WAP、BD\_WEB、BD\_WAP、BD\_APP、PAYPAL\_SANDBOX、PAYPAL\_LIVE
finish | Bool | true if refunding finished
createdat | Long | timestamp on payment's creation
optional | String | see in bill API
result | Bool| true if refunding succeed
title | String | refunding's title
total_fee | Integer | see in bill API
refund_fee | Integer | see in refund API
refund_no | String | see in refund API
updatedat | Long | timestamp on last updation of payment


## 10. Query refund by ID

#### PATH:   */1/rest/bill/{id}*
#### HTTP Method: *GET*
>NOTE: use uri format: param = JSON String, params below is in this JSON. And JSON string should be urlencoded.

>{id} was returned in bill API.

#### Common Parameters：

KEY | TYPE | MEANING | DESCRIPTION | E.G | Required
----  | ---- | ---- | ---- | ---- | ----
app_id | String | APP ID in BeeCloud | BeeCloud's distinct ID| 0950c062-5e41-44e3-8f52-f89d8cf2b6eb | YES
timestamp | Long | timestamp for signature | in milliseconds | 1435890533866 | YES
app_sign | String | signature for security | algorithm: md5(app\_id+timestamp+app\_secret)，32 hexadecimal, lowercased | b927899dda6f9a04afc57f21ddf69d69 | YES

#### Content-type of Response : *application/json*
#### Parameters of Response :

- **Common** Parameters

KEY | TYPE | DESCRIPTION
---- | ---- | ----
result_code | Integer | 0 - OK
result_msg  | String | "OK" means what it means
err_detail  | String | text of error's detail 
pay | Map | the payment item

- Parameters of pay

KEY | TYPE | DESCRIPTION
---- | ---- | ----
bill\_no | String | see in bill API
channel | String | WX、ALI、UN、JD、KUAIQIAN、BD、YEE、PAYPAL
sub\_channel | String | WX\_NATIVE、WX\_JSAPI、WX\_APP、ALI\_APP、ALI\_WEB、ALI\_QRCODE、ALI\_OFFLINE_QRCODE、ALI_WAP、UN\_APP、UN\_WEB、YEE\_WEB、YEE\_WAP、BD\_WEB、BD\_WAP、BD\_APP、PAYPAL\_SANDBOX、PAYPAL\_LIVE
createdat | Long | timestamp on payment's creation
optional | String | see in bill API
spay\_result | Bool | true if paid
title | String | payment's title
total_fee | Integer | see in bill API
updatedat | Long | timestamp on last updation of payment
channel\_trade\_no | String | third Party's transaction's ID if paid


## Contact
- Any Bug, report an [issue](https://github.com/beecloud/beecloud-rest-api/issues)
- Any feature request, report an [issue](https://github.com/beecloud/beecloud-rest-api/issues)
- You can ask questions in BeeCloud QQ-group NO.1:**321545822** or NO.2: **427128840**

## License
The MIT License (MIT).






