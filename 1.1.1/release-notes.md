# mcp协作平台 1.1.1

Windows 64 位正式渠道，源码提交 39f28c9ddf066078fa5749b2683f1ebd7ed18163。

- 修复 MySQL 会话状态丢失，保留会话参数、事务和临时表；连接失效时显式失败。
- 支持严格限定的会话锁等待参数，补齐常见结构迁移、索引和复制表的对象范围及权限检查。
- 修复 Redis GET/MGET 等已授权命令被辅助命令权限误拒，兼容无 PING 权限及 Redis 6.2 INFO 分节；保留命令策略、预算和超时检查。
- Local API 加密 SQL 分类载荷同步重建，沿用正式服信任公钥、内容密钥及已有授权绑定，无需重新激活。
- 包含此前的目录索引刷新、逻辑线文件夹、MySQL 字段 COMMENT 及文档与表格重命名修复。
- 前端、Go 全量、受保护构建与安装包解包比对通过，校验值见 SHA256SUMS.txt。
- 正式云端地址为 https://mcpcollapi2.xckjart.com，地址已锁定；沿用 1.1.0 的云端依赖要求。
- 升级保留已有配置、数据和实例身份，Local API 程序与配套 license-payload.bin 一同保留。

本包沿用 unsigned-local 构建，未进行 Windows 代码签名。
