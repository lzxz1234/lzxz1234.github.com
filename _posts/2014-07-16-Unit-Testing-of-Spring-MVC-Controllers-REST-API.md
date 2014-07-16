---
layout: post
title: 如何为 SpringMVC 编写单元测试：REST API 篇
category : junit
tags : [JUnit, Java, Testing, SpringMVC]
---
{% include JB/setup %}

SpringMVC 为开发 REST API 提供了很便捷的途径。然而，想要为它们快速并全面的编写单元测试却显得没那么容易。SpringMVC 测试框架的发布则提供了快速全面编写高可读性单元测试的可能。

这篇文章的目的就是说明如何通过 SpringMVC 为 REST API 编写单元测试。该文章中我们将会为用以提供 Todo 对象的 CRUD 操作的 Controller 方法编写单元测试。

让我们现在开始吧。

## 通过 Maven 获取相关依赖 ##

我们可以通过在 POM 文件中加入以下声明来获取到必须的测试依赖：

- Hamcrest 1.3 (hamcrest-all). 使用它为返回内容写断言。
- Junit 4.11。删掉 hamcrest-core 模块因为在前面 hamcrest-all 中已经被包含了。
- Mockito 1.9.5 (mockito-core)。
- Spring Test 3.2.3.RELEASE。
- JsonPath 0.8.1 (json-path 和 json-path-assert 模块). 我们使用 JsonPath 为 REST API 返回的 JSON 内容写断言。

相关的依赖声明如下:

{% highlight xml linenos %}
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
<dependency>
    <groupId>com.jayway.jsonpath</groupId>
    <artifactId>json-path</artifactId>
    <version>0.8.1</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>com.jayway.jsonpath</groupId>
    <artifactId>json-path-assert</artifactId>
    <version>0.8.1</version>
    <scope>test</scope>
</dependency>
{% endhighlight %}

现在让我们继续，看看如何为我们的单元测试作配置。

## 基本配置 ##

我们将要进行的单元测试会使用基于 WebApplicationContext 的配置。也就是说我们既可以通过两种方式配置 SpringMVC 基本信息，配置类方式或者 XML 文件方式。

本系列指南一开始就已经对配置相关做了详细说明，此处就不再赘述了。

有一件事我们还需要提一下。

在我们现在的例子应用中的 WEB 层配置中没有创建异常处理 Bean。我们前面用到的 SimpleMappingExceptionResolver 主要用于在抛出指定的异常时返回指定的视图。

这在普通的 SpringMVC 应用中是非常有意义的。但，在一个 REST API 的应用中，我们更希望把异常转成 HTTP 状态码。这种行为默认可以通过 ResponseStatusExceptionResolver 实现。

我们的例子应用中我们通过 `@ControllerAdvice` 注解自定义了一个异常处理类。这个类处理校验错误和应用特定异常。后面我们还会具体讨论下这个类。

现在让我们继续，看看如何为 REST API 写单元测试。

## 为 REST API 编写单元测试 ##

在这之前，我们需要一点知识储备：

- 我们需要知道 SpringMVC 测试框架的核心组件。这些组件就是在本系列指南第二部分中已经描述过。
- 我们需要知道如何通过 JsonPath 表达式为 JSON 串作断言。这部分知识需要读者自行翻阅文档。

接下来我们正式开始，为以下方法编写测试：

- 第一个方法会返回 Todo 对象的列表。
- 第二个方法会返回某个具体 Todo 对象的详情信息。
- 第三个方法会在数据库中新建并返回一条 Todo 记录。

### 列表查询 ###

第一个 Controller 方法会返回从数据库中查询到的 Todo 对象列表。让我们先看看这个方法的实现。

#### 预期行为 ####

这个方法会通过以下几步返回数据库中存储的所有 Todo 对象：

1. 接收发往 ‘/api/todo’ 地址的 GET 请求。
2. 通过调用 TodoService 接口的 `findAll()` 方法获取一个 Todo 对象的列表。这个方法会返回数据库中的全部记录。而且每次返回排序都一样。
3. 把上一步取得的列表转换成 TodoDTO 对象列表。
4. 返回包含 TodoDTO 对象的列表。

