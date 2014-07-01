---
layout: post
title: 编写干净的单元测试 - 分治法
category : junit
tags : [DSL, JUnit, Testing, CleanCode]
---
{% include JB/setup %}

一个好的测试用例只应该有一个失败原因。也就是说一个测试用例只能有一个逻辑流程。

如果我们想写干净的单元测试，我也必须要会分辨这些逻辑流程，并做到一个测试用例中只有一个流程。

这篇博客就是向我们描述如果分辨不同流程，以及如何把它们拆分到不同测试用例中。

## 代码干净还是不够好的 ##

现在我们再看看前面那个用以确保 RepositoryUserService 类的用来通过唯一邮箱或者社会化登录来实现用户注册的 `registerNewUserAccount(RegistrationForm userAccountData)` 方法能够按预期运行的测试用例源代码。

这份代码看起来是这样的：

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

    private static final String REGISTRATION_EMAIL_ADDRESS = "john.smith@gmail.com";
    private static final String REGISTRATION_FIRST_NAME = "John";
    private static final String REGISTRATION_LAST_NAME = "Smith";
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

这份代码已经足够干净了。毕竟，我们在测试方法中创建的类、方法、变量都具有了描述性的名字。我们也已经把魔数替换成了常量，并且用了面向领域的语言来创建对象和实现断言。

但是，**我们还能做的更好**.

现在这个测试用例存在的问题就是他会因为很多种原因失败。可能导致失败的原因：

1. service 方法没有确认用户注册时输入的邮箱在数据库中是不存在的。
2. 数据库中保存的用户信息和用户注册时填写的不一致。
3. 返回的 User 对象是错误的。
4. service 方法使用 PasswordEncoder 对象为用户生成了一个密码。

也就是说，这个单元测试实现四种不同的逻辑流程，这可能导致以下问题：

- 如果测试失败了，我们不一定知道为什么失败了。我们可能还是需要阅读测试用例源代码。
- 由于这个测试用例太长，可能也有点不太易读。
- 很难描述预期行为。或者说很难为测试方法指出一个明确的名字。

> 我们可以通过分辨测试用例中导致失败的不同场景做到分辨其中的不同逻辑。

这是我们为什么需要将一个测试用例拆分成四个.

## 一个测试，一个故障点 ##

我们的下一步就是把原来的测试用例拆分成四个并且确保每一个分用例测试一个逻辑流程。我们可以这样干：

1. 我们需要确保 service 方法已经对 user 提供的邮箱地址做了唯一性校验.
2. 我们需要校验据库中存储的用户对象各项信息是正确的.
3. 我们需要确保返回的 User 对象各项信息是正确的.
4. 我们需要验证 serivce 方法没有为社会化登录注册的用户创建密码.

当我们做完这些后，代码看起来应该是这样的：

{% highlight java linenos %}
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.mockito.ArgumentCaptor;
import org.mockito.Mock;
import org.mockito.invocation.InvocationOnMock;
import org.mockito.runners.MockitoJUnitRunner;
import org.mockito.stubbing.Answer;
import org.springframework.security.crypto.password.PasswordEncoder;

import static net.petrikainulainen.spring.social.signinmvc.user.model.UserAssert.assertThat;
import static org.mockito.Matchers.isA;
import static org.mockito.Mockito.times;
import static org.mockito.Mockito.verify;
import static org.mockito.Mockito.verifyZeroInteractions;
import static org.mockito.Mockito.when;

@RunWith(MockitoJUnitRunner.class)
public class RepositoryUserServiceTest {

