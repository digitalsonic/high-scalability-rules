# RESTful Web Services服务定义指南

## 1. 概述

本指南旨在介绍最基本的Restful Web Services定义约束，帮助大家定义出符合Richardson成熟度模型第2级的服务。

Richardson成熟度模型共分为4级，具体如下：

* 第0级，仅把HTTP作为传输协议
* 第1级，在服务中引入资源的概念
* 第2级，支持HTTP动词
* 第3级，超媒体驱动

遵循本指南至少应该实现到第2级，是否支持超媒体驱动则由开发者根据自身情况判断。

## 2. 适用范围

本指南聚焦于服务本身的定义，对于周边的基础设施不做约束。

由于Restful Web Services没有具体的行业标准，实现时也有众多选择，本指南不限定具体使用的框架和库。

## 3. 服务定义说明

一个Restful Web Service的定义主要可以分为以下几个步骤：

* 定义URI
* 选择HTTP METHOD
* 选择状态码
* 定义HTTP HEADER
* 定义资源表述

下面将就这些点逐一进行说明。

### 3.1 定义URI

#### 3.1.1 识别资源

REST服务的操作本质上是对资源的操作，通过操作来改变资源的状态，因此在定义URI前需要先识别出要操作的资源。

资源可以是这样的：

* 领域名词，比如余额、订单
* 集合，比如交易列表
* 函数，比如发起支付

#### 3.1.2 定义

URI中建议使用名词，不要将动词作为URI，各种动作需要使用HTTP METHOD来表示。

为了更好地将资源归类整理，可以对资源进行分组，分组由URI中的层级来表示。

如果名词较长或者由多个单词组成，可以使用`-`或`_`来对单词进行分割。

> 注意：定义URI时不要在后面加上文件扩展名，不要通过扩展名来标识资源表述格式，应该是用HTTP HEADER来表示。

例如：

* http://domain/trades
* http://domain/trades/{tradeId}
* http://domain/trades/{tradeId}/payment
* http://domain/users/{userId}/account/balance

有附加的参数需要传递时，可以在URI后以如下方式添加参数`?{param1}={value1}&{param2}={value2}`。

定义时需要考虑：

* 网络交互效率
* 资源表述的大小
* 客户端使用的便捷性

### 3.2 选择HTTP METHOD

| METHOD  | 是否安全 | 是否幂等 | 适用场景                                                  |
|---------|----------|---------|----------------------------------------------------------|
| GET     |    是    |    是    | 获取资源信息                                             |
| POST    |    否    |    否    | 创建新资源；修改已有资源；执行某些其他METHOD不适用的复杂操作 |
| DELETE  |    否    |    是    | 删除已有资源                                             |
| PUT     |    否    |    是    | 修改已有资源                                             |

还有三个METHOD不太使用，分别是HEAD、OPTIONS和TRACE，在内部服务里的使用场景不多，此处就不做说明了。

对于查询操作，直接使用GET METHOD，删除资源，直接使用DELETE METHOD。修改资源时，POST与PUT有所区别区别，POST是不幂等的，每次执行都会修改资源，PUT是幂等的，多次执行后的结果是一致的。例如，计数器的场景，要执行+1操作，选择POST，要执行赋值操作则可选择PUT。简单期间，如果不想纠结是POST还是PUT，在修改的场景中，可以统一用POST。

### 3.3 选择状态码

状态码（Status Code）也被称为响应码（Response Code），表示了请求的响应状态，必须选择合适的状态码，__严禁所有响应均使用200状态码__。

本文对一些常用的状态码的适用场景做一个说明，但实际可以使用的状态码不局限于下面列出的内容。

#### 3.3.1 请求成功处理的情况

| 状态码       | 说明                                                                 |
|--------------|---------------------------------------------------------------------|
| 200 OK       | 表示请求成功处理，也是最常用的状态码                                   |
| 201 Created  | 表示请求处理成功，同时创建了一个新的资源，该资源的URI应该放在LOCATION头里 |
| 202 Accepted | 表示请求已受理，但未必会最终成功，也可以不会最终执行，适用于异步处理的情况 |

#### 3.3.2 请求未能成功处理的情况

