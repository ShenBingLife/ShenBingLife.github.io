---
layout: post
title: Restful 接口规范调研与解析
category: 架构
tags: [restful]
---

## Restful 接口规范调研与解析

|  版本  |     日期     |  人员  |
| :--: | :--------: | :--: |
| v0.1 | 2017-11-27 |  沈兵  |

[TOC]



### 1. 简易理解

1. 基于HTTP Method对资源CRUD

   | HTTP Method | URI        | HTTP body                    | 描述         |
   | ----------- | ---------- | ---------------------------- | ---------- |
   | GET         | /users     | 无                            | 用户列表       |
   | GET         | /users/uid | 无                            | 用户信息       |
   | POST        | /users     | ``` {"name":"xx","age"18}``` | 创建用户       |
   | PATCH       | /users/uid | ```{"name":"xx"}```          | 修改用户（部分更新） |
   | PUT         | /users/uid | ```{"name":"aa","age"15}```  | 修改用户（全量更新） |
   | DELETE      | /users/uid | 无                            | 删除用户       |

   ​

2. 使用HTTP状态码准确描述相应结果

   200：ok

   201：created

   204：no content

   400: bad request

   401: not authorized

   403: forbidden

   500: server error

3. 使用Content-Type准确描述资源内容

   | Content-Type     | 描述   |
   | ---------------- | ---- |
   | application/json | JSON |
   | application/xml  | XML  |
   | text/html        | HTML |



### 2. 描述性理解

       	1. URI表示资源，使用名词而不是动词，且推荐用复数
        	2. 保证HTTP Method本身的状态表示。HEAD、GET、PUT是幂等操作；同时使用HTTP Method本身的语义操作资源。
         	3. 使用正确的HTTP Status Code表示访问状态

### 3. 论文版理解

REST风格的架构所具有的6个的主要特征（http://www.infoq.com/cn/articles/understanding-restful-style）：

- 面向资源（Resource Oriented）
- 可寻址（Addressability）
- 连通性（Connectedness）
- 无状态（Statelessness）
- 统一接口（Uniform Interface）
- 超文本驱动（Hypertext Driven）

面向资源：URI以资源抽象为核心展开，可以更好地实现面向资源的抽象、开发编程、缓存、测试。

可寻址：1. 实现从服务器端到客户端的资源状态转移，将原本在服务器端保存的资源状态用客户端保存。2. 使用带有约定性的URI资源表述，可以更方便的推断出其他的URI资源。

连通性：资源与资源间并非孤立，而是存在某种关联。使用资源状态转移的特性，在资源间通过“链接”互相联系。

无状态：服务器端不保存上一次访问的状态。实现服务端和客户端更松散的耦合，有利于测试。

统一接口：利用HTTP标准方法或约定好的使用方法，可以针对资源开发出更统一的URI资源列表及资源操作接口。

超文本驱动（Hypermedia As The Engine Of Application State，HATEOAS）：资源之间通过超链接相互关联，超链接既代表资源之间的关系，也代表可执行的状态迁移。在超媒体之中不仅仅包含数据，还包含了状态迁移的语义。这里的“超链接”代表了一个超文本资源，超文本驱动简义为使用代表超文本资源的超链接驱动资源的操作，将超链接传输到客户端（意味着资源状态迁移）取代了对服务端状态的依赖。



### 4. 名词解释：

资源状态转移与无状态服务器：

