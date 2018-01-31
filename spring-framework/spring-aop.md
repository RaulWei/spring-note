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

## 基于配置文件的AspectJ

* 使用表达式配置切入点
`execution(<访问修饰符>?<返回类型><方法名>(<参数>)<异常>)`

```Xml
<bean id="book" class="cn.itcast.aop.Book"></bean>
<bean id="adviceBook" class="cn.itcast.aop.AdviceBook"></bean>

<aop:config>
	<!-- config pointcut -->
	<aop:pointcut expression="execution(* cn.itcast.aop.Book.*(..))" id="pointcutId" />
	<!-- config aspect -->
	<aop:aspect ref="AdviceBook">
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
