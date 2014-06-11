---
layout: post
title: 编写干净的单元测试 - 用领域驱动语言取代Assert
category : junit
tags : [DSL, JUnit, Testing]
---
{% include JB/setup %}

没有断言的自动化测试是没有价值的，但问题是常规的 Junit 断言是有问题的，并且在不得不大量断言的时候就开始变得非常混乱。

如果我们想写便于理解和维护的测试，那么我们就得找一个写断言更好的方式。

这篇文章指出了标准断言通常存在的问题，也描述了如何通过用领域驱动语言代替断言来解决这个问题。

## 数据不一定是最重要的 ##

在我前面的文章中我指出过以数据为中心的测试可能导致的两个问题。尽管那篇博客讨论的是关于新对象的构造，这些问题在现在中也同样适用。

现在我们回忆下，再看看那个RepositoryUserService类中通过唯一的邮箱地址或者通过社会化登录新建用户的 `registerNewUserAccount (RegistrationForm userAccountData)` 方法能否按预期工作的单元测试的代码。

测试代码大体是这样的:

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
    public void registerNewUserAccount_SocialSignInAndUniqueEmail_ShouldCreateNewUserAccountAndSetSignInProvider() throws DuplicateEmailException {
        RegistrationForm registration = new RegistrationFormBuilder()
            .email(REGISTRATION_EMAIL_ADDRESS)
            .firstName(REGISTRATION_FIRST_NAME)
            .lastName(REGISTRATION_LAST_NAME)
            .isSocialSignInViaSignInProvider(SOCIAL_SIGN_IN_PROVIDER)
            .build();

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

正如我们能看到的，为了确认返回的用户对象的各属性值是正确的我们从测试代码中发现了大量断言。这些断言主要用于确认：

- email 属性是正确的.
- firstName 属性是正确的.
- lastName 属性是正确的.
- signInProvider 属性是正确的.
- role 属性是正确的.
- password 属性是正确的.

当然这是很明显的，但这样重复断言是很有必要的，因为我们需要通过它确认是否存在问题。这些断言就是**以数据为中心**的，并且这意味着：

- **读者必须要明白对象的不同状态**。例如，如果我们仔细想一下前面的代码，读者必须要明白，如果注册表单对象中的
email、firstName、lastName、signInProvider字段有非空的但password字段是空的，那么这就代表这个对象是来自社会化登录的注册对象。
- 如果某个对象字段非常多，那么关于它的断言就会使整个代码非常杂乱。我们需要记住，即使再想确保对象的数据是正确的，先做到**正确描述它的状态**也是更为重要的。

现在让我们看看可以如何改善这些断言。

## 将断言转化为领域驱动语言 ##

你经常会发现开发者和领域专家对相同的事物会有不同的描述。换名话说，程序员和领域专家语言不通。**这就会在大家之间导致不必要的混乱和冲突**。

领域驱动设计为这问题提供了解决办法。Eric Evans 在他的《Domain-Driven Design》一书中对这种通用语言作了详细说明。

维基百科上对于这种通用语言的定义如下:

> 通用语言是围绕领域模型特别定制的语言，它会被所有的小组成员在围绕软件的各种活动中使用。

如果我们想用一种“正确”的语言来实现断言，我们必须要做到开发者和领域专家之间的良好沟通。换句话说，我们必须创建一种面向领域的语言来实现断言。

## 实现我们的面向领域语言 ##

在真正实现之前，我们需要先设计一下。要专门为断言设计一种面向领域的语言我们需要遵循以下原则：

1. 我们必须要舍弃以数据为中心，并且多考虑下User对象代表的真实用户 
2. 我们必须要使用和领域专家一致的语言.

如果我们遵循这两条规则，我们就可以面向领域重新设计我们的代码：
- 一个用户具有 firstName, lastName 和 email 属性.
- 一个用户有一个 registeredUser 对象.
- 一个来自社会化登录的注册用户意味着他没有密码

设计出来怕后，我们就可以开始写对应实现了。我们将会根据前面的规则自定义一套 AssertJ 断言。

自定义的断言类看起来是这样的：

{% highlight java %}
import org.assertj.core.api.AbstractAssert;
import org.assertj.core.api.Assertions;

public class UserAssert extends AbstractAssert<UserAssert, User> {

    private UserAssert(User actual) {
        super(actual, UserAssert.class);
    }
    public static UserAssert assertThat(User actual) {
        return new UserAssert(actual);
    }
    public UserAssert hasEmail(String email) {
        isNotNull();

        Assertions.assertThat(actual.getEmail())
                .overridingErrorMessage( "Expected email to be <%s> but was <%s>",
                        email,
                        actual.getEmail()
                )
                .isEqualTo(email);
        return this;
    }
    public UserAssert hasFirstName(String firstName) {
        isNotNull();

        Assertions.assertThat(actual.getFirstName())
                .overridingErrorMessage("Expected first name to be <%s> but was <%s>",
                        firstName,
                        actual.getFirstName()
                )
                .isEqualTo(firstName);
        return this;
    }
    public UserAssert hasLastName(String lastName) {
        isNotNull();

        Assertions.assertThat(actual.getLastName())
                .overridingErrorMessage( "Expected last name to be <%s> but was <%s>",
                        lastName,
                        actual.getLastName()
                )
                .isEqualTo(lastName);
        return this;
    }
    public UserAssert isRegisteredByUsingSignInProvider(SocialMediaService signInProvider) {
        isNotNull();

        Assertions.assertThat(actual.getSignInProvider())
                .overridingErrorMessage( "Expected signInProvider to be <%s> but was <%s>",
                        signInProvider,
                        actual.getSignInProvider()
                )
                .isEqualTo(signInProvider);
        hasNoPassword();
        return this;
    }
    private void hasNoPassword() {
        isNotNull();

        Assertions.assertThat(actual.getPassword())
                .overridingErrorMessage("Expected password to be <null> but was <%s>",
                        actual.getPassword()
                )
                .isNull();
    }
    public UserAssert isRegisteredUser() {
        isNotNull();

        Assertions.assertThat(actual.getRole())
                .overridingErrorMessage( "Expected role to be <ROLE_USER> but was <%s>",
                        actual.getRole()
                )
                .isEqualTo(Role.ROLE_USER);
        return this;
    }
}
{% endhighlight %}

我们已经为User对象创建了一个面向领域的断言模型。然后下一步就是用这个新语言模型修改我们的测试用例。

## Replacing JUnit Assertions with a Domain-Specific Language ##

用面向领域语言的断言方式重写测试代码后看起来是这样的：

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
    public void registerNewUserAccount_SocialSignInAndUniqueEmail_ShouldCreateNewUserAccountAndSetSignInProvider() throws DuplicateEmailException {
        RegistrationForm registration = new RegistrationFormBuilder()
            .email(REGISTRATION_EMAIL_ADDRESS)
            .firstName(REGISTRATION_FIRST_NAME)
            .lastName(REGISTRATION_LAST_NAME)
            .isSocialSignInViaSignInProvider(SOCIAL_SIGN_IN_PROVIDER)
            .build();

        when(repository.findByEmail(REGISTRATION_EMAIL_ADDRESS)).thenReturn(null);
        when(repository.save(isA(User.class))).thenAnswer(new Answer<User>() {
            @Override
            public User answer(InvocationOnMock invocation) throws Throwable {
                Object[] arguments = invocation.getArguments();
                return (User) arguments[0];
            }
        });

        User createdUserAccount = registrationService.registerNewUserAccount(registration);

        assertThat(createdUserAccount)
            .hasEmail(REGISTRATION_EMAIL_ADDRESS)
            .hasFirstName(REGISTRATION_FIRST_NAME)
            .hasLastName(REGISTRATION_LAST_NAME)
            .isRegisteredUser()
            .isRegisteredByUsingSignInProvider(SOCIAL_SIGN_IN_PROVIDER);

        verify(repository, times(1)).findByEmail(REGISTRATION_EMAIL_ADDRESS);
        verify(repository, times(1)).save(createdUserAccount);
        verifyNoMoreInteractions(repository);
        verifyZeroInteractions(passwordEncoder);
    }
}
{% endhighlight %}

