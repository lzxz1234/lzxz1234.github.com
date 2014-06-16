---
layout: post
title: 编写干净的单元测试 - 小心魔数
category : junit
tags : [DSL, JUnit, Testing, CleanCode]
---
{% include JB/setup %}

魔法是易读代码的死对头，它在代码中最容易出现的形式就是魔数。

魔数会使我们的代码更混乱，并把它变成一堆不易读甚至不易维护的垃圾。

**这就是为什么我们必须不惜一切代价消灭魔数。**

这篇博客演示了魔数会给我们的测试用例造成何种影响，并且阐述了如何通过常量来消除它们。

## 常量来了 ##

我们会在代码中使用常量因为这样可以避免代码被魔数扰乱。使用魔数至少会有以下两种后果：

1. 代码更不容易阅读因为魔数只是数值而没有任何意义。
2. 当我们需要改变某个魔数的值时会发现代码更不容易维护，因为我们必须要找到那个魔数所有出现的场合并一块修改它们。

或者说，

- 常量帮助我们把魔数转换成了某种描述它们存在的意义的东西。
- 常量使我们的代码更容易维护因为如果某个常量值改变了，那我们只需要修改一个地方。

如果我们仔细思考下代码中发现的魔数，就会发现它们可以分成两类：

1. 魔数只和一个测试类有关。这种魔数典型的例子就是只是为了给测试方法内部创建的对象属性赋值。**我们应该在类的内部声明这些常量。**
2. 魔数和多个测试类有关。这种魔数典型的例子就是标志被一个 SpringMVC 控制器处理的一种请求类型。**我们应该在一个不可实例化的类中声明这些常量。**

让我们仔细看看这两种类型。

## 在测试类中声明常量 ##

首先，我们为什么要在测试类中声明常量？

毕竟，如果我们思考使用常量的好处，想到的第一件事情就是我们应该把魔数从我们的测试类移出去，然后创建一个新类持有测试需要的全部常量。例如，我们创建一个 TodoConstants 类持有了 TodoControllerTest， TodoCrudServiceTest， TodoTest 类中使用的全部常量。

**这绝对不是一个好主意。** 

尽管有些时候用这种途径分享数据是很明智的，但我们也不能掉以轻心，因为我们在测试类使用常量的初始动机只是为了避免拼写错误和魔数。

而且，如果魔数只和一个测试类相关，只是为了减少常量代码的行数而采用这种方式不存在任何意义。

在我看来，这种情况下**最简单的处理方法**就是把常量定义在测试类中。

让我们想想如何把本系列指南前面的测试用例再优化下。那个测试用例是用来测试 RepositoryUserService 类的 registerNewUserAccount() 方法的，并会校验用户通过社会化登录注册和唯一邮箱注册时能否正常工作。

这份测试用例代码如下：

{% highlight java %}
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.mockito.Mock;
import org.mockito.invocation.InvocationOnMock;
import org.mockito.runners.MockitoJUnitRunner;
import org.mockito.stubbing.Answer;
import org.springframework.security.crypto.password.PasswordEncoder;

import static org.junit.Assert.assertEquals;
import static org.junit.Assert.assertNull;
import static org.mockito.Matchers.isA;
import static org.mockito.Mockito.times;
import static org.mockito.Mockito.verify;
import static org.mockito.Mockito.verifyNoMoreInteractions;
import static org.mockito.Mockito.verifyZeroInteractions;
import static org.mockito.Mockito.when;


@RunWith(MockitoJUnitRunner.class)
public class RepositoryUserServiceTest {

     private RepositoryUserService registrationService;
     @Mock
     private PasswordEncoder passwordEncoder;
     @Mock
     private UserRepository repository;

     @Before
     public void setUp() {
         registrationService = new RepositoryUserService(passwordEncoder, repository);
     }


