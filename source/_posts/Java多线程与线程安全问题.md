---
title: Java多线程与线程安全问题
date: 2018-10-12 14:54:59
tags: [Java, 线程安全]
---
# Java中的线程问题
## Servlet是线程安全的吗？
Servlet 默认是单例模式，在web 容器中只创建一个实例，所以多个线程同时访问servlet的时候，Servlet是线程不安全的。 
那么 web 容器能为每个请求创建一个Servlet的实例吗？当然是可以的，只要Servlet实现SingleThreadModel接口，就可以了。
```java
import javax.servlet.SingleThreadModel 
public class MyServlet extends HttpServlet implements SingleThreadModel {}
```

## 变量的线程安全
只有全局变量和静态变量才会引起线程安全问题，因为它们存在多线程内存共享；
局部变量是每个对象独有的，不存在变量共享，也就没有线程安全问题；