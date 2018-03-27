# Spring Data Access

---

- [统一的数据访问异常层次体系](#统一的数据访问异常层次体系)
---

## 统一的数据访问异常层次体系

我们通常定义`DAO`来**分离数据访问和存储**，以及**屏蔽各种数据访问的差异性**！
```Java
public interface ICustomerDao {
	Customer findCustomerByPK(String customerId);
}
```

然而不同`DAO`实现可能会抛出各自**特定的受检异常**，导致`ICustomerDao`接口方法声明不断变化！
```Java
public class JdbcCustomerDao implements ICustomerDao {
	Customer findCustomerByPK(String customerId) throws SQLException;
}
public class LdapCustomerDao implements ICustomerDao {
	Customer findCustomerByPK(String customerId) throws NamingException;
}
```

```Java
public interface ICustomerDao {
	Customer findCustomerByPK(String customerId) throws SQLException, NamingException;
}
```

这样的话至少引起两个问题！
1. `ICustomerDao`接口根据实现类而变化
2. `ICustomerDao`引入了实现类相关信息，无法做到屏蔽具体实现

怎么办？
1. 把这些受检异常`catch`住，然后转换成**未受检异常**抛出，这样接口不能声明异常信息
2. `catch`住异常后，根据不同实现分析提取特定异常信息，包装成统一异常再往上抛
3. `Spring`提供了轮子！`DataAccessException`以及它的子类型
