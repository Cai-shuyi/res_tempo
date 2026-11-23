# 標準目錄結構

新專案依此結構建立。`src/` 的寫法在有新舊兩版程式時，新流程放在 `polyres/`，舊程式只作對照。

```
CLAUDE.md                              # 給 Claude 的規則、檔案地圖、修改流程
config/
├── extraction.yaml                    # 萃取參數與初始值（GUI 預設值 + params_init）
├── metadata.yaml                      # 每個輸出參數的意義、單位、netlist 名稱
└── qa.yaml                            # QA 檢查規則與門檻
data/
├── README.md                          # 資料格式與輸出命名
├── raw/                               # 量測原始資料（{溫度}c.dat，進版控）
└── processed/                         # 程式產生的結果（已加入 .gitignore）
src/
├── polyres/                           # 新流程：load、features、model、qa
└── poly_resistor_spice_extract.py     # 原程式，只作對照，不修改
```

沒有舊程式的新專案，`src/` 只放新流程即可。