| 状态码                     | 说明                        |
|---------------------------|-----------------------------|
| 400 Bad Request           | 请求的参数错误，服务端无法处理 |
| 401 Unauthorized          | 请求未经授权                 |
| 403 Forbidden             | 服务端拒绝执行请求            |
| 404 Not Found             | 未找到需要的资源              |
| 500 Internal Server Error | 服务端发生内部错误            |

### 3.4 定义HTTP HEADER

使用合适的请求和响应头能够更好地提供服务，本节提供了一些常用头

| 请求头          | 说明                                                   |
|-----------------|-------------------------------------------------------|
| Accept          | 请求接受的MIME格式，比如`application/json`              |
| Accept-Charset  | 请求接受的字符集，比如`UTF-8`，根据规范，JSON的编码类型都是UTF-8，但在实际使用时通常还会有GBK，需要自己处理编码转换问题 |
| Accept-Encoding | 请求接受的数据编码方式，比如`gzip`，如果可以压缩，建议添加 |
| Content-Type    | 请求正文的表述类型，比如`application/json`              |
| X-Forwarded-For | 用来标注客户端以及经过的代理的IP信息，帮助获取真实的IP     |

| 响应头           | 说明                                     |
|------------------|-----------------------------------------|
| Location         | 用来进行重定向，获取资源的URI              |
| Content-Type     | 当前返回的表述类型，比如`application/json` |
| Content-Encoding | 当前返回的数据编码方式，比如`gzip`         |

> 切忌不设置Content-Type，或者随便乱设Content-Type。错误的Content-Type会影响客户端对响应的解析。

如果该请求的结果可以缓存，那么在请求和响应时可以设置对应的Cache-Control头。

如果有自定义HTTP HEADER，需遵循一定的命名规范，以`X-`开头，后续每个单词均用`-`作为分隔符。

### 3.5 定义资源表述

资源表述，简单来说就是请求和响应的正文，必须和Content-Type对应，建议的类型为`application/json`。

关于响应中的资源表述，建议包含如下信息：

* 操作的响应码，虽然有HTTP状态码，但HTTP状态码可能不足以表示业务状态，因此可以在响应中再增加一个响应码。
* 错误描述，如果操作不成功，需要在响应中提供错误描述。
* 表示当前资源的URI，相当于是当前URI，rel类型为`self`。（本项可选）
* 如果是分页的列表，在表述中需要包含指向上一页和下一页的URI，rel类型分别为`prev`和`next`。（本项可选）

一个典型的分页请求的结果可能是这样的：

```
HTTP/1.1 200 OK
Location: http://domain/users?page_no=2
Content-Type: application/json
Content-Length: ...

{
  "result_code": "0",
  "error_msg": "",
  "links": [
    {"rel": "self", "href": "http://domain/users?page_no=2"},
    {"rel": "next", "href": "http://domain/users?page_no=3"},
    {"rel": "prev", "href": "http://domain/users?page_no=1"},
  ],
  "page_no": "2",
  "total_count": "20"
  "list": [
    {"id": "5", "name": "David"},
    {"id": "6", "name": "Peter"},
    {"id": "7", "name": "Paul"},
    {"id": "8", "name": "Frank"},
    {"id": "8", "name": "Mark"}
  ]
}
```

如果请求失败，则是这样的：

```
HTTP/1.1 500 Internal Server Error
Location: http://domain/users?page_no=2
Content-Type: application/json
Content-Length: ...

{
  "result_code": "50010",
  "error_msg": "can not obtain user list",
  "links": [
    {"rel": "self", "href": "http://domain/users?page_no=2"},
  ]
}
```

### 3.6 其他

#### 3.6.1 版本控制

在设计服务时，尽量保证没次变更都能向下兼容，服务端和客户端在设计时需要考虑：

* 结果中多一个属性或者少一个属性
* 结果的某个属性值增加了一个超出之前范围的值
* 操作的语义发生了变化（这种情况可以考虑新起服务）

如果存在实在无法兼容的情况，可以考虑两种做法：

* 在URI中添加版本号，例如：http://domain/users/v2?page_no=2
* 在参数中添加版本号，例如：http://domain/users?page_no=2&version=2

> 注意：如可能的话，还是不建议引入版本
