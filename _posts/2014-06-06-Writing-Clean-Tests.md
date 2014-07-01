---
layout: post
title: 编写干净的单元测试 - 从配置开始
category : junit
tags : [DSL, JUnit, Testing, CleanCode]
---
{% include JB/setup %}

当我们准备开始编写单元测试或者集成测试的第一件事情就是先做好相关配置。

如果我们想写一份干净的测试，那么也一定要按干净简单的方式去配置它们，这很明显，对吧？

不幸的是，很多开发者会以不重复的原则（DRY）为借口而选择忽视这件事情。

**这是错误的。**

这篇博客将会指出 DRY 原则存在的问题和一种更好的配置测试用例的方式。

## 存在的问题 ##

假使我们需要使用 SpringMVC 测试框架对 SpringMVC 的控制器(Controller) 编写测试用例。第一个需要测的控制器叫 TodoController，当然别的控制器对应的测试也同样需要写。

作为开发，我们知道重复的代码不是一件好事。当我们写代码的时候，要遵循 DRY 原则，它是这样描述的：

> 在一个系统中的每一项事物必须有一个唯一的，明确的，权威的表示。

我猜这也是很多程序员喜欢在他们的测试用例中使用继承的一个原因。他们认为继承是重用代码和配置最廉价和简单的方式。这也是为什么他们喜欢把公用代码和配置放在实际测试类的公有父类中的原因。

让我们看看如何通过这种方式实现配置“单元测试”。

**首先**，我们需要创建一个配置了 SpringMVC 测试框架的抽象基类，并且它的子类可以通过实现 `setUpTest(MockMvc mockMvc)` 方法来进行补充配置。

AbstractControllerTest 类的源代码看起来是这样的：

{% highlight java linenos %}
import org.junit.Before;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.test.context.web.WebAppConfiguration;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;
import org.springframework.web.context.WebApplicationContext;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = {WebUnitTestContext.class})
@WebAppConfiguration
public abstract class AbstractControllerTest {

     private MockMvc mockMvc;
     @Autowired
     private WebApplicationContext webAppContext;

     @Before
     public void setUp() {
         mockMvc = MockMvcBuilders.webAppContextSetup(webAppContext).build();
         setupTest(MockMvc mockMvc)
     }
     
     protected abstract void setUpTest(MockMvc mockMvc);
}
{% endhighlight %}

**然后**, 我们还需要实现一个负责创建需要的冒烟对象和控制器对象的实际测试类。`TodoControllerTest` 类的源代码看起来是这样的：

{% highlight java linenos %}
import org.mockito.Mockito;
import org.springframework.test.web.servlet.MockMvc;

public class TodoControllerTest extends AbstractControllerTest {

     private MockMvc mockMvc;
     @Autowired
     private TodoService serviceMock;
     
     @Override
     protected void setUpTest(MockMvc mockMvc) {
         Mockito.reset(serviceMock);
         this.mockMvc = mockMvc;
     }
     //Add test methods here
}
{% endhighlight %}

这个测试类相当简洁，但它有个明显的失陷：

**如果我们想看看我们的测试用例是如何配置的，我们必须要 `TodoControllerTest` 和 `AbstractControllerTest` 两个类的源代码。**

这看起来像是一个小问题但它会导致我们的注意力不停在测试类和测试基类间切换。这需要一个精神层面的上下文切换，然后**这个切换代价是很昂贵的**。

当然你有可能会说这种用继承实现的测试用例进行精神切换的代价很低，因为基类相当简单。这是对的，但，要知道，在真正的测试程序中不会总是这么简单。

精神切换的实际代价取决于测试类继承树的深度和配置的复杂度。

## 解决办法 ##

我们可以通过把所有的配置信息都放在实际测试类中来提高可读性。我们可以这样干：

- 在测试类上添加需要的注解（例如 @RunWith）。
- 在测试类中添加 setup 和 teardown 方法。

如果我们按这些规则重写测试类后的代码看起来是这样的：

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
@ContextConfiguration(classes = {WebUnitTestContext.class})
@WebAppConfiguration
public class TodoControllerTest {

     private MockMvc mockMvc;
     @Autowired
     private TodoService serviceMock;
     @Autowired
     private WebApplicationContext webAppContext;

     @Before
     public void setUp() {
         Mockito.reset(serviceMock);
         mockMvc = MockMvcBuilders.webAppContextSetup(webAppContext).build();
     }
     //Add test methods here
}
{% endhighlight %}

在我看来，新的配置方式比原来把配置信息拆分后放在 `TodoControllerTest` 和 `AbstractControllerTest` 两个类中的方式简单和整洁多了。

不幸的是，天下没有免费的午餐。

## 这只是一个交易 ##

每一项设计决策都是利和弊之间的较量。**现在也不例外**。

按我们的方式做测试配置至少有以下好处：

1. 我们可以在不需要阅读测试类的所有父类的情况下就能知道它的配置。这可以为我们节省大量时间因为我们不需要把我们的注意力不停在类之间切换。也就是说，**我们省了切换注意力的代价**。
2. 在测试失败的时候它也可以节省我们的时间。如果我们因为想避免代码或者配置重复而选择了继承，那么问题就是基类中的模块会和测试用例中的模块具有错综复杂的关系。或者说，我们必须要仔细分辨哪些模块会和测试失败相关，然后这可能并不容易。但当我们把所有配置信息都写在具体测试类中后，**我们很容易就会知道和失败测试用例相关的模块**。

同样，这么做也有坏处：
1. 我们必须要写部分重复代码。这比把配置信息放在测试基类中花费的时间长。
2. 如果测试依赖任何一个库有了变动，并要求修改测试配置，那么我们需要对全部测试用例做对应修改。这很明显比只修改一个测试基类来得慢。

如果我们的目标仅仅是尽快完成测试用例，很明显我们应该消除重复代码和配置。

但，这不是我唯一的目标。

至少有两个原因会让我认为这些付出是值得的：

1. 继承不是重用代码和配置的正确方法。
2. 如果某个测试用例失败了，我们必须要尽快找到并且解决问题，明确的配置可以帮我们更快达成目标。

转载注明出处：[{{page.title}}]({{permalink}})

[原文链接](http://www.petrikainulainen.net/programming/testing/writing-clean-tests-it-starts-from-the-configuration/ "Writing Clean Tests – It Starts from the Configuration")