> 例如我订阅了一个人的博客，想要获取他发表的所有文章（这里『他发表的所有文章』就是一个**资源Resource**）。于是我就向他的服务发出请求，说『我要获取你发表的所有文章，最好是atom格式的』，这时候服务器向你返回了atom格式的文章列表第一页（这里『atom格式的文章列表』就是**表征Representation**）。
>
> 你看到了第一页的页尾，想要看第二页，这时候有趣的事情就来了。如果服务器记录了应用的状态（stateful），那么你只要向服务询问『我要看下一页』，那么服务器自然就会返回第二页。类似的，如果你当前在第二页，想服务器请求『我要看下一页』，那就会得到第三页。但是REST的服务器恰恰是无状态的（stateless），服务器并没有保持你当前处于第几页，也就无法响应『下一页』这种具有状态性质的请求。因此客户端需要去维护当前应用的状态（application state），也就是『如何获取下一页资源』。当然，『下一页资源』的业务逻辑必然是由服务端来提供。服务器在文章列表的atom表征中加入一个URI超链接（hyper link），指向下一页文章列表对应的资源。客户端就可以使用**统一接口（Uniform Interface）**的方式，从这个URI中获取到他想要的下一页文章列表资源。上面的『能够进入下一页』就是应用的**状态（State）。**服务器把『能够进入下一页』这个状态以atom表征形式**传输（Transfer）**给客户端就是**表征状态传输（REpresentational State Transfer）**这个概念。
>
> https://www.zhihu.com/question/28557115



连通性与超媒体：

> 在絕大多數情況下，資源並不會孤立地存在，必然與其他資源具有某種關聯。既然我們推薦資源采用具有可尋址性的URL 來標識，那麼我們就可以利用它來將相關的資源關聯起來。比如采用XML 來表示一部電影的信息，那麼我們可以采用如下的形式利用URL 將相關的資源（導演、領銜主演、主演、編劇以及海報等）關聯在一起。當用戶得到這樣一份文檔的時候，可以利用文檔自身的內容獲得某部影片的基本信息，還可以利用相關的“鏈接”得到其他相關內容的詳細信息。
>
> ```
> <movie> 
> <name>魔鬼代言人</name> 
> <genre>劇情|懸疑|驚悚</genre> 
> <directors> 
> <add ref="http://www.artech.com/directors/taylor-hackford"> 
> 泰勒.海克福德</add> 
> </directors> 
> <starring> 
> <add ref = "http://www.artech.com/actors/al-pacino">阿爾.帕西諾</add> 
> <add ref = "http://www.artech.com/actors/keanu-reeves ">基諾.李維斯</add> 
> </starring> 
> <supportingActors> 
> <add ref = "http://www.artech.com/actors/charlize-theron ">查理茲.塞隆</add> 
> <add ref = "http://www.artech.com/actors/jeffrey-jones ">傑弗瑞.琼斯</add> 
> <add ref = "http://www.artech.com/actors/connie-nielsen">康尼.尼爾森</add> 
> </supportingActors> 
> <scriptWriters> 
> <add ref = "http://www.artech.com/scriptwriters/jonathan-lemkin"> 
> 喬納森·萊姆金  
> </add> 
> <add ref = "http://www.artech.com/scriptwriters/tony-gilroy"> 
> 托尼·吉爾羅伊</add> 
> </scriptWriters> 
> <language>英語</language> 
> <poster ref = "http://www.artech.com/images/the-devil-s-advocate"/> 
> <story>...</story> 
> </movie> 
>
>
> ```
>
> 具有上述內容的文檔可以視為一份超文本/超媒體文檔。Fielding 在他的論文中將REST定位為“分佈式超媒體應用”的架構風格，而超媒體的核心就是利用“鏈接”將相關的信息結成一個非線性的網，所以從這一點也可以看出REST 和“使用鏈接關聯相關的資源”這個特性是吻合的。
>
> http://www.wenwenti.info/article/143292

优点：

- 简单性

采用REST架构风格，对于开发、测试、运维人员来说，都会更简单。可以充分利用大量HTTP服务器端和客户端开发库、Web功能测试/性能测试工具、HTTP缓存、HTTP代理服务器、防火墙。这些开发库和基础设施早已成为了日常用品，不需要什么火箭科技（例如神奇昂贵的应用服务器、中间件）就能解决大多数可伸缩性方面的问题。

- 可伸缩性

充分利用好通信链各个位置的HTTP缓存组件，可以带来更好的可伸缩性。其实很多时候，在Web前端做性能优化，产生的效果不亚于仅仅在服务器端做性能优化，但是HTTP协议层面的缓存常常被一些资深的架构师完全忽略掉。

