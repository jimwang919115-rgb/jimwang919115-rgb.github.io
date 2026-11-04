---
name: "pixel-explore-edu-game"
description: 用來製作「像素風地圖探索」型態的教育遊戲,畫面像 GBA 時代的寶可夢:像素圖塊與角色、地圖比畫面大、鏡頭跟著玩家。地圖上放答對才收服的角色(站在地圖上或躲在高草叢)、給線索的 NPC、告示牌、寶箱、鑰匙,以及要道具或答對題目才打得開的門。當使用者想做走地圖抓怪、寶可夢式、尋寶探險、找鑰匙開門的學習遊戲,或給了一份地圖探索設計書時使用,即使只說「我要做一個走地圖抓寶可夢的遊戲考同學問題」也適用。只有一張地圖。核心工作是寫一份資料檔,用 scripts/build.js 組進範本,不是從零寫程式。
---

# 像素探索教育遊戲(快速版)

目標:**10 分鐘內交出可以玩的第一版**,之後再跟使用者對話調整。第一版簡單沒關係,一定要出得來。

## 第一版流程

1. **讀設計書一次**(單獨用讀檔工具讀,不要和別的檔案一起 cat)。抓:故事、文字地圖與圖例、NPC 台詞、帶題目的角色(題目、常見錯誤、解說)、道具與門、結局。被截斷時補讀到結局。
2. **地圖不要自己轉換**:把設計書的文字格子地圖整塊貼進 `MAP_TEXT`(座標尺、行首行尾的 y 標示可以留著),再寫 `MAP_LEGEND` 把設計書的字元換成引擎的字元(下一節)。
3. **角色只用內建圖和換色版本**(見「內建圖」),不畫新圖。設計書做不到的(主動開口的 NPC、動畫、依進度改變的台詞…)簡化,記下來放在回覆最後。不要問使用者,直接做。
4. **用建立檔案的工具寫資料檔**(例如 `game-data.js`),格式見「範例」。不要用 bash heredoc 寫(引號會壞)。
5. **組裝並檢查**:`node <skill>/scripts/build.js game-data.js 遊戲名稱.html`。它會換地圖、檢查每列長度、mark、走不走得到、鑰匙和門的順序、過關條件、`mistakes`,不需要瀏覽器。有 ✘ 就照訊息修資料檔再跑。
6. **試玩(裝得起 jsdom 才做)**:
   ```bash
   mkdir -p /tmp/pt && cd /tmp/pt && npm install jsdom --no-audit --no-fund   # 不要在 home 資料夾裝
   node <skill>/scripts/playtest.js <遊戲.html 的完整路徑>
   ```
   有 ✘ 才修。**jsdom 裝不起來就跳過試玩**,build.js 通過就交付,在回覆裡說「沒有自動試玩」。**不准安裝 Playwright、Puppeteer 或任何瀏覽器**,第一版不截圖。

第一版**不要**:讀 `assets/template.html`、讀 references/ 的長範例、畫像素圖、跑 `pixelize.py`、改引擎。

## 地圖字元(MAP_LEGEND 換成這些)

`.` 草地、`,` 高草叢(遇草叢角色)、`*` 花、`=` 路、`+` 木橋、`T` 樹、`~` 水、`O` 石頭、`#` 柵欄/石牆、`H` 房子(以上前五種能走)、`@` 起點(一個)。
- **NPC、告示牌、寶箱、站在地圖上的角色** → `1`~`9` 或 `! ? $ % & < > [ ] ^ : ; |`(最多 23 個)
- **地上撿的道具** → 小寫 `a`~`z`(小寫只能當道具;設計書用小寫字母當 NPC 的一定要換)
- **門、終點** → 大寫字母(`T` `O` `H` 除外)
- 設計書的高草叢 `"` → `,`;起點 `S` → `@`;裝飾岩、山崖 → `O` 或 `#`

`MAP_LEGEND` 只寫要換的字元,一個字元對一個字元;沒列的照原樣。

## 資料欄位

`CREATURES`:每隻 `{ id, name, where: "map" 或 "grass", mark(map 才要), sprite }`,加上:
- 題目:`question`、`choices`、`answerIndex`(從 0 算,**正解位置要分散**)、`explanation`、`mistakes: { "錯誤選項原文": "點出迷思" }`(key 要和選項一字不差)、`hint`(沒指定選項的一句提醒)。好幾題輪流用 `questions: [{…}, {…}]`。
- 台詞:`greet`(出場)、`success`(答對)、`area`(在哪一區,終點門會列出還差誰在哪)。
- **NPC / 告示牌 / 寶箱 = 不寫題目**,寫 `dialogue`(文字或陣列);告示牌 `sprite: "sign"`、寶箱 `sprite: "chest"`。NPC 永遠站在原地。`gives: "k"` 講完話或答對後送道具;`stays: true` 讓帶題目的角色答對後不消失;`floor` 是這一格底下的地形(角色讓開、門打開後也變成它),站在路上、橋上寫 `"="` / `"+"`,預設草地。

