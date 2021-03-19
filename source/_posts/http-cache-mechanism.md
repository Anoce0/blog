---
title: HTTP 缓存机制详解
tags:
  - http
  - cache
  - performance
date: 2021-03-19 13:57:05
---

### 1. 缓存的级别
从是否需要向浏览器请求资源来说，浏览器资源的缓存分成以下几个级别
#### 1.1 不缓存 - 不使用本地缓存  
每次客户端的请求都需要服务器返回完整资源  
不用缓存使用的请求头是  

```
Cache-Control: no-store
```

#### 1.2 强缓存 - 直接使用本地缓存  
不会向服务器发送请求，直接从缓存中读取资源，在chrome控制台的network选项中可以看到该请求返回200的状态码;  
强缓存的请求头是

```
Cache-Control: max-age=<seconds>
```

这里的 max-age 参数表示本地缓存的过期时间,浏览器在尝试请求资源时,会查询当前资源是否已经存在本地缓存中,并且 max-age 没有到期,如果满足这个条件, 就会直接存本地缓存中读取文件.  

#### 1.3 协商缓存 - 向浏览器确认后使用本地缓存  
向服务器发送请求，服务器会根据这个请求的request header的一些参数来判断是否命中协商缓存，如果命中，则返回304状态码并带上新的response header通知浏览器从缓存中读取资源；  
协商缓存的请求头是:  

```
Cache-Control: no-cache, max-age=<seconds>
```

在这个过程中，浏览器向服务器发起请求来判断资源是否过期有两种不同的策略  

##### 1.3.1 基于时间的判断  

服务器将资源传递给客户端时，会将资源最后更改的时间以“Last-Modified: GMT”的形式加在实体首部上一起返回给客户端。

```html
Last-Modified: Fri, 22 Jul 2020 01:47:00 GMT
```

客户端会为资源标记上该信息，下次再次请求时，会把该信息附带在请求报文中一并带给服务器去做检查，
 - 若传递的时间值与服务器上该资源最终修改时间是一致的，则说明该资源没有被修改过，直接返回304状态码，内容为空，这样就节省了传输数据量 。
 - 如果两个时间不一致，则服务器会发回该资源并返回200状态码，和第一次请求时类似。这样保证不向客户端重复发出资源，也保证当服务器有变化时，客户端能够得到最新的资源。一个304响应比一个静态资源通常小得多，这样就节省了网络带宽。  

这里的客户端传递标记给服务器去判断最终修改时间的请求报文首部字段一共有两个：

``` html
If-Modified-Since: Last-Modified-value 
If-Unmodified-Since: Last-Modified-value 
```

- `If-Modified-Since` 是比较常见的缓存用法, 用于 GET/HEAD, 返回 304 或 200(资源过期的话)  
- `If-Unmodified-Since` 通常用于校验某个任务重服务器资源没有改变, 例如编辑某个wiki 页面,验证页面在编辑过程中发生了修改,就拒绝提交, 或者断点续传过程中发现服务器资源发生了修改就停止续传,通使用状态码 412 （Precondition Failed，前置条件失败）
  
##### 1.3.2 基于 Etag 的判断  
这里的 Etag 是一个文件的标识符, 有服务器端生成, 不同的服务器使用的算法不一样, 通常用的可能是 Hash 算法, 当 Etag 校验与时间校验同时存在是, Etag 拥有 更高的优先级,时间检验会被忽略.  
Etag  校验也要两个请求头,

```html
If-Match: <etag_value> // 与 If-Modified-Since 类似
If-None-Match: <etag_value> // 与 If-Unmodified-Since 类似
```


上述的 Catche-Control 机制都出现在 HTTP 1.1 版本中,在早期的 HTTP 1.0 中,还使用了 Expires 与 Pragma 两个参数来控制缓存, Expires 与  `max-age=<seconds>` 功能类似, 只是 Expires 用的是绝对时间(基于服务器时间),这会导致服务器时间与本地不一致时, 缓存控制紊乱, 而Prama 请求头与 `no-cache` 作用基本一样.  

在两种版本的请求头同时出现的情况下, 1.1 的指令拥有更高的优先级


### 2. 缓存的角色
使用缓存的角色, 还有两种分类方法,

#### 2.1 private -(私有)浏览器缓存  

私有缓存只能用于单独用户。你可能已经见过浏览器设置中的“缓存”选项。浏览器缓存拥有用户通过 HTTP 下载的所有文档。这些缓存为浏览过的文档提供向后/向前导航，保存网页，查看源码等功能，可以避免再次向服务器发起多余的请求。它同样可以提供缓存内容的离线浏览。

#### 2.2 public - (共享)代理缓存  

共享缓存可以被多个用户使用。例如，ISP 或你所在的公司可能会架设一个 web 代理来作为本地网络基础的一部分提供给用户。这样热门的资源就会被重复使用，减少网络拥堵与延迟。

```html
Cache-Control: public, max-age=<seconds>
Cache-Control: private, max-age=<seconds>
```


> 参考:  
> - *[MDN - HTTP cache](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Caching)*  
> - *[强制缓存和协商缓存有什么区别](https://www.jianshu.com/p/1a1536ab01f1)*  
> - *[可能是最被误用的 HTTP 响应头之一 Cache-Control: must-revalidate](https://zhuanlan.zhihu.com/p/60357719)*  

