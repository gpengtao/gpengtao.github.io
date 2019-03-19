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

https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/index.html
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

下面的基于Java代码配置DispatcherServlet，它会被Servlet容器自动检测到：
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


<p>
servlet.jar 是servlet 3.0 版本之前的地址
<br>
javax.servlet-api.jar 是servlet 3.0 版本之后的地址
</p>
