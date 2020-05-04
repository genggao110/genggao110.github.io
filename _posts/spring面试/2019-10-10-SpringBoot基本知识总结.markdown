---
layout:     post
title:      "Spring及SpringBoot基础知识总结"
subtitle:   " \"Spring总结\""
date:       2019-10-10 22:16:07
author:     "ming"
catalog: true
header-img: "img/post-bg-happiness.jpg"
tags:
    - SpringBoot
    - Spring框架
---

> "To choose time is to save time."

## Spring AOP详解

### 1. 什么是AOP

AOP(Aspect-Oriented Programming)，即面向切面编程，它与OOP( Object-Oriented Programming,面向对象编程)相辅相成，提供了与OOP不同的抽象软件结构的视角。在 OOP 中, 我们以类(class)作为我们的基本单元, 而 AOP 中的基本单元是 Aspect(切面)。

### 2. 术语介绍

#### 2.1 Aspect(切面)

Aspect由Pointcut和advice组成，它既包含了横切逻辑的定义，也包括了连接点的定义。Spring AOP就是负责实施切面的框架，它将切面所定义的横切逻辑织入到切面所指定的连接点中。

AOP的工作重心在于如何将增强织入目标对象的连接点上，这里主要包含两个部分的工作：
- 如何通过pointcut和advice定位到特定的joinpoint上
- 如何在advice中编写切面代码

`可以简单的认为，使用@Aspect注解的类就是切面`。aspect用Spring的 Advisor或拦截器实现。

#### 2.2 advice(增强)

由aspect添加到特点的join point(即满足point cut规则的join point)的一段代码。许多AOP框架，包括Spring AOP,会将advice模拟为一个拦截器(interceptor)，并且在join point上维护多个advice，进行层层拦截。

例如HTTP鉴权的实现，我们可以为每个使用RequestMapping标注的方法织入advice，当HTTP请求到来的时候，首先进入到advice代码中，在这里我们可以分析这个HTTP请求是否有相应的权限，如果有，则执行Controller，如果没有，则抛出异常。这里的advice就扮演者鉴权拦截器的角色。

#### 2.3 Join Point(连接点)

> a point during the execution of a program, such as the execution of a method or the handling of an exception. In Spring AOP, a join point always represents a method execution.

程序运行中的一些时间点, 例如一个方法的执行, 或者是一个异常的处理。在 Spring AOP 中, join point 总是方法的执行点, 即只有方法连接点.

#### 2.4 point cut(切点)

匹配 join point 的谓词(a predicate that matches join points).Advice是和特定的point cut关联的，并且在point cut相匹配的join point中执行。

在Spring中，所有的方法都可以认为是joinpoint，但是我们并不希望在所有的方法上都添加Advice，而pointcut的作用就是提供一组规则(使用AspectJ pointcut expression language 来描述)来匹配joinpoint,给满足规则的joinpoint添加Advice。

#### 2.5 join point和point cut的区别

在Spring AOP中，所有的方法执行都是join point。而point cut是一个描述信息，它修饰的是join point ，通过point cut，我们就可以确定哪些join point可以被织入Advice。因此join point和point cut本质上就是两个维度上的东西。(`advice 是在 join point 上执行的, 而 point cut 规定了哪些 join point 可以执行哪些 advice`)

#### 2.6 introduction(引介)

为一个类型添加额外的方法或字段。Spring AOP允许我们为目标对象引入新的接口(和对应的实现)。例如我们可以使用introduction为一个bean实现IsModified接口，并以此来简化caching的实现。Spring中要使用Introduction, 可有通过DelegatingIntroductionInterceptor来实现通知，通过DefaultIntroductionAdvisor来配置Advice和代理类要实现的接口。

#### 2.7 target(目标对象)

织入 advice 的目标对象。目标对象也被称为 advised object。因为 Spring AOP 使用运行时代理的方式来实现 aspect, 因此 adviced object 总是一个代理对象(proxied object)注意, adviced object 指的不是原来的类, 而是织入 advice 后所产生的代理类。

#### 2.8 AOP proxy

一个类被 AOP 织入 advice, 就会产生一个结果类, 它是融合了原类和增强逻辑的代理类。在 Spring AOP 中, 一个 AOP 代理是一个 JDK 动态代理对象或 CGLIB 代理对象。

#### 2.9 Weaving(织入)

