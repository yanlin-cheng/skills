# yanlin-skills — 项目记忆

## 项目概述

合辑插件仓库：relayhand 命令 + md-project-manager 技能，市场与插件同名 yanlin-skills，GitHub 仓库 yanlin-cheng/skills。

## 工作流与技能选择

- 改命令/技能/门面 → 直接改对应文件 + DESIGN 类决策记本地日志（本仓库无 DESIGN.md，决策依据在 relay 仓库本地日志的规格书与 2026-09-22 条目）
- 发版流程：bump 两处 version → CHANGELOG 补条目 → 实测（见下）→ 提交，推送等用户明示

## 关键决策

- 合辑模式（单仓库单插件多技能），非"每工具一仓库"；决策全文见 relay 仓库本地日志 `2026-09-21-合辑仓库与md-project-manager收编设计.md`（规格书）与 DESIGN.md 2026-09-22 条目
- md-project-manager 轻拆三件套：核心 SKILL.md ~432 行 + references/ 三件（templates / automation-platforms / migration-and-editors）按需加载；description 中英触发词齐备
- 本机 ~/.claude/skills/md-project-manager/ 是 junction，指向本仓库——改仓库文件即改日常技能，无需同步
- 通用 agent 指令走 AGENTS.md（2026-09-22）：根 CLAUDE.md 内容迁入 AGENTS.md（AGENTS.md 开放标准，Codex/Cursor 等可读），CLAUDE.md 只留一行 `@AGENTS.md` 导入；Claude 侧有 CLAUDE.md 就优先读它，故行为不变。不抄 changesets 发版自动化（需 node 依赖，违背零依赖定位）与 .agents/ 目录（Claude Code 明确不读，且决策记录已有本地日志）

## 坑（勿重蹈）

- 本机个人副本占名两层：裸名 /relayhand 命令（~/.claude/commands/）与裸名 md-project-manager 技能（junction）都被个人副本占用；同名技能会遮蔽插件前缀名（yanlin-skills:md-project-manager 不存在，裸名归个人版）。测插件命令用全名 /yanlin-skills:relayhand 可行；测技能全名需临时挪开个人副本或换干净环境
- `marketplace remove` 会连带卸载其下插件，重 add 后需重 install
- 本地路径安装的插件缓存是整仓库快照（含 .gitignore 排除文件），仅本机现象；分发走 GitHub clone 无泄露
- ~/.claude/skills/ 混有大量第三方技能，绝不能把该目录整个变 git 仓库

## 待办事项

- [x] 推送 GitHub（2026-09-22 用户明示，首个正式发版）
- [ ] 真实长对话验证 relayhand 模板效果（自 relayhand 仓库继承的待办）
- [ ] 公众号文案若已提过安装命令，推送后需同步改指 yanlin-skills

## 更新日志

### 2026-09-22 建仓与收编（v1.0.0）
- 建仓：骨架（LICENSE/CHANGELOG/.gitignore，决策日志本地化）→ 迁入 relayhand 命令（脱敏校验过）→ md-project-manager 轻拆三件套（9 处悬空引用改指 references/）→ 插件配置双文件 → 双语门面 + productivity 分组说明
- Task 6 junction 切换：备份 → mklink /J → 冒烟通过（技能照常触发）→ 删备份
- Task 7 本地实测全绿：validate/add/install/details/卸载重装/bump 更新流（1.0.0→1.0.1→恢复 1.0.0）；实测全文见 本地日志/2026-09-22-实测记录.md
- 插件身份自 relayhand 仓库移交：自更新 URL 改指本仓库，插件名 yanlin-skills@yanlin-skills

### 2026-09-22 AGENTS.md 通用化 + 参考项目隔离
- 调研 mattpocock/skills 仓库（.changeset=changesets 发版工具、.agents/=作者自定文档目录、根 AGENTS.md 是一行指针）+ agents.md 标准官网 + Claude Code 官方文档（v2.1.277+ 原生支持 AGENTS.md，有 CLAUDE.md 则只读 CLAUDE.md）
- 改造：根 CLAUDE.md 内容迁入新建 AGENTS.md，CLAUDE.md 变一行导入指针；参考项目/ 进 .gitignore（第三方仓库仅本地学习，永不入库）
- 附带发现：mattpocock 的 skills/ 也用 engineering/productivity 分组，支持本仓库 productivity/ 设计
