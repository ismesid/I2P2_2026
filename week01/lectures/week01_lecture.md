# 第 1 週課堂講義 — 從 Python 到 C

> 2026 年 9 月 8 日 · C17 · 來源沿革：先前的 C 入門、
> formatted-I/O、operators 與 looping 講義，以及教師提供的
> *From C to Assembly* 講義

> Python 銜接：[第 1 週 Python 對照補充教材](week01_python_companion.md)

---

## 學習路線

- **核心：**追蹤一個程式從 source 到 executable 的過程，區分 compile/link/
  run-time failures，再撰寫 C 的 typed expressions、formatted I/O、branches 與
  loops。
- **練習：**完成[第 1 週練習](lecture_exercises/week01_ex.md)，
  再與[完整範例](examples.c)比較。
- **初讀範圍：**閱讀轉換章節時，記住 pipeline 與
  diagnostic 類別。Assembly 章節提供該模型的證據，並非
  要求背誦 instructions。
- **Python 銜接：**只有當 C 的行為難以
  與既有 Python 知識連結時，才查閱補充教材。

---

## 學習目標

完成本次課程後，你應該能夠：

1. 描述 preprocessing、compilation、assembly、linking 與 execution，並
   檢視產生的 assembly，作為轉換過程的證據。
2. 將小型 Python 程式轉換為 typed C。
3. 安全地使用 formatted input/output 與 C control flow。
4. 區分 compile-time error、link-time error 與 run-time fault。
5. 啟用 warnings 編譯，並將 diagnostics 視為有用的證據。

---

## 三小時課程規劃

| 時段 | 核心問題 | 課堂成果 |
|------|---------------|---------------------|
| 1 | typed C 如何成為 executable？ | 編譯、檢視、刻意破壞並修復一個小型程式 |
| 2 | 熟悉的 Python values 如何表示與格式化？ | Type/conversion 學習單與穩健的 input 片段 |
| 3 | 如何轉換 control flow，同時避免 C 特有的 bugs？ | 完成並測試 judge-style 分類程式 |

每小時穿插約 35–45 分鐘的講解與現場撰寫程式，以及
15–18 分鐘的核心練習。剩餘時間供討論、
銜接與短暫休息；當全班準備好時，也可利用這段緩衝時間進行
選做的延伸練習。

### 隨堂練習流程

除非另有註明較長時間，每個**立即練習**都是一至四分鐘的練習。
在小型暫存 source file 中操作，並遵循相同流程：

1. 執行 command 前，先預測結果或 diagnostic；
2. 自己完成指定修改；
3. 使用 `-std=c17 -Wall -Wextra -Wpedantic` 編譯；
4. 至少執行指定的測試；以及
5. 用一句話向同伴說明證據。

這些練習刻意保持小規模，目的是立即回想知識與
獲得回饋，而不是從講義複製完整解答。起初只會
顯示題目；自行嘗試並測試後，再開啟**展開解答**。
每個解答區塊都會指出預期的 standard output、
具代表性的 diagnostic，或說明範例為何沒有 runtime
output。

- **課堂核心：**屬於規劃中的課堂學習路線。
- **延伸：**保留在範例旁供額外練習；若時間不足，可以
  在休息時、lab 中或課後完成。

課堂核心練習在第 1 小時約需 15 分鐘、第 2 小時約需 18 分鐘，
第 3 小時約需 16 分鐘。這樣可保留銜接、提問與
短暫休息的時間，同時維持立即練習的機會。

---

## 第 1 小時 — 程式轉換與 C execution model

