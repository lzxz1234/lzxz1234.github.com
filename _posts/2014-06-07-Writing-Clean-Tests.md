---
layout: post
title: 编写干净的单元测试 - 命名很重要
category : junit
tags : [DSL, JUnit, Testing, CleanCode]
---
{% include JB/setup %}

当我们要为应用编写自动化的单元测试时，我们必须要为这些测试类、类中的方法、字段以及在方法中的本地变量命名。

如果我们想写易读的单元测试，那么我们必须要停下机械化的编码工作并把注意力放到命名上。

**当然，说起来来容易做起来难.**

这也是为什么我决定写一篇博客说一下糟糕的命名可能导致的问题，并提供一点解决这些问题的办法。

## 细节是魔鬼 ##

从一定程度上说做到看上去的代码干净还是挺容易的。然而，如果我们想做的更好一点并把我们的代码转化成可执行的描述，我们必须多关注下测试类、方法、字段和本地变量的命名工作。

让我们看看这意味着什么。

### 测试类命名 ###

我们仔细看一下项目中的测试类，就会发现这些类大体分成两种类型：

- 第一种只测试单个类中的方法。这种测试既可以是单元测试也可以是集成测试。
- 第二种包含测试某个功能是否正常工作的集成测试用例。

一个好的名字应该指出测试的类或者功能。也就是，我们应该按以下规则为我们的类命名：

1. 如果测试类属于第一种类型，我们应该按这种规则命名：[被测试类名] + Test。例如，如果我们为 `RepositoryUserService` 类写测试，那么名字应该写成 `RepositoryUserServiceTest`。这样的好处是在测试失败的时候，我们不需要读测试源码就可以知道哪个类出的问题。
2. 如果测试类属于第二种类型，我们应该按这种规则命名：[被测试功能] + Test。例如，如果我们想测试注册功能，那么测试类名字就应该是 `RegistrationTest`。这么做目的是在测试失败时，通过名字约定直接定位出错的功能。

### 测试方法命名 ###

我是 Roy Osherove 提出的命名约定的忠实拥护者。它的主旨就是通过测试方法的名字描述被测试的方法（或者功能）、预期输入或者前置状态以及预期行为。

也就是说，如果我们遵循命名约定，我们应该按这样为测试方法命名：

1. 如果我们是为单一类写的测试，我们应该按这个公式命名：[被测试方法]_[预期输入 / 前置状态]_[预期行为]。例如，我们写一个测试 `registerNewUserAccount()` 方法由于已存在邮件导致的注册失败而抛出异常的用例时，我们应该这样命名：`registerNewUserAccount_ExistingEmailAddressGiven_ShouldThrowException()`。
2. 如果我们要为某个功能写测试，我们应该按这种方式保命：[被测试功能]_[预期输入 / 前置状态]_[预期行为]。例如，我们写一个集成测试用户使用已存在邮箱导致注册失败而提示错误信息的用例时，我们应该这样全名：`registerNewUserAccount_ExistingEmailAddressGiven_ShouldShowErrorMessage()`。

符合约定的命名可以:

- 描述了特定的业务逻辑需要的前置条件。
- 描述了预期的输入（或状态）和预期的输出（或状态）.

换句话说，如果我们遵循命名约定，我们可以在不阅读测试类源代码的情况下回答以下问题：

- 应用的功能是什么？
- 某个功能或者方法在收到输入 `X` 的情况下预期的行为是什么？

同样，如果一个测试失败了，我们可以在不阅读失败用例代码的情况下对可能存在的问题有一个大体的思路。

相当酷, 是吧?

### 类的字段命名 ###

一个测试类可能包含以下字段：
- 测试依赖的冒烟对象或者桩对象。
- 一个指向被测试对象的字段引用。
- 测试用例中使用到的其它对象。

我们应该按和应用中正常代码一致的命名方式为这些字段命名。也就是，这些字段的命名应该能够描述它存在的“目的”。

这条规则听起来相当“容易”，并且貌似对我来说按这种规则为测试类或者别的类命名也确实相当容易。例如，如果我在测试类中添加了一个用来做CRUD操作的字段，我会把它命名成 crudService。当在测试类中添加用的冒烟对象或者桩对象的时候，就把它的类型加在后面。例如，如果有一个用来做CRUD操作的冒烟对象，那么我会给它命名为 crudServiceMock。

这听上去好像很不错，但这是错误的。这不是一个大问题，但问题是一个字段的名字应该用于描述它的“目的”而不是类型。所以，我们不应该把它的类型放到字段名称的后面。

### 本地变量命名 ###

当我们要为测试方法中的本地变量命名时，我们也应该遵循其它业务代码中相同的变量命名规则。

在我看来，最重要的规则主要有：

- 描述变量的意思。按通常的规则来看变量名字必须要用以描述变量内容。
- 不要使用别人可能不明白的简写命名。简写会降低可读性并且通常它也不能告诉你任何东西。
- 不要使用过于泛泛的名字，如 dto、modelObject、或者 data 等。
- 一定要一致。遵循你的编程语言规范的命名规则。当然如果你的项目有自己的命名约定，那么也要切实遵守。

