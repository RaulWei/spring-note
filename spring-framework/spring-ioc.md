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