`ITEMS`:`[{ mark: "k", name, sprite: "key", note }]`,`note` 是背包可重看的紙條。由寶箱送的道具不用放在地圖上。

`GATES`:`[{ mark: "D", name, needs: "k", consume?, lockedText, openedText, floor: "=" }]`;答題才開的門寫題目欄位;**終點**寫 `needsAllCreatures: true`(全部收服才開,打開就過關)。鑰匙不能在自己那扇門後面(build.js 會擋)。

`GAME`:`title`、`desc`(故事)、`ending`、`teacherNotes`(給老師,不寫答案);選填 `grassEncounterRate`(高草叢每步遇到的機率,預設 0.12)、`grassPity`(連續幾步沒遇到就一定遇到,預設 12)、`text`(換畫面用語,例 `{ caught: "{name} 加入了隊伍" }`)。

## 內建圖

- 動物:`leafbunny` `bubblefin` `emberchick` `pebbleshell` `starbug`,各有換色 `_2`、`_3`(例 `starbug_3`),共 15 種外觀
- 人物 NPC:`villager_green` `villager_purple` `villager_orange` `villager_pink` `villager_elder`
- 其他:`sign` `chest` `door` `key` `gem` `berry`

## 硬性限制

- 單檔 HTML,不用 CDN、外部圖片(build.js 產出的檔案已符合)。不畫或複製現有遊戲的角色。
- 資料檔不要用 `const`/`let` 宣告 `el`、`state`、`shuffle`、`SPRITES` 這類引擎用的名字。
- 不加登入、token;成績上傳只用 Google Apps Script(第二版再說)。

## 範例(資料檔的完整格式)

```js
const MAP_TEXT = `
   0123456789
0  TTTTTTTTTT
1  T..a..b.XT
2  T.S...""@T
3  TTTTTTTTTT
`;
const MAP_LEGEND = { a: "1", b: "2", '"': ",", S: "@", "@": "W" };   // 設計書的 @ 是終點 → 門 W
const CREATURES = [
  { id: "n1", name: "村長", where: "map", mark: "1", sprite: "villager_elder", dialogue: ["往東走,精靈在草叢裡。"] },
  { id: "box", name: "寶箱", where: "map", mark: "2", sprite: "chest", dialogue: "裡面有一把鑰匙!", gives: "k" },
  { id: "c1", name: "等號兔", where: "grass", sprite: "leafbunny_2", area: "東邊草叢",
    question: "x + 3 = 7,x = ?", choices: ["10", "4", "3"], answerIndex: 1,
    mistakes: { "10": "移項要變號:+3 移到右邊變 −3,7 − 3 = 4。" },
    explanation: "x = 7 − 3 = 4。驗算:4 + 3 = 7 ✓" }
];
const ITEMS = [{ mark: "k", name: "銅鑰匙", sprite: "key", note: "可以打開銅門" }];
const GATES = [
  { mark: "X", name: "銅門", needs: "k", lockedText: "要鑰匙,寶箱裡好像有。" },
  { mark: "W", name: "大天平", needsAllCreatures: true, openedText: "天平平衡了!" }
];
const GAME = { title: "平衡島", desc: "找回會解方程式的精靈!", ending: "你成功了!" };
```

## 回覆裡要寫

1. 檔案位置,和怎麼玩(三四句)。
2. 檢查結果(build.js ✔;試玩全部 ✔,或沒有跑)。
3. **第一版的簡化**:設計書的哪些內容改成了什麼(例如角色外觀用換色版本)。
4. **第二版可以加的**:條列 3~6 項(例如照設計書畫角色、主動開口的 NPC、給老師的說明、成績上傳),請使用者挑。

## 第二版以後:什麼時候讀哪一份

| 使用者要的 | 讀 |
|---|---|
| 角色照設計書的外觀畫、用自己的圖 | `references/sprites.md` |
| 地圖設計原則、設計書地圖換字元的細節 | `references/map.md` |
| NPC、門、鑰匙、題目的完整欄位(`questions`、`consume`、兩題才開的門) | `references/gates-items.md` |
| 改過關條件 `isWin()`、畫面用語、走路速度等 `GAME` 設定、引擎 | `references/engine.md`;卡住才看 `references/examples/` |
| 上傳成績到 Google 試算表 | `references/gas-deploy.md` |
| 完整品質檢查(截圖、手機版) | `references/checklist.md` |

一次只讀需要的那一份。改完照樣用 build.js 重新組裝。
