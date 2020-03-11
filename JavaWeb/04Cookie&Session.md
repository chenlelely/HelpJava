# Cookie

## 概述

Cookie由 HTTP 协议制定，由一个键和一个值构成。

Cookie 是由服务器创建， 然后通过响应发送给客户端的一个键值对。 客户端会保存Cookie， 并会标注出 Cookie 的来源（ 哪个服务器的 Cookie）。 当客户端向服务器发出请求时会把所有这个服务器 Cookie 包含在请求中发送给服务器， 这样服务器就可以识别客户端了  

### Cookie与HTTP头

Cookie是通过HTTP请求头和响应头在客户端和服务器端传递的：

- cookie：请求头，客户端发送给服务器端；
  - 格式：Cookie:a=A；b=B；c=C。即多个Cookie用分号分开；
    一个Cookie对应多个键值对
- Set-Cookie：响应头，服务器端发送给客户端；
  - 一个cookie 对象一个set-Cookie，一个键值对
    Set-Cookie:a=A
    Set-Cookie:b=B
    Set-Cookie:c=C
    比如可以用response.addHeader（“Set-Cookie\"\"a=A\"）；发送一个Cookie

## Cookie的生命周期

Cookie在客户端的有效时间，可以通过`setMaxAge(int)`来设置 cookie的有效时间。

- cookie.setMaxAge(-1):cookie的maxAge属性的默认值就是-1，表示只在浏览器内存中存活。一且关闭浏览器窗口，那么cookie就会消失。
- cookie.setMaxAge(60*60)：表示cookie对象可存活1小时。当生命大于0时，浏览器会把Cookie保存到硬盘上，就算关闭浏览器，就算重启客户端电脑，cookie也会存活1小时；
- cookie.setMaxAge(0)：cookie 生命等于0是一个特殊的值，它表示 cookie 被作废！也就是说，如果原来浏览器已经保存了这个Cookie，那么可以通过cookie的setMaxAge(0)来删除这个cookie。无论是在浏览器内存中，还是在客户端硬盘上都会删除这个Cookie。

## Cookie 的Path

可以通过设置 Cookie 的 path 来指定浏览器， 在**访问什么样的路径时， 包含什么样的Cookie**。 请求路径如果包含了 Cookie 路径， 那么会在请求中包含这个 Cookie  

设置 Cookie的路径需要使用`setPath()`方法，例如 :`cookie.setPath(\"/cookietest/servlet\");`

默认路径是访问资源的上一级路径，如果没有设置Cookie的路径，那么Cookie路径的默认值当前访问资源所在路径（上一级），例如：

- 访问http:/localhost:8080/cookietest/ASerlet时添加的 Cookie默认路径为`/cookietest`;
- 访问http:/localhost:8080/cookietest/servlet/BServlet时添加的Cookie默认路径为`/cookietest/servlet`；
- 访问http:/localhost:8080/cookietest/jsp/BServlet 时添加的Cookie默认路径为`/cookietest/jsp`;

## Cookie的domain

Cookie 的 domain 属性可以让网站中二级域共享 Cookie。  

想让`http://www.baidu.com`和`http://zhidao.baidu.com ` 主机之间共享Cookie，可以设置：

- 设置 Cookie 的 path 为`“/”`： `c.setPath(“/”)`；
- 设置 Cookie 的 domain 为`“.baidu.com”`：` c.setDomain(“.baidu.com”) ` 

# Session

Session（**服务器创建，服务器保存**）

1. session 默认过期时间，过长会怎么样：30min，过长占用内存
2. 如何防止session 被别人伪造cookie得到：Cookie的值后面跟上一段防篡改的验证串。防篡改的验证串可以由DES(**cookie的内容+盐值**），也可以用MD5(**cookie内容+密钥**），也可以是SHA1**(cookie内容+密钥**），这里的密钥只有站点本身知道，如果这个都泄漏了那就真完蛋了。这个值在服务器接收到Cookie以后，就可以用Cookie的内容+密钥重新计算一次验证串，和提交上来的做比对，如果是一致的，我们就认为cookie没有被篡改，反之，cookie 肯定被篡改过，我们就不要相信这一次提交。如果所有的Cookie都经过防篡改验证。那么也就不用担心SessionlD被冒名顶替的事情发生了。
3. 如果服务器的负载量很高内存负荷不住要怎么办：尽量规避使用，优先以Cookie（包括加密Cookie)来解决；Session要自己实现分布式储存，比如使用**redis等做储存**。

## HTTPSession

javax.servlet.http.HttpSession 接口表示一个会话， 我们可以把一个会话内需要共享的数据保存到 HttSession 对象中！

**会话**： 会话范围是某个用户从首次访问服务器开始， 到该用户关闭浏览器结束  

HttpServletRequest、 ServletContext， 它们都是域对象，HttpSession也是域对象。 它们三个是 Servlet 中可以使用的域对象，而 JSP 中可以多使用一个域对象 PageContext。

- HttpServletRequest： 一个请求创建一个 request 对象， 所以在同一个请求中可以共享request， 例如一个请求从 AServlet 转发到 BServlet， 那么AServlet 和BServlet可以共享 request域中的数据；
- ServletContext： 一个应用只创建一个 ServletContext 对象， 所以在 ServletContext中的数据可以在整个应用中共享， 只要不关闭服务器， 那么 ServletContext 中的数据就可以共享  
- HttpSession： 一个会话创建一个 HttpSession 对象， 同一会话中的多个请求中可以共享 session 中的数据；  

## Session实现原理

session底层是依赖Cookie的！

1. 当首次使用session时，服务器端要创建 session，session是保存在服务器端，而给客户端的 session的id（**一个cookie中保存了sessionld**）。**客户端带走的是sessionld，而数据是保存在session中**。

2. 当客户端再次访问服务器时，在请求中会带上sessionld，而服务器会通过sessionld找到对应的session，而无需再创建新的session。
3. 如果Servlet中没有创建 session，那么响应头中就不会有 sessionid的cookie。但是所有的JSP页面中，session可以直接使用，也就是说每个JSP页面都会自动获取session，无论是否使用，访问JSP页面一定会带回来一个sessionid。

## Session与浏览器

session 保存在服务器，而 sessionld通过cookie发送给客户端，但这个Cookie的生命不是-1，即只在浏览器内存中存在，也就是说如果用户关闭了浏览器，那么这个cookie就丢失了。
当用户再次打开浏览器访问服务器时，就不会有sessionld 发送给服务器，那么服务器会认为你没有session，所以服务器会创建一个session，并在响应中把sessionld中到Cookie中发送给客户端。
你可能会说，那原来的 session对象会怎样?当一个session长时间没人使用的话，服务器会把session删除了！这个时长在Tomcat中配置是30分钟，可以在
`$CATALANA]/conf/web.xml `找到这个配置，当然你也可以在自己的web.xml中覆盖这个配置

## URL重写

我们知道session依赖Cookie，那么session为什么依赖Cookie呢?因为服务器需要在
每次请求中获取sessionld，然后找到客户端的session对象。那么如果客户端浏览器关闭了Cookie呢?那么session是不是就会不存在了呢?
其实还有一种方法让服务器收到的每个请求中都带有sessioinld，那就是URL重写！