将aspect和其他对象连接起来，并创建adviced object的过程。根据不同的实现技术，AOP织入有三种方式：
- 编译器织入，这要求有特殊的Java编译器
- 类装载期间织入，这需要有特殊的类装载器
- 动态代理织入，在运行期为目标类添加增强(Advice)生成子类的方式。Spring采用动态代理织入，而AspectJ采用编译器织入和类装载期织入。

### 3. 内容介绍

#### 3.1 advice的类型

- before advice, 在 join point 前被执行的 advice. 虽然 before advice 是在 join point 前被执行, 但是它并不能够阻止 join point 的执行, 除非发生了异常(即我们在 before advice 代码中, 不能人为地决定是否继续执行 join point 中的代码)
- after return advice, 在一个 join point 正常返回后执行的 advice
- after throwing advice, 当一个 join point 抛出异常后执行的 advice
- after(final) advice, 无论一个 join point 是正常退出还是发生了异常, 都会被执行的 advice.
- around advice, 在 join point 前和 joint point 退出后都执行的 advice. 这个是最常用的 advice.

#### 3.2 关于AOP Proxy

Spring AOP 默认使用标准的 JDK 动态代理(dynamic proxy)技术来实现 AOP 代理, 通过它, 我们可以为任意的接口实现代理。`如果需要为一个类实现代理, 那么可以使用 CGLIB 代理.`当一个业务逻辑对象没有实现接口时, 那么Spring AOP 就默认使用 CGLIB 来作为 AOP 代理了。即如果我们需要为一个方法织入 advice, 但是这个方法不是一个接口所提供的方法, 则此时 Spring AOP 会使用 CGLIB 来实现动态代理. 鉴于此, Spring AOP 建议基于接口编程, 对接口进行 AOP 而不是类。

#### 3.3 HTTP 接口鉴权实际案例

首先我们来想象一下如下场景: 我们需要提供的 HTTP RESTful 服务, 这个服务会提供一些比较敏感的信息, 因此对于某些接口的调用会进行调用方权限的校验, 而某些不太敏感的接口则不设置权限, 或所需要的权限比较低(例如某些监控接口, 服务状态接口等).实现这样的需求的方法有很多, 例如我们可以在每个 HTTP 接口方法中对服务请求的调用方进行权限的检查, 当调用方权限不符时, 方法返回错误. 当然这样做并无不可, 不过如果我们的 api 接口很多, 每个接口都进行这样的判断, 无疑有很多冗余的代码, 并且很有可能有某个粗心的家伙忘记了对调用者的权限进行验证, 这样就会造成潜在的 bug.

我们来提炼一下需求：
- 可以定制地为某些指定的HTTP RESTful api提供权限验证功能
- 当调用方的权限不符合时，返回错误。

根据上面所提出的需求, 我们可以进行如下设计:
1. 提供一个特殊的注解`AuthChecker`，这是一个方法注解，有此注解锁标注的Controller需要进行调用方权限的认证。
2. 利用Spring AOP,以@annotation切点标识符来匹配有注解AuthChecker所标注的joinpoint
3. 在 advice 中, 简单地检查调用者请求中的 Cookie 中是否有我们指定的 token, 如果有, 则认为此调用者权限合法, 允许调用, 反之权限不合法, 范围错误。

根据上面的设计，我们来具体看一下源码：

首先给出AuthChecker 注解的定义:

```java
// 1. AuthChecker.java  注解的定义
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface AuthChecker {
}
```
AuthChecker 注解是一个方法注解, 它用于注解 RequestMapping 方法。有了注解的定义，我们来看一下aspect的实现：

```java
@Component
@Aspect
public class HttpAopAdviseDefine {
 
    // 定义一个 Pointcut, 使用 切点表达式函数 来描述对哪些 Join point 使用 advise.
    @Pointcut("@annotation(com.xys.demo1.AuthChecker)")
    public void pointcut() {
    }
 
    // 定义 advise
    @Around("pointcut()")
    public Object checkAuth(ProceedingJoinPoint joinPoint) throws Throwable {
        HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes())
                .getRequest();
 
        // 检查用户所传递的 token 是否合法
        String token = getUserToken(request);
        if (!token.equalsIgnoreCase("123456")) {
            return "错误, 权限不合法!";
        }
 
        return joinPoint.proceed();
    }
 
    private String getUserToken(HttpServletRequest request) {
        Cookie[] cookies = request.getCookies();
        if (cookies == null) {
            return "";
        }
        for (Cookie cookie : cookies) {
            if (cookie.getName().equalsIgnoreCase("user_token")) {
                return cookie.getValue();
            }
        }
        return "";
    }
}
```
在这个 aspect 中, 我们首先定义了一个 pointcut, 以 `@annotation` 切点标志符来匹配有注解 AuthChecker 所标注的 joinpoint, 即:

```java
// 定义一个 Pointcut, 使用 切点表达式函数 来描述对哪些 Join point 使用 advise.
@Pointcut("@annotation(com.xys.demo1.AuthChecker)")
public void pointcut() {
}
```
然后再定义一个 advice:

```java
// 定义 advise
@Around("pointcut()")
public Object checkAuth(ProceedingJoinPoint joinPoint) throws Throwable {
    HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes())
            .getRequest();
 
    // 检查用户所传递的 token 是否合法
    String token = getUserToken(request);
    if (!token.equalsIgnoreCase("123456")) {
        return "错误, 权限不合法!";
    }
 
    return joinPoint.proceed();
}
```
当被 AuthChecker 注解所标注的方法调用前, 会执行我们的这个 advice, 而这个 advice 的处理逻辑很简单, 即从 HTTP 请求中获取名为 user_token 的 cookie 的值, 如果它的值是 123456, 则我们认为此 HTTP 请求合法, 进而调用 joinPoint.proceed() 将 HTTP 请求转交给相应的控制器处理; 而如果user_token cookie 的值不是 123456, 或为空, 则认为此 HTTP 请求非法, 返回错误.

接下来我们写一个模拟的HTTP接口：

```java
@RestController
public class DemoController {
    @RequestMapping("/aop/http/alive")
    public String alive() {
        return "服务一切正常";
    }
 
    @AuthChecker
    @RequestMapping("/aop/http/user_info")
    public String callSomeInterface() {
        return "调用了 user_info 接口.";
    }
}
```
注意到上面我们提供了两个 HTTP 接口, 其中 接口 /aop/http/alive 是没有 AuthChecker 标注的, 而 /aop/http/user_info 接口则用到了 @AuthChecker 标注. 那么自然地, 当请求了 /aop/http/user_info 接口时, 就会触发我们所设置的权限校验逻辑。

### 4.源码解析

#### 4.1 Spring AOP组件

下面这种类图列出了Spring中主要的AOP组件：