- 松耦合

统一接口+超文本驱动，带来了最大限度的松耦合。允许服务器端和客户端程序在很大范围内，相对独立地进化。对于设计面向企业内网的API来说，松耦合并不是一个很重要的设计关注点。但是对于设计面向互联网的API来说，松耦合变成了一个必选项，不仅在设计时应该关注，而且应该放在最优先位置。

​	

缺点：

​	对实时性要求很高的应用，REST的表现不如DO和RPC。



### 5. Restful 复杂需求：

1. #### 批量删除

   * DELETE  /users/1,2,3

   * DELETE /users?id=1&id=2

   * DELETE /users?id[]=1&id[]=2

   * POST /users/deletes   （用deletes表示批量）

     ​	用POST JSON替代DELETE

     ​	request body: `{"userIds":["1", "2"]}`

     ​	

     ​	或用POST JSON创建一个ID列表的资源。 用DELETE删除这个ID集合资源表示的用户

     ​	response body: `{"id":1, "userIds":["1", "2"]}`

     ​	DELETE /users/deletes/1

   * DELETE /users

     ​	request body: `{"userIds":["1", "2"]}`

2. #### 批量更新

   * PUT  /users

     request body: ```[{"id":1, "name":"xx"}, {"id":2,"name":"ss"}]```

   * PUT /users

     request body: ```{"users":[{"id":1, "name":"xx"}, {"id":2,"name":"ss"}]}```

3. #### 多对多关联关系

   * /memberships

     1.  如果希望同时获取多对多关系本身的属性,例如team下的一些信息：教练…等。使用/memberships/shipId可以更清楚的表示出来一组关系。
     2.  关于缓存，如果使用/teams/3/players/失效时，可能同时需要/players/5/teams/失效，但是使用/memberships/，仅仅一个资源表示关联关系，可以减少服务端或者客户端关于缓存的操作。
     3.  通过/teams/tid/ players或者/players/uid/teams,如果当你需要增加一种新的关联关系，例如：过去的team成员列表, 可能需要在现有的请求体、响应体中增加历史成员列表以适应接口的变更。但是使用/memberships/{id}/表示多对多关系时，可以通过/players/5/memberships/ 和/players/5/past_memberships/来处理这些关联关系，通过增加接口数量保证原有接口的稳定性。
     4.  这里在使用/memberships时，无法获取players列表中的详细信息，而是link列表。如果需要详细信息，可以使用/users?teamId=xx，通过用户列表接口去查询用户列表。

   * /teams/tid/users

     更容易理解，但是有悖于Restful理解。/teams/tid/users应该是一组关系，但是返回值通常被定义为用户资源列表。如果理解为这是用户资源，那么POST和DELETE操作理论上应该是用户创建和用户删除。

     如果/teams/tid/users使用超媒体链接表示用户关系，那么前后台业务都会增加，很难通过链接数组获取用户列表。

4. #### 分页接口定义

   是否有必要在响应结果上区分page接口和list接口呢？

   分页接口一般存在total表示总数，更多的可以返回当前页、总页数、当前排序等其他分页相关的元数据。

   而列表接口并不存在分页信息，理论上列表接口的标准返回形式是对象数组。


   在page接口的响应体上一搬是两种情况（属于广泛使用）：

   *  带有分页元信息的响应体

   ```
{
  "page":1,
  "total":10,
  "records":[..]
}
   ```

   *  使用和列表接口一样的数据结构

   在HTTP header中增加X-Total-Count表示总数的元信息。

   增加Link header表示上一页、下一页的超链接。

```
//Header
X-Total-Count:10
X-Current-Page:2
Link: <http://xx?page=3>;rel="next",<http://xx?page=1>;rel="pre"
```

```
//Response Body
[{
  "id":"xx",
  "name":"xx"
}, {
  "id":"ss",
  "name":"ss"
}]
```