     @Test
     public void registerNewUserAccount_SocialSignInAndUniqueEmail_ShouldCreateNewUserAccountAndSetSignInProvider() throws DuplicateEmailException       {
         RegistrationForm registration = new RegistrationForm();
         registration.setEmail("john.smith@gmail.com");
         registration.setFirstName("John");
         registration.setLastName("Smith");
         registration.setSignInProvider(SocialMediaService.TWITTER);

         when(repository.findByEmail("john.smith@gmail.com")).thenReturn(null);
         when(repository.save(isA(User.class))).thenAnswer(new Answer<User>() {
             @Override
             public User answer(InvocationOnMock invocation) throws Throwable {
                 Object[] arguments = invocation.getArguments();
                 return (User) arguments[0];
             }
         });

         User createdUserAccount = registrationService.registerNewUserAccount(registration);

         assertEquals("john.smith@gmail.com", createdUserAccount.getEmail());
         assertEquals("John", createdUserAccount.getFirstName());
         assertEquals("Smith", createdUserAccount.getLastName());
         assertEquals(SocialMediaService.TWITTER, createdUserAccount.getSignInProvider());
         assertEquals(Role.ROLE_USER, createdUserAccount.getRole());
         assertNull(createdUserAccount.getPassword());

         verify(repository, times(1)).findByEmail("john.smith@gmail.com");
         verify(repository, times(1)).save(createdUserAccount);
         verifyNoMoreInteractions(repository);
         verifyZeroInteractions(passwordEncoder);
     }
}
{% endhighlight %}

这段代码存在的问题就是它在创建 RegistrationForm 对象、配置 UserRepository 桩的行为、校验返回的 User 对象是否正确、UserRepository 桩对象方法调用是否正确的时候使用了魔数。

当我们把魔数全用声明在类里的常量替换后，代码看起来是这样的：

{% highlight java %}
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.mockito.Mock;
import org.mockito.invocation.InvocationOnMock;
import org.mockito.runners.MockitoJUnitRunner;
import org.mockito.stubbing.Answer;
import org.springframework.security.crypto.password.PasswordEncoder;

import static org.junit.Assert.assertEquals;
import static org.junit.Assert.assertNull;
import static org.mockito.Matchers.isA;
import static org.mockito.Mockito.times;
import static org.mockito.Mockito.verify;
import static org.mockito.Mockito.verifyNoMoreInteractions;
import static org.mockito.Mockito.verifyZeroInteractions;
import static org.mockito.Mockito.when;


@RunWith(MockitoJUnitRunner.class)
public class RepositoryUserServiceTest {

    private static final String REGISTRATION_EMAIL_ADDRESS = "john.smith@gmail.com";
    private static final String REGISTRATION_FIRST_NAME = "John";
    private static final String REGISTRATION_LAST_NAME = "Smith";
    private static final Role ROLE_REGISTERED_USER = Role.ROLE_USER;
    private static final SocialMediaService SOCIAL_SIGN_IN_PROVIDER = SocialMediaService.TWITTER;

     private RepositoryUserService registrationService;
     @Mock
     private PasswordEncoder passwordEncoder;
     @Mock
     private UserRepository repository;

     @Before
     public void setUp() {
         registrationService = new RepositoryUserService(passwordEncoder, repository);
     }


     @Test
     public void registerNewUserAccount_SocialSignInAndUniqueEmail_ShouldCreateNewUserAccountAndSetSignInProvider() throws DuplicateEmailException       {
         RegistrationForm registration = new RegistrationForm();
         registration.setEmail(REGISTRATION_EMAIL_ADDRESS);
         registration.setFirstName(REGISTRATION_FIRST_NAME);
         registration.setLastName(REGISTRATION_LAST_NAME);
         registration.setSignInProvider(SOCIAL_SIGN_IN_PROVIDER);

         when(repository.findByEmail(REGISTRATION_EMAIL_ADDRESS)).thenReturn(null);
         when(repository.save(isA(User.class))).thenAnswer(new Answer<User>() {
             @Override
             public User answer(InvocationOnMock invocation) throws Throwable {
                 Object[] arguments = invocation.getArguments();
                 return (User) arguments[0];
             }
         });

         User createdUserAccount = registrationService.registerNewUserAccount(registration);

         assertEquals(REGISTRATION_EMAIL_ADDRESS, createdUserAccount.getEmail());
         assertEquals(REGISTRATION_FIRST_NAME, createdUserAccount.getFirstName());
         assertEquals(REGISTRATION_LAST_NAME, createdUserAccount.getLastName());
         assertEquals(SOCIAL_SIGN_IN_PROVIDER, createdUserAccount.getSignInProvider());
         assertEquals(ROLE_REGISTERED_USER, createdUserAccount.getRole());
         assertNull(createdUserAccount.getPassword());

         verify(repository, times(1)).findByEmail(REGISTRATION_EMAIL_ADDRESS);
         verify(repository, times(1)).save(createdUserAccount);
         verifyNoMoreInteractions(repository);
         verifyZeroInteractions(passwordEncoder);
     }
}
{% endhighlight %}

