---
layout: post
title: 如何为 SpringMVC 编写单元测试：从配置开始
category : junit
tags : [JUnit, Java, Testing, SpringMVC]
---
{% include JB/setup %}

为 SpringMVC 编写测试用例通常被认为是既简单又复杂的。

虽然直接写调用Controller方法的测试用例不难，但问题在于这些测试用例不够全面。

例如，仅通过直接的方法调用我们测不到 Controller 的映射、校验和异常处理。

SpringMVC 提供给我们通过 DispathServlet 调用 Controller 方法的能力解决了这个问题。

这篇文章是本系列 SpringMVC 单元测试指南的第一部分，主要说明了如何对单元测试做相关配置。

现在开始吧。

## 使用 Maven 获取相关依赖 ##

我们可以通过在 pom.xml 中声明以下来获取相关依赖：

- JUnit 4.11
- Mockito Core 1.9.5
- Spring Test 3.2.3.RELEASE

体现在 pom.xml 中的相关部分看起来应该是这样的：

{% highlight xml linenos %}
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.11</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-core</artifactId>
    <version>1.9.5</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>3.2.3.RELEASE</version>
    <scope>test</scope>
</dependency>
{% endhighlight %}

> **注意**: 如果你使用的是 Spring Framework 3.1，你可以使用 spring-test-mvc 写测试用例。这个模块从 Spring Framework 3.2 开始就默认包含了。

让我们继续看看我们的例子程序。

## 实例解析 ##

本指南中的样例程序为 Todo 实体类提供了 CRUD 的功能。为了帮助理解测试类的相关配置，我们必须对被测试类有点基本了解。

此时此刻, 我们需要知道这几个问题的答案:

- 它有哪些依赖?
- 它是如何初始化的?

我们可以看看 TodoController 类的源代码来查找问题的答案。相关的部分是这样的：

{% highlight java linenos %}
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.MessageSource;
import org.springframework.stereotype.Controller;
 
@Controller
public class TodoController {
 
    private final TodoService service;
    private final MessageSource messageSource;
 
    @Autowired
    public TodoController(MessageSource messageSource, TodoService service) {
        this.messageSource = messageSource;
        this.service = service;
    }
 
    //Other methods are omitted.
}
{% endhighlight %}

正如我们看到的，这个测试类有两个依赖：TodoService 和 MessageSource。而且，我们可以知道这个测试类使用了构造方法注入。

现在我们已经知道了我们想知道的。下面我们讨论下程序执行环境的设置。

## 执行环境设置 ##

为我们的应用程序和测试程序维护不同的执行环境是很麻烦的。而且，如果我们修改了应用程序需要的执行环境而忘了对测试环境做对应修改就可能导致某些问题。

这也是为什么我们的样例应用程序执行环境配置要做以下拆分，只有这样我们才能在测试中重用它们。

应用程序执行环境拆分规则:
- 第一个配置类是 ExampleApplicationContext，它也是我们的主要配置类。
- 第二个配置类是 WEB 展示层的配置信息。它的名字是 WebAppContext 而且它也是我们将会在我们的测试环境中可能使用的配置类。
- 第三个配置类叫 PersistenceContext ，它持有我们应用程序的持久层配置。

> **注意**: 样例程序还有一个通过 XML 配置文件定义的工作环境。和上文的几个 JAVA 配置类相同功能的配置文件是：exampleApplicationContext.xml、exampleApplicationContext-web.xml 和 exampleApplicationContext-persistence.xml。

现在让我们看看 WEB 层的环境配置，并思考一下如何为测试环境作相应配置。

## 配置 WEB 层执行环境 ##

WEB 层配置项主要有以下功能：

1. 激活 Spring MVC 注解驱动。
2. 配置 CSS 和 Javascript 等静态文件的所在目录。
3. 确保静态文件会被容器的默认 Servlet 处理。
4. 确保 Controller 类会在组件扫描过程中被找到。
5. 配置 ExceptionResolver 对象。
6. 配置 ViewResolver 对象。

现在继续，让我们看看对应的 Java 类配置和 XML 文件配置。

### JAVA 配置 ###

如果我们通过 Java 类配置，那么 WebAppContext 类的代码看起来是这样的：

