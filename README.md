# Algorithm X 解 6×6 數獨

## Reference

[Day30 -- Algorithm X and Sudoku](https://ithelp.ithome.com.tw/articles/10250123)

[X演算法](https://zh.wikipedia.org/wiki/X%E7%AE%97%E6%B3%95)

## 1. 數獨的 Exact Cover 條件

以 **6×6 數獨**為例，每個位置需要滿足 4 種條件：

1. **每個格子都必須填入一個數字**
2. **每一行的數字不能重複**
3. **每一列的數字不能重複**
4. **每一個區塊的數字不能重複**

因此共有：

```text
36 個格子
36 個 Row-Value 條件
36 個 Column-Value 條件
36 個 Block-Value 條件
────────────────────
144 個 constraints
```

也就是：

```text
6 × 6 × 4 = 144
```

---

## 2. Constraint 的編號方式

可以把 144 個 constraint 分成 4 個區域。

### Constraint 1：每格都要有數字

共有：

```text
6 × 6 = 36
```

公式：

```go
row * 6 + col
```

例如：

```text
(row, col) = (0, 2)

0 * 6 + 2 = 2
```

所以：

```text
constraint = 2
```

---

### Constraint 2：每行數字不能重複

共有：

```text
6 行 × 6 種數字 = 36
```

從 index `36` 開始。

公式：

```go
36 + row*6 + (value-1)
```

例如：

```text
(row, value) = (0, 2)

36 + 0*6 + (2-1)
= 37
```

所以：

```text
constraint = 37
```

這代表：

```text
第 0 行必須出現數字 2
```

---

### Constraint 3：每列數字不能重複

共有：

```text
6 列 × 6 種數字 = 36
```

從 index `72` 開始。

公式：

```go
72 + col*6 + (value-1)
```

例如：

```text
(col, value) = (2, 2)

72 + 2*6 + (2-1)
= 85
```

所以：

```text
constraint = 85
```

這代表：

```text
第 2 列必須出現數字 2
```

---

### Constraint 4：每個區塊數字不能重複

6×6 數獨使用 **3×2 區塊**：

```text
┌─────┬─────┐
│     │     │
│ 3×2 │ 3×2 │
├─────┼─────┤
│     │     │
│ 3×2 │ 3×2 │
├─────┼─────┤
│     │     │
│ 3×2 │ 3×2 │
└─────┴─────┘
```

共有：

```text
6 個區塊 × 6 種數字 = 36
```

從 index `108` 開始。

首先計算目前 `(row, col)` 位於哪個 block：

```go
blockRow := row / 3
blockCol := col / 2
```

再將它轉成 block index：

```go
block := blockRow*3 + blockCol
```

最後：

```go
108 + block*6 + (value-1)
```

例如：

```text
(row, col, value) = (0, 2, 2)
```

計算：

```text
blockRow = 0 / 3 = 0
blockCol = 2 / 2 = 1

block = 0*2 + 1
      = 1
```

所以：

```text
108 + 1*6 + (2-1)
= 115
```

---

## 3. Candidate

接下來把每一個：

```text
(row, col, value)
```

視為一個 **candidate**。

每個 candidate 都會對應到 **4 個 constraints**：

```go
type exactCoverCandidate struct {
    row         int
    col         int
    value       int
    constraints [4]int
}
```

也就是：

```text
(row, col, value)
        │
        ├── Cell constraint
        ├── Row-Value constraint
        ├── Column-Value constraint
        └── Block-Value constraint
```

---

## 4. 已經有數字的格子

例如：

```text
matrix[0][2] = 2
```

因為這個格子已經固定是 `2`，所以只能產生：

```text
(0, 2, 2)
```

它的 4 個 constraints：

```text
1. Cell

0 * 6 + 2
= 2

2. Row-Value

36 + 0*6 + (2-1)
= 37

3. Column-Value

72 + 2*6 + (2-1)
= 85

4. Block-Value

108 + 1*6 + (2-1)
= 115
```

因此：

```text
(0, 2, 2)

constraints = [2, 37, 85, 115]
```

---

## 5. 空白格子

例如：

```text
matrix[0][1] = 0
```

代表這個格子還沒有決定數字。

如果目前沒有其他限制，可以產生：

```text
(0, 1, 1)
(0, 1, 2)
(0, 1, 3)
(0, 1, 4)
(0, 1, 5)
(0, 1, 6)
```

假設選擇：

```text
(0, 1, 4)
```

那麼它對應：

### 1. Cell

```text
0 * 6 + 1
= 1
```

### 2. Row-Value

```text
36 + 0*6 + (4-1)
= 39
```

### 3. Column-Value

```text
72 + 1*6 + (4-1)
= 81
```

### 4. Block-Value

```text
blockRow = 0 / 3 = 0
blockCol = 1 / 2 = 0

block = 0

108 + 0*6 + (4-1)
= 111
```

因此：

```text
(0, 1, 4)

constraints = [1, 39, 81, 111]
```

如果選擇：

```text
(0, 1, 6)
```

則：

```text
constraints = [1, 41, 83, 113]
```

---

## 6. 建立 Exact Cover

建立：

```go
covered := make([]bool, 144)
```

其中：

```text
false = constraint 尚未被滿足
true  = constraint 已經被滿足
```

例如選擇：

```text
(0, 1, 4)
```

它的：

```text
constraints = [1, 39, 81, 111]
```

那麼：

```text
covered[1]   = true
covered[39]  = true
covered[81]  = true
covered[111] = true
```

代表這個 candidate 同時滿足了 4 個條件。

---

## 7. Algorithm X 的核心流程

建立所有合法 candidate 後，開始進行 Algorithm X。

### Step 1：建立所有 Candidate

針對每一個格子：

```text
(row, col)
```

如果已經有數字：

```text
只建立該數字的 candidate
```

如果是空白：

```text
建立 1~6 所有可能的 candidate
```

每個 candidate 都記錄：

```text
(row, col, value)
+
4 個 constraints
```

---

### Step 2：建立 covered

```go
covered := make([]bool, 144)
```

開始時：

```text
全部都是 false
```

---

### Step 3：找出最少 Candidate 的 Constraint

在所有：

```text
covered[i] == false
```

的 constraints 中，找出：

> **目前可以選擇的 candidate 數量最少的 constraint**

例如：

```text
Constraint A → 3 個 candidate
Constraint B → 5 個 candidate
Constraint C → 1 個 candidate
Constraint D → 4 個 candidate
```

選擇：

```text
Constraint C
```

這是 Algorithm X 很重要的最佳化。

因為：

```text
選擇越少 → 分支越少 → Backtracking 越快
```

---

### Step 4：依序嘗試 Candidate

假設選到：

```text
Constraint C
```

而它有：

```text
Candidate 1
Candidate 2
Candidate 3
```

依序嘗試。

例如：

```text
Candidate 1
    ↓
檢查它的 4 個 constraints
    ↓
是否全部 covered == false？
```

如果可以：

```text
covered[constraint1] = true
covered[constraint2] = true
covered[constraint3] = true
covered[constraint4] = true
```

並記錄：

```go
chosen = append(chosen, candidate)
```

然後：

```text
進入下一層遞迴
```

---

### Step 5：Backtracking

如果遞迴發現：

```text
沒有任何 candidate 可以選
```

代表目前選擇造成了矛盾。

因此要回溯：

```text
取消 candidate
```

例如原本：

```text
covered[1]   = true
covered[39]  = true
covered[81]  = true
covered[111] = true
```

回溯後：

```text
covered[1]   = false
covered[39]  = false
covered[81]  = false
covered[111] = false
```

並：

```go
chosen = chosen[:len(chosen)-1]
```

接著嘗試下一個 candidate。

---

### Step 6. 終止條件

當：

```text
144 個 constraints
```

全部都被覆蓋：

```text
covered[0]  == true
covered[1]  == true
...
covered[143] == true
```

代表：

> 所有 Exact Cover 條件都被滿足。

此時：

```text
chosen
```

裡面就會包含完整的數獨答案。

最後將：

```text
candidate.row
candidate.col
candidate.value
```

重新填回：

```go
matrix[row][col] = value
```

---

## 9. 整體流程

可以把整個 Algorithm X 想成：

```text
6×6 Sudoku
    │
    ▼
建立 144 個 Constraints
    │
    ▼
建立所有合法 Candidate
    │
    │ 每個 Candidate：
    │ (row, col, value)
    │       +
    │ 4 個 constraints
    ▼
covered[144]
    │
    ▼
找「最少 candidate」的 constraint
    │
    ▼
選一個 candidate
    │
    ├── 成功
    │     │
    │     ├── 標記 4 個 constraints
    │     ├── 加入 chosen
    │     └── 遞迴
    │
    └── 失敗
          │
          ├── 移除 chosen
          └── 還原 4 個 constraints
                  │
                  └── 嘗試下一個 candidate
```

## 核心概念

```text
Candidate
    ↓
同時滿足 4 個條件
    ↓
選擇 Candidate
    ↓
覆蓋 4 個 Constraints
    ↓
繼續找下一個 Constraint
    ↓
如果走不下去
    ↓
Backtracking
```
