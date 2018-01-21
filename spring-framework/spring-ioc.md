# Spring IoC

---

- [IoC底层原理](#IoC底层原理)
- [基于配置文件的IoC](#基于配置文件的IoC)
  - [对象创建](#对象创建)
  - [属性注入](#属性注入)

## IoC底层原理

`xml`记录很多`class`信息 -> 解析`xml`提取信息 -> 反射创建对象 -> 管理对象

## 基于配置文件的IoC

```Java
ApplicationContext context = new ClassPathXmlApplicationContext("xx.xml");
XXX xxx = context.getBean("xxx");
```

### 对象创建

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

### 属性注入

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

## 基于注解的IoC


### 对象创建

1. 创建`Spring`配置文件`xml`，引入约束，注意，比纯`xml`配置多引入`spring-context.xsd`
2. 开启注解扫描
2.1 `context:component-scan`扫描包里的类、方法、属性是否有注解
2.2 `context:annotation-config`只扫描包里的**属性**是否有注解

1. `@Component`
2. `@Controller` - `Web`层
3. `@Service` - 业务层
4. `@Repository` - 持久层

## 属性注入

1. `@Autowired`
1.1 `@Component`注解`Dao`类
1.2 `@Service`注解`Service`类，`Service`中定义`Dao`属性，并用`@Autowired`注解`Dao`

2. `@Resource`
2.1 `@Component(value="xxx")`注解`Dao`类
2.2 `@Service`注解`Service`类，`Service`中定义`Dao`属性，并用`@Resource(name="xxx")`注解`Dao`

`@Autowired`和`@Resource`的区别
* `@Autowired`根据类型去找装配类
* `@Resource(name="xxx")`根据组件名如`@Component(value="xxx")`中定义的`xxx`去找装配类

## 配置文件和注解混合使用
1. 创建对象操作使用配置文件方式
2. 注入属性操作使用注解方式