该功能在 TodoController 类中的相关代码如下：

{% highlight java linenos %}
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;
 
import java.util.ArrayList;
import java.util.List;
 
@Controller
public class TodoController {
 
    private TodoService service;
 
    @RequestMapping(value = "/api/todo", method = RequestMethod.GET)
    @ResponseBody
    public List<TodoDTO> findAll() {
        List<Todo> models = service.findAll();
        return createDTOs(models);
    }
 
    private List<TodoDTO> createDTOs(List<Todo> models) {
        List<TodoDTO> dtos = new ArrayList<>();
        for (Todo model: models) {
            dtos.add(createDTO(model));
        }
        return dtos;
    }
 
    private TodoDTO createDTO(Todo model) {
        TodoDTO dto = new TodoDTO();
        dto.setId(model.getId());
        dto.setDescription(model.getDescription());
        dto.setTitle(model.getTitle());
        return dto;
    }
}
{% endhighlight %}

当返回 TodoDTO 对象列表后， SpringMVC 会自动把这个列表转换成一个 JSON 串，大致是这样的：

{% highlight json linenos %}
[
    {
        "id":1,
        "description":"Lorem ipsum",
        "title":"Foo"
    },
    {
        "id":2,
        "description":"Lorem ipsum",
        "title":"Bar"
    }
]
{% endhighlight %}

现在开始为这个方法写测试，验证它能不能按照预期执行。

#### 测试: 查询到 Todo 对象时 ####

我们可以通过以下几步为这个方法编写单元测试：

1. 创建 TodoService 接口的 `findAll()` 方法被调用时返回的测试数据。我们会通过一个测试数据构造器构造这些测试数据。
2. 配置冒烟对象，让它的 `findAll()` 方法被调用时返回前面创建的测试数据。
3. 往 ‘/api/todo’ 地址发送一个 GET 请求。
4. 验证返回的 HTTP 状态码是 200。
5. 验证返回的 Content-Type 是 ‘application/json’ 并且字符编码是 ‘UTF-8′。
6. 使用 JsonPath 表达式 `$` 获取 Todo 对象集合，并确认这个集合中只包含前面返回的两个 Todo 对象。
7. 通过 JsonPath 表达式 `$[0].id`、`$[0].description` 和 `$[0].title` 获取返回的第一个对象的 id、description 和 title 属性。并验证各值是正确的。
8. 通过 JsonPath 表达式 `$[1].id`、`$[1].description` 和 `$[1].title` 获取返回的第二个对象的 id、description 和 title 属性。并验证各值是正确的。
9. 验证冒烟对象 TodoService 接口的 `findAll()` 方法仅被调用过一次。
10. 验证冒烟对象的其它方法在测试过程中没有被调用过。

这个测试的源码基本如下:

{% highlight java linenos %}
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.test.context.web.WebAppConfiguration;
import org.springframework.test.web.servlet.MockMvc;
 
import java.util.Arrays;
 
import static org.hamcrest.Matchers.*;
import static org.mockito.Mockito.*;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;
 
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = {TestContext.class, WebAppContext.class})
@WebAppConfiguration
public class TodoControllerTest {
 
    private MockMvc mockMvc;
 
    @Autowired
    private TodoService todoServiceMock;
 
    //WebApplicationContext 字段略.
    //setUp() 方法略.
 
    @Test
    public void findAll_TodosFound_ShouldReturnFoundTodoEntries() throws Exception {
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
 
        mockMvc.perform(get("/api/todo"))
                .andExpect(status().isOk())
                .andExpect(content().contentType(TestUtil.APPLICATION_JSON_UTF8))
                .andExpect(jsonPath("$", hasSize(2)))
                .andExpect(jsonPath("$[0].id", is(1)))
                .andExpect(jsonPath("$[0].description", is("Lorem ipsum")))
                .andExpect(jsonPath("$[0].title", is("Foo")))
                .andExpect(jsonPath("$[1].id", is(2)))
                .andExpect(jsonPath("$[1].description", is("Lorem ipsum")))
                .andExpect(jsonPath("$[1].title", is("Bar")));
 
        verify(todoServiceMock, times(1)).findAll();
        verifyNoMoreInteractions(todoServiceMock);
    }
}
{% endhighlight %}

