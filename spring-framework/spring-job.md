# Spring Job

---

- [Spring Quartz](#spring-quartz)

---

## Spring Quartz

`Quartz`好处
* 允许批处理任务状态的**持久化**，提供不同的持久化策略支持
* 批处理任务的远程调度
* 集群支持

`Quartz`角色划分
1. `Job` - 被调度的任务
2. `JobDetail` - 提供`Job`执行的上下文信息
3. `Trigger` - 用于触发被调度任务的执行，1个`Trigger`只能用于一个`Job`，但多个`Trigger`可以用于同一`Job`
4. `Scheduler` - 核心调度程度

### Spring简化Quartz的Job实现策略

在`Quartz`中，每个`Job`所需要的**执行上下文**信息是由对应的`JobDetail`提供的，二者通过`JobDataMap`进行通信！

```Java
JobDetail jobDetail = new JobDetail("jobName", Scheduler.DEFAULT_GROUP, HelloJob.class);
jobDetail.getJobDataMap().put("message", "hello");
jobDetail.getJobDataMap().put("counter", 10);

public class HelloJob implements Job {
	public void execute(JobExecutionContext ctx) throws JobExecutionException {
		JobDataMap jobDataMap = ctx.getJobDetail().getJobDataMap();
		String message = jobDataMap.getString("message");
		int counter = jobDataMap.getInt("counter");
	}
}
```

`Spring`提供了`QuartzJobBean`，则在实现`Job`的时候，不用直接实现`Job`接口，继承`QuartzJobBean`即可让我们在`Job`实现类中直接以属性形式访问当前`Job`的执行上下文信息！

```Java
// QuartzJobBean保证上下文信息在executeInternal(JobExecutionContext)执行之前被注入对应属性中
public class HelloJobQuartzJobBean extends QuartzJobBean {
	private String message;
	private int counter;

	@Override
	protected void executeInternal(JobExecutionContext ctx) throws JobExecutionException {
		// getMessage()并使用
		// getCounter()并使用
	}
}
```

### Spring提供更多选择去实现JobDetail

* 法1 - `JobDetailBean`

在`Quartz`中，一个`Job`实现类最终需要一个对应的`JobDetail`提供执行上下文！`JobDetail`中**必须**指定**所在组**和**组内唯一标志名**！如果我们用`JobDetailBean`来构造`JobDetail`就**可以不用**指定，因为它提供了默认值！

```xml
<bean id="fxNewsJobDetail" class="org.springframework.scheduling.quartz.JobDetailBean">
	<property name="jobClass" value="HelloJob | HelloJobQuartzJobBean" />
	<property name="jobDataAsMap">
		<map>
			<entry key="message">
				<value>Hello</value>
			</entry>
			<entry key="counter">
				<value>10</value>
			</entry>
		</map>
	</property>
</bean>
```

* 法2 - `MethodInvokingJobDetailFactoryBean`

我们只需指定调度执行时应该**调用哪个对象实例**的**哪个方法**即可！事实上，`MethodInvokingJobDetailFactoryBean`内部将构造相应的`Job`实现类，在合适的时机调用**指定的业务对象上的业务方法**！**坏处**是这样返回的`JobDetail`是**不可序列化**的，也就**无法持久化**！

```xml
<bean id="jobDetail" class="org.springframework.scheduling.quartz.MethodInvokingJobDetailFactoryBean">
	<property name="targetObject" ref="fxNewsProvider" />
	<property name="targetMethod" value="getAndPersistNews" />
</bean>
```