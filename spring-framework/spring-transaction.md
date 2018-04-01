# Spring Transaction

---

- [事务基础](#事务基础)
- [配置实现事务管理](#配置实现事务管理)
- [注解实现事务管理](#注解实现事务管理)

---

## 事务基础

1. 原子性 - 事务包含的所有操作**不可分割**，要么全部**提交**成功，要么全部失败
2. 一致性 - 事务执行**前后**，数据资源应该处于一致性状态
3. 隔离性 - 规定事务之间**相互影响的程度**，多个事务同时访问**一个**数据资源，不同隔离级别决定不同行为
4. 持久性 - 一旦事务**提交**，对数据的变更**不可逆转**，即使系统灾难也可恢复变更

假设有事务`A`和事务`B`，定义问题
1. 脏读 - 事务`A`更新数据还没提交，事务`B`可以看到更新结果，则如果事务`A`回滚，事务`B`读到的就是脏数据
2. 不可重复读 - 对**同一笔数据**，事务`A`在事务`B`**更新前后**分别读取一次，两次结果不同
3. 幻读 - 对**多笔数据**，事务`A`在事务`B`**插入前后**分别读取一次，两次结果不同

隔离级别
1. `Read Uncommited` - 一个事务能读取另一个事务**没提交**的更新结果，可能造成脏读、不可重复读、幻读
2. `Read Commited` - 一个事务的更新结果只有**提交后**，另一个事务才能读到，不会脏读，无法避免不可重复读、幻读
3. `Repeatable Read` - 不会脏读、不可重复读，无法避免幻读
4. `Serializable` - 避免所有问题，但是效率低

事务家族
1. `Resource Manager` - 存储并管理系统数据资源，如数据库服务器
2. `Transaction Processing Monitor` - 在分布式场景中协调包含多个`RM`事务处理
3. `Transaction Manager` - `TP Monitor`的核心模块，直接负责多`RM`之间事务处理的协调工作
4. `Application` - 独立形式存在的或运行于容器中的应用程序


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
