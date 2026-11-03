# mcp协作平台 1.1.0

Windows 64 位正式渠道，源码提交 092f619d18654d480fb0965da33159d5073216a1。

- 基于与 1.0.9 相同的源码重新构建，统一 Desktop、MCP 和 Local API 版本为 1.1.0。
- 包含已授权 MySQL 建表和结构迁移误拒修复，支持常见索引、表选项、生成列和多项 ALTER，保留数据库、表及账号权限检查。
- 包含此前的目录索引刷新、逻辑线文件夹、MySQL 字段 COMMENT 及文档与表格重命名修复。
- Local API 沿用 1.0.9 的加密 SQL 分类载荷、正式服信任公钥和已有授权绑定，无需重新激活。
- 前端、Go 全量、受保护构建与安装包解包比对通过，校验值见 SHA256SUMS.txt。
- 正式云端地址为 https://mcpcollapi2.xckjart.com，地址已锁定；沿用 1.0.9 的云端依赖要求。
- 升级保留已有配置、数据和实例身份，Local API 程序与配套 license-payload.bin 一同保留。

本包沿用 unsigned-local 构建，未进行 Windows 代码签名。
