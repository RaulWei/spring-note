# Spring MVC

---

- [Java Web发展历史](#java-web发展历史)
- [Spring MVC处理流程](#spring-mvc处理流程)
- [Spring MVC配置](#spring-mvc配置)
- [Spring MVC基于注解的Controller](#spring-mvc基于注解的controller)

---

## Java Web发展历史

1. `Magic Servlet`将流程控制、视图显示、业务逻辑、数据访问混杂在一起
2. `JSP`将视图渲染逻辑以独立单元抽取出来作为模板化视图标准，解放`Servlet`
3. `Magic JSP`将各种逻辑混杂在`JSP`中，避免在`web.xml`配置`URL`与具体处理`Servlet`映射的繁琐
4. `JSP Model 2`接近`MVC`，倾向使用**单一**`Servlet`作为集中控制器，底下有许多**次级**控制器
5. `Spring MVC`是一种**请求驱动**的`Web`框架

## Spring MVC处理流程

1. `DispatcherServlet`拿到`Request`，通过`HandlerMapping`映射到具体处理的`Controller`
2. `Controller`处理方法执行完毕，返回`ModelAndView`，即**模型数据**和**视图模板**
3. `DispatcherServlet`通过`ViewResolver`分析**视图模板**逻辑名称映射具体`View`实例
4. `DispatcherServlet`将**模型数据**和**视图模板**交给`View`实例生成最终视图结果并返回

## Spring MVC配置

1. `web.xml`是所有基于`Servlet`规范的`Web`应用程序**都有**的
2. 在`web.xml`中定义`ContextLoaderListener`，将为整个`Web`应用程序加载**顶层**`WebApplicationContext`，默认配置文件路径为`/WEB-INF/applicationContext.xml`，可以通过配置`contextConfigLocation`来修改默认配置文件路径
3. 在`web.xml`中定义`DispatcherServlet`，构建相应`WebApplicationContext`，将**2**中加载的`WebApplicationContext`作为父容器，默认配置文件为`/WEB-INF/<servlet-name>-servlet.xml`，`<servlet-name>-servlet.xml`中定义`HandlerMapping`，`ViewResolver`等面向`Web`层的组件，可以通过配置`DispatcherServlet`的`init-param`来修改默认配置文件路径

## Spring MVC基于注解的Controller

### 原型分析

`Spring MVC`框架类`DispatcherServlet`

1. 如何知道当前`Request`应该由哪个基于注解标注的`Controller`处理？
   
   `HandlerMapping`（官方`DefaultAnnotationHandlerMapping`）

2. 如何知道调用基于注解标注的`Controller`的哪个方法来处理？
   
   `HandlerAdaptor`（官方`AnnotationMethodHandlerAdapter`）

#### 自定义用于基于注解的`Controller`的`HandlerMapping`

1. 扫描`Classpath`遍历所有可用的基于注解的`Controller`
2. 反射获取`@RequestMapping`相应信息与请求信息匹配
3. 最终返回匹配结果

#### 自定义用于基于注解的`Controller`的`HandlerAdaptor`

1. 反射查找标注了`@RequestMapping`的方法定义并匹配
2. 反射调用方法
3. 组装`DispatcherServlet`所需要的`ModelAndView`返回

### 请求参数到方法参数的绑定

1. 默认绑定行为 - 根据**名称匹配**进行数据绑定
2. 使用`@RequestParam`明确地指定绑定关系
3. 添加自定义数据绑定规则

```Java
// 3.1 使用@InitBinder标注初始化方法

@Controller
@RequestMapping("/reporting")
public class ReportingController {
	// 只能对一个Controller对应的DataBinder做定制
	@InitBinder
	public void customizeDataBinder(WebDataBinder dataBinder) {
		PropertyEditor propEditor = ..;
		dataBinder.registerCustomEditor(SomeDataType.class, propEditor);
		...
	}
}

// 3.2 指定自定义WebBindingInitializer
public class GenericBindingInitializer implements WebBindingInitializer {
	public void initBinder(WebDataBinder binder, WebRequest request) {
		PropertyEditor propEditorOne = ..;
		binder.registerCustomEditor(SomeDataType.class, propEditorOne);

		// 如果需要可以注册更多propertyEditor
		...
	}
}

<bean class="org.springframework.web.servlet.mvc.Annotation.AnnotationMethodHandlerAdapter">
	<property name="WebBindingInitializer">
		<bean class="..GenericBindingInitializer" />
	</property>
</bean>
```