我们的测试中使用了一个在 TestUtil 类中声明的叫 APPLICATION_JSON_UTF8 的常量。这个常量的值是一个 MediaType 类型对象，并且 Content-Type 是 ‘application/json’ 字符编码是 ‘UTF-8′。

相关代码如下：

{% highlight java linenos %}
public class TestUtil {
 
    public static final MediaType APPLICATION_JSON_UTF8 = new MediaType(MediaType.APPLICATION_JSON.getType(),MediaType.APPLICATION_JSON.getSubtype(),   Charset.forName("utf8"));
}
{% endhighlight %}

### 详情查询 ###

第二个待测方法用于返回一个特定 Todo 对象的详情信息。先让我们看看这个方法是如何实现的。

#### 预期行为 ####

这个方法会通过以下几步返回单个 Todo 对象的详情信息：

1. 接收发往 ‘/api/todo/{id}’ 地址的 GET 请求。`{id}` 是一个地址变量，用于表示请求的 Todo 对象主键。
2. 通过调用 TodoService 接口的 `findById()` 方法获取被请求的 Todo 对象，方法参数是前面获取的被请求 Todo 对象的主键。这个方法会返回查询到的 Todo 对象，如果没查询到对应记录，将抛出 TodoNotFoundException 异常。
3. 把 Todo 对象转换成 TodoDTO 对象。
4. 返回前面创建的 TodoDTO 对象。

相关代码如下:

{% highlight java linenos %}
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;

@Controller
public class TodoController {
 
    private TodoService service;
 
    @RequestMapping(value = "/api/todo/{id}", method = RequestMethod.GET)
    @ResponseBody
    public TodoDTO findById(@PathVariable("id") Long id) throws TodoNotFoundException {
        Todo found = service.findById(id);
        return createDTO(found);
    }

    private TodoDTO createDTO(Todo model) {
        TodoDTO dto = new TodoDTO();
        dto.setId(model.getId());
        dto.setDescription(model.getDescription());
        dto.setTitle(model.getTitle());
        return dto;
    }
}
{% endhighlight %}

返回给客户端的 JSON 串看起来是这样的：

{% highlight json linenos %}
{
    "id":1,
    "description":"Lorem ipsum",
    "title":"Foo"
}
{% endhighlight %}
 
我们的下一个问题是:

> 抛出 TodoNotFoundException 异常时发生了什么？

我们的样例应用中有一个异常处理类，用以处理我们的 Controller 类抛出的特定异常。当抛出 TodoNotFoundException 异常时会调用这个类中的一个方法。这个方法会在日志文件中添加一条信息并给客户端返回 HTTP 状态码 404。

该功能在 RestErrorHandler 类中的相关代码如下：

{% highlight java linenos %}
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseStatus;
 
@ControllerAdvice
public class RestErrorHandler {
 
    private static final Logger LOGGER = LoggerFactory.getLogger(RestErrorHandler.class);
 
    @ExceptionHandler(TodoNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public void handleTodoNotFoundException(TodoNotFoundException ex) {
        LOGGER.debug("handling 404 error on a todo entry");
    }
}
{% endhighlight %}

这个 Controller 方法我们至少要写以下两个测试：

1. 一个测试用以确认没有找到指定 Todo 对象时我们的应用也能正常工作。
2. 一个测试用以确认正常找到请求 Todo 对象时返回给客户端的数据是都是正确的。

现在让我们看看这些测试应该如何写。

#### 测试 1: 没找到指定 Todo 对象时 ####

首先，我们需要保证找不到指定 Todo 对象时应用也能有正确执行。我们可以通过以下几步实现这个测试：

1. 配置冒烟对象让它在以参数 1L 调用 `findById()` 方法时抛出 TodoNotFoundException 异常。
2. 往 ‘/api/todo/1′ 地址发送一个 GET 请求。
3. 确认返回的 HTTP 状态码是 404。
4. 确认 TodoService 接口的 `findById()` 方法仅被调用过一次而且参数是 1L。
5. 确认 TodoService 接口的其它方法在测试过程中没被调用过。

该单元测试代码大致如下:

{% highlight java linenos %}
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.test.context.web.WebAppConfiguration;
import org.springframework.test.web.servlet.MockMvc;
 
import static org.mockito.Mockito.*;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;
 
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = {TestContext.class, WebAppContext.class})
@WebAppConfiguration
public class TodoControllerTest {
 
