---
layout: post
title: 如何为 SpringMVC 编写单元测试：普通 Controller 测试
category : junit
tags : [JUnit, Java, Testing, SpringMVC]
---
{% include JB/setup %}

前一篇文章我们已经知道如何配置使用了 SpringMVC 测试框架的单元测试。

现在我们就该亲身实践下如何为普通 Controller 编写单元测试了。

接下来一个很明显的问题就是：

> 什么是普通 Controller 

其实，就这篇文章来说普通 Controller 就是指负责渲染界面或处理请求的 Controller。

> 如果你没读过前面的配置篇，那么我建议你先读一下。

## 使用 Maven 获取必须依赖 ##

我们可以通过为我们的样例程序中的 POM 文件添加以下依赖声明来获取必须依赖：

- Jackson 2.2.1 (core 和 databind 模块)。我们使用 Jackson 把对象转化为字符串。
- Hamcrest 1.3。使用它为返回内容写断言。
- JUnit 4.11 (不需要包括 hamcrest-core 依赖).
- Mockito 1.9.5
- Spring Test 3.2.3.RELEASE

体现在 pom.xml 文件中的相关配置如下:

{% highlight xml linenos %}
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-core</artifactId>
    <version>2.2.1</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.2.1</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.hamcrest</groupId>
    <artifactId>hamcrest-all</artifactId>
    <version>1.3</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.11</version>
    <scope>test</scope>
    <exclusions>
        <exclusion>
            <artifactId>hamcrest-core</artifactId>
            <groupId>org.hamcrest</groupId>
        </exclusion>
    </exclusions>
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

接下来让我们看看如何使用 SpringMVC 测试框架为普通 Controller 写单元测试。

## 为 Controller 中方法编写单元测试 ##

我们为 Controller 中方法写的每一个测试都包括以下几个步骤：

1. 往 Controller 中方法发送一个请求。
2. 验证返回的内容是不是符合预期。

SpringMVC 为我们更便捷的实现这几步提供了几个核心类。基本描述如下：

- 我们可以使用 `MockMvcRequestBuilders` 类的静态方法构建请求对象。更确切的说， 我们可以新建一个请求构建对象并把它当作参数传递给请求方法。
- `MockMvc` 类是整个测试的入口。我们可以调用它的 `perform(RequestBuilder requestBuilder)` 方法执行请求。
- 我们可以使用 `MockMvcResultMatchers` 类的表态方法为接收到的返回对象写断言。

接下来让我们看几个如何在测试中使用它们的例子。我们会如下几个方法编写单元测试：

- 第一个 Controller 方法会为 Todo 对象渲染一个列表页。
- 第二个 Controller 方法是为单个 Todo 对象渲染详情页。
- 第三个 Controller 方法接收表单并在数据库中新建 Todo 对象记录。

### 渲染 Todo 对象列表页 ###

让我们先了解下渲染 Todo 对象列表页的代码。

#### 预期行为 ####

用来展示列表的方法实现主要包括以下几步：

1. 接收发送到 '/' 地址的 GET 请求。
2. 调用 TodoService 接口的 `findAll()` 方法获取全部 Todo 对象。这个方法会返回一个包含 Todo 对象的列表。
3. 把接收到的列表添加到 Model 对象中。
4. 返回待渲染视图名称。

相关的 TodoController 类中代码如下:

{% highlight java linenos %}
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.*;
import java.util.List;

@Controller
public class TodoController {

    private final TodoService service;
    
    @RequestMapping(value = "/", method = RequestMethod.GET)
    public String findAll(Model model) {
        List<Todo> models = service.findAll();
        model.addAttribute("todos", models);
        return "todo/list";
    }
}
{% endhighlight %}

现在我们可以开始为这个方法写单元测试了。

#### 测试: 查询到 Todo 对象列表时 ####

我们可以通过以下几步为这个 Controller 方法编写单元测试：

