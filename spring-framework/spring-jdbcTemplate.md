# Spring JdbcTemplate

---

- [增加](#增加)
- [修改](#修改)
- [删除](#删除)
- [查询](#查询)

---

1. 创建对象，设置数据库信息
2. 创建`JdbcTemplate`对象，设置数据源
3. 调用`JdbcTemplate`对象里面的方法实现操作

## 增加

```Java
JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
String sql = "insert into user values(?,?)";
int rows = jdbcTemplate.update(sql, "lucy", "250");
```

## 修改

```Java
String sql = "update user set password=? where username=?";
int rows = jdbcTemplate.update(sql, "111", "lucy");
```

## 删除

```Java
String sql = "delete from user where username=?";
int rows = jdbcTemplate.update(sql, "lucy");
```

## 查询

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