    private MockMvc mockMvc;
 
    @Autowired
    private TodoService todoServiceMock;
 
    //WebApplicationContext 字段略。
    //setUp() 方法略。
 
    @Test
    public void findById_TodoEntryNotFound_ShouldReturnHttpStatusCode404() throws Exception {
        when(todoServiceMock.findById(1L)).thenThrow(new TodoNotFoundException(""));
        mockMvc.perform(get("/api/todo/{id}", 1L))
                .andExpect(status().isNotFound());
        verify(todoServiceMock, times(1)).findById(1L);
        verifyNoMoreInteractions(todoServiceMock);
    }
}
{% endhighlight %}

#### 测试 2: 找到 Todo 对象时 ####

现在，我们需要写一个测试以保证当被查询 Todo 对象正常找到时返回的各数据都是正确的。我们可以通过这么几步实现：

1. 创建一个测试时用以返回的 Todo 对象。也是通过测试数据构造器实现。
2. 配置冒烟对象，让它的 `findById()` 方法按参数 1L 被调用时返回前面创建的 Todo 对象。
3. 往地址 '/api/todo/1' 发送一个 GET 请求。
4. 确认返回的 HTTP 状态码是 200。
5. 确认返回的 Content-Type 是 ‘application/json’ 并且字符编码是 ‘UTF-8’。
6. 使用 JsonPath 表达式 `$.id` 获取 Todo 对象的主键，并验证其值为 1。
7. 使用 JsonPath 表达式 `$.description` 获取 Todo 对象的 description 属性并验证其值为 “Lorem ipsum”。
8. 使用 JsonPath 表达式 `$.title` 获取 Todo 对象的 title 属性并验证其值为 “Foo”。
9. 确认 TodoService 接口的 `findById()` 方法在测试过程中仅被调用过一次，且参数为 1L。
10. 确认冒烟对象的其它方法在测试过程中没被调用过。

代码如下:

{% highlight java linenos %}
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.test.context.web.WebAppConfiguration;
import org.springframework.test.web.servlet.MockMvc;
 
import static org.hamcrest.Matchers.is;
import static org.mockito.Mockito.*;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.jsonPath;
 
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = {TestContext.class, WebAppContext.class})
@WebAppConfiguration
public class TodoControllerTest {
 
    private MockMvc mockMvc;
 
    @Autowired
    private TodoService todoServiceMock;
 
    //WebApplicationContext 字段略。
    //setUp() 方法略。
 
    @Test
    public void findById_TodoEntryFound_ShouldReturnFoundTodoEntry() throws Exception {
        Todo found = new TodoBuilder()
                .id(1L)
                .description("Lorem ipsum")
                .title("Foo")
                .build();
        when(todoServiceMock.findById(1L)).thenReturn(found);
        mockMvc.perform(get("/api/todo/{id}", 1L))
                .andExpect(status().isOk())
                .andExpect(content().contentType(TestUtil.APPLICATION_JSON_UTF8))
                .andExpect(jsonPath("$.id", is(1)))
                .andExpect(jsonPath("$.description", is("Lorem ipsum")))
                .andExpect(jsonPath("$.title", is("Foo")));
        verify(todoServiceMock, times(1)).findById(1L);
        verifyNoMoreInteractions(todoServiceMock);
    }
}
{% endhighlight %}

### 新建 Todo 记录 ###

第三个 Controller 方法会在数据库中新建 Todo 记录，并返回新建对象的信息。先看看其基本实现。

#### 预期行为 ####

