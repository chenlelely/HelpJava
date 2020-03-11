# Listener

## 监听器概述

在JavaWeb被监听的事件源为：**ServletContext、HttpSession、ServletRequest**,即**三大域对象**。

- 监听域对象“创建”与“销毁”的监听器；
- 监听域对象“操作域属性”的监听器；
- 监听HttpSession的监听器。

## 创建与销毁监听器
创建与销毁监听器一共有三个：

- ServletContextListener:**Tomcat启动和关闭时调用下面两个方法**
  - public void contextlnitialized(ServletContextEvent evt):ServletContext对象被创建后调用：
  - public void contextDestroyed(ServletContextEvent evt):ServletContext对象被销毁前调用；
- HttpSessionListener:**开始会话和结束会话时调用下面两个方法**
  - public void sessionCreated(HttpSessionEvent evt):HttpSession 对象被创建后调用；
  - public void sessionDestroyed(HttpSessionEvent evt):HttpSession对象被销毁前调用：
- ServletRequestListener:**开始请求和结束请求时调用下面两个方法**
  - public void requestinitiallized(ServletRequestEvent evt):ServletRequest 对象被创建后调用
  - public yoid requestDestroyed(ServletRequestEvent evt):ServletRequest 对象被销毁前调用。

### 事件对象

1. ServletContextEvent:ServletContext getServletContext()；
2. HttpSessionEvent:HttpSession getSession()；
3. ServletRequestEvent:
   1. ServletRequest getServletRequest()
   2. ServletContext getServletContext()

## 操作域属性的监听器
当对域属性进行增、删、改时，执行的监听器一共有三个：

- ServletContextAttributelistener:**在ServletContext域进行增、删、改属性时调用下面方法**。
  - public void attributeAdded(ServletContextAttributeEvent evt)
  - public void attributeRemoved(ServletContextAttributeEvent evt)
  - public void attributeReplaced(ServletContextAttributeEvent evt)
- HttpSessionAttributeListener:**在HttpSession域进行增、删、改属性时调用下面方法**
  - public void attributeAdded(HttpSessionBindingEvent evt)
  - public void attributeRemoved(HttpSessionBindingEvent evt)
  - public void attributeReplaced(HttpSessionBindingEvent evt)
- ServletRequestAttributeListener:**在ServletRequest域进行增、删、改属性时调用下面方法**
  - public void attributeAdded(ServletRequestAttributeEvent evt)
  - public void attributeRemoved(ServletRequestAttributeEvent evt)
  - public void attributeReplaced(ServletRequestAttributeEvent evt)

下面对这三个监听器的**事件对象**功能进行介绍：

- ServletContextAttributeEvent
  - String getName（)：获取当前操作的属性名；
  - Object getValue（):获取当前操作的属性值；
  - ServletContext getServletContext():获取 ServletContext对象。
- HttpSessionBindingEvent
  - String getName（):获取当前操作的属性名；
  - Object getValue():获取当前操作的属性值；
  - HttpSession getsession():获取当前操作的 session对象。
- ServletRequestAttributeEvent
  - String getName（):获取当前操作的属性名；
  - object getValue（):获取当前操作的属性值；
  - ServletContext getServletContext():获取ServletContext对象；
  - ServletRequest getServletRequest():获取当前操作的ServletRequest对象。

如果是替换，那么getValue返回的是原值；如果想拿到新值，那么去对应的域对象中取。

## HttpSession的监听器

还有两个与HttpSession相关的特殊的监听器，这两个监听器的特点如下：

- 不用在web.xml文件中部署
- 这两个监听器不是给session添加，而是给Bean添加。即**让Bean 类实现监听器接口，然后再把Bean 对象添加到session域中**。

下面对这两个监听器介绍一下：

- HttpSessionBindinglistener:当某个类实现了该接口后，可以感知本类对象添加到session中，以及感知从session中移除。例如让Person类实现HttpSessionBindingListener接口，那么当把Person 对象添加到session中，或者把Person对象从session中移除时会调用下面两个方法：
  - public void valueBound(HttpSessionBindingEvent event):当把监听器对象添加到session中会调用监听器对象的本方法；
  - public void valueUnbound(HttpSessionBindingEvent event):当把监听器对象从session中移除时会调用监听器对象的本方法；

这里要注意，HttpSessionBindinglistener监听器的使用与前面介绍的都不相同，当该监听器对象添加到session中，或把该监听器对象从session移除时会调用监听器中的方法。并且无需在web.xml文件中部署这个监听器。

**Session 有一种重生的性质。**

> 在服务器开启时保存了session，然后当服务器被关闭时，会在硬盘上保存一个session属性的序列化文件。启动服务器时会重新将该序列化文件读入内存，并删除该文件。

在conf/context.xml中进行一些配置可以取消保存session序列化文件的操作。

- HttpSessionActivationListener:Tomcat 会在session 不被使用时钝化 session对象，所谓**饨化session，就是把 session通过序列化的方式保存到硬盘文件中**。当用户再使用session时，Tomcat 还会把钝化的对象再活化 session，所谓**活化就是把硬盘文件中的 session在反序列化回内存**。当session被Tomcat钝化时，session中存储的对象也被钝化，当 session被活化时，也会把session中存储的对象活化。如果某个类实现了
  HttpSessionActivationListener 接口后，当对象随着 session被钝化和活化时，下面两个方法就会被调用：
  - public void sessionWillPassivate(HttpSessionEvent se):当对象感知被活化时调用本方法；
  - public void sessionDidActivate(HttpSessionEvent se):当对象感知被钝化时调用本方法；

HttpSessionActivationListener 监听器与HttpSessionBindingListener监听器相似，都是感知型的监听器，例如让Person 类实现了HttpSessionActivationListener 监听器接口，并把 Person对象添加到了session中后，当Tomcat钝化session时，同时也会钝化session中的Person对象，这时Person对象就会感知到自己被钝化了，其实就是调用Person对象的sessionWillPassivate（)方法。当用户再次使用session时，Tomcat会活化session，这时Person会感知到自己被活化，其实就是调用Person对象的 sessionDidActivatel）方法。

注意**JavaBean 同时需要实现Serializable接口**。

注意，因为钝化和活化session，其实就是使用序列化和反序列化技术把 session从内存保存到硬盘，和把session从硬盘加载到内存。这说明如果Person类没有实现Serializable接口，那么当session钝化时就不会钝化Person，而是把Person从session中移除再钝化！这也说明session活化后，session中就不再有Person对象了。