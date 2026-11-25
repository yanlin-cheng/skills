---
description: Relay: compress the current conversation into a task handoff for the next session, and output a copy-paste prompt
argument-hint: "[focus of the next leg / cross-product / 今晚跑(night-run) / 入库, optional]"
disable-model-invocation: true
---

# /relayhand —— Session Relay

Compress the current conversation into a "baton" so the next fresh session (or a new conversation in a different AI product) can pick up the work immediately.

Writing doctrine (from Cline's compactor system prompt): **Restrained throughout, detailed only about the next step.**

**Language: follow the conversation.** Talk to the user and write the handoff note in the language the current conversation uses (mixed conversation → prefer the user's own language). One template serves every language.

$ARGUMENTS (optional):
- Plain text = the task for the next leg. The note is then tailored to that task (see writing discipline #5)
- Contains "cross-product" (跨产品) = additionally output the embedded prompt
- Contains "今晚跑" or "夜跑" = night-run variant (see "Night-run variant" below). These trigger words are the ONLY way in — without them the normal flow runs untouched and nothing is ever enqueued
- Contains "入库" or "记日志" = sync the knowledge base this run (see Step 3.2; off by default)

## Step 0: Update check (optional, non-blocking)

If this command was installed as a plugin (`yanlin-skills@yanlin-skills`), skip this step — updates arrive through the plugin system on their own.

Otherwise (installed as a raw file) and the machine is online, quickly compare your installed copy against the official source: fetch `https://raw.githubusercontent.com/yanlin-cheng/skills/main/commands/relayhand.md` and diff it against the file this command was installed from (typically `~/.claude/commands/relayhand.md`).

- Identical → proceed silently
- Different → the update takes effect on the **next** invocation, never mid-run: ask the user "a newer relayhand is available — update now?" If yes, overwrite the installed file with the fetched content, then continue with the current instructions
- Offline or fetch fails → skip; at most one short line in the final report

## Step 1: Locate the source-of-truth transcript 【Claude Code 增强】

The full record of the current conversation is a .jsonl file under <home>/.claude/projects/. Look first in the directory matching the current project (path rule: special and non-ASCII characters in the project path are replaced with -) for the most recently modified .jsonl; if not found, use Bash to take the most recently modified one globally (the current session is being written to continuously, so it is usually the newest). Note the path — Step 2 needs it.

## Step 2: Write the handoff note 【通用核心】

Save it to relayhand-<YYYYMMDD-HHmm>.md in the OS temp directory (%TEMP% on Windows, /tmp on Linux/macOS).

### Template (drop any section that has nothing to say — never pad)

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

## Suggested skills
<skills the next agent should invoke via the Skill tool to execute "Next"; one per line, a few words on why; drop this section entirely if none>

## Next
<immediately executable actions, in order, concrete enough to run as-is>
<the $ARGUMENTS focus goes at the top of this section>

## Files
Read: <extracted from the conversation, real paths>
Edited: <extracted from the conversation, real paths; uncommitted working-tree changes count too — never claim a repo file is untouched while its changes are still pending>

## Source of truth
Full transcript: <jsonl path>. When this note lacks detail (a discussion, an error message), Grep that file.
```

### Writing discipline

1. The reader is the next agent, not the user: write a "baton", not a "retrospective"
2. Do not narrate the conversation ("we first discussed… then…"); only conclusive state
3. Facts only, no vibes; every word must help the next leg do the work
4. "Next" is the only section allowed detail; every other section serves it
5. **When $ARGUMENTS names the next task, the task is the filter**: summarize only the background, decisions, files, and pitfalls that task needs; however important, anything unrelated stays out, and the note's title is framed around that task. Only with no specified task do the general summary
6. **When the user's latest requirement changes what to do next, quote it verbatim in "Next"** — never paraphrase a direct instruction
7. **If this conversation itself began from a handoff note, add that note's path to "Source of truth"** to keep the relay chain traceable
8. File lists must be extracted from the conversation — never invented from memory
9. Redact: API keys, passwords, and personal information must not be written
10. Existing artifacts (plan docs, commits, issues): reference by path, never copy content

## Step 3: Show and sediment

1. Display the full handoff note in chat for review (rewrite on request).
2. Knowledge-base log 【Claude Code 增强】: **skipped by default — run only when the user explicitly asks**, i.e. $ARGUMENTS contains "入库"/"记日志" or the user said in this conversation that the work should be persisted to the knowledge base. Then check `H:\AI-programming-modeling\AIcoding项目进度管理\`: if it exists, read its project-level CLAUDE.md, follow its rules, and use the md-project-manager skill to log the work and update the panel (record why this was done, what pitfall was hit, what scenarios it is reusable in — never a流水账 diary). If the user did not ask, or the path does not exist: do nothing, and just carry one line in the report: "日志：未记（未要求入库）". Porters: delete this sub-step or repoint the path to your own knowledge base.

## Step 4: Output the copy-paste prompt

At the end of the reply, first a one-line note "copy the following into a new conversation", then the code block (the prompt itself is also written in the conversation's language):

**File version (default output)** — the first line is the naming seed: the session's auto-generated title summarizes the first prompt, so this line steers the title toward the relay chain:

```
Relay relayhand-<YYYYMMDD-HHmm> — <one-sentence task, in the conversation's language>
Read <full path to the handoff note>. It is the handoff note from the previous session. Read it fully, do not recap its contents, then start executing the "Next" section directly. Run `git status` first to reconcile the repo's actual state with the note; if they disagree, reconcile before acting.
```

If $ARGUMENTS carries a focus, append one line: `Focus this time: $ARGUMENTS`

Below the code block, add one human-readable line (not inside it): `After pasting, run /rename relayhand-<YYYYMMDD-HHmm> to pin the session name to the relay chain (optional, but exact).`

**Embedded version (only when $ARGUMENTS contains "cross-product" / 跨产品; self-contained, pasteable into any AI product's new conversation)** — same naming-seed first line:

```
Relay relayhand-<YYYYMMDD-HHmm> — <one-sentence task, in the conversation's language>
You are taking over a relayed task. Below is the handoff note from the previous session. Read it fully, do not recap its contents, then start executing the "Next" section directly. Run `git status` first to reconcile the repo's actual state with the note; if they disagree, reconcile before acting.
<full text of the handoff note>
```

Finally report: the note's path, whether a knowledge-base log was written (path if yes), and that the prompt is ready.

## Night-run variant 【opt-in only】

**Trigger**: $ARGUMENTS contains "今晚跑" or "夜跑" — the sole entry. Without these words everything below is inert: normal flow, temp-dir baton, zero writes to any queue file (a normal handoff must never auto-enqueue). With the trigger, run Steps 1–4 as usual — all 10 writing disciplines still apply — plus these deltas:

1. **Persistent baton path** (replaces Step 2's temp dir): save the note to `<work_dir>/docs/relayhand/night-<YYYYMMDD-HHmm>.md` (create the directory if needed). The baton is picked up hours later by an unattended runner — %TEMP% will not have survived.

2. **Night-run header 【通用核心】** — write it as the FIRST section of the baton, directly under the title. Strings marked *verbatim* are contract text consumed by the runner's worker prompt: copy them exactly, never paraphrase.

```markdown
## 夜跑头

- entry id: <YYYYMMDD-HHmm>-<project slug>（时间戳与棒文件名一致；项目用拉丁短名，如 tapexit；本分钟已入队则追加 -2/-3）
- work_dir: <项目绝对路径>
- plan: <已批准 plan/spec 的路径引用——只写路径，不抄内容>
- done_criteria: <完成标准，逐项列出，白天批准时的原话>
- sensitive_whitelist:
  - <用户白天在聊天内逐条明确批准的敏感操作，原话引用，动作+对象粒度，如 "git push 到 main">
  - （空列表 = 今晚无任何敏感操作授权）
- questions policy (verbatim): 遇到需要人决策的问题：写进本项目 docs/晨报-待决策-<日期>.md（问题+背景+选项+你的建议），跳过该步继续做别的；全部卡住则标记 blocked 收尾。不许带着方向性猜测执行。敏感操作（删除/发布/对外发送）只执行白名单内的，白名单外一律挂起写晨报。
- result sentinel (verbatim contract): 收尾写 <work_dir>/docs/relayhand/night-result-<entry id>.json，JSON 键名逐字="status"/"done_criteria"/"questions_path"；status 只认小写 "done"/"blocked"；done_criteria=逐项证据（测试输出摘录、commit 号等）；questions_path=待决策文件路径（无则 null）。无哨兵=夜跑器记 unknown。
```

**Whitelist sourcing**: enumerate from the user's approved words in this conversation (typically where the plan was approved during the day). If an approved plan contains sensitive steps but nothing was explicitly enumerated — ask the user in chat NOW, before writing the baton. An empty whitelist is legitimate only after the user confirms there are no sensitive steps tonight.

3. **Queue registration 【cc-connect 增强】**（porters: delete this delta — the queue schema is a public convention, any scheduler + headless CLI can consume it）:
   - File: `~/.cc-connect/night-queue.json`; UTF-8 **without BOM**; it lives outside every repo and must never be committed; it may mirror the whitelist but no secrets.
   - File missing → create with exactly five top-level keys: `{"version": 1, "shutdown_armed": false, "armed_date": null, "armed_note": "", "entries": []}`. File exists → read it first, append to `entries`, rewrite the whole file — never clobber other projects' entries.
   - Entry keys, exactly nine, verbatim: `id` / `project` / `work_dir` / `baton_path` / `done_criteria` / `sensitive_whitelist` / `timeout_min` / `status` / `added_at`. Values: `id`, `work_dir`, `done_criteria`, `sensitive_whitelist` mirror the night-run header; `baton_path` = absolute path of the persistent baton; `timeout_min` defaults to 120; `status` = "queued"; `added_at` = ISO 8601 with +08:00.
   - `shutdown_armed` must be a real JSON boolean (`true`/`false`), never a string.
   - **Shutdown question — never arm without an explicit yes in chat**: after enqueuing, ask the user "今晚跑完关机吗？". Only on clear agreement set `shutdown_armed: true`, `armed_date: <YYYY-MM-DD 当天>`, `armed_note: <出处：时间+渠道+用户原话>`; decline / vague / no answer → leave it false. Never touch `armed_*` for entries you are not arming.
   - **Report in chat**: entry id + 今晚队列共几项 + 白名单逐条复述供确认.

4. **The output prompt is the night-run version** — it replaces Step 4's file-version prompt (the embedded version is still only added when "cross-product" is also present):

```
夜跑接力 night-<YYYYMMDD-HHmm> — <任务一句话，用对话语言>
读取 <夜跑棒完整路径> —— 这是留给今晚无人值守工人的接力棒。先读完，不要复述其内容，然后直接开始执行「下一步」部分。先跑 git status 对账仓库实际状态与文档是否一致；不一致先对齐再动手。全程严守棒上夜跑头：问题政策按固定文案执行；敏感操作只执行白名单内的，白名单外一律挂起写晨报；收尾按契约写结果哨兵（JSON 键名逐字 "status"/"done_criteria"/"questions_path"）。
```

## Distribution notes

This command is layered so it stays shareable:
- 【通用核心】= template + writing disciplines + dual-version prompt + the night-run baton layer (persistent path, night-run header: entry id / plan / done_criteria / whitelist / questions policy / result sentinel, opt-in discipline): portable as-is to any AI product with custom commands (Cline, Cursor, …)
- 【Claude Code 增强】= Step 1's jsonl transcript pointer, Step 3.2's knowledge-base log (carries the repo owner's local KB path): delete whole blocks when porting
- 【cc-connect 增强】= Night-run delta 3's queue registration + shutdown question: delete whole block when porting — cc-connect is *our* orchestration implementation, never a prerequisite of the baton format

Other personal configuration (your own machine paths, preferences): keep it in your own local copy only — never commit it back to the repository.
