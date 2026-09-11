# 工作平台

平台契约规定跨项目统一的技术栈与部署约定，从现有应用（如 qtcloud-asset）的工程实践中提炼。契约未覆盖的选型由项目自定。

## 技术栈

平台按组件角色约定语言，不以项目为转移：

- Provider（服务端）：Go
- Studio（Web 客户端）：Flutter，路由统一使用 go_router
- CLI（管理工具）：Rust

状态管理库（如 Riverpod）属项目内选型，不纳入平台契约。

## 存储桶命名

OSS 桶按 `{产品线}-{用途}` 命名，用途四类：

- `-studio`：前端静态站点
- `-site`：公开官网
- `-private`：私密数据
- `-provider`：后端服务

## 域名

正式入口为 `{产品}.cloud.quanttide.com`。迁移期保留兼容入口 `{产品}.quanttide.com`，暂不下线。

## 基础设施

基础设施以代码管理，IaC 目录统一为 `manifests/terraform/`。

## 质量门禁

各语言使用标准工具链，本地检查从严、与 CI 一致：

| 语言 | 格式化 | 静态检查 |
|:--|:--|:--|
| Dart | dart format | flutter analyze |
| Go | gofmt | go vet |
| Rust | rustfmt | clippy |

## 可观测与安全

- 生产服务输出结构化审计日志，经 SLS 集中持久化、可查询
- 密码不落明文，本地认证密码使用 PBKDF2-SHA256 哈希
