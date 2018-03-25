# Spring AOP

---

- [AOP底层原理](#aop底层原理)
- [AOP操作术语](#aop操作术语)
- [基于配置文件的AspectJ](#基于配置文件的aspectj)
- [基于注解的AspectJ](#基于注解的aspectj)

---

## AOP底层原理

底层使用**动态代理**方式实现！

1. 针对**有接口**情况，使用`jdk`动态代理，创建**接口实现类**的代理对象，调实现类方法
2. 针对**无接口**情况，使用`cglib`动态代理，创建**子类**的代理对象，调父类方法

## AOP操作术语

* `Jointpoint` - 类里面**可以被增强**的**方法**
* `Pointcut` - 类里面**实际要增强**的**方法**
* `Advice` - 实际增强的逻辑
  - 前置增强：方法之前执行
  - 后置增强：方法之后执行
  - 异常增强：方法出现异常
  - 最终通知：后置之后执行
  - 环绕通知：方法之前和之后执行
* `Aspect` - 把`Advice`应用到`Pointcut`的过程
* `Target` - 要增强的**类**
* `Weaving` - 把`Advice`应用到`Target`的过程
* `Introduction` - 在不修改代码的前提下，`Introduction`可以在**运行期**为`Target`动态添加方法或属性
* `Proxy` - 一个`Target`被`Weaving`之后就产生一个结果代理类

## SpringAOP一世

### SpringAOP的Pointcut

`Pointcut`接口包含
1. `ClassFilter` - 类型匹配
2. `MethodMatcher` - 方法匹配

`Pointcut`分类
1. `StaticMethodMatcherPointcut` - `MethodMatcher#isRuntime()=false`，不关注匹配方法的参数，静态，性能好
2. `DynamicMethodMatcherPointcut` - `MethodMatcher#isRuntime()=true`

常见内置`Pointcut`
1. `NameMatchMethodPointcut` - 匹配方法名称
2. `JdkRegexMethodPointcut` - 正则匹配**方法签名**
3. `Perl5RegexpMethodPointcut`
4. `AnnotationMatchingPointcut`
5. `ComposablePointcut` - 支持**并**、**交**等逻辑运算
6. `ControlFlowPointcut` - 匹配**特定对象对特定方法的调用执行流程**

### SpringAOP的Advice

`per-class`类型的`Advice`是指其可以在目标对象类的所有实例间共享！
1. `BeforeAdvice` - 包括`MethodBeforeAdvice`
2. `AfterAdvice` - 包括`ThrowsAdvice`和`AfterReturningAdvice`
3. `AroundAdvice` - 包括`MethodInterceptor`

`per-instance`类型的`Advice`为不同实例对象保持各自的状态及相关逻辑
* `Introduction` - `IntroductionInterceptor`
* `DelegatingIntroductionInterceptor`和`DelegatePerTargetObjectIntroductionInterceptor`

### SpringAOP的Aspect

1. `PointcutAdvisor`
   1. `DefaultPointcutAdvisor`可以使用任何类型的`Pointcut`和`Advice`
   2. `NameMatchMethodPointcutAdvisor`限定了`Pointcut`只能使用`NameMatchMethodPointcut`，`Advice`没有限制
   3. `RegexpMethodPointcutAdvisor`限定了`Pointcut`只能使用`JdkRegexpMethodPointcut`或`Perl5RegexpMethodPointcut`
2. `IntroductionAdvisor` - 只能应用于类级别的拦截，只能使用`Introduction`型的`Advice`

某些`Advisor`的`Pointcut`匹配同一个`Joinpoint`的时候，就会在这同一个`Joinpoint`处执行多个`Advice`的横切逻辑！可能需要我们干预其执行顺序！
`Order`越小，优先级越高！

### SpringAOP的织入

`ProxyFactory`是`Spring`最基本的一个织入器！`Spring AOP`是基于代理模式的`AOP`，织入完成后会返回织入了横切逻辑的目标对象的代理对象！
使用`ProxyFactory`只需要指定两个基本：**要对其进行织入的目标对象**和**要应用到目标对象的`Aspect`**！

1. 基于接口的代理 - 如果目标对象实现了至少一个接口，`ProxyFactory`默认按照面向接口进行代理
2. 基于类的代理 - 如果满足下述3种条件之一，`ProxyFactory`将对目标类进行基于类的代理
   1. 目标类没有实现任何接口
   2. `ProxyFactory`的`proxyTargetClass`属性设置为`true`
   3. `ProxyFactory`的`optimize`属性设置为`true`

`Introduction`的织入比较特殊，它可以为**已经存在**的对象类型添加新行为，只能应用于**对象级别**而不是**方法级别**！所以在织入时不需要指定`Pointcut`，只需要指定目标接口类型！`Spring`的`Introduction`支持**只能**通过**接口**定义为当前对象添加新行为！

`ProxyFactoryBean`是`Spring`另一个织入器！如果容器中某个对象依赖于`ProxyFactoryBean`，那么它将会使用`ProxyFactoryBean#getObject`返回的代理对象！

### SpringAop的自动代理

`Spring`可用的`AutoProxyCreator`
1. `BeanNameAutoProxyCreator` - 半自动，需要指定为哪些`Bean`生成代理对象，以及要应用到目标对象的拦截器、`Advice`
2. `DefaultAdvisorAutoProxyCreator` - 全自动，自动搜索容器内所有`Advisor`，根据其拦截信息，为符合条件的容器目标对象生成代理

### TargetSource

`TargetSource`的作用是为目标对象在外面加了个壳，是目标对象的**容器**！实际上，针对目标对象的方法调用经历层层拦截而到达调用链终点时，**并不是**直接调用目标对象方法，而是通过从`TargetSource`中取得目标对象上的相应方法！

可用的`TargetSource`实现类
1. `SingletonTargetSource` - 默认的
2. `PrototypeTargetSource`
3. `HotSwappableTargetSource` - 运行时根据某种条件**动态替换**目标对象类的具体实现，如**数据库双机热备**
4. `CommonsPoolTargetSource` - 若不想每次返回新目标对象，而是想返回**有限数目**的**地位平等**的目标对象实例
5. `ThreadLocalTargetSource` - 为不同线程调用提供不同目标对象
6. 自定义`TargetSource` - `extends TargetClassAware`

## SpringAOP二世

### @AspectJ形式的Pointcut

1. `Pointcut Expression`
2. `Pointcut Signature` - 支持`&&`，`||`，`!`

`Pointcut Expression`标志符
1. `execution` - 匹配拥有**指定方法签名**的`Joinpoint`
   1. `*` - 匹配**相邻多个字符**
   2. `..` - 匹配**多层次类型声明**，匹配**0到多个参数**
2. `within` - 匹配**指定类**下所有的`Joinpoint`
3. `@within` - 指定**某种类型注解**，只要对象标注了该注解，则将匹配该对象内部所有`Joinpoint`，**静态**匹配
4. `this` - 指代调用方`caller`，**代理对象**
5. `target` - 指代被调用方`callee`，**目标对象**
6. `@target` - 指定**某种类型注解**，只要对象标注了该注解，则将匹配该对象内部所有`Joinpoint`，运行期**动态**匹配
7. `args` - 捕捉拥有指定参数类型、指定参数数量的方法级`Joinpoint`，注意它是**运行期动态检查**参数
8. `@args` - 检查当前`Joinpoint`的方法参数类型，如果该次传入参数类型拥有`@args`指定注解，则匹配
9. `@annotation` - 检查系统中**所有对象的所有方法**级别的`Joinpoint`，如果被检测方法标注有`@annotation`指定注解，则匹配

## 基于配置文件的AspectJ

* 使用表达式配置切入点
`execution(<访问修饰符>?<返回类型><方法名>(<参数>)<异常>)`

```Xml
<bean id="book" class="cn.itcast.aop.Book"></bean>
<bean id="adviceBook" class="cn.itcast.aop.AdviceBook"></bean>

<!-- adviceBook#adviceBookBefore -> book -->
<aop:config>
	<!-- config pointcut -->
	<aop:pointcut expression="execution(* cn.itcast.aop.Book.*(..))" id="pointcutId" />
	<!-- config aspect -->
	<aop:aspect ref="adviceBook">
		<aop:before method="adviceBookBefore" pointcut-ref="pointcutId" />
	</aop:aspect>
</aop:config>
```


## 基于注解的AspectJ

1. 创建对象
2. `Spring`核心配置文件中开启`AOP`操作
3. 在增强类上使用注解完成`AOP`

```Xml
<bean id="book" class="cn.itcast.aop.Book"></bean>
<bean id="adviceBook" class="cn.itcast.aop.AdviceBook"></bean>

<aop:aspectj-autoproxy></aop:aspectj-autoproxy>
```

```Java
@Aspect
public class AdviceBook {
	@Before(value="execution(* cn.itcast.aop.Book.*(..))")
	public void before() { ... }
}
```

