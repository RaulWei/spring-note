# Spring IoC

---

- [IoC底层原理](#ioc底层原理)
- [基于配置文件的IoC](#基于配置文件的ioc)
- [基于注解的IoC](#基于注解的ioc)
- [Spring整合web项目](#spring整合web项目)

---

## IoC底层原理

* `xml`记录很多`class`信息 -> 解析`xml`提取信息 -> 反射创建对象 -> 管理对象
* `Configuration`文件记录很多`Bean`信息 -> 解析提取信息 -> 反射创建对象 -> 管理对象

`Spring`提供两种容器
1. `BeanFactory` - 基础`IoC`容器，默认**延迟初始化**
2. `ApplicationContext` - 在`BeanFactory`基础上构建的，默认容器启动后**全部初始化并绑定完成**

### `BeanFactory`

`BeanFactoryPostProcessor`扩展机制允许在容器实例化对象之前，对注册到容器的`BeanDefinition`所保存的信息做相应修改！
几种内置的`BeanFactoryPostProcessor`实现
1. `PropertyPlaceholderConfigurer` - 替换`BeanDefinition`中的**占位符**，如`${jdbc.username}`，不仅会从`properties`加载配置，还会从`System`类中找
2. `PropertyOverrideConfigurer` - 覆盖`BeanDefinition`中的配置信息
3. `CustomEditorConfigurer` - `Spring`通过`JavaBean`的`PropertyEditor`来帮助`String`类型到其他类型的转换，这里允许自定义并包装`PropertyEditor`

`BeanPostProcessor`会处理容器内所有符合条件的实例化后的对象实例
1. 用标记接口来标注待处理类
2. 自定义`BeanPostProcessor`的实现类，其中根据标记接口找到要处理的类
3. 将自定义`BeanPostProcessor`注册到容器

`Spring IoC`中几个重要接口
1. `BeanFactoryAware` - 容器在实例化实现了该接口的`bean`的过程中会自动将容器本身注入该`bean`

### `ApplicationContext`

#### 统一资源加载

`Resource`
1. `ByteArrayResource`
2. `ClassPathResource`
3. `FileSystemResource`
4. `UrlResource`
5. `InputStreamResource`

`ResourceLoader`
1. `DefaultResourceLoader` - 先尝试`classpath`定位，再尝试`url`定位，最后尝试`getResourceByPath`(默认构造`ClassPathResource`)
2. `FileSystemResourceLoader` - 其他逻辑如上，只是覆盖最后一步`getResourceByPath`逻辑为从文件系统加载资源
3. `ResourcePatternResolver` - 批量加载`Resource`

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
4. `scope` - `singleton`或`prototype`或`session`

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


## 基于注解的IoC

```Java
ApplicationContext context = new AnnotationConfigApplicationContext(xx.class);
XXX xxx = context.getBean("xxx");
```

### 对象创建

1. 创建`@Configuration`注解文件
2. 在注解文件中创建`@Bean`注解方法

### 属性注入

1. `@Autowired`
   1. `@Component`注解`Dao`类
   2. `@Service`注解`Service`类，`Service`中定义`Dao`属性，并用`@Autowired`注解`Dao`
2. `@Resource`
   1. `@Component(value="xxx")`注解`Dao`类
   2. `@Service`注解`Service`类，`Service`中定义`Dao`属性，并用`@Resource(name="xxx")`注解`Dao`

`@Autowired`和`@Resource`的区别
* `@Autowired`根据类型去找装配类
* `@Resource(name="xxx")`根据组件名如`@Component(value="xxx")`中定义的`xxx`去找装配类


## Spring整合web项目

### 原理
1. 服务器启动时为每个项目创建一个`ServletContext`对象
2. 监听器可以监听`ServletContext`对象创建时机，则加载配置完成配置对象们的创建，并将其放入`ServletContext`域对象中
3. 到`ServletContext`域对象中获取所需对象

### 纯配置文件
1. `web.xml`
2. `applicationContext.xml`

### 纯注解
1. `web.xml` -> `WebApplicationInitializer`
2. `applicationContext.xml` -> `@Configuration`

### 配置文件和注解混合使用
1. 创建对象操作使用配置文件方式
   1. 创建`Spring`配置文件`xml`，引入约束，注意，比纯`xml`配置多引入`spring-context.xsd`
   2. 开启注解扫描
      * `context:component-scan`扫描包里的类、方法、属性是否有注解
      * `context:annotation-config`只扫描包里的**属性**是否有注解
2. 注入属性操作使用注解方式
   * `@Component`
   * `@Controller` - `Web`层
   * `@Service` - 业务层
   * `@Repository` - 持久层

## `BeanPostProcessor`

`BeanPostProcessor` has 2 main methods, `postProcessBeforeInitialization` and `postProcessorAfterInitialization`.
We can utilize this built-in instance to realize our custome annotation.
