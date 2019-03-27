---
layout: post
title: spring和springMvc
categories: spring
description: spring和springMvc。
keywords: spring, springMvc
---

*目录*
* Toc
{:toc}

# spring
## 先看下spring.io
<p>
从配置管理到安全性、web应用程序到大数据处理，无论你的项目需要什么样的基础设施，总有一个 Spring Project 能帮助到你构建你的项目。从小处开始，只使用你需要的部分–Spring是模块化设计的。（翻译自spring-projects首页：https://spring.io/projects）
</p>

<img src="/images/blog/spring-springMvc/some-spring-projects.png" alt="部分spring projects" width="80%" height="80%"/>

## 平时常用的spring framework
<p>
平时说的spring，其实大部分意思是这个project而已，因为这个里面包含了最核心的"IoC container"部分。

springMvc是spring的一个模块。
</p>

<img src="/images/blog/spring-springMvc/spring-framework.png" alt="spring-framework" width="100%" height="100%"/>

# springMvc
## Web on Servlet Stack
<p>
Spring Web MVC是基于Servlet API构建的原始Web框架，从一开始就被包含在Spring框架中。
正式名称“Spring Web MVC”来自它的源模块（Spring-Web MVC）的名称，但是它通常被称为“Spring MVC”。
<br>
参考：https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/index.html
</p>

<p>
与许多其他web框架一样，Spring MVC是围绕前端控制器模式（front controller pattern）设计的，
其中 DispatcherServlet 提供了一个共享的请求处理算法，而实际工作则由可配置的委托组件执行。
</p>

<p>
与任何Servlet一样，DispatcherServlet需要通过使用Java配置或在web.xml中声明和映射，并根据Servlet规范进行映射。反过来，DispatcherServlet使用Spring配置来发现它在请求映射、视图解析、异常处理等方面所需要的委托组件。
</p>

<p>
对于许多应用程序来说，拥有一个单一的WebApplicationContext是简单而足够的。
Spring Web Mvc 还支持上下文层次结构，其中一个 Root WebApplicationContext 跨多个 DispatcherServlet（或其他Servlet）实例共享。

Root WebApplicationContext 通常包含基础设施 beans，比如数据存储库和业务服务，它们可以跨多个 Servlet 实例被共享。
这些 beans 可以被继承，并且可以在特定于 Servlet 的子 WebApplicationContext 中复盖（即重新声明）。
下面的图片显示了这种关系：
</p>

<img src="/images/blog/spring-springMvc/mvc-context-hierarchy.png" alt="mvc-context-hierarchy" width="80%" height="80%"/>

## Servlet Config

下面的web.xml配置示例注册并初始化了DispatcherServlet：
```html
<web-app>

    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/root-context.xml</param-value>
    </context-param>

    <servlet>
        <servlet-name>app1</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>/WEB-INF/app1-context.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>app1</servlet-name>
        <url-pattern>/app1/*</url-pattern>
    </servlet-mapping>

</web-app>
```

基于Java代码（Java-based）的配置方式，spring推荐的方式。

下面的基于顶级接口 **WebApplicationInitializer** 配置DispatcherServlet，它会被Servlet容器自动检测到：
```java
public class MyWebApplicationInitializer implements WebApplicationInitializer {

    @Override
    public void onStartup(ServletContext servletCxt) {

        // Load Spring web application configuration
        AnnotationConfigWebApplicationContext ac = new AnnotationConfigWebApplicationContext();
        ac.register(AppConfig.class);
        ac.refresh();

        // Create and register the DispatcherServlet
        DispatcherServlet servlet = new DispatcherServlet(ac);
        ServletRegistration.Dynamic registration = servletCxt.addServlet("app", servlet);
        registration.setLoadOnStartup(1);
        registration.addMapping("/app/*");
    }
}
```
```java
import org.springframework.web.WebApplicationInitializer;

public class MyWebApplicationInitializer implements WebApplicationInitializer {

    @Override
    public void onStartup(ServletContext container) {
        XmlWebApplicationContext appContext = new XmlWebApplicationContext();
        appContext.setConfigLocation("/WEB-INF/spring/dispatcher-config.xml");

        ServletRegistration.Dynamic registration = container.addServlet("dispatcher", new DispatcherServlet(appContext));
        registration.setLoadOnStartup(1);
        registration.addMapping("/");
    }
}
```