    private static final String REGISTRATION_EMAIL_ADDRESS = "john.smith@gmail.com";
    private static final String REGISTRATION_FIRST_NAME = "John";
    private static final String REGISTRATION_LAST_NAME = "Smith";
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
    public void registerNewUserAccount_SocialSignInAndUniqueEmail_ShouldCheckThatEmailIsUnique() throws DuplicateEmailException {
        RegistrationForm registration = new RegistrationFormBuilder()
                .email(REGISTRATION_EMAIL_ADDRESS)
                .firstName(REGISTRATION_FIRST_NAME)
                .lastName(REGISTRATION_LAST_NAME)
                .isSocialSignInViaSignInProvider(SOCIAL_SIGN_IN_PROVIDER)
                .build();

        when(repository.findByEmail(REGISTRATION_EMAIL_ADDRESS)).thenReturn(null);
        registrationService.registerNewUserAccount(registration);
        verify(repository, times(1)).findByEmail(REGISTRATION_EMAIL_ADDRESS);
    }

    @Test
    public void registerNewUserAccount_SocialSignInAndUniqueEmail_ShouldSaveNewUserAccountAndSetSignInProvider() throws DuplicateEmailException {
        RegistrationForm registration = new RegistrationFormBuilder()
                .email(REGISTRATION_EMAIL_ADDRESS)
                .firstName(REGISTRATION_FIRST_NAME)
                .lastName(REGISTRATION_LAST_NAME)
                .isSocialSignInViaSignInProvider(SOCIAL_SIGN_IN_PROVIDER)
                .build();

        when(repository.findByEmail(REGISTRATION_EMAIL_ADDRESS)).thenReturn(null);
        registrationService.registerNewUserAccount(registration);

        ArgumentCaptor<User> userAccountArgument = ArgumentCaptor.forClass(User.class);
        verify(repository, times(1)).save(userAccountArgument.capture());
        User createdUserAccount = userAccountArgument.getValue();
        assertThat(createdUserAccount)
                .hasEmail(REGISTRATION_EMAIL_ADDRESS)
                .hasFirstName(REGISTRATION_FIRST_NAME)
                .hasLastName(REGISTRATION_LAST_NAME)
                .isRegisteredUser()
                .isRegisteredByUsingSignInProvider(SOCIAL_SIGN_IN_PROVIDER);
    }


    @Test
    public void registerNewUserAccount_SocialSignInAndUniqueEmail_ShouldReturnCreatedUserAccount() throws DuplicateEmailException {
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
    }

    @Test
    public void registerNewUserAccount_SocialSignInAnUniqueEmail_ShouldNotCreateEncodedPasswordForUser() throws DuplicateEmailException {
        RegistrationForm registration = new RegistrationFormBuilder()
                .email(REGISTRATION_EMAIL_ADDRESS)
                .firstName(REGISTRATION_FIRST_NAME)
                .lastName(REGISTRATION_LAST_NAME)
                .isSocialSignInViaSignInProvider(SOCIAL_SIGN_IN_PROVIDER)
                .build();

        when(repository.findByEmail(REGISTRATION_EMAIL_ADDRESS)).thenReturn(null);
        registrationService.registerNewUserAccount(registration);
        verifyZeroInteractions(passwordEncoder);
    }
}
{% endhighlight %}

为每个逻辑流程编辑单独测试用例很明显的好处是它让我们更容易知道什么导致的测试失败。而且，这样做同时还有另外两个好处：

- 更容易指出预期的行为。也就是更容易为测试方法起一个确切的名字.
- 因为这些拆分后的测试用例长度远短于原有测试用例，也就更容易看出他们需要的条件。这可以帮助我们更容易的把测试用例转化成可执行描述。

现在让我们总结下从这篇文章学到了什么.

## 总结 ##

现在我们已经成功的把原来的测试用例转拆分成了四个只负责的单流程逻辑的小测试用例。这篇博客教会了我们两件事情：

- 我们可能通过分辨导致测试失败的不同场景来分辨不同的流程逻辑
- 编写只负责一个流程逻辑的测试用例可以帮助我们更容易把测试用例转换成可执行描述。

转载注明出处：[{{page.title}}]({{permalink}})

[原文链接](http://www.petrikainulainen.net/programming/testing/writing-clean-tests-replace-assertions-with-a-domain-specific-language/ "Writing Clean Tests – Replace Assertions with a Domain-Specific Language")