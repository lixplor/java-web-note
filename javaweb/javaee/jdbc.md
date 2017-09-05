# JDBC

* JDBC, Java Data Base Connectivity, Java数据库连接
* 是一种用于执行SQL语句的Java API
* 可以为多种关系型数据库提供统一的接口
* 需要使用驱动连接数据库


## 原理

* 提供统一的操作接口, 由不同的数据库厂商依照各自特性编写各自的驱动来实现接口


## 使用步骤

步骤:
1. 注册驱动
    - 避免使用DriverManager, 缺点:
        - 硬编码, 不利于配置
        - 重复注册
    - 建议使用反射实现: `Class.forName("com.mysql.jdbc.Driver");`
2. 获取连接
    - `Connection DriverManager.getConnection(String url, String username, String password)`: 获取数据库连接
        - 需要: 用户名, 密码, 连接url
        - 数据库连接url格式: `jdbc:mysql://ip:port/db_name`
3. 获取语句执行平台
    - `PreparedStatement conn.prepareStatement(String sql)`: 避免SQL注入
    - 参数使用`?`占位符代替
    - 用`preparedStatement.setObject(int paramIndex, Object value)`设置占位符的值
4. 执行sql语句
    - `int preparedStatement.executeUpdate()`: insert, delete, update语句
    - `ResultSet preparedStatement.executeQuery()`: select语句
5. 处理结果集
    - `boolean resultSet.next()`: 移动记录指针到下一行
    - `int resultSet.getInt(String column_name)`: 获取指定列的整型记录值
6. 释放资源
    - `resultSet.close()`
    - `preparedStatement.close()`
    - `connection.close()`

```java
// 1. 注册驱动, 不要用DriverManager造成重复注册
// DriverManager.registerDriver(new com.mysql.jdbc.Driver());  // Driver中已经重复注册
// 建议用反射, 使得内部静态代码块自动注册驱动
Class.forName("com.mysql.jdbc.Driver");
// 2. 获取数据库连接
String url = "jdbc:mysql://localhost:3306/db_name";
String username = "root";
String password = "123456";
Connection conn = DriverManager.getConnection(url, username, password);
// 3. 获取语句执行平台
PreparedStatement stat = conn.prepareStatement("insert into table_name values();");
// 4. 执行sql语句
int rows = stat.executeUpdate(); // insert, delete, update
// 5. 查询结果集
PreparedStatement pst = conn.prepareStatement("select * from tb_name where username=? and password=?");
pst.setObject(0, "Tom");
pst.setObject(1, "1234");
ResultSet rs = pst.executeQuery();// select
while(rs.next()) {
    int id = rs.getInt("id");
    String name = rs.getString("name");
}
// 6. 释放资源
rs.close();
stat.close();
conn.close();
```


## SQL注入

### 注入演示

在登录时, 用户输入用户名和密码, 提交服务器查询数据库是否有匹配记录

```Java
String username = "Tom";
String password = "1234";
stat.executeQuery("SELECT * FROM users WHERE username=" + username + " AND password=" + password + ";");
```

```sql
-- 期望执行的sql
SELECT * FROM users WHERE username='Tom' AND password='1234';
```

但是用户利用SQL注入漏洞, 拼凑SQL语句的判断条件:

```Java
String username = "sdafsdf";
String password = "dfsdf' OR '1=1";
```

```sql
-- 利用SQL注入漏洞, 添加了OR条件, 导致username和password无论匹配还是不匹配, 最终都能查到结果
SELECT * FROM users WHERE username='sdafsdf' AND password='dfsdf' OR '1=1';
```

### 避免注入

* 使用`PreparedStatement`代替`Statement`



## DbUtils

* Apache Commons组件, 简化了JDBC操作
* 3个核心类
    - `QueryRunner`类: 对SQL语句操作的API
        - `update(Connection conn, String sql, Object... params)`: insert, delete, update操作
        - `query(Connection conn, String sql, ResultSetHandler<T> rsh, Object... params)`: select操作
    - `ResultSetHandler`接口: 定义select操作后如何封装结果集
        - 实现类:
            - `ArrayHandler`: 将结果集中的第一条记录封装到一个`Object[]`数组中, 数组中每个元素就是该记录的字段值
            - `ArrayListHandler`: 将结果集中每一条记录都封装到一个`Object[]`数组中, 将这些数组再封装到`List`集合中
            - `BeanHandler`: 将结果集中第一条记录封装到一个指定的JavaBean中
            - `BeanListHandler`: 将结果集中每一条记录都封装到指定的JavaBean中, 将这些JavaBean再封装到`List`集合中
            - `ColumnListHandler`: 将结果集中指定的列的字段值封装到一个`List`集合中
            - `ScalarHandler`: 用于单数据, 如`select counr(*) from table`操作
            - `MapHandler`: 将结果集第一行封装到`Map`集合中, Key列名, Value该列数据
            - `MapListHandler`: 将结果集第一行封装到`Map`集合中, Key列名, Value该列数据, 再将Map集合封装到`List`集合
    - `DbUtils`类: 工具类, 定义关闭资源和事务处理的方法

* 插入

```java
// 创建QueryRunner
QueryRunner qr = new QueryRunner();
// update
String sql = "INSERT INTO sort (sname, sprice, sdesc) VALUES(?, ?, ?)";
Object[] params = {"sdf", 324.12, "dsf"};
int row = qr.update(conn, sql, params);
DbUtils.closeQuietly(conn);
```

* 查询

