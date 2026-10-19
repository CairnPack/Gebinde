# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目

**Gebinde** 是团队 CairnPack 的 Minecraft 整合包工具链（C#，GPL-3.0-or-later）：Manifest（`gebinde.toml`）、Lockfile（`gebinde.lock`）、资源来源、解析、安装、导出、服务器测试，以及 CLI `gebinde`。整合包是受版本控制、可复现的项目，实例目录**就是**项目根目录。

另外两个产品：可复用的启动器核心 **Haytham Core**（`CairnPack/HaythamCore`）与桌面启动器 **Haytham Launcher**（以子模块引用本 repo）。跨产品的设计文档、ADR 与名词表目前放在 Haytham Launcher repo。

**现状**：只有项目骨架，还没有代码。

## 架构

| 项目 | 内容 | 可依赖 |
|---|---|---|
| `Gebinde.Sources` | 资源来源（Modrinth、CurseForge、本地文件、git） | 无 |
| `Gebinde.Pack` | Manifest、Lockfile、实例 | Sources |
| `Gebinde.Modpack` | 解析、安装（导出归 `Gebinde.Export`）；声明 Gebinde 需要的接口（`IContentStore`、`IServerRuntimes`），并列出 Lockfile 引用的对象 | Pack、Sources |
| `Gebinde.Server` | 服务器测试 | Modpack |
| `Gebinde.Cli` | CLI `gebinde`；组装 Gebinde 与 Haytham Core | 全部 |

- **Gebinde 与 Haytham Core 互不引用。** `Gebinde.*`（CLI 除外）never 引用 `Haytham.*`；Gebinde 只声明自己需要的接口，由“组装的地方”（CLI 与 Haytham Launcher 的原生库）接上 Haytham Core。接口的细节与 CLI 获取 Haytham Core 的方式尚未决定。
- **实例即整合包**：没有 `gebinde.toml` 就不是实例；Manifest 的未知字段必须保留。
- **资源版本选择不解析 semver**：按兼容性与发布渠道过滤，取发布日期最新者。

## 惯例

- 文档与领域名词用简体中文（中国大陆用语）；代码、标识符、注释与 API 文档用英文。
- Commit 信息为双语：`type(scope): 中文 / English`。作者 email 用 GitHub 的 noreply 地址，不用个人邮箱。
- 新代码只用新名称：`gebinde.toml`、`gebinde.lock`、`.haytham-lock`、`GEBINDE_`／`HAYTHAM_` 环境变量；代码中不出现 `cairn`。
- 参考 ProjBobcat、HMCL、Prism 时可参考但不复制；旧版 Cairn（Kotlin）只当行为规格，代码要重新写。
- `.claude/`、`.codex/`、`.codei18n/` 与构建产物是本地内容，不要提交。

## 重要变更必须留下笔记

非平凡变更（行为、架构、跨文件契约、流程与工具链、测试策略、落盘／网络／配置格式）前，依 `.agents/skills/write-notes-like-deepseek/SKILL.md` 在 `.agents/notes/` 撰写或更新笔记；新构想放 `proposed/`，实现时随代码移到 `implemented/`；放弃某个方案前先写出它最有说服力的理由。提交前运行 `npm run verify-notes`（Node 22.18 以上）。

## 构建

.NET 10 SDK（版本由 `global.json` 锁定）。

```powershell
dotnet build Gebinde.slnx
```

`Directory.Build.props` 打开 nullable、警告当错误与 `IsAotCompatible`；每个项目一个程序集，依赖方向由项目引用强制。测试框架尚未决定，决定后再加入 `tests/`。
