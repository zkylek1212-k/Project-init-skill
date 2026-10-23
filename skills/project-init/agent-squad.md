# 派工模型名單（本專案）

| 檔位 | worker | model | effort | 適用 |
|---|---|---|---|---|
| 頂級架構 | claude_cli | claude-opus-5-5 | high | 跨模組架構重構、整合層、需要嚴謹審查的改動 |
| 頂級邏輯 | codex_cli | gpt-6.1-sol | high | 核心演算法、困難除錯、後端邏輯 |
| 頂級邏輯 | agy_cli | gemini-3.1-pro-high | （名稱內含） | 架構重構、核心邏輯、困難除錯 |
| 敏捷主力 | agy_cli | gemini-3.8-flash-high | （名稱內含） | 一般開發、UI、串 API、樣板、測資、翻譯（不確定時預設） |

claude_cli／codex_cli 派工時須帶 `effort="high"`（需 agent-hub 含 `effort` 參數的版本）。
