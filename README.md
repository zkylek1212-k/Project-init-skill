# project-init

A **manual-only, once-per-project** kickoff skill for [Claude Code](https://claude.com/claude-code).

Opening a new project usually means the same four chores. This skill does them in one pass and
then locks itself so it can never run twice.

```
/project-init
```

That is the only way to start it. The skill will **not** fire because you mentioned "new project",
"initialize", "implement.md" or "memory" in conversation — auto-triggering is explicitly forbidden
in its description.

---

## What it does

| Step | Action | Always runs? |
| --- | --- | --- |
| 0 | Check the `.project-init.done` sentinel — if present, stop immediately and change nothing | yes |
| 1 | Wire up **agent-hub**: read `agent-squad.md` (the dispatch roster), verify the hub is alive via `get_active_workers`, drop a copy of the roster into the project | yes |
| 2 | Read `implement.md` | only if the file exists |
| 3 | Review and improve `implement.md` against an 8-point checklist (backup first, in-place edits, diff summary) | only if step 2 found a file |
| 4 | Scaffold shared project memory (`.project-memory/`, entry files, git hooks) | yes |
| 5 | Write the sentinel, report what changed, list the remaining manual git steps | yes |

No `implement.md` in the folder? Steps 2 and 3 are skipped and it goes straight to memory init.

## The implement.md checklist

Step 3 walks these eight items and **only edits what actually fails**. If the plan already passes,
it says so instead of making cosmetic changes.

| # | Check | Failing looks like | Fix |
| --- | --- | --- | --- |
| 1 | Verifiable goals | "improve performance" | Measurable done-conditions (numbers, commands, expected output) |
| 2 | Task granularity | One task spanning several modules | Split into single-responsibility subtasks |
| 3 | File lists | No mention of which files are touched | Add a `files:` line per subtask |
| 4 | Dependency order | Flat list, no sequencing | Mark `depends: none / 1,2`; parallel batches must not share files |
| 5 | Dispatchability | Can't tell solo from dispatch | Tag a tier from `agent-squad.md`, or mark "solo" |
| 6 | Over-engineering | Abstractions/config reserved for future needs | Cut it, annotate "YAGNI: add when needed" |
| 7 | Verification | No way to confirm completion | Add one runnable check |
| 8 | Open questions | Assumptions written as facts | Move to a `## Pending` block naming the decider |

The original is backed up to `implement.md.bak` before any edit, and the report is a diff summary
table — not a re-paste of the whole file.

## Install

As a plugin marketplace (recommended):

```bash
claude plugin marketplace add <your-github-user>/project-init
claude plugin install project-init@project-init
```

Or drop the skill in by hand:

```bash
cp -r skills/project-init ~/.claude/skills/
```

## Usage

From inside the project you want to set up:

```
/project-init
```

## What it writes to your project

| Path | When |
| --- | --- |
| `agent-squad.md` | Only if absent — the dispatch roster, so later sessions don't have to ask |
| `implement.md` | Edited in place, only if it already exists (backed up to `.bak` first) |
| `.project-memory/`, `AGENTS.md`, `.githooks/`, `.agents/` | Created by the memory scaffold; existing files are never overwritten |
| `.project-init.done` | The sentinel, written last |

Nothing else is touched. `implement.md` is the only pre-existing file the skill modifies.

## Customizing the dispatch roster

[`skills/project-init/agent-squad.md`](skills/project-init/agent-squad.md) is the **single source of
truth** for worker models. Edit it to match your own setup:

```
tier            worker    model                    good for
top logic       agy_cli   gemini-3.1-pro-high      architecture, core logic, hard debugging
workhorse       agy_cli   gemini-3.8-flash-high    general dev, UI, API wiring
default grunt   agy_cli   gemini-3.7-flash-high    boilerplate, fixtures, translation
```

If this file ever disagrees with a dispatch skill's inline model list, **this file wins**.

## Requirements

| Dependency | Needed for | If missing |
| --- | --- | --- |
| Claude Code | everything | — |
| [multi-agent-hub](https://github.com/zkylek1212-k/multi-agent-hub) | step 1 worker check | Step 1 reports "hub unavailable, solo mode" and continues |
| [shared-project-memory](https://github.com/zkylek1212-k/shared-project-memory) | step 4 | Step 4 is skipped and reported as skipped |
| git repo | step 4 (git hooks) | You're asked whether to `git init`; declining skips step 4 |

The skill degrades gracefully — a missing optional dependency never blocks the rest of the run.

## Re-running

Delete the sentinel:

```bash
rm .project-init.done
```

Step 1 and step 4 are idempotent (never overwrite). Step 3 will re-review `implement.md` and take
a fresh backup.

---

## 中文說明

每個專案只跑一次、**只能由使用者手動觸發**的開場設定 skill。輸入 `/project-init` 才會啟動；
即使對話中提到「新專案」「初始化」「implement.md」「memory」也不會自動啟用。

**四個步驟**

1. **掛上 agent-hub** — 讀 `agent-squad.md` 派工名單、呼叫 `get_active_workers` 確認 hub 活著，
   並把名單複製進專案（已存在就不覆蓋）。hub 連不上不會卡住，改標「solo 模式」繼續。
2. **讀 `implement.md`** — 找不到就跳過步驟 2、3，直接進步驟 4。
3. **優化 `implement.md`** — 依上面那張 8 項檢查表逐條掃，只改真的不合格的地方。
   先備份 `.bak`，就地修改，輸出 diff 摘要表而不是整份重貼。本來就過關就明講「無需修改」。
4. **memory-init** — 建立 `.project-memory/` 與 git hooks。非 git repo 會先問要不要 `git init`。

最後寫入哨兵檔 `.project-init.done`，之後重跑會被擋下。要重跑就刪掉該檔。

**硬性規則**：不得自動觸發、不得在哨兵檔存在時重跑、不得覆蓋既有檔案
（`implement.md` 是唯一會被就地修改的，且一定先備份）、不得杜撰專案進度或 worker 名單。

**自訂派工名單**：改 [`skills/project-init/agent-squad.md`](skills/project-init/agent-squad.md)。
它與任何 dispatch skill 內文的型號清單若有出入，**一律以此檔為準**。

---

## License

MIT — see [LICENSE](LICENSE).
