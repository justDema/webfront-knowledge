##  {#articleHeader0}

## **造成跨域的两种策略**

浏览器的同源策略会导致跨域，这里同源策略又分为以下两种

1. DOM同源策略：禁止对不同源页面DOM进行操作。这里主要场景是iframe跨域的情况，不同域名的iframe是限制互相访问的。
2. XmlHttpRequest同源策略：禁止使用XHR对象向不同源的服务器地址发起HTTP请求。

## **为什么要有跨域限制**

了解完跨域之后，想必大家都会有这么一个思考，为什么要有跨域的限制，浏览器这么做是出于何种原因呢。其实仔细想一想就会明白，跨域限制主要是为了安全考虑。

AJAX同源策略主要用来防止CSRF攻击。如果没有AJAX同源策略，相当危险，我们发起的每一次HTTP请求都会带上请求地址对应的cookie，那么可以做如下攻击：

1. 用户登录了自己的银行页面 
   [http://mybank.com](https://link.zhihu.com/?target=http%3A//mybank.com)
   ，
   [http://mybank.com](https://link.zhihu.com/?target=http%3A//mybank.com)
   向用户的cookie中添加用户标识。
2. 用户浏览了恶意页面 
   [http://evil.com](https://link.zhihu.com/?target=http%3A//evil.com)
   。执行了页面中的恶意AJAX请求代码。
3. [http://evil.com](https://link.zhihu.com/?target=http%3A//evil.com)向
   [http://mybank.com](https://link.zhihu.com/?target=http%3A//mybank.com)
   发起AJAX HTTP请求，请求会默认把
   [http://mybank.com](https://link.zhihu.com/?target=http%3A//mybank.com)
   对应cookie也同时发送过去。
4. 银行页面从发送的cookie中提取用户标识，验证用户无误，response中返回请求数据。此时数据就泄露了。
5. 而且由于Ajax在后台执行，用户无法感知这一过程。

DOM同源策略也一样，如果iframe之间可以跨域访问，可以这样攻击：

1. 做一个假网站，里面用iframe嵌套一个银行网站 
   [http://mybank.com](https://link.zhihu.com/?target=http%3A//mybank.com)
2. 把iframe宽高啥的调整到页面全部，这样用户进来除了域名，别的部分和银行的网站没有任何差别。
3. 这时如果用户输入账号密码，我们的主网站可以跨域访问到
   [http://mybank.com](https://link.zhihu.com/?target=http%3A//mybank.com)
   的dom节点，就可以拿到用户的输入了，那么就完成了一次攻击。

所以说有了跨域跨域限制之后，我们才能更安全的上网了。

## 1. JSONP {#articleHeader0}

首先要介绍的跨域方法必然是 JSONP。

现在你想要获取其他网站上的 JavaScript 脚本，你非常高兴的使用 XMLHttpRequest 对象来获取。但是浏览器一点儿也不配合你，无情的弹出了下面的错误信息：

```
XMLHttpRequest cannot load http://x.com/main.dat. No 'Access-Control-Allow-Origin' 
header is present on the requested resource. Origin 'http://y.com' is therefore not allowed access.
```

你心里肯定会想，我难道要用后台做个爬虫来获取这个数据吗？！\(；°○° \)  
为了避免这种蛋疼的事情发生，JSONP 就派上用场了。

`<script>`**标签是不受同源策略的限制的，它可以载入任意地方的 JavaScript 文件，而并不要求同源。**

所以 JSONP 的理念就是，我和服务端约定好一个函数名，当我请求文件的时候，服务端返回一段 JavaScript。这段 JavaScript 调用了我们约定好的函数，并且将数据当做参数传入。  
非常巧合的一点（其实并不是），JSON 的数据格式和 JavaScript 语言里对象的格式正好相同。所以在我们约定的函数里面可以直接使用这个对象。

### JSONP的优缺点 {#articleHeader3}

JSONP的优点是：它不像`XMLHttpRequest`对象实现的Ajax请求那样受到同源策略的限制；它的兼容性更好，在更加古老的浏览器中都可以运行，不需要XMLHttpRequest或ActiveX的支持；并且在请求完毕后可以通过调用callback的方式回传结果。

JSONP的缺点则是：它只支持GET请求而不支持POST等其它类型的HTTP请求；

它只支持跨域HTTP请求这种情况，不能解决不同域的两个页面之间如何进行`JavaScript`调用的问题。

安全问题，由于jsonp是通过脚本注入的方式，一旦注入源出现恶意代码就会导致被注入的页面遭受攻击。





# 2. CORS 跨域资源共享 {#page-title}

CORS是一个W3C标准，全称是"跨域资源共享"（Cross-origin resource sharing）。

CORS（Cross-Origin Resource Sharing，跨域资源共享）是 W3C 的一个工作草案，定义了在必须访问跨源资源时，浏览器与服务器应该如何沟通。CORS 背后的的基本思想，就是使用自定义的 HTTP 头部让浏览器与服务器进行沟通，从而决定请求或响应的成功和失败。

它允许浏览器向跨源服务器，发出[`XMLHttpRequest`](http://www.ruanyifeng.com/blog/2012/09/xmlhttprequest_level_2.html)请求，从而克服了AJAX只能[同源](http://www.ruanyifeng.com/blog/2016/04/same-origin-policy.html)使用的限制。

### 访问控制场景: {#访问控制场景}

#### 简单请求 {#简单请求}

所谓的简单，是指：

* 只使用 GET, HEAD 或者 POST 请求方法。如果使用 POST 向服务器端传送数据，则数据类型\(Content-Type\)只能是
  `application/x-www-form-urlencoded`,`multipart/form-data`或`text/plain`中的一种
* 不会使用自定义请求头（类似于 X-Modified 这种）

下面是一个使用简单请求的例子:

```
$.ajax({
   url: 'http://localhost:8002/listUsers',
   method: 'GET',
   success: function (data) {
       p.innerHTML = JSON.stringify(data);
       document.body.appendChild(p);
   }
});
```

Request Headers:

```
Accept:*/*
Accept-Encoding:gzip, deflate, sdch, br
Accept-Language:zh-CN,zh;q=0.8,en;q=0.6
Cache-Control:no-cache
Connection:keep-alive
Host:localhost:8002
Origin:http://localhost:63343
Pragma:no-cache
Referer:http://localhost:63343/JsPractice/src/js/ajax/json.html?_ijt=4f3ur0duo147k993set7na2733
User-Agent:Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.98 Safari/537.36       

```

上面的头信息中，Origin 字段用来说明，本次请求来自哪个源（协议 + 域名 + 端口）。服务器根据这个值，决定是否同意这次请求。  
如果 Origin 指定的源，不在许可范围内，服务器会返回一个正常的 HTTP 回应。浏览器发现，这个回应的头信息没有包含 Access-Control-Allow-Origin 字段，就知道出错了，从而抛出一个错误，被 XMLHttpRequest 的 onerror 回调函数捕获。

这种错误无法通过状态码识别，因为 HTTP 回应的状态码有可能是200。



如果Origin指定的域名在许可范围内，服务器返回的响应，会多出几个头信息字段。请看 Response Headers:

```
Access-Control-Allow-Origin:*
Connection:keep-alive
Content-Length:1138
Content-Type:application/json; charset=utf-8
Date:Sun, 20 Nov 2016 07:57:09 GMT
X-Powered-By:Express
```

Access-Control-Allow-Origin 字段是必须的。它的值要么是请求时 Origin 字段的值，要么是一个 \*，表示接受任意域名的请求。



#### 非简单请求\(预请求\) {#非简单请求-预请求}

请求以 GET, HEAD 或者 POST 以外的方法发起请求。或者，使用 POST，但请求数据为

* `application/x-www-form-urlencoded`,`multipart/form-data`或者`text/plain`以外的数据类型。比如说，用 POST 发送数据类型为`application/xml`或者`text/xml`的 XML 数据的请求。
* 使用自定义请求头（比如添加诸如 X-PINGOTHER）

不同于简单请求，“预请求”要求必须先发送一个 OPTIONS 请求给目的站点，来查明这个跨站请求对于目的站点是不是安全可接受的。

## CORS和JSONP对比 {#articleHeader4}

```
1、 JSONP只能实现GET请求，而CORS支持所有类型的HTTP请求。

2、 使用CORS，开发者可以使用普通的XMLHttpRequest发起请求和获得数据，比起JSONP有更好的错误处理。

3、 JSONP主要被老的浏览器支持，它们往往不支持CORS，而绝大多数现代浏览器都已经支持了CORS）。
```



## 3.  通过修改document.domain来跨子域 {#articleHeader5}

浏览器都有一个同源策略，其限制之一就是第一种方法中我们说的不能通过ajax的方法去请求不同源中的文档。 它的第二个限制是浏览器中不同域的框架之间是不能进行js的交互操作的。