理论已经足够了，现在开始实践。

## 把理论转化成实践 ##

让我们看一个已经修改过（改的更差了）的从 Spring Social 指南中找到的一个例子应用中的单元测试。

这个测试用例的目的是测试 `RepositoryUserService` 类中的 `registerNewUserAccount()` 方法，它用于确保目标方法在用户通过社会化登录注册或者唯一邮箱注册时能够正常工作。

这份源代码是这样的:

{% highlight java linenos %}
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

     private RepositoryUserService service;
     @Mock
     private PasswordEncoder passwordEncoderMock;
     @Mock
     private UserRepository repositoryMock;

     @Before
     public void setUp() {
         service = new RepositoryUserService(passwordEncoderMock, repositoryMock);
     }


     @Test
     public void registerNewUserAccountByUsingSocialSignIn() throws DuplicateEmailException {
         RegistrationForm form = new RegistrationForm();
         form.setEmail("john.smith@gmail.com");
         form.setFirstName("John");
         form.setLastName("Smith");
         form.setSignInProvider(SocialMediaService.TWITTER);

         when(repositoryMock.findByEmail("john.smith@gmail.com")).thenReturn(null);
                  when(repositoryMock.save(isA(User.class))).thenAnswer(new Answer<User>() {
             @Override
             public User answer(InvocationOnMock invocation) throws Throwable {
                 Object[] arguments = invocation.getArguments();
                 return (User) arguments[0];
             }
         });

         User modelObject = service.registerNewUserAccount(form);

         assertEquals("john.smith@gmail.com", modelObject.getEmail());
         assertEquals("John", modelObject.getFirstName());
         assertEquals("Smith", modelObject.getLastName());
         assertEquals(SocialMediaService.TWITTER, modelObject.getSignInProvider());
         assertEquals(Role.ROLE_USER, modelObject.getRole());
         assertNull(modelObject.getPassword());

         verify(repositoryMock, times(1)).findByEmail("john.smith@gmail.com");
         verify(repositoryMock, times(1)).save(modelObject);
         verifyNoMoreInteractions(repositoryMock);
         verifyZeroInteractions(passwordEncoderMock);
     }
}
{% endhighlight %}

这个测试用例中的问题着实不少:
- 字段名称太过抽象，它们具有Mock后缀。
- 测试方法的名称“貌似不错”但它没有指出预期的输入和行为。
- 测试方法中的变量名太糟糕了。

我们可以通过以下修改来提升这份测试用例的可读性：

1. 把 `RepositoryUserService` 字段的名字改成 registrationService（这个类的名字也太差但暂时忽略吧）。
2. 把 `PasswordEncoder` 和 `UserRepository` 字段名字中的 ‘mock’ 单词删掉。
3. 把测试方法名字改成 `registerNewUserAccount_SocialSignInAndUniqueEmail_ShouldCreateNewUserAccountAndSetSignInProvider()`。
4. 把 form 变量名字改成 registration。
5. 把 modelObject 变量名字改成 createdUserAccount。

改完后源代码看起来就这样了：

{% highlight java linenos %}
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
     public void registerNewUserAccount_SocialSignInAndUniqueEmail_ShouldCreateNewUserAccountAndSetSignInProvider() throws DuplicateEmailException {
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

很明显这份测试用例仍然有问题但我认为这些修改已经提高了不少可读性。最重要的提升就是：

1. 测试方法的名字描述了在用户通过社会化登录注册或者唯一邮箱注册时的预期行为。在老测试用例中我们获取这些信息的唯一方法是阅读测试方法的源代码。这很明显比只读方法名来的慢。也就是说，为测试方法起一个好名字可以节省时间并可以帮助我们对测试方法或功能实现快速预览。
2. 另一个变化就是把一个增删改查的测试改成了一个“测试用例”。新测试方法描述更清晰：1.这个测试用例包含哪几步，2.`registerNewUserAccount()` 方法在收到社会化登录注册或者唯一邮箱注册请求时返回什么。

在我看来，老测试方法很明显做不到这些。

> 对于 RegistrationForm 对象的名字我还是不太满意但它已经比原来的好了。

## 总结 ##

我们已经知道了命名可以在代码易读性方面有巨大影响。我们也知道了帮助我们把测试用例转化成可执行描述的一些基本规则。

然而，我们的测试用例仍然有一些问题。他们是：

- 测试用例使用了魔数。我们可以通过把这些魔数用常量替换做到进一步优化。
- 创建 RegistrationForm 对象的代码只是把创建的对象设置了正确的值。于是我们可以通过使用构造模式进一步优化代码。
- 标准的 Junit 断言可以校验返回的 User 对象的各种信息是否是正确的，但还不是很易读。另一个问题是他们只校验了返回的 User 对象的各属性值是否是正确的。我们可以通过把断言转化成面向领域语言优化代码。

在接下来的博客中我会分别介绍这些技术。

同时，我也非常希望听到你们所使用的命名约定。

转载注明出处：[{{page.title}}]({{permalink}})

[原文链接](http://www.petrikainulainen.net/programming/testing/writing-clean-tests-naming-matters/ "Writing Clean Tests – Naming Matters")