基于抽象类 **AbstractDispatcherServletInitializer** 配置DispatcherServlet:
```java
public class MyWebAppInitializer extends AbstractDispatcherServletInitializer {

    @Override
    protected WebApplicationContext createRootApplicationContext() {
        return null;
    }

    @Override
    protected WebApplicationContext createServletApplicationContext() {
        XmlWebApplicationContext cxt = new XmlWebApplicationContext();
        cxt.setConfigLocation("/WEB-INF/spring/dispatcher-config.xml");
        return cxt;
    }

    @Override
    protected String[] getServletMappings() {
        return new String[] { "/" };
    }
}
```
还可以注册 **Filter**
```java
public class MyWebAppInitializer extends AbstractDispatcherServletInitializer {

    // ...

    @Override
    protected Filter[] getServletFilters() {
        return new Filter[] {
            new HiddenHttpMethodFilter(), new CharacterEncodingFilter() };
    }
}
```

基于抽象类 **AbstractAnnotationConfigDispatcherServletInitializer** 配置DispatcherServlet:
```java
public class MyWebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

    @Override
    protected Class<?>[] getRootConfigClasses() {
        return null;
    }

    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class<?>[] { MyWebConfig.class };
    }

    @Override
    protected String[] getServletMappings() {
        return new String[] { "/" };
    }
}
```

# 什么是Servlet
## Java Servlet 是工业标准（standard）
有两个大的版本：
```java
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>servlet-api</artifactId>
    <version>2.5</version>
    <scope>provided</scope>
</dependency>
```
```java
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>3.1.0</version>
    <scope>provided</scope>
</dependency>
```
<p>
3.0版本之后用了新的 artifactId。
</p>
什么是标准，标准就是**接口**，定义了流程，定义了规范。

## Servlet Container
**Web 服务器**
<p>
Web 服务器使用 HTTP 协议来传输数据。最简单的一种情况是，用户在浏览器中输入一个URL（如，www.example.com/index.html），然后就能获取网页进行阅读。 
因此，Web服务器完成的工作就是发送网页至客户端。传输过程遵循 HTTP 协议，它指明了请求（request）消息和响应（response）消息的格式。 
</p>
<img src="/images/blog/spring-springMvc/web-server.jpg" alt="部分spring projects" width="60%" height="60%"/>
<br>

**Servlet 容器**
<p>
Servlet 容器为处理每个请求分配独立的 Java 线程。 
每一个 Servlet 都是一个拥有能处理 HTTP 请求并作出响应的 Java 类。 
Servlet 容器的主要作用是将请求转发给相应的 Servlet 进行处理，并将动态生成的结果返回至客户端。 
和所有的 Java 程序一样，Servlet 容器运行在 JVM 中。引入 Servlet 容器是为了处理复杂的 HTTP 请求。Servlet 容器负责 Servlet 的创建、执行和销毁。
</p>
<img src="/images/blog/spring-springMvc/web-server-servlet-container.jpg" alt="部分spring projects" width="60%" height="60%"/>
<br>

**目前最流行的Servlet容器**

Tomcat
<p>
Tomcat和IIS等Web服务器一样，具有处理HTML页面的功能，另外它还是一个Servlet和JSP容器，独立的Servlet容器是Tomcat的默认模式。不过，Tomcat处理静态HTML的能力不如Apache服务器。
</p>

Jetty
<p>
Jetty 是一个开源的servlet容器，它为基于Java的web容器，例如JSP和servlet提供运行环境。Jetty是使用Java语言编写的，它的API以一组JAR包的形式发布。开发人员可以将Jetty容器实例化成一个对象，可以迅速为一些独立运行（stand-alone）的Java应用提供网络和web连接。
</p>

Jboss
<p>
Jboss是一个基于J2EE的开放源代码的应用服务器。 JBoss代码遵循LGPL许可，可以在任何商业应用中免费使用。JBoss是一个管理EJB的容器和服务器，支持EJB 1.1、EJB 2.0和EJB3的规范。但JBoss核心服务不包括支持servlet/JSP的WEB容器，一般与Tomcat或Jetty绑定使用。
<p>

# 配置Servlet