1. 创建 Service 中方法被调用时的返回数据。我们使用一个叫测试数据构建器的概念来表示创建测试数据。
2. 将冒烟对象的 `findAll()` 方法被调用时的返回对象配置成前面创建的测试数据。
3. 往 ‘/’ 地址发送一个 GET 请求。
4. 确认返回的 HTTP 状态码是 200。
5. 确认返回的视图名称是 ‘todo/list’。
6. 确认请求被定向到地址 ‘/WEB-INF/jsp/todo/list.jsp’。
7. 确认 Model 对象中的 todos 属性中有两个元素。
8. 确认 Model 对象中的 todos 属性中的对象都是正确的。
9. 确认冒烟对象的 `findAll()` 方法仅被调用过一次。
10. 确认冒烟对象的其它方法在测试过程中没被调用过。

单元测试源代码如下：

{% highlight java linenos %}
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.mockito.Mockito;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.test.context.web.WebAppConfiguration;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;
import org.springframework.web.context.WebApplicationContext;

import java.util.Arrays;

import static org.hamcrest.Matchers.*;
import static org.hamcrest.Matchers.is;
import static org.mockito.Mockito.*;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.model;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = {TestContext.class, WebAppContext.class})
@WebAppConfiguration
public class TodoControllerTest {

    private MockMvc mockMvc;

    @Autowired
    private TodoService todoServiceMock;

    //此处添加 WebApplicationContext 字段
    //setUp() 方法

    @Test
    public void findAll_ShouldAddTodoEntriesToModelAndRenderTodoListView() throws Exception {
        Todo first = new TodoBuilder()
                .id(1L)
                .description("Lorem ipsum")
                .title("Foo")
                .build();
        Todo second = new TodoBuilder()
                .id(2L)
                .description("Lorem ipsum")
                .title("Bar")
                .build();

        when(todoServiceMock.findAll()).thenReturn(Arrays.asList(first, second));

        mockMvc.perform(get("/"))
                .andExpect(status().isOk())
                .andExpect(view().name("todo/list"))
                .andExpect(forwardedUrl("/WEB-INF/jsp/todo/list.jsp"))
                .andExpect(model().attribute("todos", hasSize(2)))
                .andExpect(model().attribute("todos", hasItem(
                        allOf(
                                hasProperty("id", is(1L)),
                                hasProperty("description", is("Lorem ipsum")),
                                hasProperty("title", is("Foo"))
                        )
                )))
                .andExpect(model().attribute("todos", hasItem(
                        allOf(
                                hasProperty("id", is(2L)),
                                hasProperty("description", is("Lorem ipsum")),
                                hasProperty("title", is("Bar"))
                        )
                )));

        verify(todoServiceMock, times(1)).findAll();
        verifyNoMoreInteractions(todoServiceMock);
    }
}
{% endhighlight %}

### 渲染 Todo 对象详情页 ###

在写具体测试代码前，让我们先看看待测方法的实现。

#### 预期行为 ####

用以展示单个 Todo 对象信息的 Controller 方法实现中主要包含以下几步：

1. 接收发送到 ‘/todo/{id}’ 地址的 GET 请求。{id} 是一个用于标识被请求 Todo 对象主键的地址变量。
2. 它通过调用 TodoService 接口的 `findById()` 方法获取被请求的 Todo 对象，方法参数是被请求 Todo 对象的主键。这个方法会返回找到的 Todo 对象。如果没找到对应对象，这个方法会抛出 TodoNotFoundException 异常。
3. 它会把找到的 Todo 对象添加到 Model 对象中。
4. 返回待渲染的视图对象。

Controller 方法源代码如下:

{% highlight java linenos %}
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.*;

@Controller
public class TodoController {

    private final TodoService service;

    @RequestMapping(value = "/todo/{id}", method = RequestMethod.GET)
    public String findById(@PathVariable("id") Long id, Model model) throws TodoNotFoundException {
        Todo found = service.findById(id);
        model.addAttribute("todo", found);
        return "todo/view";
    }
}
{% endhighlight %}

下一个问题就是:

> 什么时候抛出 TodoNotFoundException 异常?

在本系列指南前面我们提到过，我们为 Controller 类抛出的异常创建过一个处理类。这个处理类的配置信息大体是这样的：

{% highlight java linenos %}
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
{% endhighlight %}

