# SQL 工具函数

本文件提供 SQL 执行相关的工具函数说明。

## mycli 命令构建

根据配置构建 mycli 连接命令：

```bash
mycli -h <host> -P <port> -u <user> -p<password> -D <database> -e "<sql>"
```

参数说明：
- `-h`: 数据库主机地址
- `-P`: 端口号（大写 P）
- `-u`: 用户名
- `-p`: 密码（小写 p，密码紧跟参数，无空格）
- `-D`: 数据库名
- `-e`: 执行的 SQL 语句

**示例：**
```bash
mycli -h localhost -P 3306 -u root -proot123 -D db_a -e "SELECT * FROM users LIMIT 10"
```

## SQL 安全检查

**允许的语句类型（开头关键词）：**
- `SELECT` - 查询数据
- `SHOW` - 显示信息（如 SHOW TABLES）
- `DESCRIBE` / `DESC` - 描述表结构
- `EXPLAIN` - 解释执行计划

**禁止关键词检测：**
在执行前检查 SQL 是否包含以下关键词（不区分大小写）：
- `INSERT`
- `UPDATE`
- `DELETE`
- `DROP`
- `ALTER`
- `CREATE`
- `TRUNCATE`
- `GRANT`
- `REVOKE`

如果检测到禁止关键词，拒绝执行并提示：
> "安全限制：仅允许 SELECT/SHOW/DESCRIBE/EXPLAIN 类查询。禁止 INSERT/UPDATE/DELETE 等修改操作。"

## Mapper SQL 提取

### MyBatis XML Mapper 提取

从 `*Mapper.xml` 文件中提取 SQL：

1. 查找 `<select>` 标签
2. 提取标签内的 SQL 内容
3. 处理参数占位符

**示例 XML：**
```xml
<select id="getUserById" resultType="User">
  SELECT * FROM users WHERE id = #{id}
</select>
```

**提取结果：**
```sql
SELECT * FROM users WHERE id = #{id}
```

### MyBatis 注解方式提取

从 Java/Kotlin 代码中提取 `@Select` 注解内的 SQL：

1. 查找 `@Select("...")` 注解
2. 提取注解内的 SQL 字符串
3. 处理参数占位符

**示例代码：**
```java
@Select("SELECT * FROM users WHERE id = #{id}")
User getUserById(Long id);
```

**提取结果：**
```sql
SELECT * FROM users WHERE id = #{id}
```

## 参数替换策略

SQL 中的参数占位符需要替换为实际值才能执行：

**占位符类型：**
- `#{param}` - MyBatis 预编译参数，替换为具体值
- `${param}` - MyBatis 字符串替换，替换为具体值

**替换规则：**
- 被 systematic-debugging 调用时：AI 根据上下文推断参数值
- 独立触发时：询问用户输入具体值
- 测试场景：使用通用测试值（如 `1`、`'test'`）

**替换示例：**
原 SQL：
```sql
SELECT * FROM users WHERE id = #{id} AND status = #{status}
```

替换后（AI 推断或用户输入）：
```sql
SELECT * FROM users WHERE id = 1 AND status = 'active'
```

## 结果处理

**默认限制：**
查询结果默认限制 100 行，防止刷屏。用户可通过 SQL 中指定 LIMIT 覆盖。

**输出格式：**
mycli 输出为表格格式，直接返回给调用者。

**错误处理：**
- 连接失败：检查 host/port/user/password 配置
- SQL 错误：返回数据库报错信息，帮助修正 SQL
- 权限错误：检查用户是否有对应数据库的 SELECT 权限