![AOP组件](https://tva2.sinaimg.com/large/005CDUpdgy1g8e9tkq1idj31pa1d6k7j.jpg)

#### 4.2 如何使用Spring AOP(不是AspectJ)

关于AspectJ实现AOP的方式可以参考这篇文章：[Spring AOP 理论篇 ](https://www.cnblogs.com/kaleidoscope/p/9524698.html)

可以通过配置文件或者编程的方式来使用Spring AOP:
1.  配置ProxyFactoryBean，显式地设置advisors, advice, target等
2.  配置AutoProxyCreator，这种方式下，还是如以前一样使用定义的bean，但是从容器中获得的其实已经是代理对象
3.  通过`<aop:config>`来配置
4.  通过`<aop: aspectj-autoproxy>`来配置，使用AspectJ的注解来标识通知及切入点

也可以直接使用ProxyFactory来以编程的方式使用Spring AOP，通过ProxyFactory提供的方法可以设置target对象, advisor等相关配置，最终通过 getProxy()方法来获取代理对象。

#### 4.3 Spring AOP代理对象的生成

Spring提供了两种方式来生成代理对象: JDKProxy和Cglib，具体使用哪种方式生成由AopProxyFactory根据AdvisedSupport对象的配置来决定。默认的策略是如果目标类是接口，则使用JDK动态代理技术，否则使用Cglib来生成代理。下面我们来研究一下Spring如何使用JDK来生成代理对象，具体的生成代码放在`JdkDynamicAopProxy`这个类中，直接上相关代码：

```java
    public Object getProxy(ClassLoader classLoader) {
        if (logger.isDebugEnabled()) {
            logger.debug("Creating JDK dynamic proxy: target source is " + this.advised.getTargetSource());
        }

        Class<?>[] proxiedInterfaces = AopProxyUtils.completeProxiedInterfaces(this.advised, true);
        this.findDefinedEqualsAndHashCodeMethods(proxiedInterfaces);
        return Proxy.newProxyInstance(classLoader, proxiedInterfaces, this);
    }
```
下面的问题是，代理对象生成了，那切面是如何织入的？

我们知道InvocationHandler是JDK动态代理的核心，生成的代理对象的方法调用都会委托到InvocationHandler.invoke()方法。而通过JdkDynamicAopProxy的签名我们可以看到这个类其实也实现了InvocationHandler，下面我们就通过分析这个类中实现的invoke()方法来具体看下Spring AOP是如何织入切面的。

```java
/**
 * Implementation of {@code InvocationHandler.invoke}.
 * <p>Callers will see exactly the exception thrown by the target,
 * unless a hook method throws an exception.
 */
@Override
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    MethodInvocation invocation;
    Object oldProxy = null;
    boolean setProxyContext = false;
 
    TargetSource targetSource = this.advised.targetSource;
    Class<?> targetClass = null;
    Object target = null;
 
    try {
        if (!this.equalsDefined && AopUtils.isEqualsMethod(method)) {
            // The target does not implement the equals(Object) method itself.
            return equals(args[0]);
        }
        if (!this.hashCodeDefined && AopUtils.isHashCodeMethod(method)) {
            // The target does not implement the hashCode() method itself.
            return hashCode();
        }
        if (!this.advised.opaque && method.getDeclaringClass().isInterface() &&
                method.getDeclaringClass().isAssignableFrom(Advised.class)) {
            // Service invocations on ProxyConfig with the proxy config...
            return AopUtils.invokeJoinpointUsingReflection(this.advised, method, args);
        }
 
        Object retVal;
 
        if (this.advised.exposeProxy) {
            // Make invocation available if necessary.
            oldProxy = AopContext.setCurrentProxy(proxy);
            setProxyContext = true;
        }
 
        // May be null. Get as late as possible to minimize the time we "own" the target,
        // in case it comes from a pool.
        target = targetSource.getTarget();
        if (target != null) {
            targetClass = target.getClass();
        }
 
        // Get the interception chain for this method.
        List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);
 
        // Check whether we have any advice. If we don't, we can fallback on direct
        // reflective invocation of the target, and avoid creating a MethodInvocation.
        if (chain.isEmpty()) {
            // We can skip creating a MethodInvocation: just invoke the target directly
            // Note that the final invoker must be an InvokerInterceptor so we know it does
            // nothing but a reflective operation on the target, and no hot swapping or fancy proxying.
            Object[] argsToUse = AopProxyUtils.adaptArgumentsIfNecessary(method, args);
            retVal = AopUtils.invokeJoinpointUsingReflection(target, method, argsToUse);
        }
        else {
            // We need to create a method invocation...
            invocation = new ReflectiveMethodInvocation(proxy, target, method, args, targetClass, chain);
            // Proceed to the joinpoint through the interceptor chain.
            retVal = invocation.proceed();
        }
 
        // Massage return value if necessary.
        Class<?> returnType = method.getReturnType();
        if (retVal != null && retVal == target && returnType.isInstance(proxy) &&
                !RawTargetAccess.class.isAssignableFrom(method.getDeclaringClass())) {
            // Special case: it returned "this" and the return type of the method
            // is type-compatible. Note that we can't help if the target sets
            // a reference to itself in another returned object.
            retVal = proxy;
        }
        else if (retVal == null && returnType != Void.TYPE && returnType.isPrimitive()) {
            throw new AopInvocationException(
                    "Null return value from advice does not match primitive return type for: " + method);
        }
        return retVal;
    }
    finally {
        if (target != null && !targetSource.isStatic()) {
            // Must have come from TargetSource.
            targetSource.releaseTarget(target);
        }
        if (setProxyContext) {
            // Restore old proxy.
            AopContext.setCurrentProxy(oldProxy);
        }
    }
}
```
主流程可以简述为：获取可以应用到此方法上的通知链（Interceptor Chain）,如果有,则应用通知,并执行joinpoint; 如果没有,则直接反射执行joinpoint。而这里的关键是通知链是如何获取的以及它又是如何执行的，下面逐一分析下。

首先，从上面的代码可以看到，通知链是通过`Advised.getInterceptorsAndDynamicInterceptionAdvice()`这个方法来获取的,我们来看下这个方法的实现:

```java
    public List<Object> getInterceptorsAndDynamicInterceptionAdvice(Method method, Class<?> targetClass) {
        AdvisedSupport.MethodCacheKey cacheKey = new AdvisedSupport.MethodCacheKey(method);
        List<Object> cached = (List)this.methodCache.get(cacheKey);
        if (cached == null) {
            cached = this.advisorChainFactory.getInterceptorsAndDynamicInterceptionAdvice(this, method, targetClass);
            this.methodCache.put(cacheKey, cached);
        }

        return cached;
    }
```
可以看到实际的获取工作其实是由AdvisorChainFactory. getInterceptorsAndDynamicInterceptionAdvice()这个方法来完成的，获取到的结果会被缓存。

下面来分析这个方法的实现：

```java
@Override
public List<Object> getInterceptorsAndDynamicInterceptionAdvice(
        Advised config, Method method, Class<?> targetClass) {
 
    // This is somewhat tricky... We have to process introductions first,
    // but we need to preserve order in the ultimate list.
    List<Object> interceptorList = new ArrayList<Object>(config.getAdvisors().length);
    Class<?> actualClass = (targetClass != null ? targetClass : method.getDeclaringClass());
    boolean hasIntroductions = hasMatchingIntroductions(config, actualClass);
    AdvisorAdapterRegistry registry = GlobalAdvisorAdapterRegistry.getInstance();
 
    for (Advisor advisor : config.getAdvisors()) {
        if (advisor instanceof PointcutAdvisor) {
            // Add it conditionally.
            PointcutAdvisor pointcutAdvisor = (PointcutAdvisor) advisor;
            if (config.isPreFiltered() || pointcutAdvisor.getPointcut().getClassFilter().matches(actualClass)) {
                MethodInterceptor[] interceptors = registry.getInterceptors(advisor);
                MethodMatcher mm = pointcutAdvisor.getPointcut().getMethodMatcher();
                if (MethodMatchers.matches(mm, method, actualClass, hasIntroductions)) {
                    if (mm.isRuntime()) {
                        // Creating a new object instance in the getInterceptors() method
                        // isn't a problem as we normally cache created chains.
                        for (MethodInterceptor interceptor : interceptors) {
                            interceptorList.add(new InterceptorAndDynamicMethodMatcher(interceptor, mm));
                        }
                    }
                    else {
                        interceptorList.addAll(Arrays.asList(interceptors));
                    }
                }
            }
        }
        else if (advisor instanceof IntroductionAdvisor) {
            IntroductionAdvisor ia = (IntroductionAdvisor) advisor;
            if (config.isPreFiltered() || ia.getClassFilter().matches(actualClass)) {
                Interceptor[] interceptors = registry.getInterceptors(advisor);
                interceptorList.addAll(Arrays.asList(interceptors));
            }
        }
        else {
            Interceptor[] interceptors = registry.getInterceptors(advisor);
            interceptorList.addAll(Arrays.asList(interceptors));
        }
    }
 
    return interceptorList;
}
```
这个方法执行完成后，Advised中配置能够应用到连接点或者目标类的Advisor全部被转化成了MethodInterceptor。

接下来我们再看下得到的拦截器链是怎么起作用的。

```java
// Check whether we have any advice. If we don't, we can fallback on direct
// reflective invocation of the target, and avoid creating a MethodInvocation.
if (chain.isEmpty()) {
    // We can skip creating a MethodInvocation: just invoke the target directly
    // Note that the final invoker must be an InvokerInterceptor so we know it does
    // nothing but a reflective operation on the target, and no hot swapping or fancy proxying.
    Object[] argsToUse = AopProxyUtils.adaptArgumentsIfNecessary(method, args);
    retVal = AopUtils.invokeJoinpointUsingReflection(target, method, argsToUse);
}
else {
    // We need to create a method invocation...
    invocation = new ReflectiveMethodInvocation(proxy, target, method, args, targetClass, chain);
    // Proceed to the joinpoint through the interceptor chain.
    retVal = invocation.proceed();
}
```
从这段代码可以看出，如果得到的拦截器链为空，则直接反射调用目标方法，否则创建MethodInvocation，调用其proceed方法，触发拦截器链的执行，来看下具体代码:

```java
       public Object proceed() throws Throwable {
       //  We start with an index of -1and increment early.
       if (this.currentInterceptorIndex == this.interceptorsAndDynamicMethodMatchers.size()- 1) {
           //如果Interceptor执行完了，则执行joinPoint
           return invokeJoinpoint();
       }
  
       Object interceptorOrInterceptionAdvice =
           this.interceptorsAndDynamicMethodMatchers.get(++this.currentInterceptorIndex);
        
       //如果要动态匹配joinPoint
       if (interceptorOrInterceptionAdvice instanceof InterceptorAndDynamicMethodMatcher){
           // Evaluate dynamic method matcher here: static part will already have
           // been evaluated and found to match.
           InterceptorAndDynamicMethodMatcher dm =
                (InterceptorAndDynamicMethodMatcher)interceptorOrInterceptionAdvice;
           //动态匹配：运行时参数是否满足匹配条件
           if (dm.methodMatcher.matches(this.method, this.targetClass,this.arguments)) {
                //执行当前Intercetpor
                returndm.interceptor.invoke(this);
           }
           else {
                //动态匹配失败时,略过当前Intercetpor,调用下一个Interceptor
                return proceed();
           }
       }
       else {
           // It's an interceptor, so we just invoke it: The pointcutwill have
           // been evaluated statically before this object was constructed.
           //执行当前Intercetpor
           return ((MethodInterceptor) interceptorOrInterceptionAdvice).invoke(this);
       }
}
```

## Spring MVC详解

### 1. 什么是SpringMVC

springMVC是一个MVC的开源框架，springMVC=struts2+spring，springMVC就相当于是Struts2加上spring的整合，但是这里有一个疑惑就是，springMVC和spring是什么样的关系呢？这个在百度百科上有一个很好的解释：意思是说，springMVC是spring的一个后续产品，其实就是spring在原有基础上，又提供了web应用的MVC模块，可以简单的把springMVC理解为是spring的一个模块（类似AOP，IOC这样的模块），网络上经常会说springMVC和spring无缝集成，其实springMVC就是spring的一个子模块，所以根本不需要同spring进行整合。

### 2. 什么是MVC模式

MVC的原理图如下：

![MVC原理图](https://tva1.sinaimg.com/large/005CDUpdgy1g8g0lhaccgj30mc0a90t8.jpg)

- M-Model(模型)：完成业务逻辑，由javaBean构成，service + dao + entity
- V-View(视图)： 界面展示jsp,html，...
- C-Controller(控制器)： 接收请求-调用模型-根据结果派发页面。

### 3. SpringMVC 工作原理

给出网上比较清晰明了的几个工作流程图：

![SpringMVC](https://tva1.sinaimg.com/large/005CDUpdgy1g8g0tfsz5hj30rf0s6tf6.jpg)

![SpringMVC](https://tva1.sinaimg.com/large/005CDUpdgy1g8g0u30y47j30ue0lptam.jpg)

#### 3.1 SpringMvc工作流程

根据上图给出一个简易的请求流程如下所示：

1. 用户发起请求到前端控制器(DispatcherServlet)
2. 前端控制器请求处理映射器(HandlerMapping)去查找处理器(handler)，通过xml配置或者注解进行查找
3. 找到以后处理映射器(HandlerMapping)向前端控制器返回执行链(HandlerExecutionChain)
4. 前端控制器(DispatcherServlet)调用处理器适配器(HandlerAdapter)去执行处理器(Handler)
5. 处理器适配器执行Handler
6. Handler执行完之后给处理器适配器返回ModelAndView
7. 处理器适配器向前端控制器返回ModelAndView
8. 前端控制器请求视图解析器(ViewResolver)去进行视图解析
9. 视图解析器向前端控制器返回View
10. 前端控制器对视图进行渲染
11. 前端控制器向用户响应结果。

下面来分析一下各个组件的作用：

**1. DispatcherServlet**

作为前端控制器，整个流程控制的中心，控制其他组件的执行，统一调度，降低了组件之间的耦合性，提高每个组件的扩展性。用户请求到达前端控制器，它就相当于mvc模式中的c，dispatcherServlet是整个流程控制的中心，由它调用其它组件处理用户的请求，dispatcherServlet的存在降低了组件之间的耦合性。

**2. HandlerMapping**

处理器映射器，其作用是根据url查找Handler。HandlerMapping负责据用户请求找到Handler即处理器，springmvc提供了不同的映射器实现不同的映射方式，例如：配置文件方式，实现接口方式，注解方式等。

**3. HandlerAdapter**

处理器适配器，其作用是按照特定规则(HandlerAdater要求的规则)去执行Handler。通过HandlerAdapter对处理器进行执行，这是适配器模式的应用，通过扩展适配器可以对更多类型的处理器进行执行。

**4. Handler**

编写Handler时按照HandlerAdapter的要求去做，这样适配器才可以去正确执行Handler。Handler 是继DispatcherServlet前端控制器的后端控制器，在DispatcherServlet的控制下Handler对具体的用户请求进行处理。由于Handler涉及到具体的用户业务请求，所以一般情况需要程序员根据业务需求开发Handler。

**5. ViewResolver**

视图解析器，进行视图解析，根据逻辑视图名解析成真正的视图(view)。View Resolver负责将处理结果生成View视图，View Resolver首先根据逻辑视图名解析成物理视图名即具体的页面地址，再生成View视图对象，最后对View进行渲染将处理结果通过页面展示给用户。 springmvc框架提供了很多的View视图类型，包括：jstlView、freemarkerView、pdfView等。

一般情况下需要通过页面标签或页面模版技术将模型数据通过页面展示给用户，需要由工程师根据业务需求开发具体的页面。

**6. View**

视图是一个接口，实现类支持不同的View类型（jsp、freemarker、pdf...）。

#### 3.1 SpringMvc具体流程

1. 首先用户发送请求——>DispatcherServlet，前端控制器收到请求后自己不进行处理，而是委托给其他的解析器进行处理，作为统一访问点，进行全局的流程控制；
2. DispatcherServlet——>HandlerMapping， HandlerMapping 将会把请求映射为HandlerExecutionChain 对象（包含一个Handler 处理器（页面控制器）对象、多个HandlerInterceptor 拦截器）对象，通过这种策略模式，很容易添加新的映射策略；
3. DispatcherServlet——>HandlerAdapter，HandlerAdapter 将会把处理器包装为适配器，从而支持多种类型的处理器，即适配器设计模式的应用，从而很容易支持很多类型的处理器；
4. HandlerAdapter——>处理器功能处理方法的调用，HandlerAdapter 将会根据适配的结果调用真正的处理器的功能处理方法，完成功能处理；并返回一个ModelAndView 对象（包含模型数据、逻辑视图名）
5. ModelAndView的逻辑视图名——> ViewResolver， ViewResolver 将把逻辑视图名解析为具体的View，通过这种策略模式，很容易更换其他视图技术；
6. View——>渲染，View会根据传进来的Model模型数据进行渲染，此处的Model实际是一个Map数据结构，因此很容易支持其他视图技术；
7. 返回控制权给DispatcherServlet，由DispatcherServlet返回响应给用户，到此一个流程结束。

## Spring及SpringBoot模块架构

![Spring模块架构](https://tva2.sinaimg.com/large/005CDUpdgy1g8g1xanww2j30fp08fad0.jpg)

1. Spring Core: 主要组件就是BeanFactory,创建JavaBean的工厂，使用控制反转(IOC)模式。将应用程序的配置和依赖性规范与实际的应用程序代码分开。
2. Spring Aop: 集成了面向切面的编程功能。（AOP把一个业务流程分成几个部分，例如权限检查、业务处理、日志记录，每个部分单独处理，然后把它们组装成完整的业务流程，每个部分被称为切面）可以将声明型事务管理集成到应用程序。
3. Spring Context: 一个核心配置文件，为Spring框架提供上下文信息。
4. Spring Dao: Spring操作数据库的模块。
5. Spring ORM：Spring集成了各种orm（object relationship mapping 对象关系映射）框架的模块，集成mybatis
6. Spring web：集成各种优秀的web层框架的模块（Struts、Springmvc）
7. Spring web mvc：Spring web层框架

SpringBoot中主要的各个模块及功能：
1. spring-boot-starter: 核心SpringBoot starter,包括自动配置支持，日志和YAML
2. spring-boot-starter-actuator: 生产准备的特性，用于帮你监控和管理应用
3. spring-boot-starter-web: 对全栈web开发的支持，包括Tomact和spring-webmvc
4. spring-boot-starter-aop: 对面向切面编程的支持，包括spring-aop和AspectJ
5. spring-boot-starter-data-jpa: 对JAVA持久化API的支持，包括spring-data-jpa,spring-orm和Hibernate
6. spring-boot-starter-jdbc: 对JDBC数据库的支持
7. spring-boot-starter-security: 对spring-security的支持。