这个例子至少有以下三条好处：

1. 测试类更易读了因为订数被合理命名的常量取代了。
2. 测试类更易维护了因为我们可以修改常量的值而不需要修改任何测试代码。
3. 为 RepositoryUserService 类的 registerNewUserAccount() 方法写新测试更容易了因为我们可以用常量而不是魔数。也就是说我们不需要担心拼写错误。

然而，有时候测试类中的魔数会和很多类有关系，让我们看看这时候应该怎么处理。

## 把常量声明到不可实例化类中 ##

如果常量和多个测试类相关，那么把它们在所有的测试类中都声明一遍是没有意义的。现在就让我们看看这种情况下把常量声明到不可实例化类中有哪些好处。

假使我们需要为一个 REST API 写两个测试用例：

- 第一个测试用例确保我们不能把一个空 Todo 对象写到数据库中。
- 第二个测试用例确保我们不能把一个空 Note 对象写到数据库中。

> 这两个测试都使用了 SpringMVC 测试框架。如果你对它不熟悉，你可能需要先看看 SpringMVC 测试指南。

第一个测试用例的源代码如下：

{% highlight java %}
import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.MediaType;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.test.context.web.WebAppConfiguration;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;
import org.springframework.web.context.WebApplicationContext;

import java.nio.charset.Charset;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = {WebUnitTestContext.class})
@WebAppConfiguration
public class TodoControllerTest {

    private static final MediaType APPLICATION_JSON_UTF8 = new MediaType(
            MediaType.APPLICATION_JSON.getType(), MediaType.APPLICATION_JSON.getSubtype(),
            Charset.forName("utf8")
    );

    private MockMvc mockMvc;
    @Autowired
    private ObjectMapper objectMapper;
    @Autowired
    private WebApplicationContext webAppContext;

    @Before
    public void setUp() {
         mockMvc = MockMvcBuilders.webAppContextSetup(webAppContext).build();
    }

    @Test
    public void add_EmptyTodoEntry_ShouldReturnHttpRequestStatusBadRequest() throws Exception {
         TodoDTO addedTodoEntry = new TodoDTO();
         mockMvc.perform(post("/api/todo")
                        .contentType(APPLICATION_JSON_UTF8)
                        .content(objectMapper.writeValueAsBytes(addedTodoEntry))
         ).andExpect(status().isBadRequest());
    }
}
{% endhighlight %}

第二个测试用例源代码如下:

{% highlight java %}
import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.MediaType;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.test.context.web.WebAppConfiguration;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;
import org.springframework.web.context.WebApplicationContext;

import java.nio.charset.Charset;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = {WebUnitTestContext.class})
@WebAppConfiguration
public class NoteControllerTest {

    private static final MediaType APPLICATION_JSON_UTF8 = new MediaType(
            MediaType.APPLICATION_JSON.getType(), MediaType.APPLICATION_JSON.getSubtype(),
            Charset.forName("utf8")
    );

    private MockMvc mockMvc;
    @Autowired
    private ObjectMapper objectMapper;
    @Autowired
    private WebApplicationContext webAppContext;

