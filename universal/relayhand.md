# relayhand Universal Core Prompt (manual paste-in version)

Works with **any** AI chat product — no custom-command feature needed, and the next session may even run in a different product.

**How to use**: copy everything below the divider into the conversation you want to hand off, and send it. To assign a task to the next leg, write it at the end, under "User's task for the next leg". The AI will produce a handoff note and a copy-paste prompt block; paste that block into the new conversation.

**Self-installing agents**: instead of paste-per-session, your agent can install this command into its own command/rule system — see the agent-install prompt in the repository README. If you (the agent) installed this prompt yourself, check the [repository](https://github.com/yanlin-cheng/skills) for a newer version of this file when the command is invoked; if one exists, **ask the user before** refreshing your installed copy. The update takes effect on the next invocation, never mid-run.

---

Please organize this conversation into a "handoff note" so that I can continue working directly in a brand-new conversation.

Writing doctrine: **Restrained throughout; the "Next" section is the only place for detail.**

**Language: follow the conversation.** Write the handoff note and the copy-paste prompt in the language this conversation uses (mixed conversation → prefer the user's own language). One template serves every language.

## Template (drop any section that has nothing to say — never pad)

Section titles follow the conversation language; shown here in English:

```markdown
# Handoff: <one sentence naming the task>

## Goal
<one sentence: what is being built/fixed, and why. If it takes more than a sentence, you haven't distilled it — rewrite>

## State
- Done: <checklist, one item per line>
- In progress: <what is being worked on right now>
- Blocked: <obstacles, open questions>

## Key decisions
<only technical choices that affect later work, and why>

## Pitfalls (do not repeat)
<failed attempts and why they failed; debugging conclusions as symptom → root cause → fix>

## Next
<immediately executable actions, in order, concrete enough to run as-is>

## Files
Read: <extracted from the conversation, real paths>
Edited: <extracted from the conversation, real paths; uncommitted working-tree changes count too — never claim a repo file is untouched while its changes are still pending>
```

## Writing discipline

1. The reader is the next agent, not the user: write a "baton", not a "retrospective"
2. Do not narrate the conversation ("we first discussed… then…"); only conclusive state
3. Facts only, no vibes; every word must help the next leg do the work
4. "Next" is the only section allowed detail; every other section serves it
5. If I attach a task for the next leg (see the end of this message), the task is the filter: summarize only the background, decisions, files, and pitfalls that task needs; however important, anything unrelated stays out, and the note's title is framed around that task
6. **When the user's latest requirement changes what to do next, quote it verbatim in "Next"** — never paraphrase a direct instruction
7. If this conversation itself began from a handoff note, add that note's path to a "Source of truth" section to keep the relay chain traceable
8. File lists must be extracted from the conversation — never invented from memory
9. Redact: API keys, passwords, and personal information must not be written
10. Existing artifacts (plan docs, commits, issues): reference by path or link, never copy content

**User's task for the next leg** (omit for a general summary):
<fill in what the next leg should do here>

## Output requirements

1. Display the full handoff note in your reply for my review (I may ask for a rewrite)
2. At the end of your reply, output a prompt code block that can be copied straight into a new conversation, with this fixed content (in the conversation's language). The first line is the naming seed: many AI products auto-title a conversation from its first prompt, so this line steers the title toward the relay chain. Also suggest saving the handoff note as <relayhand-YYYYMMDD-HHmm>.md using the same timestamp so file name and session name stay aligned:

```
Relay relayhand-<YYYYMMDD-HHmm> — <one-sentence task, in the conversation's language>
You are taking over a relayed task. Below is the handoff note from the previous session. Read it fully, do not recap its contents, then start executing the "Next" section directly. Run `git status` first to reconcile the repo's actual state with the note; if they disagree, reconcile before acting.
<full text of the handoff note>
```
3. Below the code block, add one human-readable line (not inside it) telling me to rename the new conversation to relayhand-<YYYYMMDD-HHmm> after pasting — via `/rename` in Claude Code, or the product's own conversation-rename feature — so the relay chain stays traceable in the session list (optional, but exact)

## Night-run handoff (optional add-on)

If my message contains "今晚跑" or "夜跑" (night-run), produce the handoff note with these deltas on top of the normal requirements — all ten writing disciplines still apply; without those words, run the normal flow untouched:

1. **Persistent path instead of a temp file**: save (or tell me exactly where to save) the handoff note as `<project>/docs/relayhand/night-<YYYYMMDD-HHmm>.md`. An unattended runner picks it up hours later; temp directories will not have survived.
2. **Night-run header** — write it as the FIRST section under the title. Strings marked *verbatim* are contract text: copy exactly, never paraphrase.

```markdown
## 夜跑头

- entry id: <YYYYMMDD-HHmm>-<project slug>（时间戳与棒文件名一致；本分钟已入队则追加 -2/-3）
- work_dir: <项目绝对路径>
- plan: <已批准 plan/spec 的路径引用——只写路径，不抄内容>
- done_criteria: <完成标准，逐项列出，白天批准时的原话>
- sensitive_whitelist:
  - <用户白天在聊天内逐条明确批准的敏感操作，原话引用，动作+对象粒度，如 "git push 到 main">
  - （空列表 = 今晚无任何敏感操作授权）
- questions policy (verbatim): 遇到需要人决策的问题：写进本项目 docs/晨报-待决策-<日期>.md（问题+背景+选项+你的建议），跳过该步继续做别的；全部卡住则标记 blocked 收尾。不许带着方向性猜测执行。敏感操作（删除/发布/对外发送）只执行白名单内的，白名单外一律挂起写晨报。
- result sentinel (verbatim contract): 收尾写 <work_dir>/docs/relayhand/night-result-<entry id>.json，JSON 键名逐字="status"/"done_criteria"/"questions_path"；status 只认小写 "done"/"blocked"；done_criteria=逐项证据（测试输出摘录、commit 号等）；questions_path=待决策文件路径（无则 null）。无哨兵=编排器记 unknown。
```

If my approved plan mentions sensitive steps but nothing was explicitly enumerated in this conversation, ask me before writing the baton; an empty whitelist is legitimate only after I confirm there are none tonight.

3. **Output the night-run version of the prompt** (replaces the normal output prompt):

```
夜跑接力 night-<YYYYMMDD-HHmm> — <任务一句话，用对话语言>
读取 <夜跑棒完整路径> —— 这是留给今晚无人值守工人的接力棒。先读完，不要复述其内容，然后直接开始执行「下一步」部分。先跑 git status 对账仓库实际状态与文档是否一致；不一致先对齐再动手。全程严守棒上夜跑头：问题政策按固定文案执行；敏感操作只执行白名单内的，白名单外一律挂起写晨报；收尾按契约写结果哨兵（JSON 键名逐字 "status"/"done_criteria"/"questions_path"）。
```

Queue registration, shutdown pre-authorization and morning-report delivery are **orchestration-layer specifics** (cc-connect users find them in `commands/relayhand.md` 【cc-connect 增强】). Port them to your own scheduler + headless CLI against the same baton contract, or drop them.
