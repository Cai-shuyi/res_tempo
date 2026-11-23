# 萃取規格：Poly resistor SPICE 模型

## 狀態
已確認（使用者於 2026-11-23 確認 load 階段的電阻欄位規則；其餘階段依既有流程）

## 目的
從多溫度、多尺寸的量測資料（`.dat`）萃取 poly resistor 的 SPICE 模型參數，輸出 netlist 與 QA 圖形。

## 流程總覽

```
load -> features -> model -> qa
```

每個階段的輸出都包含前一階段的全部內容，所以任一階段可以單獨執行，只要前一階段的結果存在。

| 階段 | 輸入 | 輸出 |
|---|---|---|
| load | 原始 `.dat` | raw 資料（含電阻欄位） |
| features | load 的結果 | 每個 W/L/T 的 R0、Rsh0、tc1r、tc2r、vcoeff |
| model | features | 全域模型參數、netlist |
| qa | model 的結果 | 圖形與檢查數字 |

---

## 階段 1：load

### 原始檔案格式
- 每個溫度一個檔案，檔名 `{溫度}c.dat`。
- 一個檔案可以有多個區塊。每個區塊由一行標頭開始，接著是表格。
- 標頭範例：
  ```
  Page(1) {W=0.01,L=2,T=25.0,nrd=2}
  ```
- 標頭提供 `W`、`L`、`T`、`nrd`。表格提供 `I`、`V`。
- 表格的其他欄位忽略。
- 分隔符號為 Tab。
- 區塊之間以空白行分隔。
- 標頭中 `L=2.T=25.0` 這種點號寫法也要能讀出 `L=2`、`T=25`。
- `T` 以標頭為準，不從檔名解析。
- `nrd` 可以沒有，沒有時視為缺項。

### 尺寸與 contact 的對應
- 同一個 `(W, L)` 對應一個 `nrd`。同一個 `(W, L)` 的 `nrd` 不一致時，報錯。

### 計算
| 欄位 | 公式 |
|---|---|
| `R_total_data` | `V / I` |
| `R_contact` | `2 * rdc / nrd`（只在考慮 contact 時） |
| `R_main_data` | `R_total_data - R_contact`；不考慮 contact 時等於 `R_total_data` |
| `Rsh_data` | `R_main_data * (W / L)` |

- `R_total_data`、`R_main_data`、`Rsh_data` 只能由 `W`、`L`、`V`、`I`、`rdc`、`nrd` 計算，不從檔案讀取。
- `W`、`L` 乘上 `input.scale` 後再參與計算。`Rsh_data` 的比值不受 scale 影響。

### contact 的判斷
- `rdc` 來自設定檔 `preprocessing.rdc`，單位 Ohm。
- `rdc` 為空白或 0：不考慮 contact。沒有獨立的開關。
- `rdc` 有值，但有任何一列缺少 `nrd`，或 `nrd` 不為正數：整個模型都不考慮 contact，視為缺項。
- 判斷以整個模型為單位，不逐列判斷。
- 判斷結果與原因記錄在 manifest。

### 缺值處理
- `I` 為 0 或缺值的列丟棄，丟棄的列數記錄在 manifest。
- `V` 缺值的列丟棄。

### 輸出
`1_load/raw.tsv`，欄位：

| 欄位 | 說明 |
|---|---|
| `W` | 乘上 scale 後的寬度 |
| `L` | 乘上 scale 後的長度 |
| `temp` | 來自標頭 `T` |
| `I` | 電流 |
| `V` | 電壓 |
| `R_total_data` | 見上方計算 |
| `R_main_data` | 見上方計算 |
| `Rsh_data` | 見上方計算 |

`nrd` 與 `rdc` 不輸出到 raw.tsv。它們的值記錄在 manifest。

---

## 階段 2：features

- 使用 `R_main_data` 做中值濾波，不使用 `R_total_data`。
- `Rsh_data` 直接使用 load 階段的計算結果，不在這個階段重算。
- 對稱處理（`preprocessing.symmetry_flag`）在這個階段套用。
- 輸出每個 `(W, L, temp)` 的 R0、Rsh0、tc1r、tc2r、vcoeff。
- 中值濾波視窗大小來自 `preprocessing.median_window`，必須為奇數。

---

## 階段 3：model

- 依 `selection` 篩選 W、L 範圍；`null` 代表使用資料的最小值與最大值。
- 依 `fit_sizes` 選擇擬合用的尺寸；`null` 代表自動選擇極值。
- 依 `initial_values` 提供最佳化的初始猜值。
- 輸出全域參數（`model_params.yaml`）、每個元件的參數、全域參數文字檔與 netlist。

---

## 階段 4：qa

### R_total_data 檢查（必要）
每次 QA 都必須執行這項檢查，不可省略。
- **回推比對**：重新計算 `V / I`，與 `R_total_data` 比對，記錄最大相對誤差。
- **檢查 Rsh_data**：比對 `Rsh_data` 與 `R_main_data * (W / L)`，記錄最大相對誤差。
- **作圖**：`R_total_data` 與 `R_main_data` 對 V 的散點圖，依溫度分色。
- 最大誤差寫入 manifest。

### 其他圖形
- 3D 曲面：`rsh0`、`tc1r`、`tc2r`、`vcoeff` 對 log(W)、log(L)，每個溫度一張。
- Rsh 對電壓：第一階段與第二階段各一組，每個 W/L 一張。
- 百分比長條圖：`R0` 與 `vcoeff` 的模型差異。

### 門檻判定
- 目前 QA 只產生圖形與數字，不判定通過或失敗。
- 門檻定義在 `config/qa.yaml`，未確認的門檻為 `null`，不由 Claude 決定。

---

## 待確認
- `temp` 欄位名稱：load 輸出目前用 `temp`，與原始標頭的 `T` 不同。是否統一改為 `T`，待確認。
- contact 回推檢查：raw.tsv 不含 `nrd`，因此無法在 QA 中回推 `R_main_data + 2*rdc/nrd = R_total_data`。是否要在輸出中保留 `nrd`，待確認。
- `rdc` 的單位：目前假設為 Ohm，待確認。
- 原程式的已知不一致（netlist 的 `Lterm`、`np.any` 替換範圍）：修改前需先確認預期行為。
