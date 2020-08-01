# Cookie

## 概述

- 网页之间的**交互是通过HTTP协议传输数据的，**而Http协议是**无状态的协议**。无状态的协议是什么意思呢？**一旦数据提交完后，浏览器和服务器的连接就会关闭，再次交互的时候需要重新建立新的连接**。
- 服务器无法确认用户的信息，于是乎，W3C就提出了：**给每一个用户都发一个通行证，无论谁访问的时候都需要携带通行证，这样服务器就可以从通行证上确认用户的信息**。通行证就是Cookie

Cookie由 HTTP 协议制定，由一个键和一个值构成。

Cookie 是由服务器创建，浏览器访问服务器，**如果服务器需要记录该用户的状态，就使用response向浏览器发送一个Cookie**。 客户端会保存Cookie， 并会标注出 Cookie 的来源（ 哪个服务器的 Cookie）。 当客户端向服务器发出请求时会把所有这个服务器 Cookie 包含在请求中发送给服务器， 这样服务器就可以识别客户端了  

## Cookie API

- Cookie类用于创建一个Cookie对象
- response接口中定义了一个addCookie方法，它用于在其响应头中增加一个相应的Set-Cookie头字段
- request接口中定义了一个getCookies方法，它用于获取客户端提交的Cookie

常用的Cookie方法：

- public Cookie(String name,String value)
- setValue与getValue方法
- setMaxAge与getMaxAge方法
- setPath与getPath方法
- setDomain与getDomain方法
- getName方法

## 简单使用Cookie

- 创建Cookie对象，发送Cookie给浏览器

```java
//设置response的编码
response.setContentType("text/html;charset=UTF-8");
//创建Cookie对象，指定名称和值
Cookie cookie = new Cookie("username","xiaoming");
//必须设置Cookie时间
cookie.setMaxAge(1000);
//向浏览器发送Cookie
response.addCookie(cookie);
response.getWriter().write("已经发送一个Cookie");
```

## Cookie的有效期

Cookie在客户端的有效时间，可以通过`setMaxAge(int)`来设置 cookie的有效时间。

- `cookie.setMaxAge(-1)`:cookie的maxAge属性的默认值就是-1，表示只在浏览器内存中存活。一且关闭浏览器窗口，那么cookie就会消失。
- `cookie.setMaxAge(60*60)`：表示cookie对象可存活1小时。当生命大于0时，浏览器会把Cookie保存到硬盘上，就算关闭浏览器，就算重启客户端电脑，cookie也会存活1小时；
- `cookie.setMaxAge(0)`：cookie 生命等于0是一个特殊的值，它表示 cookie 被作废！也就是说，如果原来浏览器已经保存了这个Cookie，那么可以通过cookie的setMaxAge(0)来删除这个cookie。无论是在浏览器内存中，还是在客户端硬盘上都会删除这个Cookie。

## Cookie的修改和删除

- 上面我们已经知道了Cookie机制没有提供删除Cookie的方法。其实细心点我们可以发现，Cookie机制也没有提供修改Cookie的方法。那么我们**怎么修改Cookie的值呢**？
- Cookie存储的方式**类似于Map集合**，**Cookie的名称相同，通过response添加到浏览器中，会覆盖原来的Cookie**。
- 要删除该Cookie，**把MaxAge设置为0，并添加到浏览器中即可**
- 删除，修改Cookie时，**新建的Cookie除了value、maxAge之外的所有属性都要与原Cookie相同**。否则浏览器将**视为不同的Cookie，不予覆盖，导致删除修改失败**！

## Cookie 的路径Path

可以通过设置 Cookie 的 path 来指定浏览器**允许访问Cookie的路径**。 请求路径如果包含了 Cookie 路径， 那么会在请求中包含这个 Cookie  

设置 Cookie的路径需要使用`setPath()`方法，例如 :`cookie.setPath("/cookietest/servlet");`

默认路径是访问资源的上一级路径，如果没有设置Cookie的路径，那么Cookie路径的默认值当前访问资源所在路径（上一级），例如：

- 访问http:/localhost:8080/cookietest/ASerlet时添加的 Cookie默认路径为`/cookietest`;
- 访问http:/localhost:8080/cookietest/servlet/BServlet时添加的Cookie默认路径为`/cookietest/servlet`；
- 访问http:/localhost:8080/cookietest/jsp/BServlet 时添加的Cookie默认路径为`/cookietest/jsp`;

## Cookie的域名domain

Cookie的**domain属性决定运行访问Cookie的域名。domain的值规定为“.域名”**，可以让网站中二级域共享 Cookie。  