正如我们看到的，如果抛出一个 TodoNotFoundException 异常，应用会返回 404 状态码并渲染 ‘error/404′ 视图。

为这个 Controller 方法编写单元测试时很明显包括以下两步：

1. 必须有一个用例用来确保当找不到 Todo 对象时应用能正确执行。
2. 必须有一个用例用来确保当 Todo 对象被找到时也能正确执行。

现在看看这个测试应该怎么写。

#### 测试 1: 查询不到 Todo 对象时 ####

首先，我们必须要确保我们的应用在找不到被查询 Todo 对象时也能正常工作。我们可以通过以下几步来进行测试：

1. 配置冒烟对象，让它在 `findById()` 方法以参数 1L 被调用时抛出 TodoNotFoundException 异常。
2. 往 ‘/todo/1′ 地址发送一个 GET 请求。
3. 确认返回的 HTTP 状态码是 404。
4. 确保返回的视图名称是 ‘error/404′。
5. 确保请求被定向到地址 ‘/WEB-INF/jsp/error/404.jsp’。
6. 确认 TodoService 接口的 `findById()` 方法仅被调用过一次而且参数为 1L。
7. 确认冒烟对象的其它方法在测试期间没被调用过。

这个测试用例的代码如下:

{% highlight java linenos %}
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.mockito.Mockito;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.test.context.web.WebAppConfiguration;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;
import org.springframework.web.context.WebApplicationContext;

import static org.mockito.Mockito.*;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = {TestContext.class, WebAppContext.class})
@WebAppConfiguration
public class TodoControllerTest {

    private MockMvc mockMvc;

    @Autowired
    private TodoService todoServiceMock;

    //Add WebApplicationContext field here

    //The setUp() method is omitted.

    @Test
    public void findById_TodoEntryNotFound_ShouldRender404View() throws Exception {
        when(todoServiceMock.findById(1L)).thenThrow(new TodoNotFoundException(""));

        mockMvc.perform(get("/todo/{id}", 1L))
                .andExpect(status().isNotFound())
                .andExpect(view().name("error/404"))
                .andExpect(forwardedUrl("/WEB-INF/jsp/error/404.jsp"));

        verify(todoServiceMock, times(1)).findById(1L);
        verifyZeroInteractions(todoServiceMock);
    }
}
{% endhighlight %}

#### 测试 2: 查询到 Todo 对象时 ####

接上文，现在我们需要写一个确保应用在能查询到 Todo 对象时也能正确工作的测试用例。基本步骤如下：

1. 创建 Service 中方法被调用时返回的 Todo 对象。同样，我们还是通过测试数据构建器创建测试对象。
2. 配置冒烟对象，让它在 `findById()` 方法以 1L 参数被调用时返回前面创建的 Todo 对象。
3. 往 ‘/todo/1′ 地址发送 GET 请求。
4. 确认返回的 HTTP 状态码是 200。
5. 确保返回的视图名称是 ‘todo/view’。
6. 确保请求被定向到地址 ‘/WEB-INF/jsp/todo/view.jsp’。
7. 确认 Model 对象中的 Todo 对象主键是 1L。
8. 确认 Model 对象中的 Todo 对象描述字段值为 ‘Lorem ipsum’。
9. 确认 Model 对象中的 Todo 对象名称字段值为 ‘Foo’。
10. 确保冒烟对象的 `findById()` 方法仅被调用过一次，而且参数为 1L。
11. 确保冒烟对象中的其它方法在测试过程中没被调用过。

测试用例代码如下:

{% highlight java linenos %}
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.mockito.Mockito;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.test.context.web.WebAppConfiguration;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;
import org.springframework.web.context.WebApplicationContext;

import static org.hamcrest.Matchers.hasProperty;
import static org.hamcrest.Matchers.is;
import static org.mockito.Mockito.*;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = {TestContext.class, WebAppContext.class})
@WebAppConfiguration
public class TodoControllerTest {

    private MockMvc mockMvc;

    @Autowired
    private TodoService todoServiceMock;

    //Add WebApplicationContext field here

    //The setUp() method is omitted.