这个功能主要是按这么几步实现的：

1. 接收发往地址 ‘/api/todo’ 的 POST 请求。
2. 校验被当作参数的 TodoDTO 对象。如果校验失败，抛出 MethodArgumentNotValidException 异常。
3. 调用 TodoService 接口的 `add()` 方法在数据库中新建 Todo 记录，方法参数为前面的 TodoDTO 对象。这个方法会在数据库中新建 Todo 记录，并返回添加的 Todo 对象。
4. 把创建的 Todo 对象转换成 TodoDTO 对象。
5. 返回 TodoDTO 对象。

相关代码如下：

{% highlight java linenos %}
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;
 
import javax.validation.Valid;
 
@Controller
public class TodoController {
 
    private TodoService service;
 
    @RequestMapping(value = "/api/todo", method = RequestMethod.POST)
    @ResponseBody
    public TodoDTO add(@Valid @RequestBody TodoDTO dto) {
        Todo added = service.add(dto);
        return createDTO(added);
    }
 
    private TodoDTO createDTO(Todo model) {
        TodoDTO dto = new TodoDTO();
        dto.setId(model.getId());
        dto.setDescription(model.getDescription());
        dto.setTitle(model.getTitle());
        return dto;
    }
}
{% endhighlight %}

类 TodoDTO 只是一个简单的数据传输类，其源码看起来是这样的：

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
 
    //构造函数和其它方法略.
}
{% endhighlight %}

我们可以看到，这个类声明了以下几个校验约束：

1. description 属性的最大长度是 500 字符。
2. title 属性不能为空。
3. title 属性最大长度是 100 字符。

如果校验失败，异常处理组件需要确保：

1. 返回给客户端的 HTTP 状态码是 400。
2. 校验错误信息也按 JSON 格式返回给客户端。

因为已经有一篇文章描述过如何为 REST API 添加校验功能，相关功能在本文中不再赘述。

当然，我们还是需要知道当校验失败时应该返回什么内容给客户端的。信息格式稍后将会给出。

如果 TodoDTO 对象的 title 和 description 属性太长，那么将会返回以下 JSON 串：

{% highlight json linenos %}
{
    "fieldErrors":[
        {
            "path":"description",
            "message":"The maximum length of the description is 500 characters."
        },
        {
            "path":"title",
            "message":"The maximum length of the title is 100 characters."
        }
    ]
}
{% endhighlight %}

> **注意**: SpringMVC 不保证错误信息的排序。也就是说，字段错误信息顺序是随机的。在我们为该方法写单元测试时也需要考虑到这点。

另一方面，如果校验没失败，我们的 Controller 方法将向客户端返回以下 JSON 内容：

{% highlight json linenos %}
{
    "id":1,
    "description":"description",
    "title":"todo"
}
{% endhighlight %}

这个方法也至少需要两个单元测试：

1. 一个测试以确保校验失败时应用能够正常工作。
2. 一个测试以确保校验通过并正常入库时也能正常工作。

现在看看这些个测试应该怎么写。

#### 测试 1: 校验失败时 ####

第一个测试确认校验失败时应用程序也能正常工作。我们可以通过这几步实现：

1. 创建一个包含 101 字符的 title。
2. 创建一个包含 501 字符的 descrption。
3. 通过测试数据构建器新建一个 TodoDTO 对象。并设置对应的 title 和 description 属性。
4. 往 ‘/api/todo’ 地址执行一个 POST 请求。请求的 Content-Type 设置成 ‘application/json’。请求的字符编码设置成 ‘UTF-8′。把前面创建的 TodoDTO 对象转换成 JSON 字节数组并当作请求体发送。
5. 校验返回的 HTTP 状态码是 400。
6. 校验返回的 Content-Type 是 ‘application/json’ 并且它的字符编码是 ‘UTF-8’。
7. 通过 JsonPath 表达式 $.fieldErrors 获取字段错误信息，并验证错误信息数目为 2。
8. 通过 JsonPath 表达式 $.fieldErrors[*].path 获取全部可用的路径信息，并且能正常找到 title 和 description 字段的错误信息。
9. 通过 JsonPath 表达式 $.fieldErrors[*].message 获取全部可用的错误提示信息，并且能正常找到 title 和 description 字段的错误信息。
10. 验证冒烟对象的方法在测试过程中没被调用过。

