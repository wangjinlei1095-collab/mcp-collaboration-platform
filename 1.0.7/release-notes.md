# mcp协作平台 1.0.7

Windows 64 位正式渠道，源码提交 9787674947b9fb39e2a6be8c9c34f91f8a0bce28。

- 修复 MySQL CREATE TABLE、ALTER TABLE ADD COLUMN 带 COMMENT 字符串时的对象范围识别。
- 字段注释中的文本按字面量处理；数据库、表范围和账号权限继续生效，未支持的尾随语句仍拒绝。
- 加密 SQL 分类载荷同步重建，沿用原内容密钥、变体和载荷版本，兼容已有授权。
- 包含此前的文档与表格重命名、逻辑线文件夹及脚本受控代理修复。
- 前端、Go、原生与加密分类路径检查及安装包解包比对通过，校验值见 SHA256SUMS.txt。
- 正式云端地址为 https://mcpcollapi2.xckjart.com，地址已锁定；沿用既有云端版本要求。
- 升级保留已有配置、数据和实例身份。Local API 程序与 license-payload.bin 必须配套更新。

本包沿用 unsigned-local 构建，未进行 Windows 代码签名。
