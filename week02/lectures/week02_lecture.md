# 第 2 週課堂講義 — C 的 function、array 與 string

> 2026 年 9 月 15 日 · 來源沿革：先前的 function、array、string 與
> input 筆記，重新以和 Python sequence 的比較為主軸編排

> Python 銜接：[第 2 週 Python 對照補充教材](week02_python_companion.md)

---

## 學習路線

- **核心：** 撰寫具有 type 的 function，以明確的 length 走訪 array，
  建立／查詢以 boundary 為 index 的 prefix table，追蹤 lower／upper bound，並確保
  C string 維持在目的地 capacity 內。
- **練習：** 完成[第 2 週練習](lecture_exercises/week02_ex.md)，
  並在開啟[完整範例](examples.c)之前使用其 test driver。
  prefix 的實作仍未公開；範例示範與其相關的
  array、boundary search 與有界 string 技巧。
- **輔助觀念：** Big-O 詞彙與 overflow contract 可解釋設計
  選擇；先讓一般的 loop 或 query 對指定 input 正確運作。
- **Python 銜接：** 使用補充教材比較 sequence，
  而非當成第二堂必修課來閱讀。

---

## 學習目標

完成本堂課後，你應能：

1. 透過 prototype 宣告、定義並呼叫 C function。
2. 說明 pass-by-value，並以 return value 明確提供結果。
3. 閱讀使用 `&`、`*` 與 pointer parameter 的簡單 address-passing interface。
4. 走訪 array，且不讀取其 bounds 之外的位置。
5. 建立 prefix table，並用它回答 half-open range query。
6. 指定 sorted data 中的 lower 與 upper boundary，並追蹤其 binary
   search invariant。
7. 說明 C string 的 null-terminated 表示方式。
8. 設計將 array 與其 length 或 capacity 一併傳入的 interface。

---

## 三小時課程規劃

| 小時 | 核心問題 | 課堂產出 |
|------|---------------|---------------------|
| 1 | 具有 type 的 function 如何拆解程式？ | 指定並實作一小組 function |
| 2 | precomputation 如何取代重複的 query 工作？ | 追蹤 prefix 與 sorted-boundary query |
| 3 | null-terminated string 如何維持在 buffer 內？ | 建立並測試有界 string 工具 |

每小時交錯安排約 35–45 分鐘的講解與現場 coding，搭配
約 15–20 分鐘的核心練習。其餘時間用於討論、
轉場與短暫休息；若全班已準備好，也可利用這段彈性時間
進行選做練習。

### 隨堂練習流程

每個**立即練習**停點都要求你回想並應用剛剛
介紹的觀念。除非題目只要求追蹤或撰寫 contract，否則請使用一個小型的
練習 source file：

1. 在編譯前預測結果、state 變化或 diagnostic；
2. 自行完成要求的修改；
3. 使用 `-std=c17 -Wall -Wextra -Wpedantic` 編譯 C code；
4. 執行指定的一般與 boundary case；以及
5. 說明哪個 contract 或 invariant 能支持此結果。

一開始只會顯示題目。自行嘗試並測試後，再開啟**展開解答**。
解答區會提供預期輸出、追蹤過程，
或說明為何只有 declaration 的範例沒有 run-time output。它不會
公開另一份練習所使用的 prefix-table 實作。

- **課堂核心：** 屬於規劃中的課堂學習路線。
- **延伸：** 可在 lab、休息時間或日後複習時進行的額外練習。

課堂核心練習在第 1 小時合計約 16 分鐘，第 2 小時約 18 分鐘，
第 3 小時約 19 分鐘。

---

## 第 1 小時 — Function contract 與拆解

