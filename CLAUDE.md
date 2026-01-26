# 心臟病 (Heartbreak) - 雙人卡牌遊戲

## 專案概述
這是一個為 iPad 設計的雙人對戰心臟病卡牌遊戲，玩家各站一側，透過觸控按鈕進行遊戲。

## 遊戲規則
1. 系統自動喊數 1→2→3→...→K→1（循環）
2. 當翻出的牌 = 喊的數字時，要拍按鈕
3. Joker 出現時，不管喊什麼都要拍
4. 拍慢或拍錯的人收走牌堆中的所有牌
5. 收牌後，喊數重新從 1 開始
6. 54 張牌發完時，棄牌最少的玩家獲勝

## 技術架構

### 單一檔案設計
- 整個遊戲在 `index.html` 一個檔案中
- 包含 HTML、CSS、JavaScript
- 無外部依賴，方便部署到 GitHub Pages

### 音效系統
- 使用 Web Audio API 生成音效，無需外部音檔
- `playCardSound()` - 發牌音效
- `playErrorSound()` - 錯誤音效（buzzer）
- `playSuccessSound()` - 成功音效（叮咚）

### 狀態管理
```javascript
state = {
    deck: [],              // 系統牌堆（54張）
    player1Discard: [],    // 玩家1棄牌堆
    player2Discard: [],    // 玩家2棄牌堆
    centerPile: [],        // 中央牌堆
    currentNumber: 0,      // 當前喊的數字
    isPlaying: false,
    isPaused: false,
    canSlap: false,
    flipTimeout: null,
    slapRecorded: { player1: null, player2: null }
}
```

## UI 設計原則

### 玩家區域
- 左右各一個玩家區域
- 玩家1區域旋轉180度（面對面遊玩）
- 黑色背景作為初始狀態
- 紅色實體按鈕（3D 效果）
- 棄牌堆顯示亂亂散落的牌

### 反饋系統
- 按對：顯示綠色 ⭕，背景變綠
- 按錯：顯示紅色 ❌，背景變紅
- 兩人都按錯：先按紅色，後按黃色
- 反饋圖示只在各自區域內顯示

### 中央區域
- 顯示當前喊的數字
- 牌堆（每張牌有隨機偏移和旋轉，看起來亂亂的）
- 剩餘牌數
- 暫停按鈕

## 部署

### GitHub Pages
- Repository: `popcornylu/heartbreak`
- URL: https://popcornylu.github.io/heartbreak/
- 推送到 main branch 自動部署
- `.nojekyll` 檔案跳過 Jekyll 建置，加速部署

### 部署流程
```bash
git add .
git commit -m "描述"
git push
```
約 30 秒內自動更新

## 版本號規則
- 右上角顯示版本號
- 格式：`vX.Y.Z`
  - X: 大改版
  - Y: 新功能
  - Z: Bug 修復 / 小調整
- 每次部署前更新版本號

## 注意事項

### 觸控事件
- 使用 `touchstart` / `touchend` 處理觸控
- 按鈕同時支援 click 和 touch
- `preventDefault()` 防止預設行為和縮放

### 計時器管理
- 所有 `setTimeout` 都存到 `state.flipTimeout`
- 暫停時清除計時器
- callback 中檢查 `isPaused` 狀態
- 避免暫停/繼續時重複觸發

### 棄牌堆視覺
- 只在收牌時更新，不是每次發牌都重繪
- 使用追加模式，新牌疊加在上面
- 現有的牌位置保持不變
