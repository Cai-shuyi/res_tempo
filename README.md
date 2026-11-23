# 工作流程與規範

這個專案只放文件，不放程式碼。它定義從需求到交付的流程、每個階段的確認點，以及各類文件與設定的寫法。新專案開始時，依照這裡的規範建立結構，再由 Claude 產生程式碼。

## 文件地圖

| 檔案 | 內容 | 讀者 |
|---|---|---|
| `docs/workflow.md` | 流程的六個階段、每個階段的輸入與輸出 | 人、Claude |
| `docs/roles-and-gates.md` | 哪些決定由人做，哪些由 Claude 做，以及確認點 | 人、Claude |
| `docs/documentation.md` | CLAUDE.md、README、規格、config、data、skill 各自怎麼寫 | 人、Claude |
| `docs/change-management.md` | 分支、commit、PR、合併與環境注意事項 | 人、Claude |
| `docs/data-and-config.md` | 資料與設定的命名、單位、缺項、版本紀錄 | 人、Claude |
| `templates/` | 各類文件的空白範本 | 人、Claude |

## 使用方式
1. 新專案開始前，先讀 `docs/workflow.md` 與 `docs/roles-and-gates.md`。
2. 依 `templates/` 建立該專案的 CLAUDE.md、README、規格與設定檔。
3. 每個階段完成時，對照「完成標準」確認。
4. 規範有變動時，先改這裡，再同步到專案。