5. #### 查询语言的定义

   如果存在多个参数，既有精确匹配，又有模糊匹配，是否需要将模糊匹配的参数增加后缀，例如`like`

   ```
   /users?name=shen&nameLike=shen&email=shen@qq.com&emailLike=qq.com
   ```

6. #### PATCH方法的应用

   patch方法使用较少。是否在真正遇到patch时用PUT或POST替代？

7. #### 重定义HTTP Method

   在文件移动、复制等特殊行为，是否应该使用 HTTP Header **X-HTTP-Method-Override** 来覆盖POST方法表示的method.

   `X-HTTP-Method-Override:COPY`

   或者自定义更多的HTTP Method，例如`COPY`， `MOVE`

8. #### 版本号定义

   * 在URI上定义版本号

     `/v1/xxx`

   * 在查询参数上定义版本号

     `/xxx?version=v1`

   * 在Header中定义版本号

     `X-Project-Version:v1`

9. #### Token使用

   * 使用Header携带Token

     `X-Auth-Token:xxxx`

   * 使用查询参数

     `/xxx?token=xxxx`

### 附录：

#### HTTP Method：

> HEAD 和OPTIONS 相對少見。從資源操作的語義來講，一個針對某個目標資源發送的HEAD 請求一般不是為瞭獲取目標資源本身的內容，而是希望得到描述資源的元數據信息。服務器一般將對應資源的元數據置於響應的報頭集合返回給客戶端，所以這樣的響應一般不具有主體部分。OPTIONS 請求一般是一種“探測”請求，其目的在於確定針對某個目標地址的真實請求應該具有怎樣的約束（比如應該采用怎樣的HTTP 方法或者必須攜帶怎樣的自定義報頭），然後根據其約束發送符合條件的請求。比如針對“跨域資源”的預檢（Preflight）請求采用的HTTP方法就是OPTIONS。
>
> 
>
> POST 和PUT 請求一般將所加資源的內容置於請求的主體。但是對於PUT 請求來說，如果添加資源的內容完全可以由其URI 來提供，這樣的請求可以不需要主體部分。比如我們通過請求添加一個用於控制權限的角色，標識添加角色的URI 由其角色名稱來決定，並且不需要指定除角色名稱的其他信息，那麼我們隻要發送如下一個不含主體的PUT 請求即可。
>
> `PUT http://www.artech.com/roles/admin HTTP/1.1`
>
> 
>
> 除瞭進行資源的添加，PUT 請求還能用於資源的修改。由於請求包含提交資源的標識（可以放在URI 中，也可以置於主體部分的資源內容中），所以服務端能夠定位到對應的資源並予以修改。對於POST 和PUT 這兩種HTTP 方法，也存在一種一刀切的說法：POST 用於添加，PUT 用於修改。我個人比較認可的是：如果PUT 提供的資源不存在，則做添加操作，否則做修改操作。
>
> 對於發送PUT 請求以修改某個存在的資源，服務器一般會通過提供資源將原有資源整體“覆蓋”掉。如果需要進行“局部”修改，我們推薦采用PATCH 方法，因為從語義上講“PATCH”就是打補丁的意思。
>
> 關於HTTP 請求采用的這些方法，具有兩個基本特性，即“安全性”和“冪等性”。對於上述7 種HTTP 方法，GET、HEAD 和OPTIONS 均被認為是安全的方法，因為它們旨在實現對數據的獲取，並不具有“邊界效應（Side Effect④）”。至於其他4 個HTTP 方法，由於它們會導致服務端資源的變化，所以被認為是不安全的方法。
>
> 冪等性（Idempotent）是一個數學上的概念，在這裡表示發送一次和多次請求引起的邊界效應是一致的。在網速不夠快的情況下，客戶端發送一個請求後不能立即得到響應，由於不能確定請求是否被成功提交，所以它有可能會再次發送另一個相同的請求。冪等性決定瞭第二個請求是否有效。
>
> 上述3 種安全的HTTP方法（GET、HEAD和OPTIONS）均被視為冪等方法。由於DELETE和PATCH 請求操作的是現有的某個資源，所以它們都是冪等方法。對於PUT 請求，隻有在對應資源不存在的情況下服務器才會進行添加操作，否則隻進行修改操作，所以它也是冪等方法。至於POST，由於它總是進行添加操作，如果服務器接收到兩次相同的POST 操作，將導致兩個相同的資源被創建，所以這是一個非冪等的方法。
>
> http://www.wenwenti.info/article/143292

