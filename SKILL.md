---
name: commit
description: 自動分析變更並建立 git commit
allowed-tools: Bash(git *), TodoWrite
---

請依照以下步驟建立 git commit：

## 步驟一：分析變更

1. 執行 `git status` 查看所有變更檔案（不要使用 -uall）
2. 若無任何變更，告知使用者「目前沒有需要 commit 的變更」並停止
3. 執行 `git diff --name-only` 和 `git diff --staged --name-only` 快速掌握變更範圍
4. 針對有意義的變更檔案執行 `git diff <file>` 查看具體內容
5. 執行 `git log --oneline -5` 參考最近的 commit 風格

## 步驟二：規劃 commit 策略

- 判斷所有變更是否屬於**同一主題**
  - 若是 → 建立單一 commit
  - 若否 → 規劃多個 commit，依主題分批暫存與提交
- 用 `TodoWrite` 記錄每個待完成的 commit 計畫

## 步驟三：暫存檔案

- 逐一加入相關變更檔案（`git add <file>`）
- **不要**使用 `git add -A` 或 `git add .`
- **不要**加入敏感檔案（如 .env、credentials 等）

## 步驟四：撰寫 commit 訊息

使用 Conventional Commits 格式，訊息以**繁體中文**撰寫：

```
<type>(<scope>): <簡短描述>

<Body：詳細說明（可選）>

<Footer（可選）>
```

### type 類型

- `feat`：新增功能
- `fix`：修復錯誤
- `refactor`：重構（不改變功能）
- `docs`：文件變更
- `style`：格式調整（不影響邏輯）
- `test`：測試相關
- `chore`：雜務（建置、設定等）
- `perf`：改善效能
- `revert`：撤銷先前的 commit，格式：`revert: <type>(<scope>): <原描述>`
  Body 必須包含：`This reverts commit <SHA>`，並說明撤銷原因

### scope 範圍

- 使用變更的檔案名稱或模組名稱作為 scope
- 例如：`feat(auth)`, `fix(user-service)`, `docs(readme)`

### 描述規則

- 用繁體中文簡潔描述「做了什麼」
- **不超過 50 個字元**
- **不要**在 `<type>` 後加句號，例如不應寫成 `fix. 修正回上一頁邏輯`
- **不要**在描述結尾加句號
- 例如：`feat(login): 新增登入功能`、`fix(api): 修復回傳格式錯誤`

### Body 規則（可選）

- 說明「為什麼（Why）」做這個變更，以及「做了什麼（What）」
- 每行不超過 **72 個字元**
- **每個 bullet 對應一個 diff chunk（`@@` 範圍）**，說明中須標明「檔名 L起始-L結束」
- 每個項目以 ` - ` 開頭（前有一個空格）作為前綴，每項之間空一行
- 例如：
  ```
   - login.html L12-L18：統一編號欄位新增 inputmode="numeric"
  以在行動裝置彈出數字鍵盤、maxlength="8" 限制最多 8 碼輸入

   - login.html L25：密碼欄位新增 name="password"
  讓密碼管理器正確識別欄位
  ```

### Footer 規則（可選）

- 若有對應 issue，填寫 `issue #<編號>`
- 若有不兼容的破壞性變更，以 `BREAKING CHANGE:` 開頭說明

## 步驟五：執行 commit

使用 HEREDOC 格式執行 commit：

```bash
git commit -m "$(cat <<'EOF'
<type>(<scope>): <描述>
EOF
)"
```

若有多個 commit 計畫，依序重複步驟三至五。

## 步驟六：呈現 commit 結果

commit 成功後，以下列格式輸出結果：

### 格式規則

1. 執行 `git show --stat HEAD` 取得實際變更行數
2. 針對每個變更檔案執行 `git show HEAD -- <file> | grep "^@@"` 取得精確行號範圍
3. 依以下格式輸出：

---

## 📝 Commit 內容

`<type>(<scope>): <描述>`

- [<檔名>](<相對路徑>#L<起始行>-L<結束行>)
  - <chunk 1 描述>
  - <chunk 2 描述>
- [<檔名>](<相對路徑>#L<起始行>-L<結束行>)
  - <chunk 描述>

---

### 注意事項

- **上方須有 `---` 分隔線、`## 📝 Commit 內容` 標題、下方須有 `---` 分隔線**，不可省略
- 以**檔案為單位**分組，每個檔案一條連結，連結文字只顯示檔名，**不顯示行號**
- 行號只放在連結 URL 中（`#L起始-L結束`），範圍取該檔案所有 chunk 的首尾行
- 各 chunk 的變更描述列為該連結的子項目（縮排 `  - `）
- 連結路徑使用**相對路徑**（從專案根目錄起）
- 行號必須根據 `git show` 實際輸出，不可自行推測
