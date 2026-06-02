# 🎯 QA Hunter — Claude Code Skill

> 🐛 丟設計稿、丟規格、丟網址，QA 自己跑！Claude Code 技能，驗收結果自動回寫 Notion。

## 這是什麼？

**QA Hunter** 是一個 [Claude Code](https://claude.ai/code) Skill，讓你只要提供：

1. 🎨 **Figma 設計稿連結** — 截圖各 UI 狀態做視覺基準
2. 📄 **Notion 規格文件連結** — 解析功能需求與測試條件
3. 🌐 **測試網址** — UAT / staging 環境

Claude 就會自動逐項測試、遇到需要測試資料時引導你操作 DB SQL，最後把 ✅ PASS / ❌ FAIL / ⚠️ 待確認 的結果寫回 Notion。

---

## 安裝方式

### 步驟一：下載檔案

把這個 repo clone 下來，或直接下載 `SKILL.md` 和 `references/user-guide.md`。

```bash
git clone https://github.com/annahsueh-source/qa-hunter.git
```

### 步驟二：放到 Claude Code skills 資料夾

```bash
cp -r qa-hunter ~/.claude/skills/qa-hunter
```

### 步驟三：重啟 Claude Code

完全關閉後重新開啟 Claude Code（⌘+Q），在輸入框打 `/q` 就能看到 `qa-hunter` 出現。

---

## 使用方式

在 Claude Code 任意 session 中輸入：

```
/qa-hunter
```

或直接說：

- 「幫我測這個頁面」
- 「幫我 QA 驗收」
- 「這是 Figma 這是規格，幫我跑一遍」

Claude 會主動詢問缺少的資訊並開始測試流程。

---

## 需要什麼工具？

| 工具 | 用途 | 必要？ |
|------|------|--------|
| Claude Code | 執行 skill | ✅ |
| Claude in Chrome 擴充套件 | 瀏覽器自動化測試 | ✅ |
| Figma MCP | 讀取設計稿截圖 | ✅ |
| Notion MCP | 讀寫規格文件 | ✅ |
| Sequel Ace / 任何 DB 工具 | 測試資料操作（需要時） | 選用 |

詳細安裝說明請見 [`references/user-guide.md`](references/user-guide.md)。

---

## 檔案結構

```
qa-hunter/
├── SKILL.md              # Skill 主要指令（Claude 讀取）
└── references/
    └── user-guide.md     # 給使用者的說明書
```

---

Made with 🧡 by 薛安妤 — PressPlay