#### 分布式架构风格比较：

> 从架构风格的抽象高度来看，常见的分布式应用架构风格有三种：
>
> - 分布式对象（Distributed Objects，简称DO）
>
> 架构实例有CORBA/RMI/EJB/DCOM/.NET Remoting等等
>
> - 远程过程调用（Remote Procedure Call，简称RPC）
>
> 架构实例有SOAP/XML-RPC/Hessian/Flash AMF/DWR等等
>
> - 表述性状态转移（Representational State Transfer，简称REST）
>
> 架构实例有HTTP/WebDAV
>
> DO和RPC这两种架构风格在企业应用中非常普遍，而REST则是Web应用的架构风格，它们之间有非常大的差别。
>
> REST与DO的差别在于：
>
> - REST支持抽象（即建模）的工具是资源，DO支持抽象的工具是对象。在不同的编程语言中，对象的定义有很大差别，所以DO风格的架构通常都是与某种编程语言绑定的。跨语言交互即使能实现，实现起来也会非常复杂。而REST中的资源，则完全中立于开发平台和编程语言，可以使用任何编程语言来实现。
> - DO中没有统一接口的概念。不同的API，接口设计风格可以完全不同。DO也不支持操作语义对于中间组件的可见性。
> - DO中没有使用超文本，响应的内容中只包含对象本身。REST使用了超文本，可以实现更大粒度的交互，交互的效率比DO更高。
> - REST支持数据流和管道，DO不支持数据流和管道。
> - DO风格通常会带来客户端与服务器端的紧耦合。在三种架构风格之中，DO风格的耦合度是最大的，而REST的风格耦合度是最小的。REST松耦合的源泉来自于统一接口+超文本驱动。
>
> REST与RPC的差别在于：
>
> - REST支持抽象的工具是资源，RPC支持抽象的工具是过程。REST风格的架构建模是以名词为核心的，RPC风格的架构建模是以动词为核心的。简单类比一下，REST是面向对象编程，RPC则是面向过程编程。
>
> - RPC中没有统一接口的概念。不同的API，接口设计风格可以完全不同。RPC也不支持操作语义对于中间组件的可见性。
>
> - RPC中没有使用超文本，响应的内容中只包含消息本身。REST使用了超文本，可以实现更大粒度的交互，交互的效率比RPC更高。
>
> - REST支持数据流和管道，RPC不支持数据流和管道。
>
> - 因为使用了平台中立的消息，RPC风格的耦合度比DO风格要小一些，但是RPC风格也常常会带来客户端与服务器端的紧耦合。支持统一接口+超文本驱动的REST风格，可以达到最小的耦合度。
>
>   http://www.infoq.com/cn/articles/understanding-restful-style

#### 基于HTTP的API的分类

> http://www.jalg.net/classification_of_http_apis.html
>
> 表示我们目前设计的并不是Restful的API。

#### OData和GraphQL

> 开放数据协议（Open Data Protocol，缩写OData）是一种描述如何创建和访问[Restful](https://baike.baidu.com/item/Restful)服务的[OASIS](https://baike.baidu.com/item/OASIS/4235159)标准。
>
> http://www.odata.org/getting-started/basic-tutorial/
>
> 
>
> GraphQL是一种支持API的查询语言，用户提高数据交互的效率。
>
> https://github.com/airplake/GraphQL-Learn-Chinese
>
> http://www.jianshu.com/p/2ec22fc1219c

### 