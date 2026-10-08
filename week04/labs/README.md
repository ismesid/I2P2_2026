# 第 4 週 Lab — Midterm Ownership 與 Sanitizers

## 學習成果

學生能在 compiler scaffold 中區分 owners 與 borrowers、追蹤
成功及失敗時的 cleanup，並使用 diagnostics 修正 memory defect。

## Part A — 不使用 AI 的準備練習

針對提供的 allocation function，畫出 automatic-duration 與 dynamically allocated objects。
標示 owner、一個 borrower、lifetime 結束點，以及一處
dangling use。可以用 stack/heap 圖作為 implementation model，但
說明必須採用 C lifetime rules。接著在不改變
public contract 的前提下修正該 function。

## Part B — Project ownership 表

針對 tokens、token containers、AST nodes 與 temporary storage，記錄：

| Resource | 建立者 | Owner | Borrowers | 成功時的 cleanup | 失敗時的 cleanup |
|----------|---------|-------|-----------|-----------------|-----------------|

至少追蹤一個 valid input、一個部分建構後出現的 syntax error，以及
一個 semantic rejection。每次成功的 allocation，都必須在每條 path 上
有一次可到達的 release。

## Part C — Diagnostic 練習

使用 warnings 與 address/undefined-behavior
sanitizers 執行提供的預植錯誤程式。針對每份報告：

1. 找出 invalid operation；
2. 找出 allocation 或 lifetime 起點；
3. 說明違反的 ownership contract；
4. 進行保留 contract 的最小修正；
5. 加入 regression case，並重新執行提供的完整案例集。

在預植錯誤練習中，不要修改計分 project 的 TODO。

## Part D — AI 審核

將 diagnostic 與最小相關程式碼區段提供給 AI 工具。要求它
提出三個假說，而不是提供替代 function。測試這些假說，並
記錄至少一個假說為何錯誤、不完整或不適用。

## 繳交內容

- 完成的 ownership 表；
- 修正前後的 diagnostic log；
- regression test；
- AI 審核記錄。
