---
layout: post
title: 用 Javassist 获取方法参数名
category : java
tags : [Java, Javassist]
---
{% include JB/setup %}

首先，如果需要在执行过程获取方法参数名，那么代码在编译的时候需要打开调集信息，也就是 `-g` 参数。

此前，网络上普通的获取方法参数名基本代码如下：

{% highlight java linenos %}
public static String[] getMethodParameterNames(Method method) throws Exception {
	CtClass cc = getCtClass(method.getDeclaringClass());
	CtClass[] parameterCtClasses = new CtClass[method.getParameterTypes().length];
	for (int i = 0; i < parameterCtClasses.length; i++) 
	    parameterCtClasses[i] = getCtClass(method.getParameterTypes()[i]);
	
	String[] parameterNames = new String[parameterCtClasses.length];
	CtMethod cm = cc.getDeclaredMethod(method.getName(), parameterCtClasses);
	MethodInfo methodInfo = cm.getMethodInfo();
	CodeAttribute codeAttribute = methodInfo.getCodeAttribute();
	LocalVariableAttribute attr = (LocalVariableAttribute) codeAttribute.getAttribute(LocalVariableAttribute.tag);
	
	int pos = Modifier.isStatic(cm.getModifiers()) ? 0 : 1;  
	for (int i = 0; i < parameterNames.length; i++)  
	    parameterNames[i] = attr.variableName(i + pos);  
    return parameterNames;
}
{% endhighlight %}

主要逻辑在最后三行，因为读取方法变量名需要依赖字节码中的本地变量表，但这三行代码有一个很不靠谱的假设：

>方法参数在本地变量表中一定是保存在最前面的，而且有序。

**这是完全错误的**，以下本地变量表完全合法，而且正常：

{% highlight java linenos %}
LocalVariableTable:
 Start  Length  Slot   Name   Signature
    52      18     3   role   Lcom/hyxq/domain/Role;
    33      40     2     i$   Ljava/util/Iterator;
   120      15     3 roleId   I
    98      40     2     i$   Ljava/util/Iterator;
     0     146     0   this   Lcom/hyxq/service/RbacService;
     0     146     1    org   Lcom/hyxq/domain/Org;
{% endhighlight %}


在上述情况下，原代码获取到的方法参数名将会是内部变量名 `i$` 而不是 `org`。

那么正确的办法应该是怎么样的呢？

先让我们看看虚拟机规范中对本地变量表的相关描述：

> Java虚拟机使用局部变量表来完成方法调用时的参数传递，当一个方法被调用的时候，它的参数将会传递至从0开始的连续的局部变量表位置上。特别地，当一个实例方法被调用的时候，第0个局部变量一定是用来存储被调用的实例方法所在的对象的引用（即Java语言中的“this”关键字）。后续的其他参数将会传递至从1开始的连续的局部变量表位置上。

但此描述中第 0 个局部变量是指 slot 排号为 0，而不是位置为 0。

所以正确的做法就是遍历本地变量表，根据 slot 值确认是不是方法参数，体现在代码中的 API 就是 attr.index(i) 将会返回 slot。

所以一个修改版本如下：

{% highlight java linenos %}
//省略前面代码部分
int pos = Modifier.isStatic(cm.getModifiers()) ? 0 : 1;
for (int i = 0; i < attr.tableLength(); i++) {
    if(attr.index(i) >= pos && attr.index(i) < parameterNames.length + pos)
        parameterNames[attr.index(i) - pos] = attr.variableName(i);
}
//省略后面代码部分
{% endhighlight %}

上述两段代码中的 pos 变量主要用于处理实例方法前面的 this 变量，静态方法 pos 将为 0。

---

实际上述代码还会导致另一个问题，由于 long 和 double 等类型单个变量占用两个 slot，于是 attr.index(i) 是跳跃的，于是一个再次优化的版本如下：

{% highlight java linenos %}
//省略前面代码部分
TreeMap<Integer, String> sortMap = new TreeMap<Integer, String>();
for (int i = 0; i < attr.tableLength(); i++) 
    sortMap.put(attr.index(i), attr.variableName(i));
int pos = Modifier.isStatic(cm.getModifiers()) ? 0 : 1;
parameterNames = Arrays.copyOfRange(sortMap.values().toArray(new String[0]), pos, parameterNames.length + pos);
//省略后面代码部分
{% endhighlight %}


转载注明出处：[{{page.title}}]({{permalink}})