    @Test
    public void findById_TodoEntryFound_ShouldAddTodoEntryToModelAndRenderViewTodoEntryView() throws Exception {
        Todo found = new TodoBuilder()
                .id(1L)
                .description("Lorem ipsum")
                .title("Foo")
                .build();

        when(todoServiceMock.findById(1L)).thenReturn(found);
        mockMvc.perform(get("/todo/{id}", 1L))
                .andExpect(status().isOk())
                .andExpect(view().name("todo/view"))
                .andExpect(forwardedUrl("/WEB-INF/jsp/todo/view.jsp"))
                .andExpect(model().attribute("todo", hasProperty("id", is(1L))))
                .andExpect(model().attribute("todo", hasProperty("description", is("Lorem ipsum"))))
                .andExpect(model().attribute("todo", hasProperty("title", is("Foo"))));

        verify(todoServiceMock, times(1)).findById(1L);
        verifyNoMoreInteractions(todoServiceMock);
    }
}
{% endhighlight %}

### 处理表单并在数据库中添加 Todo 记录 ###

首先，在编写测试用例前还是看一下待测试 Controller 方法的预期行为。

#### 预期行为 ####

负责处理表单并入库的 Controller 方法基本实现步骤如下：

1. 接收发送到 ‘/todo/add’ 地址的 POST  请求。
2. 校验传递过来的 BingdingResult 参数是正确的，否则返回表单视图名称。
3. 以表单对象为参数调用 TodoService 接口的 `add()` 方法进行 Todo 对象入库。
4. 创建返回信息并将返回信息作为参数传递给 RedirectAttributes 中。
5. 把入库的 Todo 对象主键添加到 RedirectAttributes 中。
6. 把请求重定向到 Todo 对象详情页并渲染。

位于 TodoController 类中的相关代码如下：

{% highlight java linenos %}
import org.springframework.context.MessageSource;
import org.springframework.context.i18n.LocaleContextHolder;
import org.springframework.stereotype.Controller;
import org.springframework.validation.BindingResult;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.servlet.mvc.support.RedirectAttributes;

import javax.validation.Valid;
import java.util.Locale;

@Controller
@SessionAttributes("todo")
public class TodoController {

    private final TodoService service;
    private final MessageSource messageSource;

    @RequestMapping(value = "/todo/add", method = RequestMethod.POST)
    public String add(@Valid @ModelAttribute("todo") TodoDTO dto, BindingResult result, RedirectAttributes attributes) {
        if (result.hasErrors()) {
            return "todo/add";
        }

        Todo added = service.add(dto);
        addFeedbackMessage(attributes, "feedback.message.todo.added", added.getTitle());
        attributes.addAttribute("id", added.getId());
        return createRedirectViewPath("todo/view");
    }

    private void addFeedbackMessage(RedirectAttributes attributes, String messageCode, Object... messageParameters) {
        String localizedFeedbackMessage = getMessage(messageCode, messageParameters);
        attributes.addFlashAttribute("feedbackMessage", localizedFeedbackMessage);
    }

    private String getMessage(String messageCode, Object... messageParameters) {
        Locale current = LocaleContextHolder.getLocale();
        return messageSource.getMessage(messageCode, messageParameters, current);
    }

    private String createRedirectViewPath(String requestMapping) {
        StringBuilder redirectViewPath = new StringBuilder();
        redirectViewPath.append("redirect:");
        redirectViewPath.append(requestMapping);
        return redirectViewPath.toString();
    }
}
{% endhighlight %}

我们可以看到，这个方法用了一个 TodoDTO 对象来表示表单对象。TodoDTO 类只是一个简单的数据传输类，它的代码大致如下：

{% highlight java linenos %}
import org.hibernate.validator.constraints.Length;
import org.hibernate.validator.constraints.NotEmpty;

public class TodoDTO {

    private Long id;
    @Length(max = 500)
    private String description;
    @NotEmpty
    @Length(max = 100)
    private String title;

	//Constructor and other methods are omitted.
}
{% endhighlight %}

这个类中声明了如下一些校验约束：

- Todo 对象中的 title 属性不能为空。
- description 属性的最大长度是 500 字符。
- title 属性的最大长度是 100 字符。

