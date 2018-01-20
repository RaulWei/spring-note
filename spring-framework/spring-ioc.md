# Spring IoC

---

- [IoC底层原理](#IoC底层原理)
- [基于配置文件的IoC](#基于配置文件的IoC)
- [属性注入](#属性注入)

## IoC底层原理

`xml`记录很多`class`信息 -> 解析`xml`提取信息 -> 反射创建对象 -> 管理对象

## 基于配置文件的IoC

```Java
ApplicationContext context = new ClassPathXmlApplicationContext("xx.xml");
XXX xxx = context.getBean("xxx");
```

`Bean`实例化的3种方式

```Xml
<!-- Optional 1. Non para constructor method -->
<bean id="xx" class="complete.class.package.name"></bean>

<!-- Optional 2. Static factory constructor method -->
<bean id="xx" class="complete.factory.package.name" factory-method="methodName"></bean>

<!-- Optional 3. Instance factory constructor method -->
<!-- 3.1 Construct factory -->
<bean id="xxFactory" class="complete.factory.package.name"></bean>
<!-- 3.2 Construct fact object -->
<bean id="xx" factory-bean="xxFactory" factory-method="factoryMethodName"></bean>
```

`Bean`标签属性
1. `id` - `Bean`的标识，不能包含特殊符号
2. `class` - 要创建的对象所在**类的全路径**
3. `name` - 功能和`id`一样，能包含特殊符号
4. `scope` - `singleton`或`prototype`

## 属性注入

1. 构造注入
```Java
<bean id="xx" class="complete.class.package.name">
    <constructor-arg name="argName" value="concreteValue"></constructor-arg>
</bean>
```

2. `Set`注入
```Java
<bean id="xx" class="complete.class.package.name">
    <property name="propertyName" value="concreteValue"></property>
</bean>
```

## `BeanPostProcessor`

`BeanPostProcessor` has 2 main methods, `postProcessBeforeInitialization` and `postProcessorAfterInitialization`.

We can utilize this built-in instance to realize our custome annotation.

## Spring整合web项目原理

1. 服务器启动时为每个项目创建一个`ServletContext`对象
2. 监听器可以监听`ServletContext`对象创建时机，则加载配置完成配置对象们的创建，并将其放入`ServletContext`域对象中
3. 到`ServletContext`域对象中获取所需对象