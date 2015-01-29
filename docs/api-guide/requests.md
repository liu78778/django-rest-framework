源码: request.py

---

**注意**: 这篇文档针对的是**3.0版本**的REST framework. [2.4版本](http://tomchristie.github.io/rest-framework-2-docs/)的文档依然有效.

---

# Requests(请求对象)

> 如果你正在做一些基于REST的网络服务...你应该忽略request.POST.
>
> &mdash; Malcom Tredinnick, [Django开发组][cite]

REST framework的`Request`类继承自标准的`HttpRequest`, 加入对REST framework的请求灵活的解析以及鉴权的支持.


---

# Request 解析

REST framework的Request对象提供了对请求灵活的解析, 让你可以想处理表单数据一样来处理JSON数据或者其他媒体类型的数据.

## .data

`request.data` 获得由请求体解析来的数据.  这和标准的`request.POST`以及`request.FILES`属性很类似，除了以下几点:

* 它包含所有解析的内容, 包含*文件和非文件* 输入参数.
* 它提供了除了HTTP方法`POST`之外的其他方法的内容解析, 这意味着你可以通过`PUT`和`PATCH`请求来进行访问.
* 在对表单数据的支持之外, 它提供了REST framework请求灵活的解析.  例如你可以像处理表单数据一样处理JSON数据.

更多的细节请参考 [解析器][parsers documentation].

## .query_params

`request.query_params`是一个指代`request.GET`的属性.

为了使你的代码清晰可见, 我们推荐使用`request.query_params`来代替Django原本标准的`request.GET`. 这样做将有助于保持你的代码的可读和准确 - 任何HTTP方法的请求都可能包含查询参数, 并不仅仅是`GET`.

## .DATA and .FILES

老版本2.x中的旧API `request.data`以及`request.FILES`属性依旧可用, 但是现在有统一的`request.data`属性可用.

## .QUERY_PARAMS

老版本2.x中的旧API `request.QUERY_PARAMS`属性依旧可用, 但是现在有更符合python格式的`request.query_params`.

## .parsers

类`APIView`或者装饰器`@api_view`会确保这个属性被自动设置为`Parser`实例的列表, 基于视图中的集合`parser_classes`或者配置项`DEFAULT_PARSER_CLASSES`.

你通常不需要直接访问这个属性.

---

**注意:** 如果客户端发送的格式不正确, 这时访问`request.data`可能会触发一个错误`ParseError`.  默认情况下, REST framework的类`APIView`和装饰器`@api_view`会捕获这个错误并返回一个`400 Bad Request`的响应.

如果客户端发送的请求的内容类型不能被解析, 就会抛出一个异常`UnsupportedMediaType`, 默认情况下, 它会被捕获并返回一个`415 Unsupported Media Type`的响应.

---

# Content negotiation(内容协商)

请求对象公开了一些属性, 通过它们你可以确定内容协商阶段的结果. 这样你就可以实现一些行为, 如为不同的媒体类型选择不同的序列化方案.

## .accepted_renderer

在内容协商阶段被选中的渲染器实例对象.

## .accepted_media_type

一个用来表示被内容协商阶段所接受的媒体类型.

---

# Authentication(认证)

REST framework提供了灵活的, 依附每个请求的认证, 这为你提供了以下功能: 

* 对你的API的不同部分使用不同的认证策略.
* 支持使用多种认证策略.
* 同时提供用户和令牌(token)信息与传入的请求关联.

## .user

`request.user`通常返回一个`django.contrib.auth.models.User`的实例, 虽然这个行为取决于使用的认证策略.

如果请求是没有认证的, 那么`request.user`默认是一个`django.contrib.auth.models.AnonymousUser`的实例(匿名用户).

更多细节请参考[鉴权][authentication documentation].

## .auth

`request.auth`返回任意额外的认证上下文.  `request.auth`的确切行为依赖于正在使用的认证策略, 但它通常是一个被认证针对的请求的令牌实例.

如果请求是没有认证的, 或者没有额外的上下文出现, `request.auth`的默认值就是`None`.

更多细节请参考[鉴权][authentication documentation].

## .authenticators

类`APIView`或装饰器`@api_view`会确保这个属性被正确的设置为`Authentication`的实例, 基于视图中的几何`authentication_classes` 或者配置项`DEFAULT_AUTHENTICATORS`.

你通常不需要直接访问这个属性.

---

# Browser enhancements(浏览器增强)

REST framework支持类似基于浏览的`PUT`, `PATCH`和`DELETE`形式的浏览器增强功能.

## .method

`request.method`使用**大写的**字符串形式返回HTTP请求方法的.

透明的支持基于浏览的`PUT`, `PATCH`和`DELETE`形式.

更多细节请参考[浏览器增强][browser enhancements documentation].

## .content_type

`request.content_type`, 获得一个字符串表示的HTTP请求体的类型, 如果没有媒体类型就会是一个空字符串.

你通常不需要直接访问请求的内容类型, 因为你通常依赖于REST framework默认提供的解析.

如果你需要访问请求的内容类型, 你应该优先使用属性`.content_type`来替代`request.META.get('HTTP_CONTENT_TYPE')`, 因为它提供了基于浏览器的非内容形式的透明支持.

更多细节请参考[浏览器增强][browser enhancements documentation].

## .stream

`request.stream`返回一个表示请求体的流对象.

你通常不需要直接访问请求的内容, 因为你通常依赖于REST framework默认提供的解析.

如果你需要访问请求的内容, 你应该优先使用属性`.stream`来替代`request.content`, 因为它提供了基于浏览器的非内容形式的透明支持.

更多细节请参考[浏览器增强][browser enhancements documentation].

---

# 标准的HttpRequest属性

由于REST framework的类`Request`继承自Django的类`HttpRequest`, 所有其他标准的属性和方法都是可用的. 例如`request.META`和`request.session`可以照常使用.

需要注意的是, 由于具体实现的原因, 类`Request`并没有直接从类`HttpRequest`派生, 而是通过类组合的方式完成了继承.


[cite]: https://groups.google.com/d/topic/django-developers/dxI4qVzrBY4/discussion
[parsers documentation]: parsers.md
[authentication documentation]: authentication.md
[browser enhancements documentation]: ../topics/browser-enhancements.md