如果我们仔细思考下我们该为这个方法写的测试，就会发现我们至少有以下几点需要做的：

1. 在参数校验失败时 Controller 方法需要能正常工作。
2. 校验通过时也能正常工作并正常入库。

现在让我们看看这个测试应该怎么写。

#### 测试 1: 校验失败时 ####

首先，我们需要测一下校验失败时 Controller 方法也能正常工作。这个测试我们可以这么干：

1. 创建 title 属性包含 101 字符。
2. 创建 description 属性包含 501 字符。
3. 通过以下几步往 ‘/todo/add’ 地址发送 POST 请求：
  1. 把请求的 Content-Type 设置成 ‘application/x-www-form-urlencoded’。
  2. 把前面提到的 title 和 description 当作请求参数发送过去。
  3. 在 session 中设置一个 TodoDTO 对象。这是必须的因为我们的 Controller 有一个 `@SessionAttributes` 注解。
4. 校验返回的 HTTP 状态码是 200。
5. 校验返回的视图名称是 ‘todo/add’。
6. 校验请求被定向到地址 ‘/WEB-INF/jsp/todo/add.jsp’。
7. 校验 model 对象中有 title 和 description 参数错误的提示信息。
8. 确保 model 中的 id 属性是空。
9. 确保 model 中的 description 属性是正确的。
10. 确保 model 中的 title 属性是正确的。
11. 确保冒烟对象中的方法在测试过程中没被调用过。

这个测试的源代码大体如下：

{% highlight java linenos %}
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.mockito.Mockito;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.MediaType;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.test.context.web.WebAppConfiguration;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;
import org.springframework.web.context.WebApplicationContext;

import static org.hamcrest.Matchers.hasProperty;
import static org.hamcrest.Matchers.is;
import static org.hamcrest.Matchers.nullValue;
import static org.mockito.Mockito.*;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = {TestContext.class, WebAppContext.class})
@WebAppConfiguration
public class TodoControllerTest {

    private MockMvc mockMvc;

    @Autowired
    private TodoService todoServiceMock;

    //此处添加 WebApplicationContext 字段
    //setUp() 方法

    @Test
    public void add_DescriptionAndTitleAreTooLong_ShouldRenderFormViewAndReturnValidationErrorsForTitleAndDescription() throws Exception {
        String title = TestUtil.createStringWithLength(101);
        String description = TestUtil.createStringWithLength(501);

        mockMvc.perform(post("/todo/add")
                .contentType(MediaType.APPLICATION_FORM_URLENCODED)
                .param("description", description)
                .param("title", title)
                .sessionAttr("todo", new TodoDTO())
        )
                .andExpect(status().isOk())
                .andExpect(view().name("todo/add"))
                .andExpect(forwardedUrl("/WEB-INF/jsp/todo/add.jsp"))
                .andExpect(model().attributeHasFieldErrors("todo", "title"))
                .andExpect(model().attributeHasFieldErrors("todo", "description"))
                .andExpect(model().attribute("todo", hasProperty("id", nullValue())))
                .andExpect(model().attribute("todo", hasProperty("description", is(description))))
                .andExpect(model().attribute("todo", hasProperty("title", is(title))));

        verifyZeroInteractions(todoServiceMock);
    }
}
{% endhighlight %}

我们的测试用例调用了 TestUtil 类的 `createStringWithLength(int length)` 方法。这个方法会按照给定的长度创建并返回字符串。

TestUtil 类的源代码如下：

{% highlight java linenos %}
import com.fasterxml.jackson.annotation.JsonInclude;
import com.fasterxml.jackson.databind.ObjectMapper;

import java.util.Iterator;
import java.util.Map;
import java.util.Set;

public class TestUtil {

    public static String createStringWithLength(int length) {
        StringBuilder builder = new StringBuilder();
        for (int index = 0; index < length; index++) {
            builder.append("a");
        }
        return builder.toString();
    }
}
{% endhighlight %}

#### 测试 2: 校验通过且正常入库时 ####

现在，让我们测试可以正常入库时 Controller 能否正常工作。这个测试可以分以下几步：

