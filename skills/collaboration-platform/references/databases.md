# 关系型数据库操作

适用于 MySQL / MariaDB、PostgreSQL、SQLite 的 Desktop 数据库管理和 `db_*`。先确定连接、当前数据库、目标对象、统计口径与实际有效权限，再执行 SQL。

## 读取有效上下文

已知连接时用 `db_context_get`，否则小页 `db_connections_list`。检查 `driver`、允许 / 禁止的库、Schema 与表、脱敏列、行数 / 字节 / 超时限制、执行就绪状态及当前权限矩阵。

重点读取 `statement_permissions`、`authorization_mode`、`account_observed` / `account_unavailable` 和 `query_execute_select_only`（若返回）。工具名 `db_query_execute` 不表示任何版本都只许 SELECT，也不表示任意 SQL 都已获准。

- 当前可见实现按有效矩阵分类后决定 `allow` / `propose` / `deny`，可覆盖查询、数据写入、结构及事务等实际允许的类别。
- 若连接或旧执行端明确返回 SELECT-only，只执行其允许的读取；不能用较新版本文档覆盖运行时限制。
- 权限、隔离和所用账号路径以当前返回为准，不能把某一种账号模式的矩阵套到另一模式。

打开 `db_session_open`，在支持时指定 `current_database`，保存 `db_session_id`；后续在同一项目复用。会话打开后，不用 `USE` 跨出已选库；需要切库时按服务要求另开正确数据库的会话。

## 分类、提案与执行

`db_query_execute` 每次只提交一条 `statement_text`，不要把多条 SQL 放一个调用。参数中不包含未公布的绑定变量或批处理字段。

| 返回决策 | 解释与下一步 |
| --- | --- |
| `allow` | 检查实际执行状态与语句 ID，再读取结果；不是仅凭决策就认为执行成功 |
| `propose` | SQL 待人工处理，尚未实际执行；按返回入口交给有权成员 |
| `deny` | 说明被拒类别或对象范围，按任务与策略调整；不能换语法或账号绕过 |

明确只需待审 SQL 时使用 `db_statement_propose`。提案成功不等于执行成功，不能报告影响行数或表已创建。

写入或结构变更先核对准确对象、条件与影响范围；用户已明确授权的范围内执行，不重复索要相同确认。范围不明确或出现未涵盖的不可逆影响时，先准备具体 SQL 与对象清单再补充确认。

事务仅在当前驱动、权限与执行端实际支持时使用，同一事务保持原会话；逐条执行控制语句与操作，核对提交 / 回滚结果。断开连接、取消或关闭会话不能作为“已成功提交 / 回滚”的证据。

## MySQL 结构查询的范围规则

查询 `information_schema` 时使用支持的单一视图，以等值、IN 或 `DATABASE()` 明确限定 Schema，并保持表与外键引用在允许范围内。例如查当前库某表列信息：

```sql
SELECT column_name
FROM information_schema.columns
WHERE table_schema = DATABASE()
  AND table_name = '实际表名'
```

示例中的表名需要替换为已核对的目标。`DATABASE()` 代表打开会话时选择的库；外键视图还可能需要约束引用库与引用表，主键可涉及 `referenced_table_schema IS NULL`。

`DATABASE_METADATA_SCOPE_DENIED` / `DATABASE_SCOPE_DENIED` 表示对象或查询形状不符合范围，不是登录过期。先缩小查询和限定对象，不为此反复登录、申请全库权限或试图绕过分类器。复杂结构读取可拆成允许的单视图查询再由程序合并。

## 查询结果与未知状态

保存 `statement_id`。`db_statement_result_get` 使用原会话与语句 ID，按 `next_cursor` 读取必要页；`page_size` 按 schema，当前常见默认 50、上限 100。

- `execution_is_complete` 表示执行终态，`result_is_complete` 表示结果分页读取完成；不要混为一谈。
- `local_result.rows` 是本页结果，`row_count` 与 `truncated` 仍保留原查询语义。读完现存页面也不代表拿到了被上游截断的数据。
- `DB_RESULT_PERSIST_FAILED` 或超时可能发生在 SQL 已执行之后；查询原语句、实际对象和审计，禁止直接重发写入。

完成后关闭本任务创建且无未决操作的会话。报告连接 / 数据库、目标对象、语句 ID、实际行数或影响行数、截断与验证结论；不回贴整批明细。

## 数据处理与节省 token

在权限允许时，让 SQL 先筛选列与行、计数和聚合，避免把整表明细交给模型。统计任务优先读取表结构、明确口径、运行聚合，再取必要样本验证。

需要完整明细的迁移或审计必须处理全量数据，但可以由程序流式处理，只向 AI 返回计数、异常摘要与必要证据。分页限制单次体积，只有减少无关读取才减少总 token。
