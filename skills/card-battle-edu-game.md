---
name: "card-battle-edu-game"
description: 用來製作「卡牌對戰」型態的教育遊戲:抽卡、陣營加成、戰鬥 PK、穿插「機會命運」問答(答對加分、答錯扣分),單人對電腦或多人同一台裝置輪流(分組投影)。也能做合成、交易、多行動等自訂規則。當使用者要做抽卡、卡牌、陣營對抗類的學習遊戲,或拿卡牌桌遊規則、卡牌設計書說要「做成網頁版」時使用,即使只說「我要做一個抽卡遊戲考同學問題」也適用。核心工作是寫一份資料檔,用 scripts/build.js 組進範本,不是從零寫程式。
---

# 卡牌對戰教育遊戲(快速版)

目標:**10 分鐘內交出可以玩的第一版**,之後再跟使用者對話調整。第一版簡單沒關係,一定要出得來。

## 第一版流程

1. **讀設計書一次**(單獨用讀檔工具讀,不要和別的檔案一起 cat)。只抓:遊戲名稱與故事、陣營/派別、卡牌清單、問答題庫(題目、答案、解說、常見錯誤)。被截斷時補讀卡牌和題庫,**題庫要讀到最後一題**;平衡驗算、範例回合可以跳過。
2. **對應到內建玩法**(下一節)。設計書裡內建做不到的規則(換幕、出牌、訂單、阻力卡、道具、電腦策略…)**先簡化成最接近的內建玩法**,記下來放在回覆最後。第一版的玩法和設計書差很多是正常的:保住設計書的**卡牌內容和題庫**就夠了。不要問使用者,直接做。
3. **用建立檔案的工具寫一個資料檔**(例如 `game-data.js`),格式見「資料欄位」和「範例」。不要用 bash heredoc 寫(引號會壞)。
4. **組裝並檢查**:`node <skill>/scripts/build.js game-data.js 遊戲名稱.html`。它會檢查語法、陣營、卡片、問答(`mistakes` 要和選項一字不差)、撞名,不需要瀏覽器。有 ✘ 就照訊息修資料檔再跑。
5. **試玩一次**(只跑一次、局數少):
   ```bash
   mkdir -p /tmp/pt && cd /tmp/pt && npm install jsdom --no-audit --no-fund   # 不要在 home 資料夾裝
   node <skill>/scripts/playtest.js <遊戲.html 的完整路徑> 3
   ```
   有 ✘ 才修;全部 ✔ 就交付。**jsdom 裝不起來就跳過試玩**,build.js 通過就交付,在回覆裡說「沒有自動試玩」。**不准安裝 Playwright、Puppeteer 或任何瀏覽器。**
6. **回覆**(見最後一節)。

第一版**不要**:讀 `assets/template.html`(引擎很大,build.js 會處理)、讀 references/ 的長範例、自己裝瀏覽器或 Playwright、跑平衡模擬、截圖、改引擎。

## 內建玩法

每位玩家分到一個陣營。輪到你就按「抽下一張」:每人整局抽 `handSize` 張卡,每抽 `questionInterval` 張穿插一題問答(答對 +`correctBonus`、答錯 −`wrongPenalty`,答完顯示常見錯誤與解說)。抽完就結算。

計分 = 自己陣營的卡 × 倍率 + 別陣營的卡 + 問答加減分。自己陣營有 `role: "king"` 的卡 → ×2;再有 `role: "beast"` → ×3。另有可關掉的「戰鬥」:總分到 `battleThreshold` 可挑戰一次(`battlesPerPlayer: 0` 就沒有戰鬥)。

對應方式:設計書的派別/隊伍/卡牌分類 → `FACTIONS`(陣營是卡的分類,玩家開局隨機分到一個);每個陣營挑 1 張最核心的卡(代表人物、招牌)當 `king`、1 張次要關鍵卡當 `beast`,其他卡用自己取的 role。卡片的知識寫在 `desc`。數字沒有依據時:`baseScore` 1~10(重要的卡高),問答 `correctBonus: 2, wrongPenalty: 1`,正解位置依題號輪流放 0、1、2、3。

## 資料欄位

