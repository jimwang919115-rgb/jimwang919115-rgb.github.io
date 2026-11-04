---
name: "level-based-edu-game"
description: 用來製作「闖關 / 過關解鎖」型態的教育遊戲,包含有劇情的密室逃脫、解謎闖關:一關對應一個知識點、答對才解鎖下一關、機關謎題、線索卡、道具、分層提示、倒數或愛心。即使使用者只說「我要做一個讓學生一步步過關的遊戲」「把這份密室逃脫腳本做成網頁」也適用。內建十一種題型(選擇、複選、密碼鍵盤、文字、轉輪鎖、順序機關、計數天平、滑桿、放置配對、定位、旋轉水管)。核心工作是寫一份資料檔,用 scripts/build.js 組進範本,不是從零寫程式。
---

# 闖關教育遊戲(快速版)

目標:**10 分鐘內交出可以玩的第一版**,之後再跟使用者對話調整。第一版簡單沒關係,一定要出得來。

## 第一版流程

1. **讀設計書一次**(單獨用讀檔工具讀,不要和別的檔案一起 cat)。抓:故事、每一關的場景、線索、機關與正解、提示、常見錯誤、解說。被截斷時補讀到最後一關。
2. **每一關挑一個內建題型**(下一節)。設計書的機關內建做不到時(多根滑桿、箭頭網路、迷宮…),**改成最接近的內建題型**,記下來放在回覆最後。不要問使用者,直接做。
3. **用建立檔案的工具寫一個資料檔**(例如 `game-data.js`),格式見「範例」。不要用 bash heredoc 寫(引號會壞)。
4. **組裝並檢查**:`node <skill>/scripts/build.js game-data.js 遊戲名稱.html`。它會檢查語法、必填欄位、題型欄位、道具鏈、常見錯誤是不是寫成了正解,不需要瀏覽器。有 ✘ 就照訊息修資料檔再跑。
5. **試玩(裝得起 jsdom 才做)**:
   ```bash
   mkdir -p /tmp/pt && cd /tmp/pt && npm install jsdom --no-audit --no-fund   # 不要在 home 資料夾裝
   node <skill>/scripts/playtest.js <遊戲.html 的完整路徑>
   ```
   有 ✘ 才修。**jsdom 裝不起來就跳過試玩**,build.js 通過就交付,在回覆裡說「沒有自動試玩」。**不准安裝 Playwright、Puppeteer 或任何瀏覽器。**
6. **回覆**(見最後一節)。

第一版**不要**:讀 `assets/template.html`(build.js 會處理)、讀 references/ 的長範例、新增題型(`LEVEL_TYPES`)、截圖、改引擎。

## 內建題型

