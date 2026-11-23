# data/ 規範

實際的資料放在程式專案的 `data/` 中。這裡只說明分層與規則（詳見 `docs/data-and-config.md`）。

| 資料夾 | 用途 | 進版控 |
|---|---|---|
| `raw/` | 量測原始資料（`{溫度}c.dat`） | 小的可以進，大的另外存放 |
| `processed/` | 程式產生的各階段結果 | 否（加入 `.gitignore`） |

原始資料的格式與電阻欄位的計算，見 `docs/spec/extraction.md`。
