# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 專案

**Gebinde** 是團隊 cairnpack 的 Minecraft 整合包工具鏈（C#，GPL-3.0-or-later）：Manifest（`gebinde.toml`）、Lockfile（`gebinde.lock`）、資源來源、解析、安裝、匯出、伺服器測試，以及 CLI `gebinde`。整合包是受版本控制、可重現的專案，實例目錄**就是**專案根目錄。

另外兩個產品：可複用的啟動器核心 **Haytham Core**（`CairnPack/HaythamCore`）與桌面啟動器 **Haytham Launcher**（以子模組引用本 repo）。跨產品的設計文件、ADR 與名詞表目前放在 Haytham Launcher repo。

**現況**：只有專案骨架，還沒有程式碼。

## 架構

| 專案 | 內容 | 可依賴 |
|---|---|---|
| `Gebinde.Sources` | 資源來源（Modrinth、CurseForge、本機檔案、git） | 無 |
| `Gebinde.Pack` | Manifest、Lockfile、實例 | Sources |
| `Gebinde.Modpack` | 解析、安裝、匯出；宣告 Gebinde 需要的介面（`IContentStore`、`IServerRuntimes`），並列出 Lockfile 引用的物件 | Pack、Sources |
| `Gebinde.Server` | 伺服器測試 | Modpack |
| `Gebinde.Cli` | CLI `gebinde`；組裝 Gebinde 與 Haytham Core | 全部 |

- **Gebinde 與 Haytham Core 互不引用。** `Gebinde.*`（CLI 除外）never 引用 `Haytham.*`；Gebinde 只宣告自己需要的介面，由「組裝的地方」（CLI 與 Haytham Launcher 的原生函式庫）接上 Haytham Core。介面的細節與 CLI 取得 Haytham Core 的方式尚未決定。
- **實例即整合包**：沒有 `gebinde.toml` 就不是實例；Manifest 的未知欄位必須保留。
- **資源版本選擇不解析 semver**：依相容性與發布頻道過濾，取發布日期最新者。

## 慣例

- 文件與領域名詞用繁體中文；程式碼、識別字、註解與 API 文件用英文。
- Commit 訊息為雙語：`type(scope): 中文 / English`。作者 email 用 GitHub 的 noreply 地址，不用個人信箱。
- 新程式碼只用新名稱：`gebinde.toml`、`gebinde.lock`、`.haytham-lock`、`GEBINDE_`／`HAYTHAM_` 環境變數；程式碼中不出現 `cairn`。
- 參考 ProjBobcat、HMCL、Prism 時可參考但不複製；舊版 Cairn（Kotlin）只當行為規格，程式碼要重新寫。
- `.claude/`、`.codex/`、`.codei18n/` 與建置產物是本機內容，不要提交。

## 重要變更必須留下筆記

非平凡變更（行為、架構、跨檔案契約、流程與工具鏈、測試策略、落盤／網路／設定格式）前，依 `.agents/skills/write-notes-like-deepseek/SKILL.md` 在 `.agents/notes/` 撰寫或更新筆記；新構想放 `proposed/`，實作時隨程式碼移到 `implemented/`；放棄某個方案前先寫出它最有說服力的理由。提交前執行 `npm run verify-notes`（Node 22.18 以上）。

## 建置

.NET 10 SDK（版本由 `global.json` 釘住）。

```powershell
dotnet build Gebinde.slnx
```

`Directory.Build.props` 開啟 nullable、警告當錯誤與 `IsAotCompatible`；每個專案一個組件，相依方向由專案參考強制。測試框架尚未決定，決定後再加入 `tests/`。