该单元测试源代码如下：

{% highlight java linenos %}
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.test.context.web.WebAppConfiguration;
import org.springframework.test.web.servlet.MockMvc;
 
import static org.hamcrest.Matchers.containsInAnyOrder;
import static org.hamcrest.Matchers.hasSize;
import static org.mockito.Mockito.*;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.jsonPath;
 
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = {TestContext.class, WebAppContext.class})
@WebAppConfiguration
public class TodoControllerTest {
 
    private MockMvc mockMvc;
 
    @Autowired
    private TodoService todoServiceMock;
 
    //WebApplicationContext 字段略。
    //setUp() 方法略。
 
    @Test
    public void add_TitleAndDescriptionAreTooLong_ShouldReturnValidationErrorsForTitleAndDescription() throws Exception {
        String title = TestUtil.createStringWithLength(101);
        String description = TestUtil.createStringWithLength(501);
 
        TodoDTO dto = new TodoDTOBuilder()
                .description(description)
                .title(title)
                .build();
        mockMvc.perform(post("/api/todo")
                .contentType(TestUtil.APPLICATION_JSON_UTF8)
                .content(TestUtil.convertObjectToJsonBytes(dto))
        )
                .andExpect(status().isBadRequest())
                .andExpect(content().contentType(TestUtil.APPLICATION_JSON_UTF8))
                .andExpect(jsonPath("$.fieldErrors", hasSize(2)))
                .andExpect(jsonPath("$.fieldErrors[*].path", containsInAnyOrder("title", "description")))
                .andExpect(jsonPath("$.fieldErrors[*].message", containsInAnyOrder(
                        "The maximum length of the description is 500 characters.",
                        "The maximum length of the title is 100 characters."
                )));
 
        verifyZeroInteractions(todoServiceMock);
    }
}
{% endhighlight %}

我们的测试中使用了 TestUtil 类的两个静态方法。这些方法的基本信息如下：

- `createStringWithLength(int length)` 方法会按给定的长度创建并返回字符串。
- `convertObjectToJsonBytes(Object object)` 方法会把给定的对象转换成 JSON 字符串，并进一步按字节数组的形式转换并返回。

类 TestUtil 的源码如下：

{% highlight java linenos %}
import com.fasterxml.jackson.annotation.JsonInclude;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.http.MediaType;
 
import java.io.IOException;
import java.nio.charset.Charset;
 
public class TestUtil {
 
    public static final MediaType APPLICATION_JSON_UTF8 = new MediaType(MediaType.APPLICATION_JSON.getType(), MediaType.APPLICATION_JSON.getSubtype(), Charset.forName("utf8"));
 
    public static byte[] convertObjectToJsonBytes(Object object) throws IOException {
        ObjectMapper mapper = new ObjectMapper();
        mapper.setSerializationInclusion(JsonInclude.Include.NON_NULL);
        return mapper.writeValueAsBytes(object);
    }
 
    public static String createStringWithLength(int length) {
        StringBuilder builder = new StringBuilder();
        for (int index = 0; index < length; index++) {
            builder.append("a");
        }
        return builder.toString();
    }
}
{% endhighlight %}

#### 测试 2: Todo 对象正常入库时 ####

这个单元测试主要用以验证我们的应用在 Todo 对象正常入库时也能正常工作。这个测试我们可以这么来：

