# 第 5 週 Lab — Token Lists 與 Adversarial Tests

本次週四課程也包含 **Quiz 1**，安排在
Midterm 1 的兩週前。教學團隊將公告哪些 lab 部分使用剩餘的實體課程時間，
以及是否有任何部分採非同步方式完成。

## 學習成果

學生能追蹤 token-list invariants、驗證 list-to-array conversion，並
依據 invariants 設計 indexed sequence 的編輯操作與測試，而不依賴
完整的 reference implementation。

## Part A — 不使用 AI 的準備練習

實作或修正一個獨立的 pointer-to-pointer list operation。在 AddressSanitizer 下測試空、
head、middle、tail 與 absent-element 情況。

## Part B — Sequence-editor 設計表

使用教師提供的 playlist specification 與 integer payloads，畫出
並訂出 insert-before、remove-at、remove-if 與
reverse-range operations，但先不要完整實作。使用 zero-based positions 與 half-open `[first, last)`
ranges。針對每個 operation，找出會改變的 incoming link、
unchanged-on-failure rule，以及空／front／end／adjacent-match 情況。比較
head-pointer 與 sentinel representations，並避免混用它們的 invariants。

## Part C — Token representation 追蹤

使用未修改的 project scaffold，追蹤：

- 空或只有 whitespace 的 input；
- 一個 identifier 或 constant；
- 含有多個 operator 的 expression；
- parentheses；
- 一個 invalid character。

記錄每次 append 後的 list state，以及產生的 indexed token sequence。
說明 conversion 前後，由哪個 representation 擁有 token storage。

## Part D — 測試設計

建立包含 input、預期 tokens 或 rejection、目標規則，
以及觀察結果的表格。納入一般、boundary、invalid 與 cleanup 相關
案例。保留這些測試，供第 7 週非同步 integration checkpoint，
以及第 10 週 demo 前的驗證工作使用。

## Part E — AI 輔助 adversarial 審查

請 AI 工具找出 lexer/list 測試缺少的*類別*。將有用的
建議轉換成精確的預期 sequences。拒絕超出
已公布 language 或 scaffold contract 的建議，並記錄其中一項拒絕理由。

## 繳交內容

- token/list 手動追蹤；
- sequence-editor 圖與 contract 表；
- 至少八個涵蓋必要類別的具體測試；
- sanitizer 結果；
- 包含一項被拒絕建議的 AI 使用記錄。
