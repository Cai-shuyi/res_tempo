# 變更管理

## 分支
- 不直接修改主分支。每次變更使用一條功能分支。
- 分支名稱：`<類型>/<簡短主題>`，類型為 `feature`、`fix`、`docs`、`config`。
  - 例：`feature/load-contact-resistance`、`docs/workflow-spec`。
- 一條分支只處理一個主題。

## Commit
- 標題用一行，以動詞開頭，說明做了什麼。
- 內文說明為什麼做，以及影響的範圍。
- 不把不相關的改動放在同一個 commit。
- 不提交產生的檔案（例如 `data/processed/`、`*.db`，除非專案規定要提交）。

範例：
```
Add contact resistance option to load stage

rdc from config enables R_contact = 2 * rdc / nrd. Blank or 0 keeps the
previous behavior. Affects load and qa stages.
```

## Pull Request
- 每個 PR 只處理一個主題。
- PR 內容需要包括：
  - 變更摘要。
  - 規格變更（如有）。
  - 測試結果，包括執行的指令與結果。
  - 對 QA 或設定檔的影響。
  - 尚未確認的項目。
- 合併前，人必須審查。

## 中斷與還原
- 人要求中斷時，不再繼續推送。
- 中斷前的未提交改動，要在回報中列出，由人決定保留或還原。
- 還原使用 `git checkout -- <檔案>`，只還原指定的檔案，不使用會影響全部檔案的指令。

## 環境注意事項
- 如果 clone 是 shallow clone，推送前先設定對應分支的 fetch refspec，否則遠端追蹤分支不存在，檢查工具可能誤判為未推送。
- repo 改名後，本地 `origin` 的網址要同步更新。
- 推送失敗時，只對網路錯誤重試，最多四次，間隔依序為 2、4、8、16 秒。