> **第 1 小時路線：** [Function 是具有 type 的 contract](#1-function-是具有-type-的-contract)
> → [C 以 value 傳遞 argument](#2-c-以-value-傳遞-argument)
> → [Address-passing 銜接](#address-passing-銜接)
> → [Coding 前先拆解](#coding-前先拆解)
> → [Scope、storage duration 與 `static` local](#scopestorage-duration-與-static-local)
> → [contract 檢核](#立即練習-課堂核心--第-1-小時-contract-檢核4-分鐘)

### 1. Function 是具有 type 的 contract

Python 在程式執行時檢查 function call。C compiler 在
產生 call 之前會檢查 prototype。

```c
int clamp_value(int value, int low, int high);
```

這個 declaration 承諾：

- function 的名稱為 `clamp_value`；
- 它接收三個 `int` value；
- 它回傳一個 `int`；以及
- caller 可以在完整 definition 出現之前使用 prototype。

definition 提供實作：

```c
int clamp_value(int value, int low, int high) {
  if (value < low) {
    return low;
  }
  if (value > high) {
    return high;
  }
  return value;
}
```

precondition 是 `low <= high`。postcondition 是結果位於
從 `low` 到 `high` 的 closed interval 內：低於 interval 的 value 變成
`low`，高於 interval 的 value 變成 `high`，原本位於其中的 value 則保持不變。

保持 declaration 與 definition 一致。放在 header 中的 prototype
能讓多個 source file 共用同一份 contract。

#### 立即練習 [課堂核心] — 呼叫 contract（3 分鐘）

呼叫 `clamp_value`，分別傳入 `-3`、`7` 與 `20`，並使用 interval `[0, 10]`。先預測
三個結果，再將它們印在同一行。

<details>
<summary>展開解答</summary>

```c
#include <stdio.h>

int main(void) {
  printf("%d %d %d\n", clamp_value(-3, 0, 10),
         clamp_value(7, 0, 10), clamp_value(20, 0, 10));
  return 0;
}
```

這三次 call 分別涵蓋低於範圍、位於範圍內，以及高於範圍的情況。

**預期輸出：**

```text
0 7 10
```

prototype 本身不會產生 run-time output；它只提供 compiler
function 的名稱、parameter type 與結果 type。

</details>

---

### 2. C 以 value 傳遞 argument

每個 parameter 一開始都是對應 argument 的 copy。
return type `void` 表示 function 不提供結果 value。在
parameter list 中，例如 `main(void)`，`void` 表示 function 不接收
任何 argument。同一個 keyword 在這裡具有兩種用途。

```c
void ineffective_swap(int a, int b) {
  int temporary = a;
  a = b;
  b = temporary;
}
```

呼叫 `ineffective_swap(x, y)` 不會修改 `x` 或 `y`。稍後若需要 mutation，會傳入
它們的 address。目前則優先使用 return value 傳回結果：

```c
int absolute_value(int value) {
  if (value < 0) {
    return -value;
  }
  return value;
}
```

precondition：`value != INT_MIN`，因為 `-INT_MIN` 可能造成 overflow。interface
應透過名稱、文件或檢查，明確呈現重要的 precondition。

#### 立即練習 [課堂核心] — 區分 caller state 與 parameter copy（2 分鐘）

從 `x = 3` 與 `y = 8` 開始。呼叫 `ineffective_swap(x, y)`，再印出這兩個
variable 與 `absolute_value(-7)`。在編譯前預測 output。

<details>
<summary>展開解答</summary>

```c
int main(void) {
  int x = 3;
  int y = 8;
  ineffective_swap(x, y);
  printf("x=%d y=%d absolute=%d\n", x, y, absolute_value(-7));
  return 0;
}
```

這段程式應放在引入 `<stdio.h>` 並包含上述兩個
function definition 的檔案中。對 `a` 與 `b` 的 assignment 只會改變它們的 local
copy。

**預期輸出：**

```text
x=3 y=8 absolute=7
```

</details>

---

### Address-passing 銜接

一些常見的 C interface 必須在完整的 pointer 課程之前先介紹。目前先從操作角度閱讀
這三個符號：

```c
int value = 10;
int* address = &value; /* address points to value */
*address = 20;         /* write through the address */
```

- 在 declaration 中，`int* address` 表示「一個 `int` 的 address」。
- 在 expression 中，`&value` 取得 `value` 的 address。
- 在 expression 中，`*address` 指定所指向的 `int`。

這些知識已足以修正 swap contract：

```c
void swap(int* left, int* right) {
  int temporary = *left;
  *left = *right;
  *right = temporary;
}

void example(void) {
  int x = 1;
  int y = 2;
  swap(&x, &y);
}
```

兩個 pointer 都是 **borrowed**：`swap` 暫時取得 caller 的
object 存取權，但不擁有其 storage，也不會在 return 後保留 address。
在整個 call 期間，它們都必須指定有效的 `int` object。第 4 週課堂
筆記會建立完整模型：pointer arithmetic、nullability、array
關係、lifetime、dynamic allocation 與 ownership。在此之前，請勿
推論每個 address 都能被 dereference 或保留。

#### 立即練習 [課堂核心] — 追蹤透過 address 進行的更新（3 分鐘）

加入 `printf` call，放在 `swap(&x, &y)` 之後，位置在 `example` 中；或執行相同 call，
`main` 中執行相同 call。先預測 `x` 與 `y` 的 value，再指出究竟哪兩個
assignment 會修改 caller 的 object。

<details>
<summary>展開解答</summary>

```c
int main(void) {
  int x = 1;
  int y = 2;
  swap(&x, &y);
  printf("x=%d y=%d\n", x, y);
  return 0;
}
```

assignment `*left = *right` 與 `*right = temporary` 透過
borrowed address 進行寫入。這段程式需要 `<stdio.h>` 與上方的 `swap` definition
才能使用。

**預期輸出：**

```text
x=2 y=1
```

</details>

---

### Coding 前先拆解

先前的 function 筆記分階段建立程式。面對一題要求
讀取分數、移除一個最低分，並回報四捨五入平均值的 judge 題目時，先
撰寫 contract，再撰寫長篇 `main`：

下列 declaration 預先展示第 2 小時會使用的一種表示法：array parameter
例如 `int scores[]`，指定一個 sequence，其 element count 必須透過
另一個 parameter 傳入。這裡先從操作角度理解其用途；contiguous array
的表示方式與其調整為 pointer 的規則，會在本筆記後續任何 array
實作之前說明。`const` 用在 `const int scores[]` 中，記錄了
function 只觀察這些 element 而不修改它們；第 2 小時會在情境中
進一步說明這項承諾。

```c
int read_scores(int scores[], size_t capacity, size_t* count);
size_t index_of_minimum(const int scores[], size_t count);
void remove_at(int scores[], size_t* count, size_t index);
double mean(const int scores[], size_t count);
```

對每個 function，請說明：

- 有效 input 與 array bounds；
- 哪些 object 可以改變；
- 如何回報 failure；
- 結果的有效範圍；
- 是否允許 empty input。

C 藉由這些明確規範，取代許多隱藏在簡短
Python expression 裡的 run-time 假設。

#### 立即練習 [課堂核心] — 標註四個 interface（4 分鐘）

對每個 prototype，將每個 parameter 標註為 input、output 或 input/output。
說明 empty-input policy 與 failure channel。暫時不要撰寫 function
body。

<details>
<summary>展開解答</summary>

一組相互一致的 contract 如下：

| Function | Parameter 用途 | Empty input 與 failure policy |
|----------|-----------------|--------------------------------|
| `read_scores` | `scores` 是 output storage；`capacity` 是 input；`count` 是 output | Empty input 可在 `*count == 0` 時成功；input 格式錯誤或 capacity 不足時回傳零 |
| `index_of_minimum` | `scores` 與 `count` 是 input | 要求 `count > 0`；沒有獨立的 failure result |
| `remove_at` | `scores` 與 `count` 是 input/output；`index` 是 input | 要求 `index < *count`；prototype 沒有提供 failure result |
| `mean` | `scores` 與 `count` 是 input | 這份 contract 將 empty mean 定義為 `0.0`；第 2 小時的實作遵循這項 policy |

這些 declaration 不會產生 run-time output。此練習的重點是在
實作之前先明確寫出 contract。在正式設計中，若需要回報
invalid input，可以將 `void` 或 index 結果改成 `int`。

</details>

---

### Scope、storage duration 與 `static` local

> **輔助 C 特性：** local variable 通常只存在於一次
> function call 期間。閱讀本節，以辨識較少見的情況：
> local name 參照持續整個程式期間的 storage；一般的 local
> variable 仍是本課程的預設選擇。

**Scope** 決定名稱可以在哪裡使用。**Storage duration** 決定
具名 object 存在多久。具有 automatic
storage duration 的一般 block-local object，會在每次執行進入其 block 時存在，並在
執行離開該 block 時結束存在。相較之下，`static` local 存在於
整個程式執行期間，且在不同 call 之間保留其 value：

```c
unsigned int next_sequence(void) {
  static unsigned int value = 0;
  return ++value;
}
```

這種隱藏的 state 有時有用，但每個 caller 都會共用它，使測試
取決於順序。若 state 屬於 function contract 的一部分，優先
透過 parameter 明確傳遞；第 3 週會介紹用於整合
多個相關 state value 的 structure。

#### 立即練習 [延伸] — 呈現持續存在的 local state（2 分鐘）

呼叫 `next_sequence` 三次，放在同一個 `printf` statement 中。接著改寫
測試，改成三次獨立 call。為何第二種形式更容易推理
output？

<details>
<summary>展開解答</summary>

不要依賴 function-call argument 的 evaluation order，來將
三個 return value 對應到三個文字位置。請在
不同 statement 中儲存結果：

```c
unsigned int first = next_sequence();
unsigned int second = next_sequence();
unsigned int third = next_sequence();
printf("%u %u %u\n", first, second, third);
```

**新啟動的程式中，預期輸出：**

```text
1 2 3
```

明確的 sequence 也能清楚說明：同一個
process 中較早的 call 會改變這些數字。

</details>

---

### 立即練習 [課堂核心] — 第 1 小時 contract 檢核（4 分鐘）

為一個在 integer array 中尋找 target 的 function 撰寫 prototype 與五行 contract。
比較三種結果設計：回傳帶有 sentinel 的 index、
回傳 success 並搭配 output parameter，或回傳指向 element 的 pointer。
第三種設計會在第 4 週課堂筆記之後完整分析。

<details>
<summary>展開解答</summary>

三種可能的 interface 如下：

```c
size_t find_index_or_count(const int values[], size_t count, int target);
int find_index(const int values[], size_t count, int target, size_t* result);
const int* find_element(const int values[], size_t count, int target);
```

- 第一種可回傳 `count`，作為 past-the-end 的「找不到」sentinel。
- 第二種另外回傳 success，並且只有成功時才寫入 index。
- 第三種可回傳指向 element 的 pointer 或 `NULL`，但其 validity 與
  lifetime 需要第 4 週的 pointer 模型。`NULL` 是 C 慣用的表示法，
  表示 pointer 未指定任何 object；第 3 小時會透過 `fgets` 介紹它。

三種設計都要求具有 `count` 個 element 的有效可讀範圍，保留
array，並在存在重複值時回傳第一個 match。這些都是 declaration，
所以不會產生 run-time output。

</details>

---

## 第 2 小時 — Array layout、prefix query 與 boundary algorithm

> **第 2 小時路線：** [Array 是 contiguous fixed-size storage](#3-array-是-contiguous-fixed-size-storage)
> → [Boundary 推理](#4-boundary-推理)
> → [one-pass minimum](#示範範例one-pass-minimum)
> → [Prefix table：預先計算重複的 range query](#5-prefix-table預先計算重複的-range-query)
> → [實作之前先撰寫 build／query contract](#實作之前先撰寫-buildquery-contract)
> → [衍生 contribution 的 prefix](#衍生-contribution-的-prefix)
> → [prefix-table 檢核](#立即練習-延伸--prefix-table-檢核4-分鐘)
> → [Sorted data 中的 lower 與 upper boundary](#6-sorted-data-中的-lower-與-upper-boundary)
> → [以 monotone predicate 理解 binary search](#以-monotone-predicate-理解-binary-search)
> → [Sorting 是 precondition，並非 search 的一部分](#sorting-是-precondition並非-search-的一部分)
> → [boundary-search 檢核](#立即練習-延伸--boundary-search-檢核4-分鐘)

> **Algorithm 應用：** prefix table 與 boundary search 培養 array
> invariant 與 indexing 的嚴謹習慣。它們是解題技巧，並非
> 額外的 C syntax；在記憶任一 loop 之前，先追蹤 contract。

### 3. Array 是 contiguous fixed-size storage

```c
int scores[5] = {91, 82, 73, 94, 85};
```

array 包含五個相鄰的 `int` object，index 從 `0` 到 `4`。
與 Python list 不同，它不會記住 run-time length，也無法增長。

在與 array declaration 相同的 scope 中：

```c
size_t count = sizeof(scores) / sizeof(scores[0]);
```

如第 1 週所述，`size_t` 是用於 object size 與
array index 的標準 unsigned type；`<stddef.h>` 提供其 declaration。`sizeof(scores)` 是
以 byte 計算的總 storage，而 `sizeof(scores[0])` 是單一 element 的 storage，
所以兩者的商就是 element count。

這個 expression 在 function parameter 中**無法**正確運作。在多數 expression 中，
array 會轉換成指向第一個 element 的 pointer。因此，每個通用的
array function 都必須明確接收 length。

qualifier `const` 建立一條 read-only access path。請將
`const int values[]` 理解為「由 `int` element 組成的 array，此 function 承諾
不會透過 `values` 修改它們」。compiler 會拒絕 `values[0] = 7` 出現在
這類 function 內。它不會使 caller 的 array 永久 immutable：
caller 或其他非 `const` 的 access path 仍可修改它。第 4 週會說明
對應的 pointer type；目前請使用 `const`，只要 array
parameter 僅用於 input。

```c
double mean(const int values[], size_t count) {
  double total = 0.0;
  for (size_t i = 0; i < count; ++i) {
    total += values[i];
  }
  if (count == 0) {
    return 0.0;
  }
  return total / count;
}
```

使用 `double` 保存 running total 可避免 signed-integer overflow，但
floating-point addition 可能有 rounding。若需要精確的 integer accumulation，
contract 就必須限制 input，或在適當的
integer type 中檢查 arithmetic。此實作刻意將 empty
range 的 mean 定義為 `0.0`；其他應用也可選擇拒絕它。

```c
int maximum(const int values[], size_t count, int* result);
```

return value 可回報 maximum 是否存在；`result` 可儲存
答案。我們會在 pointer 課程中完整介紹這種 output-parameter 風格。

#### 立即練習 [課堂核心] — 區分 capacity 與 element count（2 分鐘）

印出 `scores` 的 element count，使用 `%zu`，並將 mean 印到
小數點後一位。接著想像將 `scores` 傳入 `maximum`：哪個 value 必須
與 array 一併傳入？為何 function 無法透過 `sizeof` 取得它？

<details>
<summary>展開解答</summary>

```c
int main(void) {
  int scores[5] = {91, 82, 73, 94, 85};
  size_t count = sizeof(scores) / sizeof(scores[0]);
  printf("count=%zu mean=%.1f\n", count, mean(scores, count));
  return 0;
}
```

這段程式需要 `<stddef.h>` 與 `<stdio.h>`。呼叫 `maximum` 必須
明確傳入 `count`。在 function parameter 中，array 表示法會
調整為 pointer type，所以 `sizeof(values)` 測量的是該 pointer，並非
caller 的 array。

**預期輸出：**

```text
count=5 mean=85.0
```

`maximum` prototype 本身是 declaration，不會產生 output。

</details>

---

### 4. Boundary 推理

對於 `count` 個有效 element，典型的走訪方式如下列完整
function 所示：

```c
int sum_array(const int values[], size_t count) {
  int total = 0;

  for (size_t i = 0; i < count; ++i) {
    total += values[i];
  }
  return total;
}
```

此 function 要求數學上的 sum 必須能以 `int` 表示。
後續的 integer-arithmetic 練習會為更廣的
input domain 提供具有檢查的替代方案。

對每個 loop，都要問三個問題：

1. 第一個有效 index 是什麼？
2. 第一個無效 index 是什麼？
3. loop condition 是否排除了第一個無效 index？

存取 `values[count]` 是 undefined behavior。C 沒有自動的 bounds
check，也沒有 `IndexError`。

#### 立即練習 [課堂核心] — 守住第一個無效 boundary（3 分鐘）

呼叫 `sum_array`，傳入 `{4, -1, 3}`。預測結果，再說明為何將
`i < count` 改成 `i <= count` 無法正確納入最後一個 element。
執行程式前，先修正 condition。

<details>
<summary>展開解答</summary>

```c
int main(void) {
  int values[] = {4, -1, 3};
  printf("sum=%d\n", sum_array(values, 3));
  return 0;
}
```

有效 index 是 `0`、`1` 與 `2`；condition `i < 3` 會走訪三者。
若使用 `i <= 3`，最後一次 iteration 會求值 array 之外的 `values[3]`，
而程式會出現 undefined behavior。不要執行這個有缺陷的版本。

**正確程式的預期輸出：**

```text
sum=6
```

</details>

---

### 示範範例：one-pass minimum

```c
#include <stddef.h>

int minimum(const int values[], size_t count) {
  int result = values[0];
  for (size_t i = 1; i < count; ++i) {
    if (values[i] < result) {
      result = values[i];
    }
  }
  return result;
}
```

precondition 是 `count > 0`；caller 必須在 call 之前確保它成立。
每次 iteration 開始時，`result` 是已處理
half-open range `[0, i)` 的 minimum。下一次比較將此主張擴展到 `[0, i + 1)`。
這是 **loop invariant** 的例子：在每次 iteration 前後都成立的
敘述，能解釋為何最終答案正確。

正式使用的 interface 可能需要表示 empty result。第 3 週會介紹
可整合 status 與 data 的 structure，第 4 週則會說明 output-pointer
interface。本週的版本將重點放在 array bounds、function
precondition 與走訪的證明。

#### 立即練習 [課堂核心] — 陳述並使用 loop invariant（3 分鐘）

追蹤 `minimum`，使用 `{8, -4, 6, -4}`。在每次 iteration 前，記錄已處理的
range 與 `result`，再於小型 `main` 中測試 function。不要以
empty array 呼叫它，因為這會違反其指定的 precondition。

<details>
<summary>展開解答</summary>

| Iteration 前 | 已處理 range | `result` |
|------------------|-----------------|----------|
| `i == 1` | `[0, 1)` → `{8}` | `8` |
| `i == 2` | `[0, 2)` → `{8, -4}` | `-4` |
| `i == 3` | `[0, 3)` → `{8, -4, 6}` | `-4` |
| loop 之後 | `[0, 4)` → 所有 element | `-4` |

```c
int main(void) {
  int values[] = {8, -4, 6, -4};
  printf("minimum=%d\n", minimum(values, 4));
  return 0;
}
```

這段程式需要 `<stdio.h>` 與上方的 `minimum` definition。

**預期輸出：**

```text
minimum=-4
```

</details>

---

### 5. Prefix table：預先計算重複的 range query

此處的 **precomputation** 是指在 query 到達之前，先執行一次
algorithm 的準備程序。它與在 compilation 前展開
`#include` 與 macro 的 C preprocessor 無關。

#### Running time 的基本詞彙

比較實作之前，需要先有一種方式，描述其工作量如何
隨 input 成長。令 `n` 為 array element 的數量，`q` 為
query 的數量。**Big-O notation** 描述 growth rate 的 upper bound；它
不測量秒數，通常也會省略固定 multiplier 與較小的
項。

- **O(1)，constant time：** 相關 operation 的數量不會隨
  `n` 成長。讀取一個 array element，以及相減兩個 prefix total，都是例子。
- **O(n)，linear time：** element 數量加倍時，工作量大致也會
  加倍。一次完整的 array traversal 是 linear。
- **O(log n)，logarithmic time：** 每一步都會排除剩餘
  candidate 的固定比例。Binary search 就具有這種形式。
- **O(n log n)：** 許多 comparison-based sorting algorithm 具有這種 growth
  rate。

Big-O 只是設計限制之一。兩個 O(n) loop 可能具有不同的
constant、memory access pattern 與 failure behavior。在本課程中，先
證明 algorithm 正確，再利用 growth rate 判斷
它是否在 input limit 提高時仍然實用。

假設程式只接收一次 array，接著回答許多關於
contiguous range 的問題。每個 query 都重複執行 loop，所需時間會與
各 range 的 length 成正比。prefix table 儲存每個
boundary 之前的累積 total：

```text
values:  [ 3, -1,  4,  2 ]
boundary:  0   1   2   3   4
prefix:  [ 0,  3,  2,  6,  8 ]
```

invariant 如下：

```text
prefix[i] = values[0] + values[1] + ... + values[i - 1]
```

開頭額外的零是刻意安排的。它讓 `prefix` 具有 `count + 1`
個 element，並且不需 special case 就能表示 empty prefix。因此，
half-open range `[left, right)` 的 total 是：

```text
prefix[right] - prefix[left]
```

對上方 table 而言，`[1, 4)` 的 total 為 `8 - 3 = 5`。這符合 C 常用的 loop
boundary：從 `left` 開始，只要 `i < right` 就繼續。

#### 立即練習 [課堂核心] — 相減 boundary（4 分鐘）

使用所示 prefix table，計算 `[0, 2)`、`[2, 4)` 與 `[4, 4)`。
對每個結果，也列出此 half-open
range 包含的原始 array element。

<details>
<summary>展開解答</summary>

| Range | 包含的 value | Boundary 相減 | Total |
|-------|-----------------|----------------------|-------|
| `[0, 2)` | `3, -1` | `prefix[2] - prefix[0] = 2 - 0` | `2` |
| `[2, 4)` | `4, 2` | `prefix[4] - prefix[2] = 8 - 2` | `6` |
| `[4, 4)` | 無 | `prefix[4] - prefix[4] = 8 - 8` | `0` |

這是追蹤過程而非可執行程式，所以沒有 standard
output。注意，empty range 不需要 special-case 公式。

</details>

---

### 實作之前先撰寫 build／query contract

設計兩個 interface，而不是將 precomputation 隱藏在 `main` 裡：

```c
#include <stddef.h>
#include <stdint.h>

int build_prefix(const int values[], size_t count, int64_t prefix[],
                 size_t prefix_capacity);

int query_total(const int64_t prefix[], size_t prefix_count, size_t left,
                size_t right, int64_t* result);
```

第一個要求能容納 `count + 1` 個累積 value 的空間。第二個必須
驗證 `left <= right` 與 `right < prefix_count`。兩者都應說明如何
避免或回報 arithmetic overflow；使用 `int64_t` 能涵蓋更多常見
情況，但不能從數學上證明所有可能的 input 都能容納。

對 `n` 個 value 與 `q` 個 query 而言，precomputation 搭配 constant-time query 的成本為
O(n + q)，而每個 query 都掃描其 range 時，worst case 為
O(nq)。代價是 O(n) 的額外 storage，以及在 input value 改變時，需要重建或
更新 table。

#### 立即練習 [延伸] — 檢查兩個 interface（3 分鐘）

對四個 input value，判定最小有效 `prefix_capacity`。成功
build 後，判定 `prefix_count` 傳給 `query_total` 時的 value，以及
`right` 的最大有效 value。說明每個 function 在
capacity 或 range check 失敗時應如何處理。

<details>
<summary>展開解答</summary>

- 四個 input value 需要五個 prefix entry，因此 `prefix_capacity` 必須
  至少為 `5`。
- 得到的 `prefix_count` 是 `5`。
- 由於 range contract 檢查 `right < prefix_count`，最大的有效
  right boundary 是 `4`。
- 失敗的 build 或 query 回傳零。失敗的 query 不可將 value
  透過 `result` 傳出。

這些只有 declaration 的 interface 不會產生 run-time output。prefix-table
function body 留作練習；上方的 contract 是
其實作的判定依據。

</details>

---

### 衍生 contribution 的 prefix

累積的 value 不一定要是原始 element。程式可以先
定義 contribution，例如 reading 滿足 condition 時為 `1`，
否則為 `0`，再對這些 contribution 建立 prefix，以計算任何
range 內符合條件的 element 數量。明確說明 transformation 與 range convention；改變
其中任一項，都會改變每個 query 的意義。

#### 立即練習 [課堂核心] — 為 predicate 建立 prefix（3 分鐘）

對 `{-2, 5, 0, 7}`，將每個 contribution 定義為：value 為正時是 `1`，
否則是 `0`。寫出以 boundary 為 index 的 contribution prefix，並用它
計算 `[1, 4)` 內正 value 的數量。

<details>
<summary>展開解答</summary>

```text
values:        [ -2, 5, 0, 7 ]
contribution:  [  0, 1, 0, 1 ]
prefix:        [ 0, 0, 1, 1, 2 ]
```

query 是 `prefix[4] - prefix[1] = 2 - 0 = 2`。這個手動追蹤過程沒有
standard output，並刻意不公開練習中的 C
實作。

</details>

---

### 立即練習 [延伸] — prefix-table 檢核（4 分鐘）

對 `values = {5, -2, 0, 7, -3}`，手動建立六個 boundary total。回答
`[0, 0)`、`[0, 3)`、`[2, 5)` 與 `[4, 5)`。接著指定三組
無效 boundary pair 預期會如何被拒絕。確定 table 與預期結果後，
再撰寫兩個 function body，並將它們的結果與直接執行 loop 比較。

<details>
<summary>展開解答</summary>

原始 value 的 boundary total 如下；這裡使用的是原始 value，
而非另一個練習中只計正 value 的 contribution：

```text
values:  [ 5, -2, 0, 7, -3 ]
prefix:  [ 0,  5, 3, 3, 10, 7 ]
```

| Range | 結果 |
|-------|--------|
| `[0, 0)` | `0` |
| `[0, 3)` | `3` |
| `[2, 5)` | `4` |
| `[4, 5)` | `-3` |

無效 boundary 的例子包括 `(3, 2)`，因為 `left > right`；`(0, 6)`，
因為 `right >= prefix_count`；以及 `(6, 6)`，因為兩個 boundary 都不屬於
`[0, prefix_count)`。本區提供的是預期結果，而非兩個 C
function body。這是手動追蹤過程，因此沒有 standard output。

</details>

---

### 6. Sorted data 中的 lower 與 upper boundary

在 ascending sorted array 中，相等的 value 形成一個 contiguous block 時，兩個
boundary query 可精確描述該 block：

- **lower bound：** 第一個 value 不小於 target 的位置；
- **upper bound：** 第一個 value 大於 target 的位置。

兩者都回傳 past-the-end 位置 `count`，只要沒有 element 滿足
condition。若 `lower` 與 `upper` 是這兩個結果，sorted range 就會
被分割為：

```text
[0, lower)       values < target
[lower, upper)   values equivalent to target
[upper, count)   values > target
```

對 target `-1`，下方範例 array 具體展示這些位置：

```text
index:    0   1   2   3   4   5   6
value:   -3  -1  -1  -1   2   5   5
             ^ lower = 1
                         ^ upper = 4
```

因此，`lower == upper` 表示 target 不存在，而 `upper - lower` 是
其相等 block 的大小。同樣的 boundary 也指出在保持順序下，value
可以插入的位置。

兩個 query 共用相同的 interface 形式，以下是練習
starter 使用的名稱：

```c
#include <stddef.h>

size_t lower_bound_int(const int values[], size_t size, int key);
size_t upper_bound_int(const int values[], size_t size, int key);
```

兩者都要求 `values` 包含 `size` 個以 ascending order 排列的可讀 element，
保持它們不變，並回傳 `[0, size]` 內的位置。回傳 `size`
表示沒有 element 滿足 condition。下一節將推導用於
實作兩者的單一 loop。

#### 立即練習 [課堂核心] — 辨識相等的 block（3 分鐘）

對所示 array，找出 lower bound、upper bound 與重複
數量，分別使用 target `-1` 與 `4`。目前先不要執行 binary search；只使用
三區域的定義。

<details>
<summary>展開解答</summary>

| Target | Lower bound | Upper bound | 數量 |
|--------|-------------|-------------|-------|
| `-1` | `1` | `4` | `3` |
| `4` | `5` | `5` | `0` |

對 `4` 而言，index `5` 同時是第一個 value 至少為 `4` 的位置，
也是第一個 value 大於 `4` 的位置。因此，相同的 boundary
描述一個 empty equal block。這個手動分類沒有 run-time output。

</details>

---

### 以 monotone predicate 理解 binary search

不要死記兩個幾乎相同的 loop。對 lower bound，尋找
`values[index] >= target` 首次成為 true 的 index。對 upper bound，
將 predicate 換成 `values[index] > target`。兩種情況都維持一個
由尚未分類 element 組成的 half-open interval `[low, high)`。boundary
是仍位於 `low` 到 `high` 之間的位置，包含兩端；若沒有
array element 使 predicate 成為 true，它可以等於 `count`。

設計的追蹤過程必須說明：

- `low` 之前的所有 element 已知都使 predicate 為 false；
- `[high, count)` 內每個實際 index 已知都使它為 true；若沒有實際
  element 為 true，`count` 就是 virtual true boundary，且永遠不會被存取；
- 每次比較都會從 candidate interval 排除 `mid`，或使它成為新的
  boundary，所以 interval 嚴格縮小；
- midpoint 以 `low + (high - low) / 2` 計算，避免
  `(low + high) / 2` 中的 addition overflow。

先只寫出 invariant 與 interval update。以 empty
array、單一 element、全部相等的 value、低於所有 value 的 target、高於
所有 value 的 target，以及兩端的 duplicate 測試追蹤過程。一般找到相等值就回傳的
binary search 不足以完成此任務，因為它可能找到任意 duplicate，而非
指定的 boundary。

#### 立即練習 [延伸] — 追蹤第一個 true（4 分鐘）

在所示 array 上追蹤 lower-bound predicate `values[index] >= -1`。
每次 update 前記錄 `(low, high, middle)`，再以 upper-bound
predicate `values[index] > -1` 重複一次。

<details>
<summary>展開解答</summary>

Lower bound：

| `low` | `high` | `middle` | Value | `value >= -1` | 更新 |
|-------|--------|----------|-------|---------------|--------|
| `0` | `7` | `3` | `-1` | true | `high = 3` |
| `0` | `3` | `1` | `-1` | true | `high = 1` |
| `0` | `1` | `0` | `-3` | false | `low = 1` |

interval 在 `[1, 1)` 時為空，因此 lower boundary 是 `1`。

Upper bound：

| `low` | `high` | `middle` | Value | `value > -1` | 更新 |
|-------|--------|----------|-------|--------------|--------|
| `0` | `7` | `3` | `-1` | false | `low = 4` |
| `4` | `7` | `5` | `5` | true | `high = 5` |
| `4` | `5` | `4` | `2` | true | `high = 4` |

interval 在 `[4, 4)` 時為空，因此 upper boundary 是 `4`。這些是
algorithm 的追蹤過程，而非程式 output。

</details>

---

### Sorting 是 precondition，並非 search 的一部分

Boundary search 要求 ascending sorted range。search function 應
明確陳述這個 precondition，而不是默默對 input 進行 sorting，因為 sorting
會修改順序並改變 operation 的 running time。目前請使用
已完成 sorting 的 data，或本筆記末尾的 insertion-sort 延伸
練習。第 4 週會介紹 C 的通用 `qsort` interface，時機是在 function pointer
與 comparator contract 都能妥善說明之後。

Sorting 一次，再回答 `q` 個 boundary query，成本為 O(n log n + q log n)。
每個 query 都掃描 unsorted array 的成本為 O(nq)，但能保留原始
順序，也不需要 sorting。應根據完整 workload 與 data contract 選擇，
而非只看 query operation 本身。

---

### 立即練習 [延伸] — boundary-search 檢核（4 分鐘）

對 `{-3, -1, -1, -1, 2, 5, 5}`，填寫 target
`-4`、`-1`、`0`、`5` 與 `8` 的 lower 與 upper position table。每次比較時，記錄 `[low, high)`
與相關 predicate 的 truth value。接著為兩種 search 撰寫 function contract，
暫時不要撰寫其 body。

<details>
<summary>展開解答</summary>

| Target | Lower bound | Upper bound | 相等 block 的 size |
|--------|-------------|-------------|------------------|
| `-4` | `0` | `0` | `0` |
| `-1` | `1` | `4` | `3` |
| `0` | `4` | `4` | `0` |
| `5` | `5` | `7` | `2` |
| `8` | `7` | `7` | `0` |

兩個 function 都要求包含 `count` 個可讀 element 的 ascending sorted range，
保持該 range 不變，並回傳 `[0, count]` 內的位置。lower-bound
結果是第一個 value 至少等於 target 的位置；upper-bound 結果是
第一個 value 大於 target 的位置。table 是預期的追蹤結果。請先在
練習 starter 中撰寫兩個 body；[完整範例](examples.c)
包含示範實作，可在完成後用來比較。

</details>

---

## 第 3 小時 — String 表示方式、有界 input 與 parsing

> **第 3 小時路線：** [String 是帶有 sentinel 的 character array](#7-string-是帶有-sentinel-的-character-array)
> → [Capacity 與 length](#capacity-與-length)
> → [安全讀取一行](#8-安全讀取一行)
> → [親手實作一次 library 觀念](#親手實作一次-library-觀念)
> → [處理 line 前先驗證其表示方式](#處理-line-前先驗證其表示方式)
> → [word-count 實作工坊](#立即練習-課堂核心--第-3-小時-word-count-實作工坊5-分鐘)

### 7. String 是帶有 sentinel 的 character array

```c
char language[] = "C17";
```

array 包含四個 character：`'C'`、`'1'`、`'7'`，以及作為結尾的
null character `'\0'`。library function 會掃描此
sentinel 來找到結尾。若缺少 terminator，string function 可能繼續讀取
array 之外的位置。

```c
#include <string.h>

size_t length = strlen(language); /* 3, not 4 */
```

`strlen` 是 linear time；它不知道 array capacity。

#### 立即練習 [課堂核心] — 分別計算 storage 與文字（2 分鐘）

印出 `sizeof(language)` 與 `strlen(language)`。預測為何這兩個
數字相差一。

<details>
<summary>展開解答</summary>

```c
printf("storage=%zu length=%zu\n", sizeof(language), strlen(language));
```

array 擁有四個 `char` object，但 string length 只計算第一個 null character
之前的三個 character。這段程式應放在
`language` 的 declaration 之後，且程式須引入 `<stdio.h>` 與
`<string.h>`。

**預期輸出：**

```text
storage=4 length=3
```

</details>

---

### Capacity 與 length

```c
char name[32] = "Ada";
```

- Capacity：可容納 32 個 character 的 storage。
- 目前的 string length：3 個 character。
- 額外文字可用的空間：28 個 character，因為有一個位置
  保留給 `\0`。

在每一種 sequence 表示方式中，capacity 與 logical length 都是不同的 property。
在此保持兩者分開，能讓我們準備好推理後續的 dynamic
array 與其他 container，而不依賴任何單一語言的 API。

#### 立即練習 [課堂核心] — 預留 terminator（2 分鐘）

將 declaration 改成 `char name[4] = "Ada";`。判定其 capacity、
length，以及額外可見 character 的剩餘空間。接著預測
它能否容納再 append 的一個可見 character。

<details>
<summary>展開解答</summary>

capacity 是 `4`，目前 length 是 `3`，可見文字的剩餘
capacity 是 `0`。第四個 element 已經儲存 `\0`，所以再加入一個可見
character，至少需要可容納五個 element 的 destination。

```c
printf("capacity=%zu length=%zu available=%zu\n", sizeof(name), strlen(name),
       sizeof(name) - strlen(name) - 1);
```

**預期輸出：**

```text
capacity=4 length=3 available=0
```

</details>

---

### 8. 安全讀取一行

第 1 週介紹了 numeric format contract。對 C string，`%s` 有兩種
不同但相關的用途：

- `printf("%s", text)` 從 `text` 開始讀取 character，持續印出直到
  第一個 `\0`；
- `scanf("%31s", word)` 跳過開頭的 whitespace，最多讀取 31 個 non-whitespace
  character，寫入結尾的 `\0`，並在 whitespace 處停止。

input field width 必須保留一個 array element 給 terminator。它是
format string 中寫出的 literal maximum，因此可容納 32 個 element 的 destination 搭配
`%31s`：

```c
#include <stdio.h>

int main(void) {
  char word[32];
  if (scanf("%31s", word) != 1) {
    return 1;
  }
  printf("word=%s\n", word);
  return 0;
}
```

#### 立即練習 [課堂核心] — 區分 word 與 line（3 分鐘）

以 input `Ada Lovelace` 執行程式。預測它會印出什麼，以及哪些
input 尚未讀取。再說明為何 width 能保護 destination，卻
無法將 `%s` 變成 whole-line parser。

<details>
<summary>展開解答</summary>

**預期 standard output：**

```text
word=Ada
```

分隔的 space 會結束 conversion，而 `Lovelace` 留在 `stdin` 中，等待
後續 input operation。超過 31 個 character 的 token 也只會部分
被讀取。width 能防止這次 call 寫到 `word` 之外；程式
仍須決定如何處理剩餘 input。

</details>

對完整的一行，優先採用有界 line read，再進行 parsing。這個初步版本
假設 input contract 保證整行能放進 array：

```c
#include <stdio.h>
#include <string.h>

int main(void) {
  char line[128];
  if (fgets(line, sizeof(line), stdin) == NULL) {
    return 1;
  }

  line[strcspn(line, "\n")] = '\0';
  printf("You entered %zu characters: \"%s\"\n", strlen(line), line);
  return 0;
}
```

`NULL` 是 C 慣用的 null-pointer constant：它表示 pointer 沒有
指定 object。此處，`fgets` 回傳 `NULL`，表示它無法讀取
一行。第 4 週會將 null pointer、pointer validity 與 dynamic
memory 一併說明；目前請在使用回傳的 pointer 之前，先將它與 `NULL` 比較。

成功時，`fgets` 會儲存結尾的 `\0`，因此 `strcspn` 可以安全地在
此 array 中尋找 newline。若存在 newline，將它換成 `\0` 就能移除
行尾。不過，若 input 超過 buffer 長度，`fgets` 只會讀取
一個 prefix。下方的驗證章節會示範如何區分 complete line
與遭到 truncation 的 line。

#### 立即練習 [課堂核心] — 觀察有界 line input（3 分鐘）

先以 `Ada Lovelace` 加上 Enter 執行一次程式，再以
empty line 執行一次。執行前先預測 character count 與 output。

<details>
<summary>展開解答</summary>

對 `Ada Lovelace` 後接 newline：

```text
You entered 12 characters: "Ada Lovelace"
```

對 empty line：

```text
You entered 0 characters: ""
```

newline 會被讀入 array 後再被替換，所以不包含在
任一回報的 length 中。若立刻遇到 end-of-file，程式不會產生
standard output，並回傳 nonzero status。

</details>

---

### 親手實作一次 library 觀念

在依賴 `<string.h>` 之前，先實作兩個小型 function，呈現
sentinel 與 capacity contract：

```c
#include <stddef.h>

size_t string_length(const char text[]) {
  size_t length = 0;
  while (text[length] != '\0') {
    ++length;
  }
  return length;
}

int string_copy(char destination[], size_t capacity, const char source[]) {
  size_t length = string_length(source);
  if (length >= capacity) {
    return 0;
  }

  for (size_t i = 0; i <= length; ++i) {
    destination[i] = source[i]; /* includes '\0' */
  }
  return 1;
}
```

copy loop 刻意使用 `<= length`。成功的 string copy 必須同時複製
terminator 與可見 character。這個 function 遵循
**all-or-nothing** contract：若無法容納完整 source，就回報
failure 並保持 destination 不變。本週後續的練習
刻意探討另一種會進行 truncation 的 contract，以便比較這兩種 policy。
討論為何對沒有 terminator 的 array 呼叫任一 function，
會違反其 precondition。

兩個 function 都要求有效的 array，以及 null-terminated `source`。copy
contract 不支援部分重疊的 source 與 destination range，
因為 write 可能改變尚未讀取的 source character。

#### 立即練習 [課堂核心] — 測試恰好容納與拒絕情況（4 分鐘）

使用可容納四個 element 的 destination。先 copy `"C17"`，再嘗試將 `"C17!"`
copy 到相同 destination。預測每次 call 的 status，以及 call 後
destination 的文字。

<details>
<summary>展開解答</summary>

```c
int main(void) {
  char destination[4] = "old";
  int first_status = string_copy(destination, sizeof(destination), "C17");
  printf("status=%d text=%s\n", first_status, destination);

  int second_status = string_copy(destination, sizeof(destination), "C17!");
  printf("status=%d text=%s\n", second_status, destination);
  return 0;
}
```

這段程式需要 `<stdio.h>` 與上方的兩個 definition。`"C17"`
包含 `\0` 後，剛好需要四個 array element。第二個 source 需要五個，
所以 all-or-nothing check 會在改變 destination 之前失敗。

**預期輸出：**

```text
status=1 text=C17
status=0 text=C17
```

</details>

---

### 處理 line 前先驗證其表示方式

成功的 `fgets` call 不保證整個 logical line 都能放進
array。尋找 `\n`。若存在，就換成 `\0`；若
不存在，且程式尚未到達 end-of-file，就表示 input line 超過
buffer，必須明確選擇拒絕或丟棄剩餘部分。

這個驗證順序示範一項可重複運用的原則：

1. 確認有效 data 在哪裡結束；
2. 確認其表示方式完整；
3. 完成後才解讀其內容。

下列 helper 處理 array 剛好在 newline
之前填滿的 boundary case。它假設 `fgets` 已成功。若未儲存 newline，
它會再讀取一個 character：newline 或真正的 end-of-file 表示
line 恰好能容納，其他 character 則證明 line 已遭到 truncation。
遇到 truncation 時，它會丟棄該 logical line 的剩餘部分，讓下一次
read 從乾淨的 boundary 開始。

```c
#include <stdio.h>
#include <string.h>

int finish_bounded_line(char line[]) {
  size_t end = strcspn(line, "\n");
  if (line[end] == '\n') {
    line[end] = '\0';
    return 1;
  }

  int next = fgetc(stdin);
  if (next == '\n') {
    return 1;
  }
  if (next == EOF) {
    return feof(stdin) != 0;
  }

  while (next != '\n' && next != EOF) {
    next = fgetc(stdin);
  }
  return 0;
}
```

#### 立即練習 [延伸] — 區分恰好容納與 truncation（3 分鐘）

使用 `char line[8]`，追蹤 `fgets` 後接 `finish_bounded_line`，分別傳入
input `Ada\n`、`1234567\n` 與 `12345678\n`。記錄回傳的 status 與
儲存在 `line` 中的文字。

<details>
<summary>展開解答</summary>

| Input | `fgets` 最初儲存的文字 | Helper status | 意義 |
|-------|-----------------------------------|---------------|---------|
| `Ada\n` | `"Ada\n"` | `1` | Newline 已儲存，並被換成 `\0` |
| `1234567\n` | `"1234567"` | `1` | helper 讀取後續 newline；七個可見 character 恰好能容納 |
| `12345678\n` | `"1234567"` | `0` | 後續的 `8` 證明已發生 truncation；helper 會一路丟棄到 newline |

helper 本身不會產生 standard output。caller 只能在回傳 status
為 `1` 時使用儲存的 line。

</details>

安全地將 substring 轉換成 number，必須同時判定
numeric text 的結尾，以及其 value 是否可表示。第 7 週會在
lexer 中建立一個明確的解法：每次累積一個 digit，在 multiplication 前檢查
每一步是否位於可表示的範圍內，並在成功讀取
token 後，讓 scan position 停在第一個不屬於
number 的 character。在此之前，請遵循第 1 週 `scanf`
contract 指定的可表示性假設來讀取 numeric input。

---

### 立即練習 [課堂核心] — 第 3 小時 word-count 實作工坊（5 分鐘）

為 null-terminated character array 撰寫 `count_words`。word 由一個或多個
non-whitespace character 組成，任何連續 whitespace 都會分隔 word。對
例如 `inside_word` 的 Boolean state 進行追蹤，涵蓋 empty string、開頭／結尾的
space，以及重複 separator。重要的技巧是辨識
從「外部」到「內部」的 transition，而非記憶 library function。

<details>
<summary>展開解答</summary>

```c
#include <ctype.h>
#include <stddef.h>
#include <stdio.h>

size_t count_words(const char text[]) {
  size_t count = 0;
  int inside_word = 0;

  for (size_t i = 0; text[i] != '\0'; ++i) {
    int whitespace = isspace((unsigned char)text[i]) != 0;
    if (whitespace) {
      inside_word = 0;
    } else if (!inside_word) {
      ++count;
      inside_word = 1;
    }
  }
  return count;
}

int main(void) {
  printf("empty=%zu\n", count_words(""));
  printf("words=%zu\n", count_words("  C  arrays\tand strings "));
  return 0;
}
```

`count` 只在 outside-to-inside transition 時改變。cast 會提供
`isspace` 一個位於其要求的 `unsigned char` domain 內的 value，避免 undefined
behavior 因負的 plain `char` value 而發生。

**預期輸出：**

```text
empty=0
words=4
```

</details>

---

## 自我檢核

1. 為何 `sizeof(parameter) / sizeof(parameter[0])` 在 function 中會失敗？
2. 將 string `"tree"` 儲存為 `char` array，需要多少 byte？
3. 為何 `count` 個 value 的 prefix table 包含 `count + 1` 個 entry？
4. 將數學上包含兩端的範圍 `left` 到 `right` 表達為 C 風格的
   half-open range，並檢查 boundary conversion 的 overflow。
5. 陳述 lower 與 upper bound 定義的三個 sorted region。
6. 為何一般 binary search 對 duplicate 可能回傳錯誤位置？
7. 設計一個 reverse array 的 function。其 contract 必須包含什麼？
8. 找出 `for (i = 0; i <= count; ++i)` 中的 off-by-one error。
9. string-building function 除了目前 length，還應知道什麼？

---

## 重點整理

- Prototype 讓 compiler 能取得 function contract。
- Argument 以 value 傳遞；mutation 需要明確的 indirection。
- C array 是 contiguous，且沒有 run-time length metadata。
- Prefix precomputation 將重複的 range total 計算轉為 boundary 相減。
- Lower 與 upper bound 找出 sorted data 中相等 block 的邊界。
- C string 是一種 array convention：character 後接 `\0`。
- 每個 array 都搭配其 length，每個 output buffer 都搭配其 capacity。

---

## 選讀補充與 lab 延伸

下列主題是相同表示方式與
boundary 規則的實用應用，但不屬於三小時課堂的核心內容。

### Two-dimensional array 與 row-major layout

```c
#define COLUMN_COUNT 4

int matrix[3][COLUMN_COUNT] = {0};
```

element 逐 row 儲存。傳遞此 array 時，compiler 必須知道
column stride：

```c
int sum_matrix(size_t rows, const int matrix[][COLUMN_COUNT]) {
  int total = 0;
  for (size_t r = 0; r < rows; ++r) {
    for (size_t c = 0; c < COLUMN_COUNT; ++c) {
      total += matrix[r][c];
    }
  }
  return total;
}
```

這種可攜的 fixed-column 形式，讓 row stride 成為 function type 的一部分。
只要 `rows > 0`，就要求有效的 matrix，且數學上的 sum
必須能以 `int` 表示。接受不同 column count 的 function，需要
不同的表示方式，或在支援的 implementation 上使用
明確標示的 variable-length-array interface。

`matrix[r][c]` 在概念上的 byte offset 是
`(r * COLUMN_COUNT + c) * sizeof(int)`。將 `2 x 4` matrix 畫成八個
連續 cell，並說明為何 column count 是 interface
contract 的一部分。

#### 立即練習 [延伸] — 將 matrix index 展平（3 分鐘）

初始化 `2 x 4` matrix，使用 `1` 到 `8` 的 value。計算
`matrix[1][2]` 的 linear element offset，再對兩個 row 呼叫 `sum_matrix`。

<details>
<summary>展開解答</summary>

element offset 是 `1 * 4 + 2 = 6`，所以 `matrix[1][2]` 是第七個儲存的
element，內容為 `7`。

```c
int main(void) {
  int values[2][COLUMN_COUNT] = {{1, 2, 3, 4}, {5, 6, 7, 8}};
  printf("value=%d sum=%d\n", values[1][2], sum_matrix(2, values));
  return 0;
}
```

這段程式需要 `<stdio.h>` 與上方的 definition。

**預期輸出：**

```text
value=7 sum=36
```

</details>

---

### 實作簡單的 sort

```c
#include <stddef.h>

void insertion_sort(int values[], size_t count) {
  for (size_t i = 1; i < count; ++i) {
    int current = values[i];
    size_t position = i;
    while (position > 0 && values[position - 1] > current) {
      values[position] = values[position - 1];
      --position;
    }
    values[position] = current;
  }
}
```

在 iteration `i` 之前，`[0, i)` 已完成 sorting，且包含原始 prefix 的
value。追蹤 `{4, 2, 2, 1}`，並找出什麼讓相等 element 保持 stable。

#### 立即練習 [延伸] — 追蹤 stable insertion（4 分鐘）

對 `{4, 2, 2, 1}` 執行 function，並印出結果。在追蹤過程中，
仍要區分第一個 `2` 與第二個，即使儲存的 integer
value 相等。哪個比較能保留它們的 relative order？

<details>
<summary>展開解答</summary>

```c
int main(void) {
  int values[] = {4, 2, 2, 1};
  insertion_sort(values, 4);
  for (size_t i = 0; i < 4; ++i) {
    if (i > 0) {
      printf(" ");
    }
    printf("%d", values[i]);
  }
  printf("\n");
  return 0;
}
```

這段程式需要 `<stdio.h>` 與上方的 function。

**預期輸出：**

```text
1 2 2 4
```

loop 只在 `values[position - 1] > current` 時進行 shift。相等的 value
不會彼此越過，所以它們的原始 relative order 會被保留。

</details>

---

## 參考資料與來源教材

- [Functions](<https://github.com/htchen/i2p-nthu/blob/master/程式設計一/function/function.md>)
- [Arrays](<https://github.com/htchen/i2p-nthu/blob/master/程式設計一/array/array.md>)
- [C strings](<https://github.com/htchen/i2p-nthu/blob/master/程式設計一/Printf%20and%20Scanf/String%20type.md>)
- [Input and output](<https://github.com/htchen/i2p-nthu/blob/master/程式設計一/Input%20and%20output/Input%20and%20output.md>)
