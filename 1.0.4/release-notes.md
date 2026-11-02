# mcp协作平台 1.0.4

Windows 64 位正式渠道，源码提交 baa041414be34e2507d6c70c9acc7d91c819df5a。

- 重新构建 Desktop、MCP 程序和 Local API，替换同版本旧安装包。
- 修复数据库 SQL 入口按有效权限执行，统一桌面、MCP 与脚本执行入口的权限判断。
- 包含按邮箱添加项目成员、连接状态隔离及跨工作区已授权共享项目发现。
- 前端检查、Go 测试、受保护构建及安装包解包比对通过，校验值见 SHA256SUMS.txt。
- 正式云端地址为 https://mcpcollapi2.xckjart.com，地址已锁定；云端需配套包含本次权限修复。
- 升级保留已有配置、数据和实例身份。Local API 沿用已有授权载荷。

本包沿用 unsigned-local 构建，未进行 Windows 代码签名。