```java
// ArrayHandler
QueryRunner qr = new QueryRunner();
String sql = "SELECT * FROM table where id=?;"
Object[] params = {1};
Object[] objArr = qr.query(conn, sql, new ArrayHandler(), params);
conn.close();

// ArrayListHandler
QueryRunner qr = new QueryRunner();
String sql = "SELECT * FROM table where id>?;";
Object[] params = {1};
List<Object[]> list = qr.query(conn, sql, new ArrayListHandler(), params);
for(Object[] record : list) {

}
conn.close();

// BeanHandler
QueryRunner qr = new QueryRunner();
String sql = "SELECT * FROM table where id=?;"
Object[] params = {1};
User user = qr.query(conn, sql, new BeanHandler<User>(User.class), params);
conn.close();

// BeanListHandler
QueryRunner qr = new QueryRunner();
String sql = "SELECT * FROM table where id>?;"
Object[] params = {1};
List<User> list = qr.query(conn, sql, new BeanListHandler<User>(User.class), params);
for(User user : list) {

}
conn.close();

// ColumnListHandler
QueryRunner qr = new QueryRunner();
String sql = "SELECT * FROM table where id>?;"
Object[] params = {1};
List<String> list = qr.query(conn, sql, new ColumnListHandler<String>(), params);
for(String str : list) {

}
conn.close();

// ScalarHandler
QueryRunner qr = new QueryRunner();
String sql = "SELECT count(*) FROM table;"
Object[] params = {};
Integer count = qr.query(conn, sql, new ScalarHandler<Integer>(), params);
conn.close();
```


## 数据库连接池

* `创建连接`和`释放资源`是比较消耗系统资源的两个操作, 为了解决这个问题, 采用连接池技术来共享数据库连接
* 通过连接池来管理Connection, 初始化时创建多个Connection放入池中, 通过连接池获取Connection, 关闭连接实际是将Connection归还到连接池中, 达到复用目的, 减少创建和释放的资源消耗
* `javax.sql.DataSource`接口, 各厂商实现该接口实现连接池
* 常见连接池:
    - `DBCP`: Apache Commons组件, 是Tomcat内置的连接池
        - `BasicDataSource`
        - `BasicDataSourceFactory`
    - `C3P0`: 开源的JDBC连接池, Hibernate, Spring都在使用

### DBCP

```java
Connection conn = null;
PreparedStatement pst = null;

// 创建连接池:
BasicDataSource dataSource = new BasicDataSource();
dataSource.setDriverClassName(DB_CLASS_NAME);
dataSource.setUrl(DB_URL);
dataSource.setUsername(DB_USERNAME);
dataSource.setPassword(DB_PASSWORD);

try {
    // 获取连接
    conn = dataSource.getConnection();
    // 执行sql
    String sql = "insert into table values (null, ?);";
    pst = conn.prepareStatement(sql);
    pst.setString(1, "sdf");
    pst.executeUpdate();
} catch (Exception e) {

} finally {
    // 释放资源
}
```

* 配置文件: dbcp.properties

```text
# 连接设置
driverClassName=com.mysql.jdbc.Driver
url=jdbc:mysql://ip:port/db_name
username=root
password=1234

# 初始连接池大小
initialSize=10

# 最大连接数量
maxActive=50

# 最大空闲连接
maxIdle=20

# 最小空闲连接
minIdle=5

# 超时等待时间毫秒
maxWait=6000
```

### C3P0

```java
Connection conn = null;
PreparedStatement pst = null;
try {
    // 创建连接池
    ComboPooledDataSource source = new ComboPooledDataSource();
    // 设置参数
    /*source.setDriverClass("com.mysql.jdbc.Driver");
    source.setJdbcUrl("jdbc:mysql://ip:port/db_name");
    source.setUser("root");
    source.setPassword("1234");*/
    // 也可以使用配置文件配置以上信息, C3P0会自动去寻找c3p0-config.xml文件

    conn = source.getCOnnection();
    String sql = "INSERT INTO table VALUES (null, ?);";
    pst = conn.prepareStatement(sql);
    pst.setString(1, "sdaf");
    pst.executeUpdate();
} catch (Exception e) {

} finally {
    // 释放资源
}
```

* 配置文件: c3p0-config.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<c3p0-config>
    <default-config>
        <property name="driverClass">com.mysql.jdbc.Driver</property>
        <property name="jdbcUrl">jdbc:mysql://ip:port/dbname</property>
        <property name="user">user </property>
        <property name="password">password</property>
    </default-config>
</c3p0-config>
```

### Druid

@todo


## JDBC工具类
    - 针对DBCP, C3P0等连接池不同的配置需要进行修改

```Java
public class JDBCUtil {

    private static final String KEY_DRIVER_CLASS = "driverClass";
    private static final String KEY_URL = "url";
    private static final String KEY_USERNAME = "username";
    private static final String KEY_PASSWORD = "password";

    private static Connection conn;

    private JDBCUtil(){}

    // 加载驱动
    public static void loadDriver(String propertiesPath) {
        try {
            Properties properties = new Properties();
            properties.load(new FileInputStream(propertiesPath));
            String driverClass = properties.getProperty(KEY_DRIVER_CLASS);
            String url = properties.getProperty(KEY_URL);
            String username = properties.getProperty(KEY_USERNAME);
            String password = properties.getProperty(KEY_PASSWORD);
            Class.forName(driverClass);
            conn = DriverManager.getConnection(url, username, password);
        } catch (Exception e) {
            throw new RuntimeException("Database connect error!");
        }
    }

    // 获取数据库连接
    public static Connection getConnection() {
        return conn;
    }

    // 关闭连接
    public static void close(ResultSet resultSet, Statement statement, Connection connection) {
        if(resultSet != null) {
            try {
                resultSet.close();
            } catch (SQLException e) {

            }
        }
        if(statement != null) {
            try {
                statement.close();
            } catch (SQLException e) {

            }
        }
        if(connection != null) {
            try {
                connection.close();
            } catch (SQLException e) {

            }
        }
    }
}
```
