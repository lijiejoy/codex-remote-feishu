# 仓库分析与规划说明（2026-06）

> Type: `draft`
> Updated: `2026-06-21`
> Summary: 基于当前仓库现状补一份面向项目理解的分析说明，梳理目标、结构、风险点与后续整理方向。

## 1. 项目定位

`codex-remote-feishu` 是一个把 Codex 工作现场投影到飞书的本地项目。
它的核心不是单纯做一个聊天壳，而是把工作区、thread、消息流、图片、停止控制和 WebSetup 管理面整合成一套可接管的工作流。

从仓库现状看，这个项目已经不是早期原型，而是一个有明确分层、明确文档体系、并且持续收敛到统一二进制入口的成熟工程。

## 2. 现有实现轮廓

我从 README、`go.mod`、`web/package.json`、`cmd/codex-remote/main.go`、`internal/app/daemon/app.go`、`internal/adapter/feishu/gateway.go` 看到了几条清晰主线：

1. Go 是主实现语言，仓库当前只维护 Go 版本。
2. 产品入口收敛到统一二进制 `codex-remote`。
3. `daemon` 是组合根，负责 orchestrator、gateway、relay、preview、外部接入与各类 runtime。
4. `web/` 仍保留独立前端管理界面，路由上按 setup/admin 分流。
5. `docs/` 已经被当作长期设计与约束的主载体在维护。

## 3. 业务目标

这个项目解决的核心问题是：让飞书成为 Codex 的远程控制面和投影面。

它强调的不是“发消息”，而是：

- 保留 thread 语义
- 保留工作目录语义
- 支持继续已有对话
- 支持图片、停止、steer、compact
- 支持按需接入 VS Code
- 支持安装、升级、管理和可视化状态

这说明项目的产品定位偏“工作台桥接层”，不是单一机器人。

## 4. 结构判断

当前仓库的结构大致可理解为：

- `cmd/`：统一入口与兼容入口
- `internal/core/`：协议、状态、控制面与渲染等基础层
- `internal/adapter/`：Codex、Feishu、relayws 等平台/传输适配
- `internal/app/`：daemon、install、launcher、wrapper 等应用编排
- `internal/runtime/`：进程与生命周期管理
- `web/`：管理端 UI
- `docs/`：架构、产品与实现约束

## 5. 当前观察到的风险点

这次只做仓库级分析，不改业务逻辑。当前最明显的维护性热点是：

1. `internal/adapter/codex/translator.go` 偏大，协议翻译容易变成高耦合区。
2. `internal/adapter/feishu/markdown_preview.go` 偏大，预览链路复杂。
3. `web/src/routes/SetupRoute.tsx` 偏大，setup 交互很集中。
4. `internal/app/daemon/app.go` 偏大，组合根承载过多职责。
5. `internal/adapter/feishu/gateway.go` 偏大，平台交互与状态管理混在一起。

另外，`internal/runtime` 的覆盖率在先前审查里明显偏低，说明生命周期管理仍值得补测试。

## 6. 产品层面的判断

这个项目最大的优点是边界意识比较强：文档很多，而且已经把 release/install、daemon、gateway、preview、管理 UI 分开谈。

但它也有典型的成熟工程问题：

- 核心文件偏大
- 前端关键流程缺少更细的行为级测试
- 管理 UI 与 setup 页有一定重复
- 一些“保存后热应用”链路的状态表达还可以更清楚

这些都不算致命问题，但说明仓库已经进入“继续扩功能前，先整理结构”的阶段。

## 7. 后续建议

如果继续按 `plan-docs` 的路径推进，建议下一步优先做这三件事：

1. 把 `03-索引.md` 当作单一事实来源，补出更细的模块树。
2. 给大文件对应的模块建立更明确的微观文档。
3. 选择一个最值得下手的热点模块，先做一次结构收敛，再谈功能扩展。