| 名稱 | 內容 |
|---|---|
| `FACTIONS` | `[{ key, label, color }]`,2~4 個 |
| `ROLES` | `[{ key, label }]`。`king`、`beast` 這兩個 key 有加成,label 隨意改 |
| `CARD_POOL` | `[{ id, faction, role, name, baseScore, desc, copies }]`。`baseScore` 是數字;`desc` 一行知識(選填);`copies` 幾張(預設 1)。每個陣營的卡要比 `handSize` 多;多人時總張數(含 copies)≥ 人數 × `handSize` |
| `QUESTION_CARDS` | `[{ id, question, choices, answerIndex, correctBonus, wrongPenalty, explain, mistakes }]`。`answerIndex` 從 0 算,**正解位置要分散**;`explain` 每題一句;`mistakes` 是 `{ "錯誤選項原文": "迷思說明" }`,設計書有就照抄,沒有就省略 |
| `GAME` | 必寫 `title`、`desc`、`howToPlay`(`<ol>` 條列);建議 `story`、`theme`(`"dark"` `"warm"` `"paper"` `"scifi"`,歷史用 paper、低年級用 warm、理化用 scifi)。選填:`handSize`(預設 5)、`questionInterval`(預設 2)、`battlesPerPlayer`、`modes`(預設單人+多人)、`maxPlayers`、`bots`(電腦玩家,例 `{ solo: [1, 3] }`,會自動抽牌答題)、`text`(改用詞,例 `{ questionTitle: "📜 史料題", questionName: "史料題" }`) |

每局每人約考 `handSize ÷ questionInterval` 題。題庫比這多沒關係,每局隨機抽。

## 硬性限制

- 單檔 HTML,不用 CDN、外部字型、外部圖片(build.js 產出的檔案已符合)。
- 資料檔只放資料和 `function` 宣告;不要用 `const`/`let` 宣告 `el`、`state`、`shuffle`、`cur` 這類短名字(會和引擎撞名)。
- 不加登入、token;成績上傳只用 Google Apps Script(第二版再說)。

## 範例(資料檔的完整格式)

```js
const FACTIONS = [
  { key: "mammal", label: "哺乳類隊", color: "#e0556f" },
  { key: "bird",   label: "鳥類隊",   color: "#5b8def" }
];
const ROLES = [
  { key: "king",   label: "代表物種" },   // 有自己隊的代表物種 → 同隊卡 ×2
  { key: "beast",  label: "稀有物種" },   // 再加稀有物種 → ×3
  { key: "common", label: "常見物種" }
];
const CARD_POOL = [
  { id: 1, faction: "mammal", role: "king",   name: "臺灣黑熊", baseScore: 8, desc: "胎生、用乳汁哺育" },
  { id: 2, faction: "mammal", role: "beast",  name: "穿山甲",   baseScore: 6, desc: "有鱗片,仍是哺乳類" },
  { id: 3, faction: "mammal", role: "common", name: "蝙蝠",     baseScore: 3, desc: "會飛的哺乳類", copies: 2 },
  { id: 4, faction: "bird",   role: "king",   name: "臺灣藍鵲", baseScore: 8, desc: "卵生、有羽毛" }
  // ……每個陣營的卡要比 handSize 多
];
const QUESTION_CARDS = [
  { id: 1, question: "蝙蝠屬於哪一類動物?", choices: ["鳥類", "哺乳類", "爬蟲類"], answerIndex: 1,
    correctBonus: 3, wrongPenalty: 1,
    explain: "蝙蝠胎生、用乳汁哺育幼體,所以是哺乳類。",
    mistakes: { "鳥類": "會飛不代表是鳥;鳥類有羽毛、卵生。" } }
];
const GAME = {
  title: "🐾 動物分類大對決",
  desc: "抽卡收集動物,答對分類問題加分!",
  story: "保育中心要組兩支調查隊……",
  theme: "warm",
  howToPlay: "<ol><li>按「抽下一張」。</li><li>收集自己隊的代表物種,同隊卡分數 ×2。</li><li>答對問答加分。</li></ol>",
  handSize: 6, questionInterval: 2, battlesPerPlayer: 0
};
```

## 回覆裡要寫

1. 檔案位置,和怎麼玩(三四句)。
2. 試玩結果(全部 ✔,或沒有跑)。
3. **第一版的簡化**:設計書的哪些規則改成了什麼。
4. **第二版可以加的**:條列 3~6 項(例如照設計書的特殊規則、電腦對手、答錯解說、給老師的說明、成績上傳),請使用者挑。

## 第二版以後:什麼時候讀哪一份

| 使用者要的 | 讀 |
|---|---|
| 調外觀、用詞、故事、給老師說明、答錯解說、電腦對手、卡面樣式、改數字 | `references/second-version.md` |
| 設計書的特殊規則(換幕、阻力卡、出牌、合成、交易、自訂電腦策略) | `references/custom-rules.md`;卡住才看 `references/examples/` 的完整範例 |
| 上傳成績到 Google 試算表 | `references/gas-deploy.md` |
| 要完整品質檢查(平衡模擬、截圖、逐項驗收) | `references/checklist.md` |

一次只讀需要的那一份。改完照樣用 build.js 重新組裝、playtest 跑 3 局。