Cookie的隐私安全机制决定Cookie是不可跨域名的。也就是说www.baidu.com和www.google.com之间的Cookie是互不交接的。**即使是同一级域名，不同二级域名也不能交接**，也就是说：www.goole.com和www.image.goole.com的Cookie也不能访问

想让`http://www.baidu.com`和`http://zhidao.baidu.com ` 主机之间共享Cookie，可以使用domain进行设置：

- ```java
  Cookie cookie = new Cookie("username","xiaoming");
  cookie.setMaxAge(1000);
  cookie.setDomain(".baidu.com");
  response.addCookie(cookie);
  //只要一级域名是“.baidu.com”就可以访问
  ```



## Cookie的应用

### 显示用户上次访问的时间

其实就是**每次登陆的时候，取到Cookie保存的值，再更新下Cookie的值**。

```java
SimpleDateFormat simpleDateFormat =new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
response.setContentType("text/html;charset=UTF-8");
PrintWriter printWriter = response.getWriter();
//获取网页上所有的Cookie
Cookie[] cookies = request.getCookies();
//判断Cookie的值是否为空
String cookieValue =null;
for(int i = 0; cookies !=null&& i < cookies.length; i++){
//获取到以time为名的Cookie
    if(cookies[i].getName().equals("time")) {
        printWriter.write("您上次登陆的时间是：");
        cookieValue = cookies[i].getValue();
        printWriter.write(cookieValue);
        cookies[i].setValue(simpleDateFormat.format(new Date()));
        response.addCookie(cookies[i]);
    //既然已经找到了就可以break循环了
        break;
    }
}
//若果cookieValue的值为空，则为第一次登陆
if(cookieValue==null){
    Cookie cookie = new Cookie("time",simpeDateFormat.format(new Date()));
    cookie.setMaxAge(20000);
    response.addCookie(cookie);
    printtWriter.write("第一次访问");
}
```



# Session

Session 是另一种记录浏览器状态的机制。不同的是Cookie保存在浏览器中，Session保存在服务器中。用户使用浏览器访问服务器的时候，服务器把用户的信息以某种的形式记录在服务器，这就是Session

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
- HttpSession： **一个会话**创建一个 HttpSession 对象， 同一会话中的多个请求中可以共享 session 中的数据；  

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

我们知道session依赖Cookie，那么session为什么依赖Cookie呢?因为服务器需要在每次请求中获取sessionld，然后找到客户端的session对象。那么如果客户端浏览器关闭了Cookie呢?那么session是不是就会不存在了呢?

其实还有一种方法让服务器收到的每个请求中都带有sessioinld，那就是URL重写！

- HttpServletResponse类提供了两个URL地址重写的方法：

- - **encodeURL(String url)**
  - **encodeRedirectURL(String url)**

```java
String url = "/omg/servlet2";
response.sendRedirect(response.encodeURL(url));
```

> 需要值得注意的是：这两个方法会自动判断该浏览器是否支持Cookie，如果支持Cookie，重写后的URL地址就不会带有jsessionid了【当然了，即使浏览器支持Cookie，第一次输出URL地址的时候还是会出现jsessionid（因为没有任何Cookie可带）】

URL地址重写的原理：**将Session的id信息重写到URL地址中**。**服务器解析重写后URL，获取Session的id**。这样一来，即使浏览器禁用掉了Cookie，但Session的id通过服务器端传递，还是可以使用Session来记录用户的状态。

## Session API

- long getCreationTime();【获取Session被创建时间】
- **String getId();【获取Session的id】**
- long getLastAccessedTime();【返回Session最后活跃的时间】
- ServletContext getServletContext();【获取ServletContext对象】
- **void setMaxInactiveInterval(int var1);【设置Session超时时间】**
- **int getMaxInactiveInterval();【获取Session超时时间】**
- **Object getAttribute(String var1);【获取Session属性**】
- Enumeration getAttributeNames();【获取Session所有的属性名】
- **void setAttribute(String var1, Object var2);【设置Session属性】**
- **void removeAttribute(String var1);【移除Session属性】**
- **void invalidate();【销毁该Session】**
- boolean isNew();【该Session是否为新的】

## Session作为域对象

从上面的API看出，Session有着request和ServletContext类似的方法。其实**Session也是一个域对象**。Session作为一种记录浏览器状态的机制，**只要Session对象没有被销毁，Servlet之间就可以通过Session对象实现通讯**

## Session的生命周期和有效期

