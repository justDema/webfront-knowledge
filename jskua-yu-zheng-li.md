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

**`<script>`标签是不受同源策略的限制的，它可以载入任意地方的 JavaScript 文件，而并不要求同源。**

所以 JSONP 的理念就是，我和服务端约定好一个函数名，当我请求文件的时候，服务端返回一段 JavaScript。这段 JavaScript 调用了我们约定好的函数，并且将数据当做参数传入。  
非常巧合的一点（其实并不是），JSON 的数据格式和 JavaScript 语言里对象的格式正好相同。所以在我们约定的函数里面可以直接使用这个对象。

### JSONP的优缺点 {#articleHeader3}

JSONP的优点是：它不像`XMLHttpRequest`对象实现的Ajax请求那样受到同源策略的限制；它的兼容性更好，在更加古老的浏览器中都可以运行，不需要XMLHttpRequest或ActiveX的支持；并且在请求完毕后可以通过调用callback的方式回传结果。

JSONP的缺点则是：它只支持GET请求而不支持POST等其它类型的HTTP请求；

它只支持跨域HTTP请求这种情况，不能解决不同域的两个页面之间如何进行`JavaScript`调用的问题。

安全问题，由于jsonp是通过脚本注入的方式，一旦注入源出现恶意代码就会导致被注入的页面遭受攻击。