| type | 機關 | 欄位 |
|---|---|---|
| `choice`(預設) | 選擇題、選一個檔位 | `choices`, `answerIndex`(從 0 算,正解位置要分散) |
| `multi` | 複選、同時接通幾條線 | `choices`, `answerIndexes: [0, 2]` |
| `passcode` | 數字密碼鎖 | `answer: "3576"`(一律字串,只能 0-9 * #),`slots?` 多給幾格才不洩漏位數 |
| `text` | 輸入單字、有小數點的數 | `answer: "apple"` 或陣列(多個都算對) |
| `dial` | 字母鎖、時鐘、多個旋鈕 | `answer: "GOLD"`;或 `answer: ["11","45"]` + `wheels: [[…],[…]]` + `sep: ":"`;`labels?` 每個轉輪的小標 |
| `sequence` | 照順序按、踩地磚 | `buttons`, `answer: [按鈕文字…]`, `cols?`, `strict?`(按錯立刻重來,mistakes 寫到錯的那一步),`blocked?: ["C3"]`(不能按的格子) |
| `counter` | + − 調數量、天平、量杯 | `answer: 10`, `min?`, `max?`, `step?`, `unit?`, `visual?: "balance"` 或 `"cup"` |
| `slider` | 拖滑桿調角度、溫度(一關一根) | `answer`, `min`, `max`, `step?`, `unit?`, `tolerance?`, `visual?: "compass"` |
| `assign` | 放進格子、配對、分類 | `slots`, `items`(可多放干擾物), `answer: [每格的物件或 null]`;分類用 `multiple: true` + `answer: [[…],[…]]` |
| `locate` / `pipes`,`assign` 疊在圖上、線索卡疊圖 `align` | 在地圖上指位置、旋轉水管、拖到圖上、疊格片讀數 | 需要畫 SVG:**先讀 `references/art.md`**,欄位見 `references/types.md` |

每一關共同欄位(都選填,`question` 必寫):`title`、`scene`(場景一兩句)、`question`(玩家要做什麼)、`clues`(線索卡:`"文字"` 或 `{ title, text, reveal: "翻開的按鈕文字" }`)、`hints`(三層:方向 → 推理 → 答案)、`success`(過關劇情)、`explanation`(答對後的解說,會停住)、`mistakes`(`{ "錯誤答案": "點出迷思" }`)。文字裡 `\n` 換行;長文字用反引號字串。

`mistakes` 的錯誤答案寫成學生操作出來的樣子:選擇題寫選項文字;複選用「、」接起選到的選項(`"陽光、菇"`);順序用「→」接起按鈕(`"A1→B2"`);放置依格子順序用「、」接起,空格寫「空」、同一格多個用「+」;密碼、文字、計數、滑桿寫輸入的內容或數字;轉輪寫轉出來的樣子(含 `sep`)。寫錯格式 build.js 會擋下。設計書有常見錯誤就照抄;算不出具體錯誤答案的,寫進 `hints`。

道具(選填):`const ITEMS = { lens: { name: "放大鏡", icon: "🔍", desc: "…" } };`,某一關 `reward: ["lens"]`,後面的線索卡 `needs: "lens"`。`needs` 的道具一定要是前面關卡給的。

`GAME`:必寫 `title`、`desc`;建議 `story`、`ending`、`theme`(`"cute"` 低年級、`"warm"` 烘焙農場、`"parchment"` 歷史古宅、`"scifi"` 實驗室太空、`"terminal"` 駭客駕駛艙)、`icon`(emoji)。選填:`timeLimit`(秒,0 不限)、`lives`(0 不限)、`failTo`(`"level"` 從這關重來,預設;`"start"` 從頭)。沒寫的 build.js 會補。

## 硬性限制

- 單檔 HTML,不用 CDN、外部字型、外部圖片(build.js 產出的檔案已符合)。圖示用 emoji。
- 插圖常數(`const MAP = \`<svg…>\``)要寫在 `LEVELS` 前面。
- 不要用 `const`/`let` 宣告 `el`、`mk`、`state`、`shuffle` 這類短名字(會和引擎撞名)。
- 不加登入、token;成績上傳只用 Google Apps Script(第二版再說)。

## 範例(資料檔的完整格式)

```js
const ITEMS = { key: { name: "銅鑰匙", icon: "🗝️", desc: "可以打開書櫃的抽屜" } };
const LEVELS = [
  { type: "passcode", title: "第一關:黑板上的算式",
    scene: "教室門被鎖住了,黑板上留著一道算式。",
    clues: [{ title: "黑板", text: "3/4 + 1/8 = ?/8\n分子就是門鎖的密碼。" }],
    question: "輸入門鎖的密碼。", answer: "7",
    hints: ["先把 3/4 改成分母是 8 的分數。", "3/4 = 6/8,再加 1/8。", "6/8 + 1/8 = 7/8,密碼是 7。"],
    mistakes: { "4": "分母不同不能直接把分子相加:3/4 要先通分成 6/8。" },
    success: "喀噠一聲,門開了!", reward: ["key"],
    explanation: "異分母分數相加,要先通分成同分母,再把分子相加。" },
  { type: "dial", title: "第二關:書櫃的轉輪鎖",
    clues: [{ title: "抽屜", text: "鎖上寫著:1/2 + 1/3 = ?/6", needs: "key" }],
    question: "把轉輪轉到正確的分子。", answer: "5", labels: ["分子"],
    hints: ["公分母是 6。", "1/2 = 3/6、1/3 = 2/6。", "3 + 2 = 5。"],
    mistakes: { "2": "1/2 + 1/3 不是把分子、分母各自相加(2/5)喔。" } }
];
const GAME = {
  title: "🏫 分數密室", desc: "解開兩道分數機關,逃出教室!", theme: "warm", icon: "🧮",
  story: "放學後你被反鎖在教室裡……", ending: "你成功逃出教室!", lives: 3
};
```

## 回覆裡要寫

1. 檔案位置,和怎麼玩(三四句)。
2. 檢查結果(build.js ✔;試玩全部 ✔,或沒有跑)。
3. **第一版的簡化**:設計書的哪些機關改成了什麼。
4. **第二版可以加的**:條列 3~6 項(例如照設計書的機關、插圖、道具鏈、倒數、給老師的說明、成績上傳),請使用者挑。

## 第二版以後:什麼時候讀哪一份

| 使用者要的 | 讀 |
|---|---|
| 答錯解說、提示、故事、給老師說明、道具、倒數愛心、外觀 | `references/second-version.md` |
| 某個題型的選填欄位、`mistakes` 格式、腳本的機關該用哪個題型 | `references/types.md`(grep 題型名稱) |
| 插圖、地圖定位 `locate`、拖到圖上 `board`、疊圖 `align`、水管 `pipes` | `references/art.md` |
| 內建題型做不到的機關(新增題型) | `references/custom-types.md`;卡住才看 `references/examples/` |
| 上傳成績到 Google 試算表 | `references/gas-deploy.md` |
| 完整品質檢查 | `references/checklist.md` |

一次只讀需要的那一份。改完照樣用 build.js 重新組裝。
