---
layout: post
title: Restful约定
tags: 约定
category: Restful
---

# Restful约定

[TOC]

Restful的复杂之处不在于它的定义与难以理解，而在与它未定义的部分。为了编写一个完全Restful的接口，我们可能要写出很多古怪而复杂的代码。所以为了统一开发人员的理解与开发过程，这里约定一些在Restful的规范，它不是标准的Restful定义，但是尽量利用Restful的优点。

<!-- more -->

### URI命名

* 以资源路径命名，路径名称必须是名词

* 名词表示资源集合，使用复数形式； 单数资源，可以使用单数形式

* 子资源：/资源名/id/子资源名/id

* uri命名建议：不用大写，用中杠 **-** 不用下杠 **_** 

* 示例：

  > /users
  >
  > /users/1
  >
  > /users/1/name
  >
  > /users/validation
  >
  > /users/name-validation
  >
  > /orgs/1/users

### 基于HTTP Method语义对资源CRUD

| HTTP Method | URI        | HTTP body               | 描述                 |
| ----------- | ---------- | ----------------------- | -------------------- |
| GET         | /users     | 无                      | 用户列表             |
| GET         | /users/uid | 无                      | 用户信息             |
| POST        | /users     | `{"name":"xx","age"18}` | 创建用户             |
| PATCH       | /users/uid | `{"name":"xx"}`         | 修改用户（部分更新） |
| PUT         | /users/uid | `{"name":"aa","age"15}` | 修改用户（全量更新） |
| DELETE      | /users/uid | 无                      | 删除用户             |



### 使用HTTP状态码准确描述相应结果

200：ok

201：created

204：no content

400: bad request

401: not authorized

403: forbidden

500: server error



### 使用Content-Type准确描述资源内容

| Content-Type     | 描述 |
| ---------------- | ---- |
| application/json | JSON |
| application/xml  | XML  |
| text/html        | HTML |

### 统一的错误返回值

```
{
    "message":"用户未认证",
    "code": "1001",
    "data": null
}
```

### 批量删除

* 第一种，优先考虑使用

  `DELETE  /users/1,2,3`

* 第二种

  `DELETE /users?id=1&id=2`

### 批量更新

request:

`PUT  /users`

body: 

```json
[{"id":1, "name":"xx"}, {"id":2,"name":"ss"}]
```

### 分页接口

* 第一种，常用，优先考虑使用

  带有分页元信息的响应体

  ```json
  {
    "page":1,
    "total":10,
    "records":[..]
  }
  ```

  ​

* 第二种

   在HTTP header中增加X-Total-Count表示总数的元信息。

     增加Link header表示上一页、下一页的超链接。

  ```
  //Header
  X-Total-Count:10
  X-Current-Page:2
  Link: <http://xx?page=3>;rel="next",<http://xx?page=1>;rel="pre"
  ```

  ```json
  //Response Body
  [{
    "id":"xx",
    "name":"xx"
  }, {
    "id":"ss",
    "name":"ss"
  }]
  ```

### 查询语言的定义

如果存在多个参数，既有精确匹配，又有模糊匹配

`/users?name=shen&nameLike=shen&email=shen@qq.com&emailLike=qq.com`

### PATCH与POST

从现在开始尽量使用PATCH，旧版本可以用POST替代PATCH

### 版本号定义

* 在URI上定义版本号，优先考虑使用

  `/v1/xxx`

* 在Header中定义版本号

  `X-Project-Version:v1`

### Token使用

- 使用Header携带Token，优先考虑使用

  `X-Auth-Token:xxxx`

- 使用查询参数

  `/xxx?token=xxxx`

### 多对多关联关系

* /teams/tid/users，优先考虑使用

  更容易理解，但是有悖于Restful理解。/teams/tid/users应该是一组关系，但是返回值通常被定义为用户资源列表。如果理解为这是用户资源，那么POST和DELETE操作理论上应该是用户创建和用户删除。

  如果/teams/tid/users使用超媒体链接表示用户关系，那么前后台业务都会增加，很难通过链接数组获取用户列表。

- /memberships
  1. 如果希望同时获取多对多关系本身的属性,例如team下的一些信息：教练…等。使用/memberships/shipId可以更清楚的表示出来一组关系。
  2. 关于缓存，如果使用/teams/3/players/失效时，可能同时需要/players/5/teams/失效，但是使用/memberships/，仅仅一个资源表示关联关系，可以减少服务端或者客户端关于缓存的操作。
  3. 通过/teams/tid/ players或者/players/uid/teams,如果当你需要增加一种新的关联关系，例如：过去的team成员列表, 可能需要在现有的请求体、响应体中增加历史成员列表以适应接口的变更。但是使用/memberships/{id}/表示多对多关系时，可以通过/players/5/memberships/ 和/players/5/past_memberships/来处理这些关联关系，通过增加接口数量保证原有接口的稳定性。