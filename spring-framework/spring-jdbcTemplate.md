# Spring Jdbc

---

- [JdbcTemplate](#jdbctemplate)

--

## JdbcTemplate

贡献
1. 封装所有基于`JDBC`数据访问代码，以**统一格式和规范**使用`JDBC API`
2. 对`SQLException`提供的异常信息在框架内进行**统一转义**，统一**数据接口定义**，简化客户端对数据访问异常的处理

`JdbcTemplate`中心思想
* **模板方法模式** + 回调函数

`DataSource`
* `DriverManagerDataSource` - 每次返回新的连接
* `SingleConnectionDataSource` - 只管理并返回一个
* `ComboPooledDataSource` - 配备缓冲池，`close`相当于把连接还给缓冲池

`JdbcTemplate`
* `NamedParameterJdbcTemplate` - 使用名称替代原先`sql`中的`?`，包装了`JdbcTemplate`
* `SimpleJdbcTemplate` - 包装`NamedParameterJdbcTemplate`

`JdbcTemplate`使用流程
1. 创建`DataSource`对象，设置数据库信息
2. 创建`JdbcTemplate`对象，设置数据源
3. 调用`JdbcTemplate`对象里面的方法实现操作

### 增加

```Java
JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
String sql = "insert into user values(?,?)";
int rows = jdbcTemplate.update(sql, "lucy", "250");
```

### 修改

```Java
String sql = "update user set password=? where username=?";
int rows = jdbcTemplate.update(sql, "111", "lucy");
```

### 删除

```Java
String sql = "delete from user where username=?";
int rows = jdbcTemplate.update(sql, "lucy");
```

### 查询

`JdbcTemplate`实现查询，有接口`RowMapper`，`JdbcTemplate`针对该接口没有提供实现类，得到不同类型数据需要自己封装！

* 查询返回某一个值

```Java
String sql = "select count(*) from user";
int count = jdbcTemplate.queryForObject(sql, Integer.class);
```

* 查询返回对象

```Java
String sql = "select * from user where username=?";
User user = jdbcTemplate.queryForObject(sql, new MyRowMapper(), "mary");

class MyRowMapper implements RowMapper<User> {
	@Override public User mapRow(ResultSet rs, int num) throws SQLException {
		// 1. 从结果集取出数据
		// 2. 把数据封装进对象当中
	}
}
```

* 查询返回列表

```Java
String sql = "select * from user";
List<User> users = jdbcTemplate.query(sql, new MyRowMapper());
```