- Session在用户**第一次访问服务器Servlet，jsp等动态资源就会被自动创建，Session对象保存在内存里**，这也就为什么可以**直接使用request对象获取得到Session对象**。
- 如果访问HTML,IMAGE等静态资源Session不会被创建。
- Session生成后，只要用户继续访问，服务器就会更新Session的最后访问时间，无论**是否对Session进行读写，服务器都会认为Session活跃了一次**。
- 由于会有越来越多的用户访问服务器，因此Session也会越来越多。**为了防止内存溢出，服务器会把长时间没有活跃的Session从内存中删除，这个时间也就是Session的超时时间**。
- Session的超时时间默认是30分钟，有三种方式可以对Session的超时时间进行修改
  - **第一种方式：**在tomcat/conf/web.xml文件中设置，时间值为20分钟，**所有的WEB应用都有效**
  - **第二种方式：**在单个的web.xml文件中设置，对单个web应用有效，**如果有冲突，以自己的web应用为准**。
  - **第三种方式：**通过setMaxInactiveInterval()方法设置

- Session的有效期与Cookie的是不同的
  - session周期指的是不活动的时间，如果我们设置session是10s，在10s内，没有访问session，session中属性失效，如果在9s的时候，你访问session，则重新计时
  - 如果重启了tomcat，或者reload web应用，或者关机了，session也会失效
    我们也可以通过函数让session失效，`invalidate()`该方法是让session中的所有属性失效，常常用于安全退出
  - 如果你希望某个session属性失效，可以使用方法`removeAttribute()`
  - cookie的生命周期就是按累计的时间来算的。不管用户有没有访问过session

# Session和Cookie的区别

- **从存储方式上比较**

- - Cookie只能存储字符串，如果要存储非ASCII字符串还要对其编码。
  - Session可以**存储任何类型的数据**，可以把Session看成是一个容器

- **从隐私安全上比较**

  - *Cookie存储在浏览器中，对客户端是可见的**。信息容易泄露出去。如果使用Cookie，最好将Cookie加密

- **从有效期上比较**

  - Cookie保存在硬盘中，只需要设置maxAge属性为比较大的正整数，即使关闭浏览器，Cookie还是存在

- 

  **从对服务器的负担比较**

- - Session是保存在服务器的，每个用户都会产生一个Session，如果是并发访问的用户非常多，是不能使用Session的，Session会消耗大量的内存。
  - Cookie是保存在客户端的。不占用服务器的资源。像baidu、Sina这样的大型网站，一般都是使用Cookie来进行会话跟踪。

- **从浏览器的支持上比较**

- - 如果浏览器禁用了Cookie，那么Cookie是无用的了！
  - 如果浏览器禁用了Cookie，Session可以通过URL地址重写来进行会话跟踪。

- **从跨域名上比较**

- - Cookie可以设置**domain属性来实现跨域名**
  - Session只在当前的域名内有效，**不可夸域名**

------

# Cookie和Session共同使用

**如果仅仅使用Cookie或仅仅使用Session可能达不到理想的效果**。这时应该尝试一下同时使用Session和Cookie

那么，什么时候才需要同时使用Cookie和Session呢？

Session来进行简单的购物，功能也的确实现有一个问题：**我在购物的途中，不小心关闭了浏览器。当我再返回进去浏览器的时候，发现我购买过的商品记录都没了！！为什么会没了呢？原因也非常简单：服务器为Session自动维护的Cookie的maxAge属性默认是-1的，当浏览器关闭掉了，该Cookie就自动消亡了。当用户再次访问的时候，已经不是原来的Cookie了。**

我们现在想的是：**即使我不小心关闭了浏览器了，我重新进去网站，我还能找到我的购买记录**。

要实现该功能也十分简单，问题其实就在：服务器为Session自动维护的Cookie的maxAge属性是-1，Cookie没有保存在硬盘中。我现在要做的就是：**把Cookie保存在硬盘中，即使我关闭了浏览器，浏览器再次访问页面的时候，可以带上Cookie，从而服务器识别出Session。**

- **第一种方式：**只需要在处理购买页面上创建Cookie，Cookie的值是Session的id返回给浏览器即可

```
Cookie cookie = new Cookie("JSESSIONID",session.getId());      
cookie.setMaxAge(30*60);        
cookie.setPath("/shopping/");        
response.addCookie(cookie);
```

- **第二种方式：** **在server.xml文件中配置，将每个用户的Session在服务器关闭的时候序列化到硬盘或数据库上保存**。但此方法不常用，知道即可！