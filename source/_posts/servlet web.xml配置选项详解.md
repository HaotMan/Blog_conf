---
title: servlet web.xml配置选项详解
date: 2018/10/31 19:52:58 
categories: 

- [web后端, servlet, java]

tags: 

- java
- web后端
- servlet

permalink: servlet-web-xml

---

一般的web工程中都会用到web.xml，web.xml主要包括一些配置标签，例如Filter、Listener、Servlet等，可以用来预设容器的配置，可以方便的开发web工程。但是web.xml并不是必须的，一个web工程可以没有web.xml文件

<!--more--> 

#### \<web-app\>

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.4" 
    xmlns="http://java.sun.com/xml/ns/j2ee" 
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee 
        http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd">
</web-app>
```
这是整个配置文件的根标签，web.xml的模式文件是由Sun公司定义的，它必须标明web.xml使用的是哪个模式文件。

----
#### \<display-name\>

```xml
<display-name>serTest</display-name>
```
它标注了该web项目的名字，提供GUI工具可能会用来标记这个特定的Web应用的一个名称

---
#### \<welcome-list-file\>

```xml
  <welcome-file-list>
    <welcome-file>index.html</welcome-file>
    <welcome-file>index.htm</welcome-file>
    <welcome-file>index.jsp</welcome-file>
    <welcome-file>default.html</welcome-file>
    <welcome-file>default.htm</welcome-file>
    <welcome-file>default.jsp</welcome-file>
  </welcome-file-list>
```
\<welcome-file-list\>定义了首页文件，也就是用户直接输入域名时跳转的页面（如http://localhost:8080/）

----
#### \<servlet>
用来声明一个servlet的数据，主要有以下子元素：

- **\<servlet-name>** 
  指定servlet的名称
- **\<servlet-class>** 
  指定servlet的类名称
- **\<jsp-file></jsp-file>** 
  指定web站台中的某个JSP网页的完整路径
- **\<init-param></init-param>** 
  用来定义初始化参数，可有多个init-param。在servlet类中通过ServletConfig对象传入init函数，通过- getInitParamenter(String name)方法访问初始化参数
  例如使用\<init-param>来初始化数据库连接参数
```java
public void init(ServletConfig config) throws SevletException{
	super(config);
	String driver = config.getInitParameter("driver");
	String url = config.getInitParameter("url");
	String username = config.getInitParameter("username");
	String passwd = config.getInitParameter("passwd");
	try{
		Class.forName(driver).newInstance();
		this.conn = DriverManager.getConnection(url, username, passwd);
		System.out.println("Connection successful...");
	} catch(SQLExceprion se){
		System.out.println("se");
	} catch(Exception e){
		e.printStackTrace():
	}
	
}
```
此时servlet配置为

```xml
<servlet>
    <servlet-name>myServlet</servlet-name>
    <servlet-class>*.myservlet</servlet-class>
    <init-param>
        <param-name>driver</param-name>
        <param-value>com.mysql.jdbc.Driver</param-value>
    </init-param>
    <init-param>
        <param-name>url</param-name>
        <param-value>jdbc:mysql://localhost:3306/myDatabase</param-value>
    </init-param>
    <init-param>
        <param-name>username</param-name>
        <param-value>tang</param-value>
    </init-param>
    <init-param>
        <param-name>passwd</param-name>
        <param-value>whu</param-value>
    </init-param>
</servlet>
```

- **\<load-on-startup>** 
  指定当Web应用启动时，装载Servlet的次序。当值为正数或零时：Servlet容器先加载数值小的servlet，再依次加载其他数值大的servlet。当值为负或未定义：Servlet容器将在Web客户首次访问这个servlet时加载它。
- **\<servlet-mapping>** 
  用来定义servlet所对应的URL，包含两个子元素
- **\<servlet-name>** 
  指定servlet的名称
- **\<url-pattern>** 
  指定servlet所对应的URL

```xml
<!-- 基本配置 -->
<servlet>
    <servlet-name>myServlet</servlet-name>
    <servlet-class>*.myservlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>myServlet</servlet-name>
    <url-pattern>/myServlet</url-pattern>
</servlet-mapping>


<!-- 高级配置 -->
<servlet>
    <servlet-name>myServlet</servlet-name>
    <servlet-class>*.myservlet</servlet-class>
    <init-param>
        <param-name>foo</param-name>
        <param-value>bar</param-value>
    </init-param>
    <run-as>
        <description>Security role for anonymous access</description>
        <role-name>tomcat</role-name>
    </run-as>
</servlet>
<servlet-mapping>
    <servlet-name>myServlet</servlet-name>
    <url-pattern>/myServlet</url-pattern>
</servlet-mapping>
```
需要特别注意的是\</servlet-mapping>元素，这个元素规定了一个servlet-name和url-pattern，如果请求的url能够匹配该url-pattern，则使用servlet-name指定的servlet处理该请求。

**url-pattern匹配规则**

按照优先级从高到低排列：
1. 精确匹配：类似于/myServlet的精确路径
2. 通配符匹配：/*
3. 扩展名匹配：\*.html，\*.jpg ，.do ，.action之类的
4. 默认匹配（/）——当之前匹配都不成功时

当servlet收到来自客户端的url请求时，先把请求url减去当前项目上下文路径，然后再与url-pattern进行匹配，匹配按照上面列出的优先级顺序进行，只要有一个匹配成功就停止，不再继续匹配其他的。
**（注：/*.action这种匹配式是错误的，容器无法识别同时拥有两种匹配规则的pattern）**

例子：
```xml
<!-- 配置-->
<servlet>
    <servlet-name>myServlet</servlet-name>
    <servlet-class>*.myservlet</servlet-class>
</servlet>
myServlet
    <servlet-name>myServlet</servlet-name>
    <url-pattern>/myServlet</url-pattern>
</servlet-mapping>
```

收到来自客户端的请求http://localhost:8080/serTest/myServlet/index.html，首先将该请求url减去项目上下文得到/myServlet/index.html，之后与url-pattern进行匹配，发现/myServlet匹配成功，之后便把该请求交由servlet-name指定的myServlet处理。

另一方面，\<servlet-mapping>并不是必须的，例如在如下项目中：
![project目录树](https://img-blog.csdnimg.cn/20181031184925645.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01TRE5fdGFuZw==,size_16,color_FFFFFF,t_70)
src文件夹中是定义的两个servlet类（Test和Img）和一个逻辑事务处理类（Output），如果我们都不设置\<servlet-mapping>，则可以像如下方法进行请求
请求使用Test处理：http://localhost:8080/serTest/Test
请求使用Img处理：http://localhost:8080/serTest/Img
请求静态图片文件：http://localhost:8080/serTest/img/img.jpg

----

#### \<filter>
过滤器元素将一个名字与一个实现javax.servlet.Filter接口的类相关联。

----

#### \<listener>
 Listener元素指出事件监听程序类

----
#### \<session-config>
配置会话超时，单位是分钟

```xml
<session-config>
    <session-timeout>120</session-timeout>
</session-config>
```
----

#### \<error-page>
 在返回特定HTTP状态代码时，或者特定类型的异常被抛出时，能够制定将要显示的页面。 


更多元素参见 https://blog.csdn.net/xiuwu0423/article/details/54861184