> **第 1 小時路線：**[machine model](#1-相同-algorithms不同-machine-model)
> → [轉換 pipeline](#2-轉換-pipeline)
> → [現場 diagnostic build](#第-1-小時現場-build分類-diagnostic)
> → [以 assembly 為證據](#assembly-是觀察視窗)
> → [第一個完整程式](#3-第一個程式)

### 1. 相同 algorithms，不同 machine model

你已經熟悉 sequencing、selection、iteration、functions 與 values。C 要求
你更明確地表達 representation。

| Python | C |
|--------|---|
| 名稱綁定到 object | variable 具有宣告的 type 與 storage |
| Integers 依需要擴大 | Integer types 具有固定的 ranges |
| Lists 可動態調整大小 | Arrays 通常具有固定大小 |
| Exceptions 回報許多錯誤 | 某些錯誤會造成 undefined behavior |
| interpreter 執行程式 | compiler 與 linker 建立 executable |

核心問題從單純的「這個 expression 產生什麼 value？」
變成「產生什麼 value、具有什麼 type、存在哪裡、存放多久？」

---

### 2. 轉換 pipeline

對於名為 `hello.c` 的 source file：

```sh
cc -std=c17 -Wall -Wextra -Wpedantic -g hello.c -o hello
./hello
```

概念上，build 會在 execution 前進行四個轉換階段：

1. **Preprocess：**展開 `#include` 與 `#define` 等 directives。
2. **Compile：**檢查 C，並轉換為目標 assembly。
3. **Assemble：**將 assembly instructions 與 data 編碼成 object file。
4. **Link：**將 object files 與 libraries 合併為一個 executable。

在 run time，operating system loader 將 executable 與所需的
libraries 映射到 memory，建立 process environment，並
透過 language implementation 將控制權交給 `main`。像
`cc` 這樣的 compiler driver 通常會替我們執行其中幾個工具，但也可以在各個
階段結束後停止：

```sh
cc -std=c17 -E hello.c -o hello.i  # preprocessed C
cc -std=c17 -O0 -S hello.c -o hello.s
cc -std=c17 -c hello.c -o hello.o
cc hello.o -o hello
```

本週使用的 command-line 各部分含義如下：

| Command 或 option | 用途 |
|-------------------|---------|
| `cc` | 執行系統的 C compiler driver |
| `-std=c17` | 選擇 C17 language version |
| `-Wall -Wextra -Wpedantic` | 啟用有用的 warning groups |
| `-g` | 保留 debugger 使用的資訊 |
| `-o filename` | 指定 output file 名稱 |
| `-E` | 在 preprocessing 後停止 |
| `-S` | 在產生 assembly text 後停止 |
| `-c` | 在產生 object file 後停止 |
| `-O0` | 將 optimization 降至最低，方便觀察 source 結構 |
| `-O2` | 啟用常用且程度較高的 optimization level |
| `./hello` | 從目前目錄執行名為 `hello` 的檔案 |

<details>
<summary>補充 — 四種 build artifacts 的樣貌</summary>

使用下方現場 build 章節中的完整 `hello.c` 程式。前
兩個 outputs 是可在 editor 中閱讀的 text files。後兩個是
binary files，因此請使用開發工具檢視，不要在 terminal 中印出它們的
raw bytes。成功時，這四個 `cc` commands 通常不會印出
任何內容：結果就是 `-o` 後指定名稱的檔案。下列 commands 讓每個
結果都能被觀察。

#### 1. Preprocessed C：`hello.i`

```sh
cc -std=c17 -E hello.c -o hello.i
```

preprocessor 會在一般 C compilation 前展開 directives。尤其是
`#include <stdio.h>`，會被 implementation 提供的 declarations
取代。產生的檔案通常比 `hello.c` 長得多。
搜尋程式自己的 function，不必從頭開始閱讀：

```sh
grep -n "int twice" hello.i
```

這裡的 `grep -n` 會印出符合的文字及其行號。
`hello.i` 的開頭通常包含類似以下的行：

```text
# 1 "hello.c"
# 1 "<built-in>" 1
# 1 "/.../include/stdio.h" 1 3 4
```

以 `#` 開頭的行是 **line markers**。即使 preprocessing 合併了
多個檔案，它們仍讓後續 diagnostics 能對應到適當的 source 或 header。
Paths 與尾端的 marker numbers 依 implementation 而異。
在較後面的地方，仍有一小段看起來像原本的 C：

```c
int twice(int value);

int main(void) {
  printf("%d\n", twice(21));
  return 0;
}
```

原本的 `#include <stdio.h>` 已不再是要求包含
檔案的 instruction：該 header 的 declarations 現在已出現在 translation unit 中。Macro
的使用處也已被 expansions 取代，comments 可能已被
移除。Function bodies、declarations 與 expressions 仍然是 C，
而非 assembly 或 machine code。確切的 header declarations 與 line-marker
寫法不是課程要求；可跨平台觀察到的是，
preprocessing 會產生另一個 C translation unit。

#### 2. Assembly text：`hello.s`

```sh
cc -std=c17 -O0 -S hello.c -o hello.s
```

compiler 將 preprocessed C 轉換為目前
機器的 assembly。使用以下 command 找出 function labels：

```sh
grep -n "twice" hello.s
```

示意用的 ARM/macOS 片段可能包含：

```text
        .globl  _main
_main:
        ... prepare the argument 21 ...
        bl      _twice
        ... prepare the format string and result ...
        bl      _printf
        ret

        .globl  _twice
_twice:
        ... load value ...
        lsl     w0, w8, #1
        ret

        .asciz  "%d\n"
```

這個 output 同時包含 instructions 與 assembler directives：

- `.globl` 讓 linker 能看見 symbol；
- `_main:` 與 `_twice:` 是標示 instruction 位置的 labels；
- 在這個 ARM target 上，`bl` 呼叫另一個 function，`ret` 則返回；
- `lsl` 將 bits 向左移，可用來實作乘以二；以及
- `.asciz` 儲存 format string，並在其後接上結尾的 zero byte。

x86 compiler 可能改用沒有前導 underscores 的 labels、以 `call` 表示
function call，並使用不同的 registers 或 arithmetic instructions。即使在
`-O0` 下，compiler 也不必將每個 C operator 轉換為
同名 instruction：用 shift 實作乘以二，仍能保留 C 的
結果。在這個階段，辨識 function boundaries、calls 與 constants 即可；
不要背誦某個 target 的 instruction 寫法。

#### 3. Relocatable object file：`hello.o`

```sh
cc -std=c17 -c hello.c -o hello.o
```

`hello.o` 包含編碼後的 machine instructions、data、symbol table，以及
linker 仍需要的資訊。`file` command 可以描述
binary，而不直接傾印內容：

```sh
file hello.o
nm hello.o
```

具代表性的 `file` 描述包括：

```text
hello.o: Mach-O 64-bit object arm64
hello.o: ELF 64-bit LSB relocatable, x86-64, ...
```

關鍵字是 `relocatable`：code 與 data 已存在，但最終
addresses 還沒確定。這個 object 不是完整的 executable，仍然
包含等待 linker 解析的 references。

`nm` command 會列出 object file 已知的 symbols。具代表性的
macOS 結果如下：

```text
0000000000000000 T _main
                 U _printf
0000000000000048 T _twice
0000000000000060 s l_.str
```

左欄是以 hexadecimal 表示的 offsets。在中間欄位，
`T` 表示定義於 code section、全域可見的 symbol，`U`
表示在此 object 中尚未定義，小寫 `s` 則通常表示 local
section symbol。因此 `main` 與 `twice` 的 code 在此，而 `printf` 必須在
linking 時連接到 C library。Linux 通常省略前導
underscores，symbol letters 也可能略有不同。Symbol 寫法與
offsets 是某個 toolchain 的證據，並非 source-level C 規則。

link 步驟會合併並重新定位相關部分，將 external
references 連接到 libraries。採用 dynamic linking 時，部分連接資訊會
先被記錄，等程式啟動時由 loader 完成。

#### 4. Linked executable：`hello`

```sh
cc hello.o -o hello
```

linker 解析剩餘的 references，產生 operating system
能載入的檔案。先檢視，再執行：

```sh
file hello
./hello
```

具代表性的描述包括：

```text
hello: Mach-O 64-bit executable arm64
hello: ELF 64-bit LSB pie executable, x86-64, ...
```

與 relocatable object 不同，這個檔案包含啟動
process 所需的 metadata。它通常比 `hello.o` 大，因為還包含
headers、loader information 與其他 link-time metadata。檔案大小會變動，
不能用來衡量撰寫了多少 C statements。

執行程式，再檢視 shell 儲存的 exit status，會得到：

```text
$ ./hello
42
$ echo $?
0
```

`42` 是 `printf` 寫出的普通 program output；
`"%d\n"` 中的 newline 讓 terminal 移到下一行。程式不會印出
最後的零。shell 儲存此 status，是因為 `main` 回傳 `0`，
而 `echo $?` 會顯示它。依慣例，nonzero status 表示失敗。

完整流程如下：

| Artifact | Representation | 適用的檢視方式 | 還缺少什麼？ |
|----------|----------------|-------------------|------------------------|
| `hello.i` | preprocessed C text | editor、`grep` | C compilation |
| `hello.s` | target assembly text | editor、`grep` | 將 assembly 轉成 binary instructions |
| `hello.o` | relocatable binary object | `file`、`nm` | 最終 addresses 與 external definitions |
| `hello` | linked executable binary | `file`、`./hello` | 在正常 loading 與 execution 前已無缺少的步驟 |

</details>

> **立即練習 [課堂核心] — 說出 artifact 名稱（2 分鐘）：**先不要執行
> commands，在每行後面寫下預期的 output filename。稍後使用
> 下方完整的 `hello.c` 程式執行這些 commands，並修正你的
> 預測。哪個 command 產生的東西可以直接執行？

<details>
<summary>展開解答</summary>

| Command 停止的階段 | Output | 能直接執行嗎？ |
|---------------------|--------|----------------------|
| Preprocessing | `hello.i` | 否 |
| Compilation 到 assembly | `hello.s` | 否 |
| Assembly 到 object code | `hello.o` | 否 |
| Linking | `hello` | 是 |

執行完整 build 的 compiler driver command 也會產生
由 `-o` 指定名稱的 executable。

**預期 terminal output：**這四個成功的 `cc` commands 通常不會印出
任何內容；各自寫入 `-o` 後指定名稱的檔案。完成 linking 後，執行
程式會產生：

```text
42
```

</details>

編譯時出現 warning 的程式不一定安全。閱讀第一個
diagnostic，找出它指向的 source，判斷是 code 還是
所述 contract 有問題。

> **初讀時至少要掌握：**source code 會先經過檢查與
> 轉換才執行；linker 將分別轉換的部分合併；
> compilation、linking 與 execution 階段的 failures 是不同的證據。
> 目前不必背誦 file suffixes、loader 細節或 assembly
> instructions。用上述 commands 觀察各個 stages，等寫完
> 第一個 C 程式後，再回頭閱讀較低層的細節。

---

### 第 1 小時現場 build：分類 diagnostic

從下方第一個程式開始，每次只引入一個 defect：

`int twice(int value);` 是 **declaration**：在 call 被編譯前，先告訴 compiler
function 的名稱、parameter type 與 result type。
後方以 braces 包住的區塊是 **definition**，提供實際操作。這個
最基本的區別已足夠用來觀察今天的 compilation 與 linking；第 2 週會
詳細說明 function contracts、parameter passing 與 decomposition。

```c
#include <stdio.h>

int twice(int value);

int main(void) {
  printf("%d\n", twice(21));
  return 0;
}

int twice(int value) {
  return value * 2;
}
```

#### 立即練習 [課堂核心] — 找出 failure stage（10 分鐘）

1. 移除 `return value * 2` 後的 semicolon。哪個 stage 最先拒絕這個
   程式？
2. 保留 prototype，但移除 definition。現在是哪個 stage 失敗？
3. 執行 `cc -E`，在 preprocessed declarations 中找出原始 source。
4. 執行 `cc -S`，找出 `twice` 的 code，再與 `-O2`
   build 比較，但不要預期會逐行對應。
5. 執行 `cc -c`，檢視 object filename，再用另一個 command 進行 link。

<details>
<summary>展開解答</summary>

1. 缺少 semicolon 讓 C translation unit 的 syntax 無效，因此
   compilation 會在產生 object file 前失敗。
2. call 符合可見的 declaration，因此 compilation 可以成功。
   Linking 會失敗，因為沒有任何已連結的 object 提供 `twice` 的 definition。
3. `hello.i` 包含引入的 declarations，以及可辨識的
   原始檔案內容。
4. 未經 optimization 的 build 通常含有對應 `twice` 的 code；
   經 optimization 的 build 可能簡化或 inline 該 call，同時保留結果。
5. `cc -c hello.c -o hello.o` 產生 `hello.o`，而
   `cc hello.o -o hello` 產生 executable。

**具代表性的 diagnostics 與 output：**diagnostic 的用詞依
compiler 與 linker 而異，但觀察結果應符合以下形式：

| 實驗 | 具代表性的 terminal 證據 | Runtime output |
|------------|----------------------------------|----------------|
| 缺少 semicolon | `error: expected ';' after return statement` | 無；compilation 停止 |
| 缺少 definition | `undefined reference to 'twice'` 或 `Undefined symbols ... _twice` | 無；linking 停止 |
| 已還原的有效程式 | 成功的 build commands 不會印出內容 | `42` |

`-E`、`-S` 與 `-c` commands 成功時通常也不會印出內容；
可觀察到的 outputs 分別是 `hello.i`、`hello.s` 與 `hello.o`。

</details>

學生應記錄 stage、diagnostic 證據與最小修正。
目標是找出 pipeline 中負責的環節，而非背誦訊息。

---

### Assembly 是觀察視窗

> **輔助觀察：**使用產生的 assembly 作為 C 經過
> 轉換的證據，但不要背誦 instruction 名稱、executable sections 或
> 特定機器的 encodings。必須掌握的是轉換 stages 與 diagnostic
> 類別。

產生的 assembly 揭示 compiler 的選擇，並不是可跨平台套用的轉換
步驟。Instruction 名稱、register 名稱、symbol 寫法、calling conventions，
以及 section 名稱，都取決於 target architecture、object format、compiler、
options 與 optimization level。在 x86 target 上，`-masm=intel` 可要求
Intel syntax；但並非每個 target 都適用。

宣告在所有 functions 之外的 object 具有 **static storage duration**：
它存在於程式的整段 execution 期間，且未寫
initializer 時會初始化為零。一般 block-local object 具有 **automatic
storage duration**：execution 位於該 block 期間它才存在，且沒有
自動指定的初始值。在 file scope，keyword `static` 也會讓
名稱只供此 source file 使用。這些 lifetime 規則才是 C 的概念；
下方的 section 名稱只是常見的 implementation 證據。

常見的 object-file 區域讓 C storage duration 變得可觀察：

| 常見 section | 典型內容 |
|----------------|------------------|
| `.text` | 可執行的 machine instructions |
| `.rodata` | read-only constants，包含部分 string literals |
| `.data` | 具有 nonzero initial data 的可寫 static-storage objects |
| `.bss` | 以精簡方式表示的 zero-initialized static-storage objects |

這些名稱常見於 ELF-based systems，並非 C 的保證。具有 static storage duration、
未初始化或明確初始化為零的 object，
即使 executable 沒有儲存每個 zero byte，也會從零開始。
automatic local variable 的 duration 不同，不會只因
平台恰好向 operating system 取得 stack memory 就被初始化。

分別使用 `-O0 -S` 與 `-O2 -S` 編譯這個檔案：

```c
static int zero_count;
static int initial_count = 7;

int counts_total(void) {
  return zero_count + initial_count;
}

int add_one(int value) {
  int result = value + 1;
  return result;
}
```

#### 立即練習 [延伸] — 觀察而不背誦（4 分鐘）

在 `-O0` 下，找出兩個 static objects 與兩項計算的證據。在
`-O2` 下，判斷哪些名稱或 storage locations 仍存在，哪些可能已被
constants 或較簡單的 instructions 取代。即使 instruction sequences、
registers 與 labels 不同，哪些 semantic 觀察仍然成立？

<details>
<summary>展開解答</summary>

由於 `counts_total` 會讀取兩個 objects，典型的 `-O0` assembly file
會把 `zero_count` 保留在 `.bss` 等 zero-initialized region，並把
`initial_count` 保留在 `.data`。在 `-O2` 下，compiler 能證明它們在
此 translation unit 中的總和永遠是七；它可能讓 `counts_total` 回傳這個
constant，並省略兩個 private objects。同樣地，具名 local `result` 在
經 optimization 的 `add_one` 中，可能沒有 memory location。

可跨平台成立的觀察是：`counts_total()` 回傳七，且只要
數學結果能以 `int` 表示，`add_one(value)` 就回傳
比 `value` 多一的值。確切的 sections、symbols、registers 與 instruction
sequences 是 implementation 證據，而非 C language 的保證。

**Runtime output：**無。這個 source 刻意不含 `main` function，
只使用 `-S` 轉換。它的 output 是 assembly file。典型的 `-O0`
檔案含有兩個 objects 的 storage 或 symbol 證據，以及兩個 functions 的
instructions；`-O2` 檔案則可能只含簡化的 function bodies。

</details>

---

### 3. 第一個程式

```c
#include <stdio.h>

int main(void) {
  int courses_completed = 1;
  printf("Programming courses completed: %d\n", courses_completed);
  return 0;
}
```

#### 立即練習 [課堂核心] — 修改、編譯、執行（3 分鐘）

修改 `courses_completed`，使其符合你的經驗，並修改印出的
label，但不要改動 `%d`。編譯並執行程式。接著移除一個
semicolon，預測哪個轉換 stage 會拒絕該檔案，再將它還原。

<details>
<summary>展開解答</summary>

一種可能的修改如下：

```c
int courses_completed = 2;
printf("Previous programming courses: %d\n", courses_completed);
```

你的數字與文字可能不同。`%d` 仍然正確，因為對應的
argument 仍是 `int`。移除必要的 semicolon 會使 compilation
失敗；還原後，translation unit 的 syntax 就再次有效。

**上述修改的預期輸出：**

```text
Previous programming courses: 2
```

修改前，原始程式印出：

```text
Programming courses completed: 1
```

移除 semicolon 後，沒有 runtime output，因為 compilation
會因 syntax diagnostic 而停止。

</details>

- `#include <stdio.h>` 讓 standard I/O functions 的 declarations 可見。
- `int main(void)` 定義程式的 entry point。此處 `void` 表示這個
  版本不接受 arguments，而 `int` 表示它會回報 exit status。
- Braces 界定 block；semicolons 結束 statements。
- `int courses_completed` 宣告 storage 及其解讀方式。
- 依慣例，回傳零向 operating system 表示成功。

---

## 第 2 小時 — Types、representation、conversion 與 formatted I/O

> **第 2 小時路線：**[types 與 expressions](#4-types-與-expressions)
> → [operators](#基本-operators-與-precedence)
> → [division 與 conversion](#integer-division-與-conversion)
> → [truth values](#truth-values)
> → [integer ranges](#補充參考--integer-ranges-與-signedunsigned-交互作用)
> → [formatted I/O](#5-formatted-io)
> → [format contracts](#format-contract-參考)
> → [檢核點](#立即練習-課堂核心--第-2-小時檢核點5-分鐘)

### 4. Types 與 expressions

第一週的核心 scalar types 如下：

- `char` 儲存一個 character 大小的 integer value；
- `int` 是一般的 whole-number type；
- `double` 儲存 floating-point approximation；以及
- `_Bool` 儲存零或一。在 C17 中，`<stdbool.h>` 提供較容易閱讀的
  `bool`、`false` 與 `true` 寫法，分別對應 `_Bool`、零與一。

```c
#include <stdbool.h>

char grade = 'A';
int count = 42;
double average = 87.5;
bool passed = true;
```

若要觀察 `char`、`int` 與 `double` values，請使用 `printf` function，
它已在第一個程式介紹。第一個 argument 是 format string；每個
以 `%` 開頭的 conversion 描述其後對應的 value：
如下表所示。

| C value type | 首個 output conversion | 含義 |
|--------------|-------------------------|---------|
| `int` | `%d` | 印出 decimal integer |
| `double` | `%.1f` | 印出 decimal point 後一位數字 |
| `char` | `%c` | 印出 character |

本小時稍後的完整 format-contract 參考涵蓋 input 與
其他 types。

#### 立即練習 [課堂核心] — 選擇 representation（2 分鐘）

使用前面的第一個程式作為 scaffold。加入 variables，分別表示整數的
學生人數、含小數的溫度與字母成績。先選擇 type，
再設定 initial value，然後依上表分別使用 `%d`、`%.1f` 與 `%c`
印出它們。編譯並執行；不要複製
程式不會使用的示範 variables。

<details>
<summary>展開解答</summary>

一種可能的程式如下：

```c
#include <stdio.h>

int main(void) {
  int student_count = 40;
  double temperature = 26.5;
  char letter_grade = 'A';
  printf("students=%d temperature=%.1f grade=%c\n", student_count,
         temperature, letter_grade);
  return 0;
}
```

實際 values 可以不同。Types 表達了重要的保證：whole
number、帶有小數的 numeric value，以及一個 character。

**上述完整程式的預期輸出：**

```text
students=40 temperature=26.5 grade=A
```

先前只有 declarations 的片段本身不會印出內容；只有在
解答將這些 values 傳給 `printf` 後，才會出現 output。

</details>

使用 `sizeof value` 詢問 object 佔用多少 bytes。除了 `char`，
基本 types 的確切大小可能取決於 implementation。

> **補充 type 名稱：**`size_t` 是用於 object sizes 的 unsigned type，
> 在第 2 週的 arrays 中會變得重要。像
> `int32_t` 這樣的 exact-width types，適用於 external data contract 要求恰好具有該
> width 的 code；它們是參考材料，而非預設用來取代
> `int` 的選擇。

```c
#include <stddef.h>
#include <stdint.h>

size_t length = 10;
int32_t exact_width = 1000;
```

<details>
<summary>Output 說明 — 只有 declarations 不會印出 values</summary>

**Runtime output：**無。這些行宣告並初始化兩個 objects，卻
沒有呼叫 output function。即使把它們放進完整程式，
也要等後續 statements 將它們的 values 傳給
`printf` 等 operation，程式才會產生 output。

</details>

---

### 基本 operators 與 precedence

底層 operations 與熟悉的 Python 類似，但部分寫法與
type 規則不同。先從以下幾組開始：

| 用途 | C operators | 重要規則 |
|---------|-------------|----------------|
| Arithmetic | `+`, `-`, `*`, `/`, `%` | `/` 依 operand types 運算；`%` 要求 integer operands |
| Comparison | `<`, `<=`, `>`, `>=`, `==`, `!=` | 結果是 `0` 或 `1` |
| Logic | `&&`, `\|\|`, `!` | `&&` 與 `\|\|` 由左至右進行 short-circuit |
| Assignment | `=`, `+=`, `-=`, `*=`, `/=`, `%=` | compound assignment 會讀取、計算並儲存 |
| 加減一 | `++`, `--` | 它們會修改 object；初學時請作為獨立 statements 使用 |

Multiplication、division 與 remainder 的結合優先於 addition 與
subtraction。Comparison 在 arithmetic 之後，`&&` 在 comparison 之後，
`||` 又在 `&&` 之後。只要預期的 grouping 不是
一目了然，就優先使用 parentheses：

```c
int quotient = 7 / 3;          /* 2: both operands are int */
int remainder = 7 % 3;         /* 1 */
int precedence = 2 + 3 * 4;    /* 14 */
int grouped = (2 + 3) * 4;     /* 20 */

int score = 10;
score += 5;                    /* score is now 15 */
++score;                       /* score is now 16 */
```

#### 立即練習 [課堂核心] — 印出前先預測（3 分鐘）

將這個片段放入 `main`，再將兩個 division 相關運算改成 `11 / 4` 與
`11 % 4`；將 precedence 對照組改成 `5 + 2 * 6` 與 `(5 + 2) * 6`；並
將 `score` 初始化為 7，再加 4 並 increment。印出五個
最終 values 時使用 `%d`，並在執行前先預測。

<details>
<summary>展開解答</summary>

修改後的 values 是 `quotient == 2`、`remainder == 3`、
`precedence == 17`、`grouped == 42` 與 `score == 12`。沒有 parentheses 的
expression 先執行 multiplication。第二個 expression 的 parentheses 讓 addition
先執行。

**一種可能的 output 行：**若依上述順序印出五個 values，
並以空格分隔，output 為：

```text
2 3 17 42 12
```

</details>

Prefix 與 postfix `++`/`--` 在較大的
expression 中使用其 value 時，行為不同。在入門程式中，這種差異很少值得犧牲
可讀性：優先使用獨立的 `++index;` 或 `--count;` statement，
不要在同一個 expression 中多次修改同一個 object。

---

### Integer division 與 conversion

```c
double wrong = 5 / 2;         /* 2.0: division happened as int */
double right = (double)5 / 2; /* 2.5 */
```

#### 立即練習 [課堂核心] — 移動 conversion（2 分鐘）

印出兩個 values，保留 decimal point 後一位。接著測試兩組 operands：
`-5` 與 `2`，以及 `5` 與 `-2`，每次都先預測再執行。最後，
將 cast 從 numerator 移到 denominator，判斷這樣是否
改變結果。

<details>
<summary>展開解答</summary>

對正數 operands，`wrong` 印出 `2.0`，`right` 印出 `2.5`。C 的 Integer
division 朝零截斷，所以 `-5 / 2` 與 `5 / -2` 都先產生 `-2`，
再轉成 `double`；在 division 前加上 cast，則產生
`-2.5`。將任一 operand cast 為 `double` 都足夠，因此
`5 / (double)2` 也產生 `2.5`。

**預期輸出：**使用 `printf("%.1f %.1f\n", wrong, right)`，三組
operand pairs 會產生：

| Operands | 輸出 |
|----------|--------|
| `5` 與 `2` | `2.0 2.5` |
| `-5` 與 `2` | `-2.0 -2.5` |
| `5` 與 `-2` | `-2.0 -2.5` |

</details>

C 的 Conversions 可能丟失資訊。啟用 warnings 編譯，若 conversion 是刻意的，
就明確寫出來。

Integer remainder 遵循相同的 division 規則。對 nonzero divisor，只要
quotient 可表示，C 就選擇 `/` 與 `%`，使得：

```text
(left / right) * right + left % right == left
```

Integer division 朝零截斷，因此 nonzero remainder 與左側
operand 同號。例如 `-5 / 2` 是 `-2`，`-5 % 2` 是 `-1`。
這也解釋了為什麼測試 `value % 2 == 0` 能正確辨識偶數的
negative integers。以零進行 division 或 remainder 是 undefined。Signed division
也要求 quotient 可表示；下方的 integer-range 參考
會說明重要的 boundary case。

---

### Truth values

在 condition 中，零是 false，任何 nonzero scalar value 都是 true。Relational
與 logical operators 產生 `0` 或 `1`。`if` statement 會計算
parentheses 內的 condition；若 condition 為 true，執行第一個
以 braces 包住的 block。可選的 `else` 提供另一個 block。

```c
int age = 18;
bool has_id = true;
bool eligible = age >= 18 && has_id;
```

#### 立即練習 [課堂核心] — 測試 boundary（2 分鐘）

加入 `if`/`else`，印出 `eligible` 或 `not eligible`。執行由
年齡 17、18 及 ID values `false`、`true` 組成的四種組合。指出
每個未通過案例是被 expression 的哪個部分拒絕。

<details>
<summary>展開解答</summary>

```c
if (eligible) {
  printf("eligible\n");
} else {
  printf("not eligible\n");
}
```

只有年齡 18 且 `has_id == true` 才符合資格。年齡 17 時，
`&&` 的左 operand 為 false，因此 short-circuit evaluation 不需要右 operand。
年齡 18 但沒有 ID 時，右 operand 為 false。

**預期輸出：**

| `age` | `has_id` | 輸出 |
|-------|----------|--------|
| `17` | `false` | `not eligible` |
| `17` | `true` | `not eligible` |
| `18` | `false` | `not eligible` |
| `18` | `true` | `eligible` |

</details>

不要混淆 assignment（`=`）與 comparison（`==`）。

---

### 補充參考 — integer ranges 與 signed/unsigned 交互作用

初讀時，記住 C integer types 具有有限的 ranges，且
兩個 operands 的 types 都會影響計算。下方確切的 limit macros
與 mixed signed/unsigned conversion 規則是有用的 diagnostic
參考，但延伸練習不必在核心
課堂路線中完成。

將 `sizeof` 與 limits headers 連結，而不要假設使用固定的機器：

```c
#include <limits.h>
#include <stdio.h>

printf("int: %zu bytes, range %d through %d\n", sizeof(int), INT_MIN, INT_MAX);
printf("unsigned int maximum: %u\n", UINT_MAX);
```

#### 立即練習 [延伸] — 詢問 implementation（2 分鐘）

將 calls 放入 `main`，再擴充程式，印出 `long` 的 size 與
range，使用 `LONG_MIN` 與 `LONG_MAX`。用 `%zu` 印出 `sizeof` 的結果，
用 `%ld` 印出兩個 `long` limits。不要猜測 `long` 與
`int` 大小相同；編譯，讓目前的 implementation 回答。

<details>
<summary>展開解答</summary>

在 `main` 中加入以下內容；請先引入 `<limits.h>` 與 `<stdio.h>`：

```c
printf("long: %zu bytes, range %ld through %ld\n", sizeof(long), LONG_MIN,
       LONG_MAX);
```

數值的 size 與 limits 是 implementation 的結果。必須從
程式的 output 讀取，不能沿用其他機器的假設。

**常見 64-bit Unix-like system 上的示意輸出：**包含原本兩個
calls 與新增的 `long` call，會得到：

```text
int: 4 bytes, range -2147483648 through 2147483647
unsigned int maximum: 4294967295
long: 8 bytes, range -9223372036854775808 through 9223372036854775807
```

這個確切結果無法跨平台保證。符合標準的 implementation 可以讓 `long`
具有不同的 size 與 range；程式自身的 output 才是
目前環境的答案。

</details>

Unsigned arithmetic 以最大值加一為 modulus 進行 wrap。Signed
overflow 是 undefined behavior。在典型的 two's-complement implementation 上，
`INT_MIN / -1` 與 `INT_MIN % -1` 也都是 undefined，因為數學上的
quotient 無法以 `int` 表示。混用 signed 與 unsigned values
可能將負數轉換為非常大的 unsigned value：

```c
#include <stddef.h>
#include <stdio.h>

int main(void) {
  int index = -1;
  size_t count = 10;

  printf("index=%d count=%zu\n", index, count);
  /* Uncomment only after predicting the result. */
  /* printf("%d\n", index < count); */
  return 0;
}
```

#### 立即練習 [延伸] — 顯示 mixed-domain bug（3 分鐘）

取消 comparison 的 comment，並使用課程 warning flags 編譯。先預測
結果。選擇能表示相同預期 domain 的 types 來修正 comparison；
不要只為了消除 warning 而加入 cast。

<details>
<summary>展開解答</summary>

```c
printf("%d\n", index < count);
```

在目前常見的 implementations 上，comparison 印出零，因為 `index`
被轉換為 `size_t`；將 `-1` 轉換為該 unsigned type，會得到其
最大值，而最大值不小於 10。使用課程 warning flags 時，常見的
compilers 會診斷出兩個不同的 numeric domains 被混用。若這個
小問題確實用 `-1` 作為 sentinel，且所有 counts 都能以 `int` 表示，
一種一致的修正如下：

```c
int index = -1;
int count = 10;
printf("%d\n", index < count);
```

對實際的 container API，獨立的 success flag 或其他明確的 absence
representation，通常比將 negative sentinel 與
unsigned size 混用更清楚。

**上述常見 implementations 的具代表性 output：**原本的
mixed-type 程式先印出 values，在取消 comparison 的
comment 後，印出零：

```text
index=-1 count=10
0
```

將 `count` 改成 `int` 後，修正的 comparison 比較兩個 signed
values，並印出：

```text
1
```

</details>

#### 補充參考 — integer literal suffixes

integer-literal suffix 會參與決定 expression 的 type。Suffix
`U` 表示「選擇 unsigned integer type」；對較小的 literals `0U` 與
`1U`，該 type 是 `unsigned int`。因此 `0U - 1U` 的兩個 operands 都是
unsigned，subtraction 會 wrap 到 `UINT_MAX`。相關 suffixes 包括
`L`、`LL`，以及 `ULL` 等組合。當所需的 type
屬於 contract 的一部分時，才使用 suffix，不要只為了消除 conversion warning。

不要用 cast「修正」每個 warning。先判斷
程式要表示哪個 domain。Array sizes 的 loop indices 通常使用 `size_t`；
必須表示 `-1` 的 values，則需要 signed type 或不同的 absence representation。

---

### 5. Formatted I/O

每個 C 程式啟動時都有三個 standard text streams：

- `stdin` 提供一般 input；
- `stdout` 接收一般 output；以及
- `stderr` 接收 diagnostics，與一般 output 分開。

`scanf` 從 `stdin` 讀取，`printf` 寫入 `stdout`。相關的 call
`fprintf(stderr, ...)` 使用與 `printf` 相同形式的 format string，但
將訊息送到 diagnostic stream。對 online judge 而言，
這個區分很重要，因為 diagnostics 不能成為要求答案的一部分。
每個 formatted call 中，conversion specifiers 都必須符合
對應的 argument types。從 `main` 回傳零表示成功；
回傳 nonzero value 則表示程式無法完成其
contract。

```c
int score = 95;
double ratio = 0.875;
printf("score=%d ratio=%.2f\n", score, ratio);
```

#### 立即練習 [課堂核心] — 控制呈現方式（1 分鐘）

將 precision 從 decimal point 後兩位改成四位，再於
每個 value 前加入描述性 label。確認 formatting 只改變
output 文字，不改變儲存的 `ratio`。

<details>
<summary>展開解答</summary>

```c
printf("student score=%d success ratio=%.4f\n", score, ratio);
```

output 變成 `student score=95 success ratio=0.8750`。新增的
位數與 labels 只影響呈現；`ratio` 仍是相同的 `double`。

**預期輸出：**

```text
student score=95 success ratio=0.8750
```

進行指定的 formatting 修改前，在一般的 round-to-nearest 環境下，原本的 call 印出
`score=95 ratio=0.88`。

</details>

對簡單的 judge input，請檢查 `scanf` 的結果：

```c
int a;
int b;
if (scanf("%d %d", &a, &b) != 2) {
  fprintf(stderr, "expected two integers\n");
  return 1;
}
printf("%d\n", a + b);
```

#### 立即練習 [課堂核心] — 測試 input contract（3 分鐘）

將片段放入 `main` 中，並在程式引入 `<stdio.h>`。依序使用
`10 20`、`10 x`，以及只有一個 integer 隨即
end-of-file 的 input 執行。暫時將每個案例的 `scanf` 結果存入
`int conversions` variable 並記錄。理解三種結果後，
還原精簡的 condition。

<details>
<summary>展開解答</summary>

diagnostic 版本的開頭如下：

```c
int conversions = scanf("%d %d", &a, &b);
printf("conversions=%d\n", conversions);
if (conversions != 2) {
  fprintf(stderr, "expected two integers\n");
  return 1;
}
```

Input `10 20` 產生兩次 conversions，允許印出總和。
`10 x` 只轉換第一個 integer，因此 count 是一。一個 integer
後接 end-of-file，也只產生一次 conversion。若一開始就遇到 end-of-file，
會產生 `EOF`，而不是成功的 conversion count。

**各 stream 的預期輸出：**假設原本計算總和的 statement 保留在
diagnostic 片段後方。`EOF` 使用的 numeric value 屬於 implementation-
defined，通常是 `-1`。

| Input | Standard output | Standard error | Exit status |
|-------|-----------------|----------------|-------------|
| `10 20` | `conversions=2` 後接 `30` | 無 | `0` |
| `10 x` | `conversions=1` | `expected two integers` | nonzero |
| `10` 後接 EOF | `conversions=1` | `expected two integers` | nonzero |
| 一開始即遇到 EOF | `conversions=<EOF value>` | `expected two integers` | nonzero |

</details>

`scanf` 需要 `a` 與 `b` 的 **addresses**，才能修改它們。我們會在
第 4 週課堂講義解釋 addresses。在此之前，將 format
string 與每個對應的 argument 視為一組需要檢查的配對。

---

### Format-contract 參考

| Value type | `printf` | `scanf` |
|------------|----------|---------|
| `int` | `%d` | `%d` 搭配 `&integer_variable` |
| `unsigned int` | `%u` | `%u` 搭配 `&unsigned_variable` |
| `long` | `%ld` | `%ld` 搭配 `&long_variable` |
| `long long` | `%lld` | `%lld` 搭配 `&long_long_variable` |
| `size_t` | `%zu` | `%zu` 搭配 `&size_variable` |
| `double` | `%f` | `%lf` 搭配 `&double_variable` |
| character | `%c` | `%c` 搭配 `&character_variable` |
| `bool` | integer promotion 後使用 `%d` | 沒有直接的 conversion；讀取並驗證 `int` |

對 `printf`，`float` argument 會 promoted 成 `double`，因此使用 `%f`。對
`scanf`，`%f` 要求 `float` 的 address，而 `%lf` 要求
`double` 的 address。這種不對稱是 memory corruption 的常見來源。
String 與 pointer formatting 要等第 2 週建立
array representation、第 4 週建立 pointer model 後才介紹。

將 `bool` 傳給 `printf` 時，會 promoted 成 `int`，所以 `%d` 印出
零或一。不要將 `bool*` 傳給 `scanf` 的 `%d`：`%d` 要求
`int*`。先讀入 `int`，驗證允許的 values，再將
結果指派給 `bool`。

#### 立即練習 [延伸] — 解讀 format warning（2 分鐘）

回到第 1 小時的 `twice` 程式。只將 output conversion 從
`%d` 改為 `%f`，再使用課程 warning flags 編譯。`%f` 要求什麼 type？
`twice(21)` 產生什麼 type？為何應在執行程式前還原
正確的 conversion？

<details>
<summary>展開解答</summary>

對 `printf`，`%f` 要求對應的 `double`，但 `twice` 宣告為
回傳 `int`。啟用 warnings 的 compiler 因此能診斷此 mismatch。
`printf` 依賴 format string 來決定如何解讀後續的每個
argument。提供錯誤的 type 會使 call 具有 **undefined behavior**，
表示 C 沒有規定必須產生什麼結果；第 7 節會深入說明此概念。
看似合理的 output 也不能讓 call 變得正確。請還原 `%d`，
因為程式的目的是印出 `twice(21)` 的 integer 結果。

**有缺陷程式的 Runtime output：**不應嘗試取得；不要
執行它。具代表性的 compilation diagnostic 如下：

```text
warning: format specifies type 'double' but the argument has type 'int'
```

還原 `%d` 後，warning 消失，程式印出：

```text
42
```

</details>

#### 立即練習 [延伸] — 建立 format 檢查表（2 分鐘）

從表格選擇五列，包含 `size_t` 與 `bool`，並撰寫
單一 `printf` call 印出它們。加入一個 `scanf` call，讀取 `double`。
與同伴交換 code，在編譯前檢查每個 specifier 是否符合
對應的 argument。

<details>
<summary>展開解答</summary>

一種可能的檢查表程式如下：

```c
#include <stdbool.h>
#include <stddef.h>
#include <stdio.h>

int main(void) {
  int signed_value = -3;
  unsigned int unsigned_value = 3U;
  size_t item_count = 5;
  double real_value = 3.5;
  bool ready = true;

  printf("%d %u %zu %.1f %d\n", signed_value, unsigned_value, item_count,
         real_value, ready);

  double input_value;
  if (scanf("%lf", &input_value) != 1) {
    return 1;
  }
  printf("input=%.1f\n", input_value);
  return 0;
}
```

審查的重點是位置對應：每個 conversion specifier 都必須符合
對應 argument 的 type。

**Input `2.5` 的預期輸出：**

```text
-3 3 5 3.5 1
input=2.5
```

</details>

---

### 立即練習 [課堂核心] — 第 2 小時檢核點（5 分鐘）

編譯前，先預測 type 與 value：

```c
int a = 7;
int b = 2;
double x = a / b;
double y = (double)a / b;
```

接著編譯一個印出這四個 values 的程式。解釋兩個
floating-point 結果為何不同，不要只停留在數字答案。

<details>
<summary>展開解答</summary>

`a` 與 `b` 是 `int`。因此 Integer division 先產生 3，`x` 再將
該 value 儲存為 `3.0`。Cast 使第二次 division 的一個 operand 變成 `double`，
所以 `y` 是 `3.5`。

對應的 output statement 如下：

```c
printf("a=%d b=%d x=%.1f y=%.1f\n", a, b, x, y);
```

**預期輸出：**

```text
a=7 b=2 x=3.0 y=3.5
```

</details>

---

## 第 3 小時 — Selection、iteration、EOF 與 judge-style 轉換

> **第 3 小時路線：**[selection 與 iteration](#6-selection-與-iteration)
> → [input-driven loops](#input-driven-loops-與-eof)
> → [引導式轉換](#立即練習-課堂核心--第-3-小時引導式轉換8-分鐘)
> → [undefined behavior](#7-undefined-behavior-不是-exception)

### 6. Selection 與 iteration

Python indentation 轉換成明確的 braces：

```python
limit = 10
total = 0
for value in range(1, limit + 1):
    if value % 2 == 0:
        total += value
```

```c
int limit = 10;
int total = 0;
for (int value = 1; value <= limit; ++value) {
  if (value % 2 == 0) {
    total += value;
  }
}
```

C `for` loop 有三個以 semicolons 分隔的 control clauses。在此，
`int value = 1` 在 loop 前執行一次，`value <= limit` 在
每次 iteration 前檢查，`++value` 在每次完成的 iteration 後執行。`if`
statement 決定該 iteration 是否更新 `total`。由於 `value` 是
在 `for` statement 中宣告，其名稱只能在該 loop 中使用。

#### 立即練習 [課堂核心] — 改變一個規則（3 分鐘）

將 C 片段放進 `limit = 10` 的完整程式，並印出
結果。接著改成加總可被三整除的 values，取代
可被二整除的 values。執行程式前，先預測兩個總和。

<details>
<summary>展開解答</summary>

原本的 loop 加總 `2 + 4 + 6 + 8 + 10`，產生 30。修改後的 loop
可以寫成：

```c
int limit = 10;
int total = 0;
for (int value = 1; value <= limit; ++value) {
  if (value % 3 == 0) {
    total += value;
  }
}
printf("%d\n", total);
```

它印出 18，因為納入的 values 是 3、6 與 9。

**預期輸出：**

| 版本 | 輸出 |
|---------|--------|
| 原本的偶數規則 | `30` |
| 修改後的可被三整除規則 | `18` |

</details>

C 也提供 `while` 與 `switch`；接下來兩個範例會展示各個 construct 的
具體用途。`switch` 計算 controlling expression 一次，
再跳到符合的 `case`。`default` label 處理所有未符合的
values，`break` 則離開 `switch`。即使 body 只有一個 statement，
也優先使用 braces，避免後續修改時出錯。

```c
char command = 'h';

switch (command) {
  case 'q':
    printf("quit\n");
    break;
  case 'h':
    printf("help\n");
    break;
  default:
    fprintf(stderr, "unknown command\n");
    break;
}
```

#### 立即練習 [延伸] — 讓 fallthrough 可見（3 分鐘）

將片段放入 `main` 中，並在程式引入 `<stdio.h>`。新增一個
`r` command，讓它印出 `reset`。暫時省略其 `break`，將它放在
`h` case 前，預測 `r` 會印出的兩行。執行一次，再還原
`break`，確認只剩下預期的動作。

<details>
<summary>展開解答</summary>

缺少第一個 `break` 時，command `r` 同時印出 `reset` 與 `help`，因為
execution 會繼續進入下一個 case。修正後的 case 如下：

```c
case 'r':
  printf("reset\n");
  break;
case 'h':
  printf("help\n");
  break;
```

還原 `break` 後，command `r` 只印出 `reset`。

**原始範例在 `command == 'h'` 時的預期輸出：**

```text
help
```

**缺少第一個 `break` 時的預期輸出：**

```text
reset
help
```

**還原 `break` 後的預期輸出：**

```text
reset
```

</details>

缺少 `break` 時，execution 會繼續進入下一個 `case`。只有在刻意使用
並有文件說明時，才使用 fallthrough。

---

### Input-driven loops 與 EOF

Judge data 有時包含數量未知的 records。`while`
statement 在每次 iteration 前檢查 parentheses 內的 condition，且
只在 condition 為 true 時繼續。在 Python 中，對 input
stream 的 iteration 會自然結束。在 C 中，`scanf` 回報要求的 conversions 有多少次
成功，所以 `scanf("%d", &value) == 1` 表示「已讀取一個 integer；
處理它」。

第一個範例的 input contract 允許最多 100 個 numeric tokens，
每個都能以 `int` 表示，且位於 `[-30000, 30000]`。因此，總和的
絕對值最多是 `100 * 30000`，也就是 3,000,000，落在
`long long` 保證的最小 range 內。此證明讓範例聚焦於
input-loop behavior。假設課程 judge 提供可讀取的 input
stream；偵測 device-level I/O error 不在本練習範圍。當 input 代表的 value 超出範圍時，`%d`
conversion 不提供安全的 recovery path；也就是說，若該 value
無法以 `int` 表示，就沒有安全恢復的保證，所以 representability 在此是明確的
precondition。第 7 週會發展 digit-by-digit conversion，在每個 arithmetic 步驟前
檢查 range。

loop 結束後，`feof(stdin)` 只有在失敗的 read 遇到
end-of-file 時才是 nonzero。若下一個 token 不是 integer，conversion count 為零，
`feof(stdin)` 也維持零。因此程式能區分
一般的 input 結束與 invalid token：

```c
#include <stddef.h>
#include <stdio.h>

int main(void) {
  int value;
  long long total = 0;
  size_t count = 0;

  while (scanf("%d", &value) == 1) {
    if (count == 100) {
      fprintf(stderr, "too many integers\n");
      return 1;
    }
    if (value < -30000 || value > 30000) {
      fprintf(stderr, "integer is outside the supported range\n");
      return 1;
    }
    total += value;
    ++count;
  }

  if (!feof(stdin)) {
    fprintf(stderr, "invalid token after %zu integers\n", count);
    return 1;
  }
  printf("count=%zu total=%lld\n", count, total);
  return 0;
}
```

#### 立即練習 [課堂核心] — 從 shell 驅動 loop（5 分鐘）

編譯程式，再測試 valid sequence、empty input，以及
兩個 integers 後接 `x` 的 sequence。例如，使用
`printf '10 -2 5\n' | ./program` 將文字 pipe 進程式。也測試每個 numeric
boundary 上的 value、剛超出 boundary 的 value，以及第 101 個 integer。說明
每個被拒絕的案例違反了 input contract 的哪個部分。

<details>
<summary>展開解答</summary>

- `10 -2 5` 在三次成功的 conversions 後到達 end-of-file，並印出
  `count=3 total=13`。
- Empty input 不進行任何 conversions，正常到達 end-of-file，並印出
  `count=0 total=0`。
- `10 -2 x` 進行兩次 conversions，接著在 `x` 停止。由於 failure
  不是 end-of-file，程式回報 `invalid token after 2 integers`，並
  回傳失敗。
- Values `-30000` 與 `30000` 符合包含端點的 numeric boundary，
  而任一緊鄰的外側 value 都會被拒絕。
- 前 100 個 integers 會被處理；成功讀取的第 101 個 integer，
  會在加入總和前被拒絕。

EOF 是此程式一般的結束 condition。Noninteger token 違反
input contract，不能默默當成相同的 condition 處理。

**各 stream 的預期輸出：**

| Input | Standard output | Standard error | Exit status |
|-------|-----------------|----------------|-------------|
| `10 -2 5` | `count=3 total=13` | 無 | `0` |
| Empty input | `count=0 total=0` | 無 | `0` |
| `10 -2 x` | 無 | `invalid token after 2 integers` | nonzero |
| `-30000 30000` | `count=2 total=0` | 無 | `0` |
| `30001` | 無 | `integer is outside the supported range` | nonzero |
| 101 個 `1` | 無 | `too many integers` | nonzero |

</details>

只要求一次 conversion 時，`scanf` 在轉換 integer 後回傳 `1`，
下一個 token 不符合時回傳 `0`，或回傳 `EOF`，表示 input 在
conversion 前已結束。絕不要寫 `while (!feof(stdin))`：只有 read 嘗試
失敗後才會觀察到 EOF，因此這種寫法常會多處理一次 stale data。

---

### 立即練習 [課堂核心] — 第 3 小時引導式轉換（8 分鐘）

轉換這個 Python 程式所表達的正數平方和。Python
版本讀取一行；C 練習刻意將 input 推廣成
以 whitespace 分隔、持續到 end-of-file 的 integers。兩個版本
對 valid input 都印出一個答案：

```python
values = [int(token) for token in input().split()]
answer = sum(value * value for value in values if value > 0)
print(answer)
```

每讀到一個 integer 就處理，不儲存 array。接受最多 100 個
inputs，要求每個 value 位於 `[-30000, 30000]`，累加到
`long long`，並區分 end-of-file 與 invalid token。如同典型
使用 `%d` 的 judge input，假設每個 numeric token 都能以
`int` 表示。第 7 週的 lexer 會移除此假設：只有在確認
每個 arithmetic 步驟的結果仍可表示後，才累加 digits。測試：

- empty line/end-of-file；
- 全部為負數的 values；
- 零混合正數；
- 恰好 100 個 values；
- 第 101 個 value；
- 一個 noninteger token；
- 所述 range 兩端的 values。

最後的討論應區分 algorithm 的轉換，以及
C 要求的新 representation 與 range 決策。

<details>
<summary>展開解答</summary>

下列解答實作明確修訂後的 stream contract：

```c
#include <stddef.h>
#include <stdio.h>

int main(void) {
  int value;
  size_t count = 0;
  long long answer = 0;

  while (scanf("%d", &value) == 1) {
    if (count == 100) {
      fprintf(stderr, "too many values\n");
      return 1;
    }
    if (value < -30000 || value > 30000) {
      fprintf(stderr, "value is outside the supported range\n");
      return 1;
    }
    ++count;

    if (value > 0) {
      long long wide_value = value;
      answer += wide_value * wide_value;
    }
  }

  if (!feof(stdin)) {
    fprintf(stderr, "invalid integer input\n");
    return 1;
  }
  printf("%lld\n", answer);
  return 0;
}
```

不需要 array，因為每個 value 只貢獻一次，之後不再
需要。最多 100 個 30000 的平方相加為 90,000,000,000，落在
`long long` 保證的最小 range 內。因此 input bounds 能確保
arithmetic safety，不必用進階的 overflow formulas
打斷核心 loop。

**各 stream 的預期輸出：**

| Input | Standard output | Standard error | Exit status |
|-------|-----------------|----------------|-------------|
| Empty input | `0` | 無 | `0` |
| `-3 -1 0` | `0` | 無 | `0` |
| `0 3 4` | `25` | 無 | `0` |
| `-30000 30000` | `900000000` | 無 | `0` |
| `30001` | 無 | `value is outside the supported range` | nonzero |
| `1 x` | 無 | `invalid integer input` | nonzero |
| 101 個 `1` | 無 | `too many values` | nonzero |

</details>

---

### 7. Undefined behavior 不是 exception

Python 通常會停止並回報 out-of-range list access 等錯誤。
C standard 則不為某些 invalid operations 定義意義。
例如：

- 讀取 uninitialized automatic variable；
- signed integer overflow；
- 將 integer 除以零；
- 存取超出 object 有效 bounds 的 storage，第 2 週會配合 arrays
  深入說明；
- 使用不符合的 `printf` format。

compiler 可以假設 undefined behavior 永遠不會發生。因此，「曾經
成功執行一次」不能作為程式正確的證據。

#### 立即練習 [延伸] — 執行前先修正（4 分鐘）

將以下片段放入 `main`，但**先不要執行**：

```c
int denominator = 0;
int uninitialized_value;
printf("%d\n", 100 / denominator);
printf("%d\n", uninitialized_value);
```

找出兩個被違反的 preconditions。修改 inputs 或加上 guard，
使每個實際計算的 division 與 scalar read 都是 defined。完成後，
才編譯並執行修正版本。

<details>
<summary>展開解答</summary>

Integer division 要求 nonzero denominator，automatic scalar 則必須
先取得 value 才能被讀取。一種使用 guard 的修正如下：

```c
int initialized_value = 25;

if (denominator != 0) {
  printf("%d\n", 100 / denominator);
} else {
  fprintf(stderr, "denominator must not be zero\n");
}
printf("%d\n", initialized_value);
```

initialization 與 guard 很重要，因為它們避免 invalid operations
被計算；事後才印出錯誤訊息就太晚了。

**原本 `denominator == 0` 時的預期輸出：**diagnostic 與
一般結果會寫入不同的 streams。

```text
standard error: denominator must not be zero
standard output: 25
```

</details>

---

## 完整範例：分類 integer

第 2 小時的 `if`/`else` 形式可以擴充為 `else if` chain。每個
condition 由上到下檢查，只有第一個為 true 的 branch 會執行。
這個程式使用 chain 區分負數、正數與零的 values，
再用獨立的 `if`/`else` 分類 parity：

```c
#include <stdio.h>

int main(void) {
  int value;
  if (scanf("%d", &value) != 1) {
    return 1;
  }

  printf("%d is ", value);
  if (value < 0) {
    printf("negative");
  } else if (value > 0) {
    printf("positive");
  } else {
    printf("zero");
  }

  if (value % 2 == 0) {
    printf(" and even\n");
  } else {
    printf(" and odd\n");
  }
  return 0;
}
```

追蹤每個 input 選中的 condition。為何計算 `value % 2` 仍是 defined，
即使 `value` 是負數？零會得到什麼特殊 output？這個版本
只使用 integer values 與 control flow。第 2 週介紹 character arrays，
第 4 週則解釋指向 strings 的 pointer-valued references。

### 立即練習 [延伸] — 擴充而不重複（4 分鐘）

擴充程式，讓它也回報 value 是否可被三整除。
重複使用同一個 `value`；不要新增 input operation。預測並測試
`-3`、`0`、`4` 與 invalid input 的完整 output。

<details>
<summary>展開解答</summary>

延後 parity 結果後的 newline，再加入一個獨立測試：

```c
if (value % 2 == 0) {
  printf(" and even");
} else {
  printf(" and odd");
}

if (value % 3 == 0) {
  printf(", divisible by three\n");
} else {
  printf(", not divisible by three\n");
}
```

**延伸前的預期輸出：**原本的 classifier 產生：

```text
-3 is negative and odd
0 is zero and even
4 is positive and even
```

產生的描述如下：

- `-3 is negative and odd, divisible by three`
- `0 is zero and even, divisible by three`
- `4 is positive and even, not divisible by three`

Invalid input 仍會在印出任何 classification 前回傳。

**三次分別執行有效案例的預期 standard output：**

```text
-3 is negative and odd, divisible by three
0 is zero and even, divisible by three
4 is positive and even, not divisible by three
```

對 invalid input，standard output 為空，程式回傳 nonzero
status。

</details>

---

## 自我檢核

1. 「undefined reference」diagnostic 出現在 pipeline 的哪個位置？
2. `-E`、`-S` 與 `-c` 分別產生哪些不同的 artifacts？
3. `7 / 3` 與 `(double)7 / 3` 的 values 是多少？
4. 為什麼 `%d` 的 argument 必須具有預期的 integer type？
5. 將 Python `while` loop 轉換為 C，使它重複讀取直到遇到 `0`。
6. **延伸：**為什麼 `-O2` 產生的 assembly 可以省略具名的 local
   variable？

---

## 重點整理

- 既有的程式設計知識仍適用；C 讓 types、storage 與 failures 更明確。
- C 程式先經過 preprocessing、compilation、assembly 與 linking，再 execution。
- 產生的 assembly 是依 target 與 option 而異的證據，並非 C
  language 的定義。
- Declarations、format strings 與 conversions 都是 contracts。
- Warnings、exit status 與 tests 都是正常開發的一部分。
- 避免 undefined behavior 是 correctness 的必要條件。

---

## 參考資料與來源教材

- [教師講義：*From C to Assembly*](../../assets/references/from_c_to_assembly.pdf)
- [程式設計入門](<https://github.com/htchen/i2p-nthu/blob/master/程式設計一/Introduction%20to%20programming/README.md>)
- [Operators、expressions 與 statements](<https://github.com/htchen/i2p-nthu/blob/master/程式設計一/Operators%2C%20Expressions%2C%20and%20Statements/README.md>)
- [Looping](<https://github.com/htchen/i2p-nthu/blob/master/程式設計一/Looping/README.md>)
- [`printf` 與 `scanf` 重點整理](<https://github.com/htchen/i2p-nthu/blob/master/程式設計一/Printf%20and%20Scanf/總整理.md>)