{% highlight java linenos %}
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.ViewResolver;
import org.springframework.web.servlet.config.annotation.DefaultServletHandlerConfigurer;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.config.annotation.ResourceHandlerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;
import org.springframework.web.servlet.handler.SimpleMappingExceptionResolver;
import org.springframework.web.servlet.view.InternalResourceViewResolver;
import org.springframework.web.servlet.view.JstlView;
 
import java.util.Properties;
 
@Configuration
@EnableWebMvc
@ComponentScan(basePackages = {
        "net.petrikainulainen.spring.testmvc.common.controller",
        "net.petrikainulainen.spring.testmvc.todo.controller"
})
public class WebAppContext extends WebMvcConfigurerAdapter {
 
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/static/**").addResourceLocations("/static/");
    }
 
    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }
 
    @Bean
    public SimpleMappingExceptionResolver exceptionResolver() {
        SimpleMappingExceptionResolver exceptionResolver = new SimpleMappingExceptionResolver();
 
        Properties exceptionMappings = new Properties();
        exceptionMappings.put("net.petrikainulainen.spring.testmvc.todo.exception.TodoNotFoundException", "error/404");
        exceptionMappings.put("java.lang.Exception", "error/error");
        exceptionMappings.put("java.lang.RuntimeException", "error/error");
 
        exceptionResolver.setExceptionMappings(exceptionMappings);
 
        Properties statusCodes = new Properties();
 
        statusCodes.put("error/404", "404");
        statusCodes.put("error/error", "500");
 
        exceptionResolver.setStatusCodes(statusCodes);
 
        return exceptionResolver;
    }
 
    @Bean
    public ViewResolver viewResolver() {
        InternalResourceViewResolver viewResolver = new InternalResourceViewResolver();
 
        viewResolver.setViewClass(JstlView.class);
        viewResolver.setPrefix("/WEB-INF/jsp/");
        viewResolver.setSuffix(".jsp");
 
        return viewResolver;
    }
}
{% endhighlight %}

### XML 文件配置 ###

如果我们通过 XML 文件配置，exampleApplicationContext-web.xml 文件中的内容看起来是这样的：

{% highlight xml linenos %}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-3.1.xsd
       http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.1.xsd">
 
    <mvc:annotation-driven/>
 
    <mvc:resources mapping="/static/**" location="/static/"/>
    <mvc:default-servlet-handler/>
 
    <context:component-scan base-package="net.petrikainulainen.spring.testmvc.common.controller"/>
    <context:component-scan base-package="net.petrikainulainen.spring.testmvc.todo.controller"/>
 
    <bean id="exceptionResolver" class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
        <property name="exceptionMappings">
            <props>
                <prop key="net.petrikainulainen.spring.testmvc.todo.exception.TodoNotFoundException">error/404</prop>
                <prop key="java.lang.Exception">error/error</prop>
                <prop key="java.lang.RuntimeException">error/error</prop>
            </props>
        </property>
        <property name="statusCodes">
            <props>
                <prop key="error/404">404</prop>
                <prop key="error/error">500</prop>
            </props>
        </property>
    </bean>
 
    <bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <property name="suffix" value=".jsp"/>
        <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
    </bean>
</beans>
{% endhighlight %}

## 配置测试执行环境 ##

测试环境配置项主要有以下功能：

1. 它声明了我们的 Controller 和 SpringMVC 用到的 MessageSource 对象。我们之所以这么做的主要原因就是 MessageSource 在应用程序配置中是位于主配置文件中的。
2. 它创建了一个 TodoService 的冒烟对象来注入到 Controller 类中。

现在让我们看看如何通过 Java 类和 XML 分别配置我们的测试环境。

### Java 配置 ###

如果我们通过 Java 类配置，那么 TestContext 类的代码看起来是这样的：

{% highlight java linenos %}
import org.mockito.Mockito;
import org.springframework.context.MessageSource;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.support.ResourceBundleMessageSource;
 
@Configuration
public class TestContext {
 
    @Bean
    public MessageSource messageSource() {
        ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();
        messageSource.setBasename("i18n/messages");
        messageSource.setUseCodeAsDefaultMessage(true);
        return messageSource;
    }
 
    @Bean
    public TodoService todoService() {
        return Mockito.mock(TodoService.class);
    }
}
{% endhighlight %}

### XML 文件配置 ###

如果我们通过 XML 文件配置，testContext.xml 文件中的内容看起来是这样的：

{% highlight xml linenos %}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
 
    <bean id="messageSource" class="org.springframework.context.support.ResourceBundleMessageSource">
        <property name="basename" value="i18n/messages"/>
        <property name="useCodeAsDefaultMessage" value="true"/>
    </bean>
 
    <bean id="todoService" name="todoService" class="org.mockito.Mockito" factory-method="mock">
        <constructor-arg value="net.petrikainulainen.spring.testmvc.todo.service.TodoService"/>
    </bean>
</beans>
{% endhighlight %}

## 配置测试类 ##

我们可以通过下面这些方式来对我们的测试类作相关配置：

1. 独立配置允许我们以编程的方式注册一个或者多个 Controller（通过`@Controller` 注解） 并对 SpringMVC 做些基本设置。在配置简单和直接的情况下这种方式也是可行的。
2. 基于 WebApplicationContext 的配置允许我们通过对 WebApplicationContext 的完全初始化对 SpringMVC 做基本设置。当我们的配置相当复杂，而且不再适合使用独立配置的情况下这种方式是更好的。

现在让我们看看如何将这两种配置方式付诸实践。

### 使用独立配置 ###

我们可以遵循以下几步：

1. 使用 `@RunWith` 注解注释类，并确保测试类会被通过 MockitoJUnitRunner 执行。
2. 在测试类中添加一个 MockMvc 字段。
3. 在测试类中添加一个 TodoService 字段并使用 `@Mock` 注解注释类。
4. 在测试类中添加一个私有的 `exceptionResolver()` 方法。这个方法生成一个配置良好的 SimpleMappingExceptionResolver 对象并返回。
5. 在测试类中添加一个私有的 `messageSource()` 方法。这个方法生成一个配置良好的 ResourceBundleMessageSource 对象并返回。
6. 在测试类中添加一个私有的 `validator()` 方法。这个方法生成并返回一个 LocalValidatorFactoryBean 对象。
7. 在测试类中添加一个私有的 `viewResolver()` 方法。这个方法生成一个配置良好的 InternalResourceViewResolver 对象并返回。
8. 在测试类中添加一个 `setUp()` 方法并使用 `@Before` 注释注解它。这个注解可以确保这个方法会在每个测试执行前被调用。这个方法通过调用 MockMvcBuilders 的 `standaloneSetup()` 方法生成一个 MockMvc 对象并对它做好对应基本配置。

测试类的源代码如下:

{% highlight java linenos %}
import org.junit.Before;
import org.junit.runner.RunWith;
import org.mockito.Mock;
import org.mockito.runners.MockitoJUnitRunner;
import org.springframework.context.MessageSource;
import org.springframework.context.support.ResourceBundleMessageSource;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;
import org.springframework.validation.beanvalidation.LocalValidatorFactoryBean;
import org.springframework.web.servlet.HandlerExceptionResolver;
import org.springframework.web.servlet.ViewResolver;
import org.springframework.web.servlet.handler.SimpleMappingExceptionResolver;
import org.springframework.web.servlet.view.InternalResourceViewResolver;
import org.springframework.web.servlet.view.JstlView;
 
import java.util.Properties;
 
@RunWith(MockitoJUnitRunner.class)
public class StandaloneTodoControllerTest {
 
    private MockMvc mockMvc;
    @Mock
    private TodoService todoServiceMock;
 
    @Before
    public void setUp() {
        mockMvc = MockMvcBuilders.standaloneSetup(new TodoController(messageSource(), todoServiceMock))
                .setHandlerExceptionResolvers(exceptionResolver())
                .setValidator(validator())
                .setViewResolvers(viewResolver())
                .build();
    }
 
    private HandlerExceptionResolver exceptionResolver() {
        SimpleMappingExceptionResolver exceptionResolver = new SimpleMappingExceptionResolver();
 
        Properties exceptionMappings = new Properties();
 
        exceptionMappings.put("net.petrikainulainen.spring.testmvc.todo.exception.TodoNotFoundException", "error/404");
        exceptionMappings.put("java.lang.Exception", "error/error");
        exceptionMappings.put("java.lang.RuntimeException", "error/error");
 
        exceptionResolver.setExceptionMappings(exceptionMappings);

        Properties statusCodes = new Properties();
        statusCodes.put("error/404", "404");
        statusCodes.put("error/error", "500");
        exceptionResolver.setStatusCodes(statusCodes);
        return exceptionResolver;
    }
 
    private MessageSource messageSource() {
        ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();
        messageSource.setBasename("i18n/messages");
        messageSource.setUseCodeAsDefaultMessage(true);
        return messageSource;
    }
 
    private LocalValidatorFactoryBean validator() {
        return new LocalValidatorFactoryBean();
    }
 
    private ViewResolver viewResolver() {
        InternalResourceViewResolver viewResolver = new InternalResourceViewResolver();
 
        viewResolver.setViewClass(JstlView.class);
        viewResolver.setPrefix("/WEB-INF/jsp/");
        viewResolver.setSuffix(".jsp");
 
        return viewResolver;
    }
}
{% endhighlight %}

使用独立配置主要有以下两个问题：

1. 即使我们的 SpringMVC 配置再简单也会导致测试类看起来相当复杂。通常我们会通过把创建 SpringMVC 基础对象的代码挪到一个单独的类中来对它做简化。这步操作就留给读者了。
2. 我们不得不对 SpringMVC 的基础组件做重复配置。这意味着如果我们修改主程序的执行环境，那么必须要对全部测试类做对应修改。

### 使用基于 WebApplicationContext 的配置 ###

基本步骤如下：

1. 使用 `@RunWith` 注解注释类，并确保测试类会被通过 MockitoJUnitRunner 执行。
2. 使用 `@ContextConfiguration` 注解，并确保使用正确的配置类（或者XML配置文件）。如果我们用 Java 类做相关配置，那么就把类名设到 `classes` 属性上。类似，如果我们用 XML 做配置，那么就把配置文件设到 `locations` 属性上。
3. 使用 `@WebAppConfiguration` 注释类。这个注解可以确保为我们的单元测试加载的执行环境是一个 WebApplicationContext。
4. 在测试类中添加一个 MockMvc 字段。
5. 在测试类中添加一个 TodoService 字段，并添加 `@Autowired` 注解。
6. 在测试类中添加一个 WebApplicationContext 字段，并添加 `@Autowired` 注解。
7. 在测试类中添加一个 `setUp()` 方法并添加 `@Befor` 注解。它可以确保这个方法在每个测试执行前被调用。这个方法的主要职责有：在每个测试执行前重置冒烟对象并通过调用 MockMvcBuilders 类的 `webAppContextSetup()` 方法重新生成 MockMvc 对象。

测试类源代码如下:

{% highlight java linenos %}
import org.junit.Before;
import org.junit.runner.RunWith;
import org.mockito.Mockito;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.test.context.web.WebAppConfiguration;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;
import org.springframework.web.context.WebApplicationContext;
 
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = {TestContext.class, WebAppContext.class})
//@ContextConfiguration(locations = {"classpath:testContext.xml", "classpath:exampleApplicationContext-web.xml"})
@WebAppConfiguration
public class WebApplicationContextTodoControllerTest {
 
    private MockMvc mockMvc;
    @Autowired
    private TodoService todoServiceMock;
    @Autowired
    private WebApplicationContext webApplicationContext;
 
    @Before
    public void setUp() {
        //We have to reset our mock between tests because the mock objects
        //are managed by the Spring container. If we would not reset them,
        //stubbing and verified behavior would "leak" from one test to another.
        Mockito.reset(todoServiceMock);
 
        mockMvc = MockMvcBuilders.webAppContextSetup(webApplicationContext).build();
    }
}
{% endhighlight %}

这个测试类的配置信息比通过独立配置的代码简洁不少。但是，它的弊端就是为每个测试都使用了完全的 SpringMVC 基础配置。如果我们测试依赖确实较少的情况下确实会比较消耗性能。

## 总结 ##

我们已经通过独立配置和基于 WebApplicationContext 对我们的测试类做了相应配置。这篇文章也教了我们两件事情：

- 我们知道了通过把执行环境的配置项按易于在测试类中重用的方式做拆分是相当重要的。
- 我们知道了独立配置和基于 WebApplicationContext 配置的区别。




[原文链接](http://www.petrikainulainen.net/programming/spring-framework/unit-testing-of-spring-mvc-controllers-configuration/ "Unit Testing of Spring MVC Controllers: Configuration")