1. 创建一个 Todo 对象，它会在 TodoService 接口的 `add()` 方法被调用时返回。
2. 配置冒烟对象让它在 `add()` 方法按给定 TodoDTO 对象为参数调用时返回前面创建的 Todo 对象。
3. 通过以下几步往 ‘/todo/add’ 地址发送 POST 请求：
  1. 把请求的 Content-Type 设置成 ‘application/x-www-form-urlencoded’。
  2. 把 Todo 对象的 description 和 title 字段当做参数发送请求。
  3. 在 session 中添加一个 TodoDTO 对象。因为我们的 Controller 类上包含一个 `@SessionAttributes` 注解。
4. 确认返回的 HTTP 状态码是 302。
5. 确认返回的视图名称是 ‘redirect:todo/{id}’。
6. 确认请求被定向到地址 ‘/todo/1’。
7. 确认 model 中 id 属性值为 ‘1’。
8. 确认返回信息已正确设置。
9. 确认冒烟对象的 `add()` 方法仅被调用过一次而且参数是 TodoDTO 对象。使用 ArgumentCaptor 为使用的参数做一个快照。
10. 确认冒烟对象的其它方法在测试过程中没被调用过。
11. 确认 TodoDTO 对象的和字段值是正确的。

这个单元测试的源代码如下:

{% highlight java linenos %}
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.mockito.ArgumentCaptor;
import org.mockito.Mockito;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.MediaType;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.test.context.web.WebAppConfiguration;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;
import org.springframework.web.context.WebApplicationContext;

import static org.hamcrest.Matchers.is;
import static org.junit.Assert.assertNull;
import static org.junit.Assert.assertThat;
import static org.mockito.Matchers.isA;
import static org.mockito.Mockito.*;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = {TestContext.class, WebAppContext.class})
@WebAppConfiguration
public class TodoControllerTest {

    private MockMvc mockMvc;
    @Autowired
    private TodoService todoServiceMock;

    //此处添加 WebApplicationContext 字段
    //setUp() 方法

    @Test
    public void add_NewTodoEntry_ShouldAddTodoEntryAndRenderViewTodoEntryView() throws Exception {
        Todo added = new TodoBuilder()
                .id(1L)
                .description("description")
                .title("title")
                .build();
        when(todoServiceMock.add(isA(TodoDTO.class))).thenReturn(added);

        mockMvc.perform(post("/todo/add")
                .contentType(MediaType.APPLICATION_FORM_URLENCODED)
                .param("description", "description")
                .param("title", "title")
                .sessionAttr("todo", new TodoDTO())
        )
                .andExpect(status().isMovedTemporarily())
                .andExpect(view().name("redirect:todo/{id}"))
                .andExpect(redirectedUrl("/todo/1"))
                .andExpect(model().attribute("id", is("1")))
                .andExpect(flash().attribute("feedbackMessage", is("Todo entry: title was added.")));
		ArgumentCaptor<TodoDTO> formObjectArgument = ArgumentCaptor.forClass(TodoDTO.class);
		verify(todoServiceMock, times(1)).add(formObjectArgument.capture());
		verifyNoMoreInteractions(todoServiceMock);

		TodoDTO formObject = formObjectArgument.getValue();
		assertThat(formObject.getDescription(), is("description"));
		assertNull(formObject.getId());
		assertThat(formObject.getTitle(), is("title"));
    }
}
{% endhighlight %}

## 总结 ##

我们现在已经使用 SpringMVC 测试框架写了好几个普通 Controller 的测试用例了。通过这篇指南我们学会了：

- 如何为被测试 Controller 方法创建请求对象。
- 如何为被测试 Controller 方法的返回做断言。
- 如何为渲染视图的 Controller 写单元测试。
- 如何为处理表单的 Controller 写单元测试。

本系列指南接下来的部分将会告诉我们如何为 REST API 写单元测试。



[原文链接](http://www.petrikainulainen.net/programming/spring-framework/unit-testing-of-spring-mvc-controllers-normal-controllers/ "Unit Testing of Spring MVC Controllers: “Normal” Controllers")