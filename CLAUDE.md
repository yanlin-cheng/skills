# yanlin-skills —— 工作流合辑仓库

## 项目概述

Yanlin 的 Claude Code 工作流合辑：单仓库单插件多技能（superpowers/mattpocock 同构），收编 relayhand 命令（会话接力）与 md-project-manager 技能（Markdown 项目管理），一次安装随更新自动同步。

## 技术栈

纯 markdown 提示词资产 + git + Windows junction（mklink /J），无代码无 CI。

## 项目结构

```
skills/
├── README.md                              # 英文主门面（与中文版同构）
├── README-ZH.md                           # 简体中文门面
├── CHANGELOG.md                           # 版本记录
├── .claude-plugin/                        # 仓库即市场：plugin.json + marketplace.json（市场/插件同名 yanlin-skills）
├── commands/relayhand.md                  # 会话接力命令（英文指令 + 语言跟随）
└── skills/productivity/md-project-manager/  # 技能三件套：SKILL.md 核心 + references/ 三件按需加载
```

## 开发规范

- 发版必 bump `.claude-plugin/plugin.json` 与 `marketplace.json` 两处 version 字段（不 bump 插件用户拿不到更新）
- 门面双语同构：README.md 与 README-ZH.md 逐节对应，改动双份同步
- 本地日志/ 不进库（决策日志只在本机维护，第一天即进 .gitignore）
- 模板/技能结构改动先实测再定稿；插件实测用 `claude plugin` CLI + `claude -p`（本机个人副本占名，见 .claude/CLAUDE.md 坑栏）
- 提交信息用中文，说清"改了什么、为什么"；推送 GitHub 等用户明示

## 当前阶段

2026-09-22 建仓并收编两工具（relayhand 命令自 relayhand 仓库迁入，插件身份随之移交；md-project-manager 轻拆三件套），v1.0.0 本地实测全绿。待办：推送 GitHub（等用户明示）、真实长对话验证 relayhand 模板效果。
