---
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
- **以檔案為單位分組**，每個檔案一條 ` - ` 開頭的 bullet，**不顯示行號**
- 同一檔案有多個 chunk 時，依以下原則決定是否展開子項目：
  - 若多個 chunk **屬於同一功能或目的** → 合併為一條描述，不展開子項目
  - 若多個 chunk **屬於不同功能或目的** → 各 chunk 描述列為縮排子項目（`   - `，前有三個空格）
- 每個頂層 bullet 之間空一行
- 例如（合併）：
  ```
   - MGSection_becomeSupplier.vue：新增品牌名稱（BrandName）欄位，
     支援多選 chips 輸入，舊資料向下相容
  ```
- 例如（展開）：
  ```
   - login.html：修正表單輸入體驗
     - 統一編號欄位新增 inputmode="numeric" 以在行動裝置
  彈出數字鍵盤、maxlength="8" 限制最多 8 碼輸入
     - 密碼欄位新增 name="password" 讓密碼管理器正確識別欄位
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

- **嚴禁**在 commit 訊息中附加 `Co-Authored-By` 或任何 AI 署名行

若有多個 commit 計畫，依序重複步驟三至五。

## 步驟六：呈現 commit 結果

commit 成功後，以下列格式輸出結果：

### 格式規則

1. 執行 `git show --stat HEAD` 取得實際變更行數
2. 針對每個變更檔案執行 `git show HEAD -- <file> | grep "^@@"` 取得每個 chunk 的精確行號範圍
3. **先決定每個檔案的描述策略（合併或展開）**，再同時套用到 commit message body 與 📝 輸出，確保兩者完全一致
4. 依以下格式輸出（合併時僅顯示一條子項目，連結指向主要變更區段）：

---

## 📝 Commit 內容

`<type>(<scope>): <描述>`

- [<檔名>](<相對路徑>)
  - [<chunk 1 描述>](<相對路徑>#L<chunk1起始行>-L<chunk1結束行>)
  - [<chunk 2 描述>](<相對路徑>#L<chunk2起始行>-L<chunk2結束行>)
- [<檔名>](<相對路徑>)
  - [<chunk 描述>](<相對路徑>#L<起始行>-L<結束行>)

---

### 注意事項

- **上方須有 `---` 分隔線、`## 📝 Commit 內容` 標題、下方須有 `---` 分隔線**，不可省略
- 以**檔案為單位**分組，每個檔案一條連結，連結文字只顯示檔名，**不加行號錨點**
- 各 chunk 的描述列為子項目，**每個子項目都是可點擊的連結**，連結到該 chunk 的具體行號範圍
- 行號必須根據 `git show` 實際輸出，不可自行推測
- 連結路徑使用**相對路徑**（從專案根目錄起）
- **commit message body 中的描述文字必須與 📝 輸出的子項目描述完全一致**，不可各寫各的
- 合併/展開的決策須與 Body 規則一致：同一功能的多個 chunk 合併為一條子項目，不同功能才展開為多條
