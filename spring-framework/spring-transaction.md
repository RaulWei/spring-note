# Spring Transaction

---

- [配置实现事务管理](#配置实现事务管理)
- [注解实现事务管理](#注解实现事务管理)

---

1. 编程式事务管理
2. 声明式事务管理
   1. 基于`xml`配置文件实现
   2. 基于注解实现


## 配置实现事务管理

第一步，配置事务管理器

```xml
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
	<!-- 注入dataSource -->
	<property name="dataSource" ref="dataSource"></property>
</bean>
```

第二步，配置事务增强

```xml
<tx:advice id="txAdvice" transaction-manager="transactionManager">
	<tx:attributes>
		<tx:method name="xx" />
	</tx:attributes>
</tx:advice>
```

第三步，配置切面

```xml
<aop:config>
	<!-- 切入点 -->
	<aop:pointcut expression="execution(* xx.xx.xx.xxService.*(..))" id="pointcut" />
	<!-- 切面 -->
	<aop:advisor advice-ref="txAdvice" pointcut-ref="pointcut" />
</aop:config>
```

## 注解实现事务管理

第一步，配置事务管理器

```xml
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
	<property name="dataSource" ref="dataSource"></property>
</bean>
```

第二步，开启注解事务

```xml
<tx:annotation-driven transaction-manager="transactionManager" />
```

第三步，在要使用事务的方法所在类上面添加注解

```Java
@Transactional
public class xxService { ... }
```
