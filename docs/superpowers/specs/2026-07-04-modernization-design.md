# Picasign 全面稳定升级设计

## 目标

将项目中明确版本化且已经落后的运行时、GitHub Actions 和 NuGet 依赖升级到 2026-07-04 可用的最新稳定主版本或稳定版本，同时保持现有多账号签到行为不变，并以可重复的本地检查验证改动。

## 升级范围

- 将目标框架从 `net8.0` 升级为最新 LTS `net10.0`。
- 将 `actions/checkout` 从 `v2` 升级为使用 Node.js 24 的 `v6`。
- 将 `actions/setup-dotnet` 从 `v1` 升级为使用 Node.js 24 的 `v5`，并安装 `10.0.x` SDK。
- 将 `Newtonsoft.Json` 从 `13.0.1` 升级为 `13.0.4`。
- 将 `picacomic` 从 `1.0.4` 升级为 `1.0.7`。
- 更新 README 中过时的 VS2019 和 .NET 5 信息，并修复当前中文文本的编码表现。

不重写签到业务逻辑，不改变 `ACCOUNTS` secret 的格式，不引入新的部署方式或额外服务。

## 实现方案

项目继续保持单个控制台应用结构。项目文件负责锁定目标框架和直接依赖版本；GitHub Actions 工作流负责在 `ubuntu-latest` 上安装 .NET 10、构建并按现有计划运行程序；README 准确描述当前环境和配置方式。

工作流使用稳定主版本标签，以自动接收同一主版本内的安全修复。目标框架和 NuGet 包使用明确版本，保证项目构建基线清晰。现有定时任务、push 触发条件、时区和 secret 名称保持不变。

## 兼容性与错误处理

升级后通过编译验证 `picacomic 1.0.7` 的公开 API 仍兼容现有调用。程序现有的缺少账号参数、账号格式错误、登录失败和签到失败行为保持原样；本次不扩展业务错误处理，以控制变更范围。

GitHub 托管 runner 满足新版 Actions 的 runner 要求。工作流显式安装 `10.0.x`，避免依赖 runner 镜像当前预装的 SDK。

## 验证标准

1. `dotnet restore` 成功。
2. `dotnet build -c Release --no-restore` 成功且无编译错误。
3. `dotnet list package --outdated --include-transitive` 不报告可升级包。
4. `dotnet list package --vulnerable --include-transitive` 不报告已知漏洞。
5. `dotnet list package --deprecated` 不报告弃用包。
6. 不提供账号参数运行程序时，仍以预期的配置错误退出，证明入口可启动且参数校验未丢失。
7. 工作流不再引用 Node.js 20 或更早运行时的旧 Action 主版本，并与 `net10.0` 一致。

真实签到需要有效的 `ACCOUNTS` secret 和外部服务可用性；本地验证不使用或索取用户凭据，因此将其明确列为 GitHub Actions 端到端验证边界。
