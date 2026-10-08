# 第 3 週 Lab — Midterm Scaffold 建置與 Code Map

## 學習成果

完成本次 lab 後，學生能建置提供的 C project、重現
baseline 執行、找出各個 compiler stage，並說明未完成
functions 的 contracts，而不實作它們。

## Part A — 不使用 AI 的準備練習

針對提供的雙檔案 C 程式及 header，修正一項 declaration/definition mismatch
與一項 link error。記錄各個錯誤屬於 preprocessing、
compilation，還是 linking。

## Part B — 重現 baseline

1. 取得已公布的 midterm scaffold，並記錄其 revision。
2. 使用課程指定的 C standard 與 warning flags 建置。
3. 在不修改 source 的情況下，執行每個 public testcase。
4. 記錄完整 commands、compiler version、預期行為，以及觀察到的
   行為。只要清楚記錄，失敗或尚未完成的 baseline 也可以接受。

## Part C — Pipeline 與 TODO map

追蹤一個 expression 經過以下流程：

```text
input → tokens/list → parser → AST → semantic check → instructions → cleanup
```

針對每個 stage，記錄其 input、output、failure signal、allocation behavior，
以及 caller。針對每個 TODO，寫出 precondition 與 postcondition。在這個 milestone 中，
先不要撰寫實作。

## Part D — AI 輔助程式碼閱讀

請 AI 工具僅根據相關的 declaration、
definition 與 call site 解釋一個 function。提問前，先預測該 function 的 contract。
對照實際執行的 trace 驗證回覆，並記錄一項缺乏依據或
錯誤的假設。

## 繳交內容

- 可重現的 build 記錄；
- 一頁的 pipeline/TODO map；
- baseline public-test 表；
- `AI_USAGE.md` 中的第一筆記錄。

使用 [`project_templates/`](../../project_templates/) 中的範本。
