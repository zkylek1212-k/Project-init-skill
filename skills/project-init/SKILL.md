---
name: project-init
description: 「手動專用」新專案開場設定：掛上 agent-hub 派工名單、讀取並優化 implement.md、建立 shared project memory。只有在使用者親自輸入 /project-init 時才執行；嚴禁自動觸發、嚴禁因為使用者提到「新專案」「初始化」「implement.md」「memory」就自己啟用。每個專案只跑一次。
allowed-tools: Bash, Read, Edit, Write, Glob, Grep, Skill, ToolSearch
---

# Project Init（開場一次性設定）

## 啟用規則（最高優先）

- **只由使用者執行**：唯一合法觸發是使用者輸入 `/project-init`。其他情況一律不啟用，
  即使對話提到新專案、初始化、implement.md、memory、agent-hub 也一樣。
- **每個專案只跑一次**：起手先檢查哨兵檔 `.project-init.done`。存在就**立刻停止**，
  回一句「此專案已初始化（見 .project-init.done），如需重跑請先刪除該檔」，不做任何修改。
- 全程在**當前專案根目錄**操作，不動其他專案。

## 起手：哨兵檢查

```bash
test -f .project-init.done && cat .project-init.done
```

有輸出 → 停止（見上）。沒有 → 往下跑步驟 1。

---

## 步驟 1：掛上 agent-hub（永遠執行）

1. 讀本 skill 目錄下的 `agent-squad.md`，那是**派工模型名單的唯一真相來源**
   （與 multi-agent-dispatch skill 內文若有出入，以 `agent-squad.md` 為準）。
2. 確認 hub 活著：載入並呼叫 `get_active_workers`
   - Codex：工具若是 deferred，搜尋 `get_active_workers`；完整名稱是
     `mcp__agent_hub__get_active_workers`。
   - Claude Code / Antigravity：保持既有工具載入方式；Claude Code 可用
     `ToolSearch` query `select:mcp__plugin_multi-agent-hub_agent-hub__get_active_workers`。
   - 回傳 worker 名單 → 記下實際可用的 worker 名稱。
   - 連不上／空名單 → **不要卡住**，標記「agent-hub 目前不可用，本專案先以 solo 模式進行」，繼續步驟 2。
3. 把名單落地到專案：若專案根目錄沒有 `agent-squad.md`，複製本 skill 的那份過去
   （已存在就不覆蓋）。這樣之後的 session 不必再問。
4. 回報一張表：檔位 / worker / model / 適用，並標注哪些 worker 這次真的在線。

> 派工 SOP 本身不在這裡重寫——真要派工時走 `multi-agent-dispatch` skill（含 solo-first triage）。

---

## 步驟 2：讀 implement.md（條件執行）

```bash
ls implement.md IMPLEMENT.md docs/implement.md 2>/dev/null
```

- **找不到** → 跳過步驟 2、3，直接進步驟 4，並在最後回報寫明「無 implement.md，略過優化」。
- **找到** → 完整讀完（不要只讀開頭），再進步驟 3。

---

## 步驟 3：優化 implement.md（只在步驟 2 有讀到時）

先備份：`cp implement.md implement.md.bak`（已有 .bak 就加時間戳）。

用這份檢查表逐條掃，**只改真的有問題的地方**，不要重寫整份、不要為了顯得有做事而加篇幅：

| # | 檢查項 | 不合格的樣子 | 怎麼改 |
|---|---|---|---|
| 1 | 目標可驗收 | 「優化效能」「改善體驗」 | 換成可測的完成條件（數字、指令、預期輸出） |
| 2 | 任務顆粒度 | 一條任務橫跨多個模組 | 拆成單一責任的子任務 |
| 3 | 檔案清單 | 沒寫會動到哪些檔 | 每個子任務補上 `檔案:` 一行 |
| 4 | 依賴順序 | 全部平鋪、看不出先後 | 標 `依賴: 無 / 1,2`；同批平行任務**檔案不得重疊** |
| 5 | 可派工性 | 無法判斷該 solo 還是 dispatch | 對照 `agent-squad.md` 標建議檔位，或標「solo 即可」 |
| 6 | 過度設計 | 為未來需求預留的抽象層、設定檔、介面 | 砍掉並在該行註明「YAGNI：需要時再加」 |
| 7 | 驗證方式 | 沒說怎麼確認做完了 | 補一行可執行的檢查（測試指令 / 手動步驟） |
| 8 | 未決事項 | 假設被寫成事實 | 獨立 `## 待確認` 區塊列出，標明誰來決定 |

改法：用 Edit 就地修改，**保留原作者的結構與用字**，不重排章節、不換語言。
改完輸出一張 diff 摘要表：`項次 | 原本 | 改成 | 理由`（一行一項，不貼整份檔案）。

若 implement.md 本來就過關（沒有任何一條不合格）→ 明講「已檢查 8 項，無需修改」，
**不要硬生生改幾個字來交差**。

---

## 步驟 4：memory-init（永遠執行）

1. 確認是 git repo：`git rev-parse --git-dir`。
   - 不是 → 問使用者要不要 `git init`（memory-init 需要 git hooks）。使用者說不要就跳過本步驟，
     在回報中標明「未建立 shared memory：非 git repo」。
2. 是 git repo → 依執行環境載入 memory-init：
   - Codex：執行 `shared-project-memory:source-command-memory-init` skill。
   - Claude Code / Antigravity：保持既有 `shared-project-memory:memory-init` 流程。
   兩者都必須走完原流程；初始化本身 idempotent，不覆蓋既有檔案。
3. 填 `.project-memory/STATE.md` 與 `handoff.md` 時，只寫**從 README / 建置檔 / `git log` 真的看得到的事實**；
   推測的一律標 `Unverified`。**不得杜撰進度或任務。**

---

## 收尾

1. 寫哨兵檔：

```bash
printf 'project-init done: %s\n' "$(date -Iseconds)" > .project-init.done
```

2. 回報（**精簡，四行以內＋兩張表**）：
   - 步驟 1：hub 狀態 ＋ 派工名單表
   - 步驟 2/3：有無 implement.md；有的話貼 diff 摘要表
   - 步驟 4：memory scaffold 建了什麼 / 跳過的原因
   - 剩下的手動動作：`git add .project-memory .githooks .gitattributes CLAUDE.md AGENTS.md .agents agent-squad.md .project-init.done && git commit`；
     其他機器 clone 後要跑 `git config core.hooksPath .githooks`

## 硬性禁止

- 不得在沒有使用者明確 `/project-init` 的情況下啟用。
- 不得在哨兵檔存在時重跑。
- 不得覆蓋既有檔案（`agent-squad.md`、`.project-memory/*` 一律「不存在才建」；
  `implement.md` 是唯一會被就地修改的檔，且必須先備份）。
- 不得杜撰專案進度、測試結果或 worker 名單。
