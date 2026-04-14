---
name: db-query
description: 数据库查询助手，支持多环境（local/sit/uat）SELECT 查询，自动识别项目和解析 MyBatis Mapper
triggers:
  - keywords: [local, sit, uat, local环境, sit环境, uat环境]
---

<ANNOUNCEMENT>
**调用此 skill 时必须首先打印：**
> 🔍 正在使用 **db-query** skill 执行数据库查询...
</ANNOUNCEMENT>

# db-query - 数据库查询助手

在不同环境（local/sit/uat）下自动连接数据库执行 SELECT 查询。主要用于 systematic-debugging 调试流程，支持 MyBatis Mapper SQL 提取。

## 触发条件

**自动触发：** 用户消息包含环境关键词时自动触发：
- `local` / `local环境`
- `sit` / `sit环境`
- `uat` / `uat环境`

**手动触发：** `/db-query` 命令调用

## 项目识别

通过当前工作目录路径判断项目名：
- 解析当前目录路径，提取项目名
- 比如当前在 `/Users/viscum/projects/project_a/src/` 下 → 识别为 `project_a`
- 根据项目名查询 `config.yaml` 获取对应环境的数据库名

## 执行流程

### 流程图

```
用户触发 skill
    ↓
识别项目（当前目录）
    ↓
识别环境（关键词）
    ↓
获取数据库配置（config.yaml）
    ↓
接收/提取 SQL
    ↓
安全检查（仅 SELECT 类）
    ↓
构建 mycli 命令
    ↓
执行查询
    ↓
返回结果
```

### 详细步骤

1. **识别环境**：从触发消息中提取环境名（local/sit/uat）
2. **识别项目**：从当前工作目录路径提取项目名
3. **获取配置**：读取 `config.yaml`，获取对应环境的连接信息和数据库名
4. **准备 SQL**：
   - 被 systematic-debugging 调用时：接收传入的 SQL
   - 独立触发时：询问用户查询内容，或从 Mapper 文件提取
5. **安全检查**：检查 SQL 是否为 SELECT 类查询，禁止修改操作
6. **执行查询**：使用 mycli 连接数据库执行 SQL
7. **返回结果**：将查询结果返回给调用者

## 安全限制

**允许的操作：**
- SELECT - 查询数据
- SHOW - 显示信息
- DESCRIBE/DESC - 描述表结构
- EXPLAIN - 执行计划

**禁止的操作：**
- INSERT/UPDATE/DELETE - 数据修改
- DROP/ALTER/CREATE/TRUNCATE - 结构修改
- GRANT/REVOKE - 权限操作

检测到禁止操作时拒绝执行并提示用户。

## Mapper SQL 提取

支持两种 MyBatis Mapper 格式：

**XML Mapper** (`*Mapper.xml`):
- 查找 `<select>` 标签
- 提取标签内 SQL
- 处理参数占位符

**注解 Mapper** (`@Select`):
- 查找 `@Select("...")` 注解
- 提取注解内 SQL
- 处理参数占位符

详细方法见 `lib/sql-utils.md`。

## 配置文件

配置存放在 `config.yaml`，包含：
- `environments`: 各环境的连接信息（host/port/user/password）
- `projects`: 项目在各环境的数据库名映射

修改配置直接编辑 `config.yaml`，无需修改 skill 文件。

## 使用示例

**被 systematic-debugging 调用：**
```
用户: sit环境返回结果不对
AI: 分析相关 Mapper，提取 SQL，调用 db-query 执行查询
db-query: 返回查询结果
AI: 根据结果继续调试分析
```

**独立触发：**
```
用户: 查一下 local 环境的 users 表
db-query: 识别项目 → 获取配置 → 构建查询 → 执行 → 返回结果
```

**从 Mapper 提取 SQL：**
```
用户: sit 环境，执行 UserMapper.xml 的 getUserById
db-query: 读取 Mapper 文件 → 提取 SQL → 替换参数 → 执行 → 返回结果
```

## 工具函数

详细的 SQL 工具函数说明见 `lib/sql-utils.md`：
- mycli 命令构建
- SQL 安全检查规则
- Mapper SQL 提取方法
- 参数替换规则
- 错误处理

## 错误处理

| 错误类型 | 处理方式 |
|---------|---------|
| 连接失败 | 提示检查 host/port/user/password 配置 |
| SQL 错误 | 返回数据库报错，帮助修正 SQL |
| 权限错误 | 提示检查用户 SELECT 权限 |
| 禁止操作 | 拒绝执行，提示仅允许 SELECT 类查询 |