    @Before
    public void setUp() {
         mockMvc = MockMvcBuilders.webAppContextSetup(webAppContext).build();
    }

    @Test
    public void add_EmptyNote_ShouldReturnHttpRequestStatusBadRequest() throws Exception {
         NoteDTO addedNote = new NoteDTO();

         mockMvc.perform(post("/api/note")
                        .contentType(APPLICATION_JSON_UTF8)
                        .content(objectMapper.writeValueAsBytes(addedNote))
         ).andExpect(status().isBadRequest());
    }
}
{% endhighlight %}

这两个测试类都声明了一个叫 APPLICATION_JSON_UTF8 的常量。这这常量标志请求的传输类型。而且，很明显我们所有测试 Controller 方法的类中都需要使用这个变量。

这是不是就是我们真应该在每个测试类中都声明这个常量？

**答案很明显不是的！**

我们应该把这个常量挪到一个不可实例化的类中因为：

1. 它和多个测试类相关。
2. 把它挪到一个单独类中更利于我们为 Controller 方法编写新的测试用例，也利于维护已经存在的。

两个创建一个不可变的 WebTestConstants 类，把 APPLICATION_JSON_UTF8 常量挪进去，并为它添加一个私有的构造方法。

源代码如下：

{% highlight java %}
import org.springframework.http.MediaType;

public final class WebTestConstants {
     public static final MediaType APPLICATION_JSON_UTF8 = new MediaType(
             MediaType.APPLICATION_JSON.getType(), MediaType.APPLICATION_JSON.getSubtype(), 
             Charset.forName("utf8")
     );
     
     private WebTestConstants() {
     }
}
{% endhighlight %}

这些工作完成后，我们就可以把原先测试用例中的 APPLICATION_JSON_UTF8 常量移除了。现在的源代码是这样的：

{% highlight java %}
import com.fasterxml.jackson.databind.ObjectMapper;
import net.petrikainulainen.spring.jooq.config.WebUnitTestContext;
import net.petrikainulainen.spring.jooq.todo.dto.TodoDTO;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.test.context.web.WebAppConfiguration;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;
import org.springframework.web.context.WebApplicationContext;

import java.nio.charset.Charset;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = {WebUnitTestContext.class})
@WebAppConfiguration
public class TodoControllerTest {

     private MockMvc mockMvc;
     @Autowired
     private ObjectMapper objectMapper;
     @Autowired
     private WebApplicationContext webAppContext;

     @Before
     public void setUp() {
         mockMvc = MockMvcBuilders.webAppContextSetup(webAppContext).build();
     }

     @Test
     public void add_EmptyTodoEntry_ShouldReturnHttpRequestStatusBadRequest() throws Exception {
         TodoDTO addedTodoEntry = new TodoDTO();
         mockMvc.perform(post("/api/todo")
                        .contentType(WebTestConstants.APPLICATION_JSON_UTF8)
                        .content(objectMapper.writeValueAsBytes(addedTodoEntry))
         ).andExpect(status().isBadRequest());
     }
}
{% endhighlight %}

我们已经把冗余的代码都删除了，并且在写新测试代码的时候也不需要重新声明了。相当 Cool，对吧？

> 如果我们修改了常量类中的常量值，它会影响所有使用它的测试类。这就是为什么我们应该减少这个类中的常量个数的原因。

## 总结 ##

我们已经知道常量能帮助我们编写洁净的测试，也能减少编写新测试或者维护老代码时的工作。当我们在把这篇博客中的建议付诸实践时还有几件事需要记住：

- 要给常量类和常量起个好名字。如果我们不这么干，我们不能充分发挥它的好处。
- 在我们想明白添加一个常量能带来哪些好处前不要随意添加它们。现实中的代码往往比博客中的例子复杂的多。如果我们不经思考的写代码，很有可能就会错过最佳的解决方案。

转载注明出处：[{{page.title}}]({{permalink}})

[原文链接](http://www.petrikainulainen.net/programming/testing/writing-clean-code-beware-of-magic/ "Writing Clean Tests – Beware of Magic")
