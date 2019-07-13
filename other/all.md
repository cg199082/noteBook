# Api Documentation


<a name="overview"></a>
## 概览
Api Documentation


### 版本信息
*版本* : 1.0


### 许可信息
*许可证* : Apache 2.0  
*许可网址* : http://www.apache.org/licenses/LICENSE-2.0  
*服务条款* : urn:tos


### URI scheme
*域名* : 47.101.10.214  
*基础路径* : /


### 标签

* CallBackController : Call Back Controller
* DriverController : Driver Controller
* WxPortalController : Wx Portal Controller




<a name="paths"></a>
## 资源

<a name="callbackcontroller_resource"></a>
### CallBackController
Call Back Controller


<a name="closeoildrumusingpost"></a>
#### 油桶关箱回调
```
POST /wechatApp/callback/closeOilDrum
```


##### 说明
油桶关箱回调


##### 参数

|类型|名称|说明|类型|
|---|---|---|---|
|**Body**|**dto**  <br>*必填*|dto|[OilDrumCloseDTO](#oildrumclosedto)|


##### 响应

|HTTP代码|说明|类型|
|---|---|---|
|**200**|OK|[Result«string»](#e249bf1902de7f75aaed353ffea96339)|
|**201**|Created|无内容|
|**401**|Unauthorized|无内容|
|**403**|Forbidden|无内容|
|**404**|Not Found|无内容|


##### 消耗

* `application/json`


##### 生成

* `\*/*`


##### HTTP请求示例

###### 请求 path
```
/wechatApp/callback/closeOilDrum
```


###### 请求 body
```json
{
  "capacity" : 0.0,
  "driverId" : "string",
  "oilDrumId" : "string"
}
```


##### HTTP响应示例

###### 响应 200
```json
{
  "code" : 0,
  "data" : "string",
  "message" : "string"
}
```


<a name="oildrumreachlimitusingpost"></a>
#### 油桶满油通知
```
POST /wechatApp/callback/oilDrumReachLimit
```


##### 说明
油桶满油通知


##### 参数

|类型|名称|说明|类型|
|---|---|---|---|
|**Query**|**oilDrumId**  <br>*必填*|oilDrumId|string|


##### 响应

|HTTP代码|说明|类型|
|---|---|---|
|**200**|OK|[Result«string»](#e249bf1902de7f75aaed353ffea96339)|
|**201**|Created|无内容|
|**401**|Unauthorized|无内容|
|**403**|Forbidden|无内容|
|**404**|Not Found|无内容|


##### 消耗

* `application/json`


##### 生成

* `\*/*`


##### HTTP请求示例

###### 请求 path
```
/wechatApp/callback/oilDrumReachLimit?oilDrumId=string
```


##### HTTP响应示例

###### 响应 200
```json
{
  "code" : 0,
  "data" : "string",
  "message" : "string"
}
```


<a name="drivercontroller_resource"></a>
### DriverController
Driver Controller


<a name="activeoildrumusingpost"></a>
#### 激活油桶
```
POST /wechatApp/driver/activeOilDrum
```


##### 说明
返回值code所代表状态：  
 0:成功  
 1:内部错误  
 2:超时  
 3:参数不正确  
 4:油箱未激活


##### 参数

|类型|名称|说明|类型|
|---|---|---|---|
|**Body**|**dto**  <br>*必填*|dto|[ActiveOilDrumDTO](#activeoildrumdto)|


##### 响应

|HTTP代码|说明|类型|
|---|---|---|
|**200**|OK|[Result«string»](#e249bf1902de7f75aaed353ffea96339)|
|**201**|Created|无内容|
|**401**|Unauthorized|无内容|
|**403**|Forbidden|无内容|
|**404**|Not Found|无内容|


##### 消耗

* `application/json`


##### 生成

* `\*/*`


##### HTTP请求示例

###### 请求 path
```
/wechatApp/driver/activeOilDrum
```


###### 请求 body
```json
{
  "capacity" : 0.0,
  "driverId" : "string",
  "oilDrumId" : "string",
  "storeId" : "string"
}
```


##### HTTP响应示例

###### 响应 200
```json
{
  "code" : 0,
  "data" : "string",
  "message" : "string"
}
```


<a name="bindoildrumusingpost"></a>
#### 绑定油桶
```
POST /wechatApp/driver/bindOilDrum
```


##### 说明
返回值code所代表状态：  
 0:成功  
 1:内部错误  
 2:超时  
 3:参数不正确  
 4:查询不存在


##### 参数

|类型|名称|说明|类型|
|---|---|---|---|
|**Body**|**dto**  <br>*必填*|dto|[DriverBindOilDrumDTO](#driverbindoildrumdto)|


##### 响应

|HTTP代码|说明|类型|
|---|---|---|
|**200**|OK|[Result«string»](#e249bf1902de7f75aaed353ffea96339)|
|**201**|Created|无内容|
|**401**|Unauthorized|无内容|
|**403**|Forbidden|无内容|
|**404**|Not Found|无内容|


##### 消耗

* `application/json`


##### 生成

* `\*/*`


##### HTTP请求示例

###### 请求 path
```
/wechatApp/driver/bindOilDrum
```


###### 请求 body
```json
{
  "capacity" : 0.0,
  "currentCapacity" : 0.0,
  "driverId" : "string",
  "oilDrumId" : "string",
  "storeId" : "string",
  "storeName" : "string"
}
```


##### HTTP响应示例

###### 响应 200
```json
{
  "code" : 0,
  "data" : "string",
  "message" : "string"
}
```


<a name="checkstoreusingget"></a>
#### 查询门店是否存在
```
GET /wechatApp/driver/checkStore
```


##### 说明
返回值code所代表状态：  
 0:成功  
 1:内部错误  
 2:超时  
 3:参数不正确  
 4:门店不存在


##### 参数

|类型|名称|说明|类型|
|---|---|---|---|
|**Query**|**storeName**  <br>*必填*|storeName|string|


##### 响应

|HTTP代码|说明|类型|
|---|---|---|
|**200**|OK|[Result«List«StoreDTO»»](#ad4bdd1cd55d091aa31396fdd3c485b5)|
|**401**|Unauthorized|无内容|
|**403**|Forbidden|无内容|
|**404**|Not Found|无内容|


##### 消耗

* `application/json`


##### 生成

* `\*/*`


##### HTTP请求示例

###### 请求 path
```
/wechatApp/driver/checkStore?storeName=string
```


##### HTTP响应示例

###### 响应 200
```json
{
  "code" : 0,
  "data" : [ {
    "address" : "string",
    "createDate" : "string",
    "createTime" : "string",
    "oilDrumStr" : "string",
    "oilDrums" : [ {
      "capacity" : 0.0,
      "createTime" : "string",
      "id" : 0,
      "isDelete" : 0,
      "oilDrumId" : "string",
      "updateTime" : "string"
    } ],
    "storeId" : "string",
    "storeName" : "string"
  } ],
  "message" : "string"
}
```


<a name="driverloginusingpost"></a>
#### 司机登陆
```
POST /wechatApp/driver/driverLogin
```


##### 说明
返回值code所代表状态：  
 0:成功  
 1:内部错误  
 2:超时  
 3:参数不正确


##### 参数

|类型|名称|说明|类型|
|---|---|---|---|
|**Body**|**dto**  <br>*必填*|dto|[DriverLoginDTO](#driverlogindto)|


##### 响应

|HTTP代码|说明|类型|
|---|---|---|
|**200**|OK|[Result«DriverDTO»](#1c00a821d27f72c98b3ab5d07d3fe96f)|
|**201**|Created|无内容|
|**401**|Unauthorized|无内容|
|**403**|Forbidden|无内容|
|**404**|Not Found|无内容|


##### 消耗

* `application/json`


##### 生成

* `\*/*`


##### HTTP请求示例

###### 请求 path
```
/wechatApp/driver/driverLogin
```


###### 请求 body
```json
{
  "openId" : "XXXXXXXX",
  "password" : "123456a",
  "telNo" : "14010000001",
  "unionId" : "XXXXXXXX"
}
```


##### HTTP响应示例

###### 响应 200
```json
{
  "code" : 0,
  "data" : {
    "driverId" : "string",
    "telNo" : "14010000001"
  },
  "message" : "string"
}
```


<a name="driverlogoutusingpost"></a>
#### 退出登录
```
POST /wechatApp/driver/driverLogout
```


##### 说明
退出登录后，服务器将清除该账号和微信OpenID的关联，该用户所关注的油桶满油后将无法通知到用户，请务必提醒用户。  
返回值code所代表状态：  
 0:成功  
 1:内部错误  
 2:超时  
 3:参数不正确  
 4:查询不存在


##### 参数

|类型|名称|说明|类型|
|---|---|---|---|
|**Body**|**dto**  <br>*必填*|dto|[DriverBindWechatDTO](#driverbindwechatdto)|


##### 响应

|HTTP代码|说明|类型|
|---|---|---|
|**200**|OK|[Result«string»](#e249bf1902de7f75aaed353ffea96339)|
|**201**|Created|无内容|
|**401**|Unauthorized|无内容|
|**403**|Forbidden|无内容|
|**404**|Not Found|无内容|


##### 消耗

* `application/json`


##### 生成

* `\*/*`


##### HTTP请求示例

###### 请求 path
```
/wechatApp/driver/driverLogout
```


###### 请求 body
```json
{
  "driverId" : "string",
  "openId" : "string"
}
```


##### HTTP响应示例

###### 响应 200
```json
{
  "code" : 0,
  "data" : "string",
  "message" : "string"
}
```


<a name="driverlogoutbyunionidusingget"></a>
#### 根据unionId退出登录
```
GET /wechatApp/driver/driverLogoutByUnionId
```


##### 说明
退出登录后，服务器将清除该账号和微信OpenID的关联，该用户所关注的油桶满油后将无法通知到用户，请务必提醒用户。  
返回值code所代表状态：  
 0:成功  
 1:内部错误  
 2:超时  
 3:参数不正确  
 4:查询不存在


##### 参数

|类型|名称|说明|类型|
|---|---|---|---|
|**Query**|**unionId**  <br>*必填*|unionId|string|


##### 响应

|HTTP代码|说明|类型|
|---|---|---|
|**200**|OK|[Result«string»](#e249bf1902de7f75aaed353ffea96339)|
|**401**|Unauthorized|无内容|
|**403**|Forbidden|无内容|
|**404**|Not Found|无内容|


##### 消耗

* `application/json`


##### 生成

* `\*/*`


##### HTTP请求示例

###### 请求 path
```
/wechatApp/driver/driverLogoutByUnionId?unionId=string
```


##### HTTP响应示例

###### 响应 200
```json
{
  "code" : 0,
  "data" : "string",
  "message" : "string"
}
```


<a name="getbindoildrumusingpost"></a>
#### 获取我的油桶
```
POST /wechatApp/driver/getBindOilDrum
```


##### 说明
获取我的油桶


##### 参数

|类型|名称|说明|类型|
|---|---|---|---|
|**Body**|**dto**  <br>*必填*|dto|[DriverBindOilDrumPage](#driverbindoildrumpage)|


##### 响应

|HTTP代码|说明|类型|
|---|---|---|
|**200**|OK|[PageInfo«DriverBindOilDrumDTO»](#551b127e32e34ce4a41365957ce87f3c)|
|**201**|Created|无内容|
|**401**|Unauthorized|无内容|
|**403**|Forbidden|无内容|
|**404**|Not Found|无内容|


##### 消耗

* `application/json`


##### 生成

* `\*/*`


##### HTTP请求示例

###### 请求 path
```
/wechatApp/driver/getBindOilDrum
```


###### 请求 body
```json
{
  "driverId" : "string",
  "pageNum" : 1,
  "pageSize" : 20
}
```


##### HTTP响应示例

###### 响应 200
```json
{
  "endRow" : 0,
  "hasNextPage" : true,
  "hasPreviousPage" : true,
  "isFirstPage" : true,
  "isLastPage" : true,
  "list" : [ {
    "capacity" : 0.0,
    "currentCapacity" : 0.0,
    "driverId" : "string",
    "oilDrumId" : "string",
    "storeId" : "string",
    "storeName" : "string"
  } ],
  "navigateFirstPage" : 0,
  "navigateLastPage" : 0,
  "navigatePages" : 0,
  "navigatepageNums" : [ 0 ],
  "nextPage" : 0,
  "pageNum" : 0,
  "pageSize" : 0,
  "pages" : 0,
  "prePage" : 0,
  "size" : 0,
  "startRow" : 0,
  "total" : 0
}
```


<a name="getbindrelationbyopenidusingget"></a>
#### 获取微信绑定关系
```
GET /wechatApp/driver/getBindRelationByOpenId
```


##### 说明
返回值code所代表状态：  
 0:成功  
 1:内部错误  
 2:超时  
 3:参数不正确  
 4:查询不存在


##### 参数

|类型|名称|说明|类型|
|---|---|---|---|
|**Query**|**openId**  <br>*必填*|微信openId|string|


##### 响应

|HTTP代码|说明|类型|
|---|---|---|
|**200**|OK|[Result«DriverBindWechatDTO»](#7bee99fd8df79a0a9981710536dd4ac4)|
|**401**|Unauthorized|无内容|
|**403**|Forbidden|无内容|
|**404**|Not Found|无内容|


##### 消耗

* `application/json`


##### 生成

* `\*/*`


##### HTTP请求示例

###### 请求 path
```
/wechatApp/driver/getBindRelationByOpenId?openId=string
```


##### HTTP响应示例

###### 响应 200
```json
{
  "code" : 0,
  "data" : {
    "driverId" : "string",
    "openId" : "string"
  },
  "message" : "string"
}
```


<a name="getrecoverlogusingpost"></a>
#### 分页查询回收记录
```
POST /wechatApp/driver/getRecoverLog
```


##### 说明
分页查询回收记录


##### 参数

|类型|名称|说明|类型|
|---|---|---|---|
|**Body**|**page**  <br>*必填*|page|[DriverRecoverLogPage](#driverrecoverlogpage)|


##### 响应

|HTTP代码|说明|类型|
|---|---|---|
|**200**|OK|[PageInfo«DriverRecoverLogVO»](#7a56d1610fea4a9c3d42e8b841ad99bb)|
|**201**|Created|无内容|
|**401**|Unauthorized|无内容|
|**403**|Forbidden|无内容|
|**404**|Not Found|无内容|


##### 消耗

* `application/json`


##### 生成

* `\*/*`


##### HTTP请求示例

###### 请求 path
```
/wechatApp/driver/getRecoverLog
```


###### 请求 body
```json
{
  "driverId" : "string",
  "pageNum" : 1,
  "pageSize" : 20
}
```


##### HTTP响应示例

###### 响应 200
```json
{
  "endRow" : 0,
  "hasNextPage" : true,
  "hasPreviousPage" : true,
  "isFirstPage" : true,
  "isLastPage" : true,
  "list" : [ {
    "createTime" : "string",
    "oilDrumId" : "string",
    "recoverCapacity" : 0.0,
    "storeId" : "string",
    "storeName" : "string"
  } ],
  "navigateFirstPage" : 0,
  "navigateLastPage" : 0,
  "navigatePages" : 0,
  "navigatepageNums" : [ 0 ],
  "nextPage" : 0,
  "pageNum" : 0,
  "pageSize" : 0,
  "pages" : 0,
  "prePage" : 0,
  "size" : 0,
  "startRow" : 0,
  "total" : 0
}
```


<a name="openoildrumusingpost"></a>
#### 油桶开箱
```
POST /wechatApp/driver/openOilDrum
```


##### 说明
返回值code所代表状态：  
 0:成功  
 1:内部错误  
 2:超时  
 3:参数不正确  
 4:油桶未激活


##### 参数

|类型|名称|说明|类型|
|---|---|---|---|
|**Body**|**dto**  <br>*必填*|dto|[DriverBindOilDrumDTO](#driverbindoildrumdto)|


##### 响应

|HTTP代码|说明|类型|
|---|---|---|
|**200**|OK|[Result«string»](#e249bf1902de7f75aaed353ffea96339)|
|**201**|Created|无内容|
|**401**|Unauthorized|无内容|
|**403**|Forbidden|无内容|
|**404**|Not Found|无内容|


##### 消耗

* `application/json`


##### 生成

* `\*/*`


##### HTTP请求示例

###### 请求 path
```
/wechatApp/driver/openOilDrum
```


###### 请求 body
```json
{
  "capacity" : 0.0,
  "currentCapacity" : 0.0,
  "driverId" : "string",
  "oilDrumId" : "string",
  "storeId" : "string",
  "storeName" : "string"
}
```


##### HTTP响应示例

###### 响应 200
```json
{
  "code" : 0,
  "data" : "string",
  "message" : "string"
}
```


<a name="unbindoildrumusingpost"></a>
#### 取消关注油桶
```
POST /wechatApp/driver/unBindOilDrum
```


##### 说明
返回值code所代表状态：  
 0:成功  
 1:内部错误  
 2:超时  
 3:参数不正确  
 4:查询不存在


##### 参数

|类型|名称|说明|类型|
|---|---|---|---|
|**Body**|**dto**  <br>*必填*|dto|[DriverBindOilDrumDTO](#driverbindoildrumdto)|


##### 响应

|HTTP代码|说明|类型|
|---|---|---|
|**200**|OK|[Result«string»](#e249bf1902de7f75aaed353ffea96339)|
|**201**|Created|无内容|
|**401**|Unauthorized|无内容|
|**403**|Forbidden|无内容|
|**404**|Not Found|无内容|


##### 消耗

* `application/json`


##### 生成

* `\*/*`


##### HTTP请求示例

###### 请求 path
```
/wechatApp/driver/unBindOilDrum
```


###### 请求 body
```json
{
  "capacity" : 0.0,
  "currentCapacity" : 0.0,
  "driverId" : "string",
  "oilDrumId" : "string",
  "storeId" : "string",
  "storeName" : "string"
}
```


##### HTTP响应示例

###### 响应 200
```json
{
  "code" : 0,
  "data" : "string",
  "message" : "string"
}
```


<a name="wxportalcontroller_resource"></a>
### WxPortalController
Wx Portal Controller


<a name="postusingpost"></a>
#### post
```
POST /wechatApp/portal/{appid}
```


##### 参数

|类型|名称|说明|类型|
|---|---|---|---|
|**Path**|**appid**  <br>*必填*|appid|string|
|**Query**|**encrypt_type**  <br>*可选*|encrypt_type|string|
|**Query**|**msg_signature**  <br>*可选*|msg_signature|string|
|**Query**|**nonce**  <br>*必填*|nonce|string|
|**Query**|**openid**  <br>*必填*|openid|string|
|**Query**|**signature**  <br>*必填*|signature|string|
|**Query**|**timestamp**  <br>*必填*|timestamp|string|
|**Body**|**requestBody**  <br>*必填*|requestBody|string|


##### 响应

|HTTP代码|说明|类型|
|---|---|---|
|**200**|OK|string|
|**201**|Created|无内容|
|**401**|Unauthorized|无内容|
|**403**|Forbidden|无内容|
|**404**|Not Found|无内容|


##### 消耗

* `application/json`


##### 生成

* `application/xml;charset=UTF-8`


##### HTTP请求示例

###### 请求 path
```
/wechatApp/portal/string?nonce=string&openid=string&signature=string&timestamp=string
```


###### 请求 body
```json
{ }
```


##### HTTP响应示例

###### 响应 200
```json
"string"
```


<a name="authgetusingget"></a>
#### authGet
```
GET /wechatApp/portal/{appid}
```


##### 参数

|类型|名称|说明|类型|
|---|---|---|---|
|**Path**|**appid**  <br>*必填*|appid|string|
|**Query**|**echostr**  <br>*可选*|echostr|string|
|**Query**|**nonce**  <br>*可选*|nonce|string|
|**Query**|**signature**  <br>*可选*|signature|string|
|**Query**|**timestamp**  <br>*可选*|timestamp|string|


##### 响应

|HTTP代码|说明|类型|
|---|---|---|
|**200**|OK|string|
|**401**|Unauthorized|无内容|
|**403**|Forbidden|无内容|
|**404**|Not Found|无内容|


##### 消耗

* `application/json`


##### 生成

* `text/plain;charset=utf-8`


##### HTTP请求示例

###### 请求 path
```
/wechatApp/portal/string
```


##### HTTP响应示例

###### 响应 200
```json
"string"
```




<a name="definitions"></a>
## 定义

<a name="activeoildrumdto"></a>
### ActiveOilDrumDTO
油桶激活


|名称|说明|类型|
|---|---|---|
|**capacity**  <br>*必填*|油桶容量  <br>**样例** : `0.0`|number|
|**driverId**  <br>*必填*|司机id  <br>**样例** : `"string"`|string|
|**oilDrumId**  <br>*必填*|油桶硬件id  <br>**样例** : `"string"`|string|
|**storeId**  <br>*必填*|门店id  <br>**样例** : `"string"`|string|


<a name="driverbindoildrumdto"></a>
### DriverBindOilDrumDTO
司机绑定油桶


|名称|说明|类型|
|---|---|---|
|**capacity**  <br>*必填*|油桶容量  <br>**样例** : `0.0`|number|
|**currentCapacity**  <br>*必填*|油桶当前容量  <br>**样例** : `0.0`|number|
|**driverId**  <br>*必填*|司机id  <br>**样例** : `"string"`|string|
|**oilDrumId**  <br>*必填*|油桶硬件ID  <br>**样例** : `"string"`|string|
|**storeId**  <br>*必填*|门店Id  <br>**样例** : `"string"`|string|
|**storeName**  <br>*必填*|门店名称  <br>**样例** : `"string"`|string|


<a name="driverbindoildrumpage"></a>
### DriverBindOilDrumPage
分页查询司机绑定油桶DTO


|名称|说明|类型|
|---|---|---|
|**driverId**  <br>*必填*|司机id  <br>**样例** : `"string"`|string|
|**pageNum**  <br>*必填*|页码  <br>**样例** : `1`|integer (int32)|
|**pageSize**  <br>*必填*|每页查询数量  <br>**样例** : `20`|integer (int32)|


<a name="driverbindwechatdto"></a>
### DriverBindWechatDTO
司机微信绑定关系


|名称|说明|类型|
|---|---|---|
|**driverId**  <br>*必填*|司机id  <br>**样例** : `"string"`|string|
|**openId**  <br>*必填*|微信openId  <br>**样例** : `"string"`|string|


<a name="driverdto"></a>
### DriverDTO
司机信息


|名称|说明|类型|
|---|---|---|
|**driverId**  <br>*必填*|司机id  <br>**样例** : `"string"`|string|
|**telNo**  <br>*必填*|司机手机号  <br>**样例** : `"14010000001"`|string|


<a name="driverlogindto"></a>
### DriverLoginDTO
登陆信息


|名称|说明|类型|
|---|---|---|
|**openId**  <br>*必填*|微信openId  <br>**样例** : `"XXXXXXXX"`|string|
|**password**  <br>*必填*|登录密码  <br>**样例** : `"123456a"`|string|
|**telNo**  <br>*必填*|司机手机号  <br>**样例** : `"14010000001"`|string|
|**unionId**  <br>*必填*|公众平台统一id  <br>**样例** : `"XXXXXXXX"`|string|


<a name="driverrecoverlogpage"></a>
### DriverRecoverLogPage
分页查询司机回收记录


|名称|说明|类型|
|---|---|---|
|**driverId**  <br>*必填*|司机id  <br>**样例** : `"string"`|string|
|**pageNum**  <br>*必填*|页码  <br>**样例** : `1`|integer (int32)|
|**pageSize**  <br>*必填*|每页查询数量  <br>**样例** : `20`|integer (int32)|


<a name="driverrecoverlogvo"></a>
### DriverRecoverLogVO
司机回收油桶记录


|名称|说明|类型|
|---|---|---|
|**createTime**  <br>*必填*|回收时间  <br>**样例** : `"string"`|string (date-time)|
|**oilDrumId**  <br>*必填*|油桶ID  <br>**样例** : `"string"`|string|
|**recoverCapacity**  <br>*必填*|回收容量  <br>**样例** : `0.0`|number|
|**storeId**  <br>*必填*|门店Id  <br>**样例** : `"string"`|string|
|**storeName**  <br>*必填*|门店名称  <br>**样例** : `"string"`|string|


<a name="oildrum"></a>
### OilDrum

|名称|说明|类型|
|---|---|---|
|**capacity**  <br>*可选*|**样例** : `0.0`|number|
|**createTime**  <br>*可选*|**样例** : `"string"`|string (date-time)|
|**id**  <br>*可选*|**样例** : `0`|integer|
|**isDelete**  <br>*可选*|**样例** : `0`|integer (int32)|
|**oilDrumId**  <br>*可选*|**样例** : `"string"`|string|
|**updateTime**  <br>*可选*|**样例** : `"string"`|string (date-time)|


<a name="oildrumclosedto"></a>
### OilDrumCloseDTO
油桶关箱回调DTO


|名称|说明|类型|
|---|---|---|
|**capacity**  <br>*必填*|当前油量  <br>**样例** : `0.0`|number|
|**driverId**  <br>*必填*|司机id  <br>**样例** : `"string"`|string|
|**oilDrumId**  <br>*必填*|油桶硬件ID  <br>**样例** : `"string"`|string|


<a name="551b127e32e34ce4a41365957ce87f3c"></a>
### PageInfo«DriverBindOilDrumDTO»

|名称|说明|类型|
|---|---|---|
|**endRow**  <br>*可选*|**样例** : `0`|integer (int32)|
|**hasNextPage**  <br>*可选*|**样例** : `true`|boolean|
|**hasPreviousPage**  <br>*可选*|**样例** : `true`|boolean|
|**isFirstPage**  <br>*可选*|**样例** : `true`|boolean|
|**isLastPage**  <br>*可选*|**样例** : `true`|boolean|
|**list**  <br>*可选*|**样例** : `[ "[driverbindoildrumdto](#driverbindoildrumdto)" ]`|< [DriverBindOilDrumDTO](#driverbindoildrumdto) > array|
|**navigateFirstPage**  <br>*可选*|**样例** : `0`|integer (int32)|
|**navigateLastPage**  <br>*可选*|**样例** : `0`|integer (int32)|
|**navigatePages**  <br>*可选*|**样例** : `0`|integer (int32)|
|**navigatepageNums**  <br>*可选*|**样例** : `[ 0 ]`|< integer (int32) > array|
|**nextPage**  <br>*可选*|**样例** : `0`|integer (int32)|
|**pageNum**  <br>*可选*|**样例** : `0`|integer (int32)|
|**pageSize**  <br>*可选*|**样例** : `0`|integer (int32)|
|**pages**  <br>*可选*|**样例** : `0`|integer (int32)|
|**prePage**  <br>*可选*|**样例** : `0`|integer (int32)|
|**size**  <br>*可选*|**样例** : `0`|integer (int32)|
|**startRow**  <br>*可选*|**样例** : `0`|integer (int32)|
|**total**  <br>*可选*|**样例** : `0`|integer (int64)|


<a name="7a56d1610fea4a9c3d42e8b841ad99bb"></a>
### PageInfo«DriverRecoverLogVO»

|名称|说明|类型|
|---|---|---|
|**endRow**  <br>*可选*|**样例** : `0`|integer (int32)|
|**hasNextPage**  <br>*可选*|**样例** : `true`|boolean|
|**hasPreviousPage**  <br>*可选*|**样例** : `true`|boolean|
|**isFirstPage**  <br>*可选*|**样例** : `true`|boolean|
|**isLastPage**  <br>*可选*|**样例** : `true`|boolean|
|**list**  <br>*可选*|**样例** : `[ "[driverrecoverlogvo](#driverrecoverlogvo)" ]`|< [DriverRecoverLogVO](#driverrecoverlogvo) > array|
|**navigateFirstPage**  <br>*可选*|**样例** : `0`|integer (int32)|
|**navigateLastPage**  <br>*可选*|**样例** : `0`|integer (int32)|
|**navigatePages**  <br>*可选*|**样例** : `0`|integer (int32)|
|**navigatepageNums**  <br>*可选*|**样例** : `[ 0 ]`|< integer (int32) > array|
|**nextPage**  <br>*可选*|**样例** : `0`|integer (int32)|
|**pageNum**  <br>*可选*|**样例** : `0`|integer (int32)|
|**pageSize**  <br>*可选*|**样例** : `0`|integer (int32)|
|**pages**  <br>*可选*|**样例** : `0`|integer (int32)|
|**prePage**  <br>*可选*|**样例** : `0`|integer (int32)|
|**size**  <br>*可选*|**样例** : `0`|integer (int32)|
|**startRow**  <br>*可选*|**样例** : `0`|integer (int32)|
|**total**  <br>*可选*|**样例** : `0`|integer (int64)|


<a name="7bee99fd8df79a0a9981710536dd4ac4"></a>
### Result«DriverBindWechatDTO»

|名称|说明|类型|
|---|---|---|
|**code**  <br>*可选*|**样例** : `0`|integer (int32)|
|**data**  <br>*可选*|**样例** : `"[driverbindwechatdto](#driverbindwechatdto)"`|[DriverBindWechatDTO](#driverbindwechatdto)|
|**message**  <br>*可选*|**样例** : `"string"`|string|


<a name="1c00a821d27f72c98b3ab5d07d3fe96f"></a>
### Result«DriverDTO»

|名称|说明|类型|
|---|---|---|
|**code**  <br>*可选*|**样例** : `0`|integer (int32)|
|**data**  <br>*可选*|**样例** : `"[driverdto](#driverdto)"`|[DriverDTO](#driverdto)|
|**message**  <br>*可选*|**样例** : `"string"`|string|


<a name="ad4bdd1cd55d091aa31396fdd3c485b5"></a>
### Result«List«StoreDTO»»

|名称|说明|类型|
|---|---|---|
|**code**  <br>*可选*|**样例** : `0`|integer (int32)|
|**data**  <br>*可选*|**样例** : `[ "[storedto](#storedto)" ]`|< [StoreDTO](#storedto) > array|
|**message**  <br>*可选*|**样例** : `"string"`|string|


<a name="e249bf1902de7f75aaed353ffea96339"></a>
### Result«string»

|名称|说明|类型|
|---|---|---|
|**code**  <br>*可选*|**样例** : `0`|integer (int32)|
|**data**  <br>*可选*|**样例** : `"string"`|string|
|**message**  <br>*可选*|**样例** : `"string"`|string|


<a name="storedto"></a>
### StoreDTO

|名称|说明|类型|
|---|---|---|
|**address**  <br>*可选*|**样例** : `"string"`|string|
|**createDate**  <br>*可选*|**样例** : `"string"`|string|
|**createTime**  <br>*可选*|**样例** : `"string"`|string (date-time)|
|**oilDrumStr**  <br>*可选*|**样例** : `"string"`|string|
|**oilDrums**  <br>*可选*|**样例** : `[ "[oildrum](#oildrum)" ]`|< [OilDrum](#oildrum) > array|
|**storeId**  <br>*可选*|**样例** : `"string"`|string|
|**storeName**  <br>*可选*|**样例** : `"string"`|string|





