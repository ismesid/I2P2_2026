# 第 2 週 Lab — Arrays、Strings 與 Interface Contracts

## 學習成果

學生能實作 array/length interfaces、區分 capacity 與
logical length、驗證 prefix 與 sorted-boundary queries，並處理
null-terminated strings，而不假設它們具有 Python 式的 bounds 或 resizing。

## Part A — 不使用 AI 的準備練習

針對空、singleton 與 full-capacity inputs，追蹤提供的 array function。
找出一處 off-by-one access，並在保留 function contract 的前提下修正。

## Part B — Array 與 string functions

實作一個 numeric array operation 與一個 bounded string operation。針對
每個 interface，說明 pointer/address parameters、logical length、capacity、
mutability、return/failure behavior 與 null-termination requirements。

## Part C — Prefix-table 設計與驗證

在進行整體驗證前，先為
教師提供的 integer sequence 建立 boundary-indexed prefix table。訂出至少四個有效的 half-open
queries，包含一個 empty range，以及三組無效的 boundary pairs。先寫出預期答案，
才實作已公布的 interfaces，再將每個
有效結果與直接執行的 range loop 比較。使用較寬的 accumulation type，並說明
仍然需要的 overflow assumption。

## Part D — Sorted-boundary 設計追蹤

針對教師提供、含有 duplicates 的 sorted array，訂出 lower
與 upper boundary contracts，並針對存在及不存在的 targets，追蹤 half-open candidate interval。
納入空、全部相同、低於最小值，以及
高於最大值的情況。在 invariant 與
預期 boundary table 完成審查前，先不要實作 search。

## Part E — 一般 adversarial 驗證

測試 zero length、單一 element、exact capacity、insufficient capacity、適用時的內嵌
whitespace，以及 invalid input。啟用 warnings 編譯，並在 sanitizers 下執行
與 memory 相關的案例。

## Part F — AI 輔助測試審查

寫出預期結果後，請 AI 找出缺少的 boundary 類別。拒絕
假設了所述 interface 範圍以外操作的測試，並獨立驗證保留的
案例。

## 繳交內容

- source 與 interface-contract 表；
- 附註解的 prefix table 與 direct-loop 比較；
- lower/upper boundary 表與 binary-search invariant 追蹤；
- boundary-test 表；
- warning/sanitizer 證據；
- 一項接受及一項拒絕的 AI 建議。
