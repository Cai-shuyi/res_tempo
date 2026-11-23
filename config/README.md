# config/ 規範

實際的設定檔放在程式專案的 `config/` 中。這裡只說明每個檔案的用途，以及寫法的規則（詳見 `docs/data-and-config.md`）。

| 檔案 | 用途 |
|---|---|
| `extraction.yaml` | 萃取參數與初始值（GUI 預設值與 params_init） |
| `metadata.yaml` | 每個輸出參數的意義、單位、netlist 名稱 |
| `qa.yaml` | QA 檢查規則與門檻；未確認的門檻為 `null` |

規則：
- 每個項目都要有註解，寫明用途、單位、預設值與來源。
- 數值用帶小數點的科學記號，例如 `1.0e-6`。
- 設定檔不放密碼、token 或金鑰。