这种解决方案至少会有以下好处:

- 这种方式写成的断言是可以被领域专家理解的。这意味着我们的测试用例是一个可执行的定义，它很容易被理解也容易更新。
- 我们不需要浪费时间去定位为什么测试失败了。我们自定义的错误信息足以让我们知道失败的原因。
- 如果 User 类的 API 变更了，我们不需要挨个修改对 User 对象做过断言的方法。唯一需要修改的是 UserAssert 类。换句话说，把实际的断言逻辑从测试方法中移出去使我们的测试类更健壮和易于维护。

让我们花点时间总结下从这篇文章学到了什么。

## 总结 ##

我们已经顺利把断言改造成了面向领域语言的。这篇博客教会我们三件事：

- 以数据为中心会导致开发和领域专家间不必要的混乱和冲突
- 为断言创建一种面向领域的语言会使我们的测试类更健壮因为我们把所有的断言逻辑都移到自定义断言类里去了.
- 如果我们用面向领域语言实现断言，我们就可以把测试类转换成一个更容易阅读的可执行的定义，这样也更容易被领域专家理解。

转载注明出处：[{{page.title}}]({{permalink}})

[原文链接](http://www.petrikainulainen.net/programming/testing/writing-clean-tests-replace-assertions-with-a-domain-specific-language/ "Writing Clean Tests – Replace Assertions with a Domain-Specific Language")
