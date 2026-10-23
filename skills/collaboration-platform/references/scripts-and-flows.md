# 参数化脚本与流程测试

用于 Desktop“脚本管理”“流程测试”与 `script_*` / `flowtest_*`。脚本和流程的实际编排、运行依赖发起方真实 Desktop 及运行权限；MCP 网关在线不代表执行端已就绪。

## 先复用，再新增

只运行已有对象时，使用已知 ID 或相关列表确认用途与参数；不要固定把所有脚本源码读一遍。需要写脚本、选连接或编排流程时再读取 `script_context_get` / `flowtest_context_get`。

| 对象 | 最短读取路径 |
| --- | --- |
| 脚本用途与参数摘要 | `script_list`，不含源码 |
| 一份脚本的源码、参数、运行配置和修订 | `script_get` |
| 流程用途与公共参数 | `flowtest_list` / `flowtest_get`，不含完整图 |
| 流程节点、连线、断点与可执行原因 | `flowtest_graph_get` |
| 流程说明文档 | `flowtest_docs_list` |

选择与任务匹配的现有脚本或流程；没有合适对象或确实需要修改时，再读取源码或完整图。

## 编写与更新脚本

`script_create` 提供有意义的名称和必填 Markdown `description`；`params_schema` 中每个参数带 `description`，说明单位、取值、边界和用途。`params_doc` 描述参数关系，`notes_doc` 记录运行前提、可见副作用与结果解释。

`script_update` 修改元数据、参数或运行配置；`script_source_set` 是源码整体替换，修改前读取当前源码并保留无关逻辑，带最新支持的 `base_revision`。不要虚构局部源码 patch 工具。

使用平台随包 Python 与公开 `platform_sdk`，不重写认证、凭据提取或内部 HTTP 协议。已公开的常用入口包括 `params`、`log`、`result`、`mysql`、`redis`、`pool`、`load` 和取消检查；不要据名称推测不存在的数据库 SDK。

最小的纯计算源码示例，仅演示参数与结构化结果：

```python
from platform_sdk import params, result

values = params['values']
result({'count': len(values), 'total': sum(values)})
```

为该脚本定义名为 `values` 的必填数值数组参数，并提供描述。`params` 是映射，确需普通字典时用 `dict(params)`。日志只输出关键进度与错误，结果用 `result` 返回计数、摘要、指标和必要样本，不返回全部原始数据。

数据库与 Redis 连接 ID 从对应上下文取得。当前 SDK 的 `mysql` 与 `pool` 用于 MySQL；受控代理与直连由管理员对连接配置，脚本不能自行开通。代理逐条分类与审计，直连的性能与审计粒度不同，不能将代理延迟直接当成数据库极限吞吐。

使用动态 SQL 时按已支持驱动与连接模式处理参数。当前受控代理不接受驱动式参数化占位符，不把用户输入直接拼入 SQL；能用固定语句则固定，动态输入需按类型、范围或白名单验证并正确引用，不支持时说明限制。只有已授权且已启用的直连才使用其驱动能力。

需要第三方包时先查 `script_packages_list`，缺少实际必需包再按指定名称 / 版本 `script_package_install`。安装与直连配置是否属于任务范围，以用户现有授权为准，不为简单任务额外改环境。

## 启动与观察

1. 以 `script_run_start` 或 `flowtest_run_start` 传 schema 已声明的参数；未声明参数会被拒绝。保存真实 `run_id`，不要为确认状态重复启动。
2. 优先 `script_run_status` / `flowtest_run_status` 查看状态、结构化结果、错误与指标；确需日志时再读输出。
3. 脚本用 `after` 续读，流程用 `cursor` 续读；流程可指定 `node_id`。空页但 `is_complete=false` 时仍可能有后续输出，合理等待而非密集空轮询。
4. 以终态、退出码、错误、必要结果和验收条件判断成功。任务需要完整输出时继续到输出完成，否则只读取必要结果并说明范围。
5. 按需取消运行并确认状态；取消可能终止进程树，但不会撤销已写入外部系统的数据。

授权以当前目录授权、项目绑定、AI 委托与执行时校验为准；是否另有对象编辑 / 执行开关，以当前界面和返回为准，不能假定每个版本都必须人工审批一次。

## 编排流程

使用 `flowtest_create` 创建元信息，`flowtest_graph_set` 保存完整图。编辑已有图必须先取得现状并保留未涉及节点、边、断点和 ID，不能用局部图覆盖剩余流程。

- 只能有一个 `start`，图不可成环，所有节点从开始节点可达。
- 节点类型包括 `start`、`end`、`script`、`branch`、`delay`、`assert`、`manual`；只有 `script` 执行 Python 并引用 `script_id`，其他类型由编排器处理。
- 参数映射、分支和断言表达式沿用当前图返回或软件说明的结构，不猜测开放对象字段。
- 公共参数的随机生成作用域为 `run`、`vu` 或 `call`，分别对应整次运行、每个虚拟用户、每次调用。按真实业务选择，避免把每用户一个账号变成每请求换账号。
- `flowtest_doc_set` 新增说明用父流程最新修订，修改已有说明用该说明自身修订；保留真实 `doc_id`。

先单次调试再进行已授权的压力测试。`mode=debug` 可在启用断点暂停；暂停后用 `flowtest_run_resume` 进行 `continue`、`step` 或 `abort`，必要时按 schema 修改变量。`mode=stress` 忽略断点，检查 `breakpoints_ignored`；断点不能作为压测的停止保护。

并发、持续时间、请求量、目标数据与停止条件从用户目标和有效授权确定。没有明确压测授权时，不把调试任务自行升级为高负载运行。

## 复用与完成报告

重复任务尽量复用脚本和既有流程，只传参数、读取终态与结构化结果，避免每次重新生成整段代码和逐节点手动调度。首次编写、依赖安装、调试和必要日志仍计入 token。

报告脚本 / 流程 ID、实际修订、运行 ID、关键参数、终态、结果及关键指标。压测区分目标错误、超时、限流与执行链路开销；只输出结论与必要证据，不堆砌逐请求日志。
