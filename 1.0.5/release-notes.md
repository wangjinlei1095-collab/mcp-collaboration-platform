# mcp协作平台 1.0.5

Windows 64 位正式渠道，本次重新发布于 2026-11-11。
源码基础提交 be0b0a29e1f10cea2159858a1fb9969b73a7d212，包含当前未提交工作区修改。
构建摘要：sha256:ccd9dfa8c3d7dd76d3888e6264b60a9ce514b5a75a64265e381c52b1d0aa50d1。

- 更新 Desktop、MCP 程序和 Local API，包含当前侧栏搜索与宽度调整、Markdown 图表及文档工作流改进。
- 包含多客户端授权保持和短暂网络失败恢复修复。
- 修复 MySQL 十六进制 SQL 的对象范围识别；加密 SQL 分类载荷同步重建，并验证已有授权兼容。
- 保留正式服信任公钥、内容密钥、实例绑定及授权方式。
- 正式云端地址为 https://mcpcollapi2.xckjart.com，地址已锁定。协作与授权功能需要配套云端服务支持。
- 升级保留已有配置、数据和实例身份；Local API 程序与 license-payload.bin 配套使用。
- 安装包解包与文件哈希比对通过，校验值见 SHA256SUMS.txt。

按用户指定使用版本号 1.0.5，既有 1.1.1 包仍保留在仓库。此次包与历史 1.0.5 不同，请核对 SHA256SUMS.txt。
本包沿用 unsigned-local 构建，未进行 Windows 代码签名。
