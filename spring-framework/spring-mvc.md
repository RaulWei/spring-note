# Spring MVC

---

- [Java Web发展历史](#javaweb发展历史)

---

## Java Web发展历史

1. `Magic Servlet`将流程控制、视图显示、业务逻辑、数据访问混杂在一起
2. `JSP`将视图渲染逻辑以独立单元抽取出来作为模板化视图标准，解放`Servlet`
3. `Magic JSP`将各种逻辑混杂在`JSP`中，避免在`web.xml`配置`URL`与具体处理`Servlet`映射的繁琐
4. `JSP Model 2`接近`MVC`，倾向使用**单一**`Servlet`作为集中控制器，底下有许多**次级**控制器
5. `Spring MVC`是一种**请求驱动**的`Web`框架