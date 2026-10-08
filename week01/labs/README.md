# 第 1 週 Lab — Compiler 與 Python-to-C 轉換

## 學習成果

學生能編譯並執行小型 C 程式、解讀 diagnostics，並在考量
static types 與
undefined behavior 的情況下，將熟悉的 Python control flow 轉換為 C。

## Part A — 環境與不使用 AI 的準備練習

記錄 compiler 及其版本。使用課程指定的
standard、warnings、debug information 與 sanitizers 編譯提供的程式。使用 AI 前，先閱讀 diagnostic，
修正一個 syntax error 與一個 warning。

## Part B — 對照轉換

將小型 Python loop/condition 程式轉換為 C。執行前，先預測
integer division、conversions 與一個 boundary input 的結果。使用
formatted input 時，必須檢查 conversion counts。

## Part C — 驗證

測試一般、零、負數與 invalid-input 情況。在
sanitizers 下執行有效案例，並記錄完整 build/test commands，供後續 labs 重複使用。

## Part D — AI 輔助審查

請 AI 審查 C 轉換結果，找出一項與 Python 的 semantic difference，以及
一項缺少的測試。透過編譯或查閱已公布的
language/tool contract 驗證這兩項說法；記錄任何缺乏依據的說法。

## 繳交內容

- source 與可重現的 command 記錄；
- 預測／觀察對照表；
- 無 warning 與無 sanitizer 錯誤的結果；
- 簡短的 AI 審查筆記。
