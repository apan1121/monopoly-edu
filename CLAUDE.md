# CLAUDE.md

此檔案為 Claude Code (claude.ai/code) 提供在此程式碼庫工作時的指引。

## 專案概述

這是一款專為課堂教學設計的互動式大富翁遊戲，採用單一 HTML 檔案架構（包含 CSS 和 JavaScript），讓老師能夠透過遊戲讓學生完成真實學習任務。

**核心技術：**
- Vanilla JavaScript（無框架依賴）
- CSS3（含 3D Transforms、Grid、Flexbox）
- HTML5（LocalStorage、FileReader API、Blob API）

**專案特點：**
- 單檔案應用程式（`index.html`，約 1760 行）
- 無需建置工具或套件管理
- 可直接在瀏覽器開啟使用
- 無外部資源依賴

## 開發指令

### 本地開發與測試
```bash
# 直接在瀏覽器開啟檔案
open index.html              # macOS
start index.html             # Windows
xdg-open index.html          # Linux

# 或使用簡易 HTTP 伺服器（避免某些瀏覽器的 CORS 限制）
python3 -m http.server 8000  # Python 3
# 然後在瀏覽器開啟 http://localhost:8000
```

### Git 操作
```bash
# 查看狀態
git status

# 提交變更
git add index.html
git commit -m "描述變更內容"

# 推送到遠端
git push origin main
```

## 核心架構

### 檔案結構
```
index.html (約 1760 行)
├── CSS 樣式 (約 850 行)
│   ├── 基礎樣式與響應式設計
│   ├── 設定頁面樣式
│   ├── 遊戲棋盤樣式（Grid 佈局）
│   ├── 3D 骰子樣式（preserve-3d）
│   ├── 側邊欄與控制面板
│   ├── 任務彈窗
│   └── 結果頁面
├── HTML 結構 (約 150 行)
│   ├── 設定頁面 (#setup-page)
│   ├── 遊戲頁面 (#game-page)
│   ├── 任務彈窗 (#task-modal)
│   └── 結果頁面 (#result-page)
└── JavaScript 邏輯 (約 760 行)
    ├── 遊戲狀態管理
    ├── 玩家與任務設定
    ├── 棋盤初始化與渲染
    ├── 骰子系統（3D 動畫）
    ├── 玩家移動邏輯
    ├── 任務系統
    ├── 計分系統
    └── 匯入/匯出功能
```

### 主要狀態變數（行 1061-1074）
```javascript
let gameConfig = { cellCount, playerCount, taskCount, taskPoints, hiddenMode };
let players = [];              // 玩家陣列：{name, color, position, score}
let currentPlayerIndex = 0;    // 當前玩家索引
let board = [];                // 棋盤陣列：{type, task}
let isRolling = false;         // 骰子滾動狀態鎖
let visitedCells = new Set();  // 追蹤已訪問格子（迷霧模式）
let diceStats = {1:0, 2:0, 3:0, 4:0, 5:0, 6:0}; // 骰子統計
```

### 關鍵函數

**遊戲初始化：**
- `startGame()` (行 1260) - 從設定頁面啟動遊戲
- `initBoard()` (行 1334) - 初始化棋盤狀態與任務分配
- `renderBoard()` (行 1385) - 渲染蛇形棋盤 UI

**棋盤佈局：**
- 使用 CSS Grid 佈局
- 蛇形排列演算法：奇數行從左到右，偶數行從右到左
- 響應式：桌面版自動計算列數，手機版固定 4 列
- 不完整行使用 `gridColumnStart` 對齊

**骰子系統：**
- `createDice(number)` (行 1522) - 創建 3D 骰子 DOM
- `getDiceRotation(number)` (行 1553) - 計算骰子旋轉角度
- `rollDice()` (行 1567) - 擲骰子邏輯（含動畫與移動）
- 使用 CSS 3D transforms 實現立體效果

**玩家移動：**
- `movePlayer(steps)` (行 1615) - 逐格移動動畫（每格 0.3 秒）
- `scrollToCurrentPlayer()` (行 1505) - 自動追蹤玩家視角
- 移動過程中移除迷霧（`visitedCells.add(position)`）

**任務系統：**
- `showTask(taskText)` (行 1650) - 顯示任務彈窗
- `completeTask(success)` (行 1659) - 處理任務完成/失敗
- LocalStorage 持久化：`saveTasksToStorage()` / `loadTasksFromStorage()`

