# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目

Gebinde 是 CairnPack 的 Minecraft 整合包工具链，介绍见 [README](README.md)。另外两个产品：可复用的启动器核心 **Haytham Core**（`CairnPack/HaythamCore`）与桌面启动器 **Haytham Launcher**（以子模块引用本 repo）。跨产品的设计文档、ADR 与名词表在 Haytham Launcher repo，它在正式版发布前不公开。

## 架构

| 项目 | 内容 |
|---|---|
| `Gebinde.Sources` | 资源来源（Modrinth、CurseForge、本地文件、git） |
| `Gebinde.Pack` | Manifest、Lockfile（`gebinde.lock`）、实例 |
| `Gebinde.Modpack` | 解析、安装（导出归 `Gebinde.Export`）；声明 Gebinde 需要的接口（`IContentStore`、`IServerRuntimes`），并列出 Lockfile 引用的对象 |
| `Gebinde.Server` | 服务器测试 |
| `Gebinde.Cli` | CLI `gebinde`；组装 Gebinde 与 Haytham Core |

- **Gebinde 与 Haytham Core 互不引用。** 除了 `Gebinde.Cli` 与转接项目 `Gebinde.Hosting.Haytham`，`Gebinde.*` never 引用 `Haytham.Core.*`。Gebinde 只声明自己需要的接口，实现写在转接项目里，由 CLI 与 Haytham Launcher 的原生库共用。引用 Haytham Core 时，本 repo 单独构建用 `PackageReference`；在 Haytham Launcher 里构建时由 MSBuild 属性换成指向子模块的 `ProjectReference`，保证进程里只有一份 Haytham Core。
- **实例即整合包**：没有 Manifest（`gebinde.toml`）就不是实例；Manifest 的未知字段必须保留。
- **资源版本选择不解析 semver**：按兼容性与发布渠道过滤，取发布日期最新者。

## 惯例

- 文档与领域名词用简体中文（中国大陆用语）；代码、标识符、注释与 API 文档用英文。
- Commit 信息为双语：`type(scope): 中文 / English`。作者 email 用 GitHub 的 noreply 地址，不用个人邮箱。
- 新代码只用新名称：`gebinde.toml`、`gebinde.lock`、`.haytham-lock`、`GEBINDE_`／`HAYTHAM_` 环境变量；代码中不出现旧产品名 `cairn`（团队名 CairnPack 不受影响）。
- 参考 ProjBobcat、HMCL、Prism 时可参考但不复制。旧版 Cairn 在 Haytham Launcher repo 的 `legacy` 分支（本 repo 没有这个分支），只当参考，不是规格：行为以本文件、本 repo 的笔记，以及 Haytham Launcher repo 的 `docs/design.md` 与开发者指南为准，代码要重新写。

## 重要变更必须留下笔记

非平凡变更前，先按 `.agents/skills/write-notes-like-deepseek/SKILL.md` 在 `.agents/notes/` 撰写或更新笔记；什么算非平凡、放在哪个目录、何时归档都以那份 skill 为准。提交前运行 `npm run verify-notes`，失败先修正再提交。
