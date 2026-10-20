# Agent Note: CLAUDE.md 只写本 repo 里查不到的规则

Status: implemented

## Problem

CLAUDE.md 每个任务都会整份加载，agent 照着它做事，它过期了也没有东西会提醒。如果它抄写本 repo 已有的文件，或者写“只有项目骨架”“尚未决定”这类状态，就会出现两种错：

- 副本和原文走偏。例如构建一节写“测试框架尚未决定，决定后再加入 `tests/`”，测试框架定下之后 agent 仍会以为不该加测试。
- 规则跟不上决定。例如“`Gebinde.*`（CLI 除外）never 引用 Haytham Core”漏掉了转接项目 `Gebinde.Hosting.Haytham`，agent 会把转接代码写进 CLI；“旧版在 `legacy` 分支”会让 agent 在本 repo 找一个不存在的分支。

## Decision

判断一行该不该写进 CLAUDE.md，问两件事：

1. 在本 repo 里能不能查到（代码、`.csproj`、`Directory.Build.props`、`global.json`、`.gitignore`、`package.json`、CI、README、笔记与笔记 skill）？
2. 没有这一行，agent 会不会做错？

查得到的只留一句指针，或者不写；查不到、又会让 agent 做错的才写正文。

- **“查得到”只算本 repo 的文件。** 本 repo 公开、会被单独 clone；跨产品的设计文档、ADR 与 Haytham Launcher 的开发者指南都在 Haytham Launcher repo，它在正式版发布前不公开。所以只写在那边的规则（互不引用、实例即整合包、不解析 semver、只用新名称、旧版只当参考、commit 格式、笔记规则）在这里写正文，不改成指向那边的指针。
- **不写状态。** 不写“只有骨架”、进度或“尚未决定”。已经决定的写成规则（例如单独构建用 `PackageReference` 取得 Haytham Core、在 Haytham Launcher 里换成 `ProjectReference`），还没决定的不写。
- **架构表是代码地图。** 只列每个项目放什么；依赖方向看各项目的 `.csproj`。加项目时同时在表里加一行。

### 删掉的内容在哪里

| 删掉的内容 | 在本 repo 的哪里 |
|---|---|
| 产品介绍（工具链的内容、实例目录就是项目根目录） | `README.md`；实例即整合包仍写在架构一节 |
| 现状：只有项目骨架 | 删除，不写状态 |
| 架构表的“可依赖”一栏 | 各项目 `.csproj` 的 `ProjectReference` |
| `.claude/`、`.codex/`、`.codei18n/` 与构建产物不要提交 | `.gitignore` |
| 笔记规则的细节（什么算非平凡、`proposed/` 与 `implemented/`、先写被否方案最有力的理由） | `.agents/skills/write-notes-like-deepseek/SKILL.md` |
| Node 22.18 以上 | `package.json` 的 `description` |
| .NET 10 SDK 与版本锁定 | `global.json`、`README.md` |
| `dotnet build Gebinde.slnx` | `README.md`、`.github/workflows/build.yml` |
| nullable、警告当错误、`IsAotCompatible` | `Directory.Build.props` |
| 测试框架尚未决定 | 删除，不写状态 |

## Alternatives considered

- **只删“现状”一句，其余不动** — 最有力的理由：改动最小，审阅最省事，和 Haytham Core 的 CLAUDE.md 也容易保持一致。不采用：构建一节与 `.gitignore` 那一行仍是副本，“测试框架尚未决定”“CLI 获取 Haytham Core 的方式尚未决定”同样是会过期的状态，只删一句等于留着其他会过期的地方。
- **只写在开发者指南里的规则也改成指针** — 最有力的理由：规则正文只在开发者指南里有一份，改规则只改一处。不采用：外部贡献者与单独 clone 的 agent 打不开 Haytham Launcher repo，指针对他们等于没写。
- **保留“可依赖”一栏** — 最有力的理由：`.csproj` 只记录现在引用了谁，这一栏写的是允许引用谁。不采用：现有五个项目之间，任何反向引用都会形成循环引用，MSBuild 会拒绝构建；新项目的依赖要在加项目时决定，这一栏本来也没列。

## Consequences

- **收益**：CLAUDE.md 从 3,336 字节降到 2,796 字节，删掉的每一条都能在本 repo 的文件里找到；文件里没有会过期的状态。
- **代价与已知上限**：与 Haytham Launcher repo 重复的规则有两份，那边改规则时这边要跟着改，没有脚本检查。这里的规则是压缩版，完整的理由与 C# 注意事项在 Haytham Launcher 的开发者指南里，外部贡献者看不到；如果发现 agent 反复踩到某个只写在那边的坑，就把那一条补进这里。

## Verification

- 在 repo 根目录运行 `rg -n "现状|尚未|再定|骨架" CLAUDE.md`，没有命中。
- 删掉的内容逐项能找到：`rg -n "\.claude/" .gitignore`、`rg -n "dotnet build" README.md .github/workflows/build.yml`、`rg -n "TreatWarningsAsErrors|IsAotCompatible" Directory.Build.props`、`rg -n "22.18" package.json`，每条都有命中。
- `npm run verify-notes` 通过。