**設定匯入/匯出：**
- `exportSettings()` (行 1719) - 匯出 JSON 設定檔
- `importSettings(event)` (行 1757) - 匯入 JSON 設定檔

### 特殊機制

**蛇形棋盤演算法（行 1385-1498）：**
```javascript
// 計算列數：桌面版自動，手機版固定 4 列
const cols = isMobile ? 4 : Math.ceil(Math.sqrt(cellCount));
const rows = Math.ceil(cellCount / cols);

// 從最後一行往上渲染（蛇形排列）
for (let row = rows - 1; row >= 0; row--) {
  const isRightToLeft = (rows - 1 - row) % 2 === 1;
  // 奇偶行交替方向...
}
```

**迷霧模式：**
- 啟用時，未訪問格子覆蓋 `.fog` 樣式（`:before` 偽元素）
- 玩家經過即移除迷霧（不需停留）
- 使用 `Set` 資料結構追蹤已訪問格子

**防抖處理：**
- 視窗大小改變事件（行 1822）使用 300ms 防抖
- 避免頻繁重新渲染棋盤

## 開發注意事項

### 修改程式碼時

1. **單檔案架構**：所有程式碼在 `index.html`，修改時注意行數變化

2. **CSS 與 JavaScript 位置**：
   - CSS：`<style>` 標籤內（行 76-1035）
   - JavaScript：`<script>` 標籤內（行 1036-1850）

3. **響應式設計**：
   - 斷點：768px
   - 手機版特殊處理：棋盤固定 4 列、字體縮小
   - 使用 `window.innerWidth` 判斷設備類型

4. **動畫時序**：
   - 骰子滾動：0.6 秒
   - 玩家移動：每格 0.3 秒
   - 側邊欄滑動：0.3 秒
   - 修改時需考慮時序協調

5. **狀態管理**：
   - 使用全域變數（無狀態管理框架）
   - 修改狀態後需手動觸發重新渲染
   - `isRolling` 變數防止重複擲骰

6. **LocalStorage 使用**：
   - 鍵名：`monopoly_tasks`
   - 僅儲存任務內容（不儲存完整遊戲狀態）
   - JSON 序列化/反序列化

### 測試注意事項

**功能測試項目：**
- 不同格子數量（20-50）的棋盤渲染
- 不同玩家數量（2-6）的遊戲流程
- 任務成功/失敗的計分正確性
- 迷霧模式的視覺效果與移除邏輯
- 設定匯入/匯出的完整性
- 響應式：桌面、平板、手機版面

**瀏覽器相容性：**
- 需支援 CSS Grid、3D Transforms
- 需支援 ES6 語法（let/const、箭頭函數、模板字串）
- 需支援 LocalStorage、FileReader API

**效能測試：**
- 大量格子（50 格）的渲染流暢度
- 快速連續擲骰的防抖效果
- 視窗大小改變的重新渲染

### 常見問題處理

**骰子點數顯示錯誤：**
- 檢查 `faces` 陣列（行 1529）點數排列
- 檢查 `getDiceRotation()` 的旋轉角度對應

**棋盤排列異常：**
- 檢查蛇形演算法的奇偶判斷
- 檢查不完整行的 `gridColumnStart` 計算

**玩家視角追蹤失效：**
- 檢查 `scrollToCurrentPlayer()` 的執行時機
- 確認元素已渲染完成再執行 `scrollIntoView()`

**側邊欄滾動問題：**
- 確認 Flexbox 佈局正確（固定區 + 可滾動區）
- 檢查 `overflow-y: auto` 設定

## 程式碼風格

**現有風格慣例：**
- 縮排：2 空格
- 字串：單引號為主，HTML 屬性用雙引號
- 函數：使用 `function` 宣告（非箭頭函數）
- 變數命名：駝峰式命名（camelCase）
- 註解：中文註解說明關鍵邏輯
- 表情符號：UI 文字使用表情符號增加親和力

**修改時請保持一致性。**

## 除錯技巧

**Console 輸出：**
- 骰子統計：`rollDice()` 會輸出點數分佈
- 可在關鍵函數加入 `console.log` 追蹤狀態

**開發者工具：**
- 檢查元素：查看 Grid 佈局、3D Transform
- 效能監控：測試動畫流暢度
- LocalStorage：Application 面板查看儲存內容

**常用除錯點：**
- 遊戲初始化：`startGame()`, `initBoard()`
- 棋盤渲染：`renderBoard()` 的蛇形邏輯
- 玩家移動：`movePlayer()` 的逐格動畫
- 任務判定：`completeTask()` 的計分邏輯