1. 使用测试数据构建器新建一个 TodoDTO 对象。并在 title 和 description 字段上设置合法内容。
2. 创建 TodoService 接口的 `add()` 方法被调用时返回的 Todo 对象。
3. 配置冒烟对象，让它在 `add()` 方法被调用时返回前面创建的 Todo 对象。
4. 往地址 ‘/api/todo’ 执行一个 POST 请求。设置请求的 Content-Type 是 ‘application/json’。设置请求的字符编码集是 ‘UTF-8’。把创建的 TodoDTO 对象转换成 JSON 字节数组并当作请求体发送。
5. 校验返回的 HTTP 状态码是 200。
6. 校验返回的 Content-Type 是 ‘application/json’，并且它的字符编码是 ‘UTF-8’。
7. 使用 JsonPath 表达式 $.id 获取返回的 Todo 对象的 id 属性，校验其值为 1。
8. 使用 JsonPath 表达式 $.description 获取返回的 Todo 对象的 description 属性，校验其值为 “description”。
9. 使用 JsonPath 表达式 $.title 获取返回的 Todo 对象的 title 属性，校验其值为 “title”。
10. 创建 ArgumentCaptor 对象为 TodoDTO 作镜像。
11. 校验 TodoService 接口的 `add()` 方法仅被调用过一次而且参数与前面 TodoDTO 镜像相同。
12. 确认冒烟对象的其它方法在测试过程没被调用过。
13. 验证 TodoDTO 镜像对象的 id 属性是 null。
14. 验证 TodoDTO 镜像对象的 description 属性值是 “description”。
15. 验证 TodoDTO 镜像对象的 title 属性值是 “title”。

该单元测试基本代码如下：

{% highlight java linenos %}
import org.junit.Test;
import org.junit.runner.RunWith;
import org.mockito.ArgumentCaptor;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.test.context.web.WebAppConfiguration;
import org.springframework.test.web.servlet.MockMvc;
 
import static junit.framework.Assert.assertNull;
import static org.hamcrest.Matchers.is;
import static org.junit.Assert.assertThat;
import static org.mockito.Mockito.*;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.jsonPath;
 
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = {TestContext.class, WebAppContext.class})
@WebAppConfiguration
public class TodoControllerTest {
 
    private MockMvc mockMvc;
 
    @Autowired
    private TodoService todoServiceMock;
 
    //WebApplicationContext 字段略。
    //setUp() 方法略。
 
    @Test
    public void add_NewTodoEntry_ShouldAddTodoEntryAndReturnAddedEntry() throws Exception {
        TodoDTO dto = new TodoDTOBuilder()
                .description("description")
                .title("title")
                .build();
        Todo added = new TodoBuilder()
                .id(1L)
                .description("description")
                .title("title")
                .build();
 
        when(todoServiceMock.add(any(TodoDTO.class))).thenReturn(added);
        mockMvc.perform(post("/api/todo")
                .contentType(TestUtil.APPLICATION_JSON_UTF8)
                .content(TestUtil.convertObjectToJsonBytes(dto))
        )
                .andExpect(status().isOk())
                .andExpect(content().contentType(TestUtil.APPLICATION_JSON_UTF8))
                .andExpect(jsonPath("$.id", is(1)))
                .andExpect(jsonPath("$.description", is("description")))
                .andExpect(jsonPath("$.title", is("title")));
 
        ArgumentCaptor<TodoDTO> dtoCaptor = ArgumentCaptor.forClass(TodoDTO.class);
        verify(todoServiceMock, times(1)).add(dtoCaptor.capture());
        verifyNoMoreInteractions(todoServiceMock);
 
        TodoDTO dtoArgument = dtoCaptor.getValue();
        assertNull(dtoArgument.getId());
        assertThat(dtoArgument.getDescription(), is("description"));
        assertThat(dtoArgument.getTitle(), is("title"));
    }
}
{% endhighlight %}

## 总结 ##

现在我们已经使用 SpringMVC 测试框架为 REST API 写过数个单元测试了。本文主要讲了：

- 如何为从数据库读信息的 Controller 方法写单元测试。
- 如何为往数据库插数据的 Controller 方法写单元测试。
- 如何把 DTO 对象转换成 JSON 字节数组并把转换后的内容当作请求体发送。
- 如何使用 JsonPath 表达式为 JSON 串写断言。

转载注明出处：[{{page.title}}]({{permalink}})

[原文链接](http://www.petrikainulainen.net/programming/spring-framework/unit-testing-of-spring-mvc-controllers-rest-api/ "Unit Testing of Spring MVC Controllers: REST API")