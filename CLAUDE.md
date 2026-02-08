# 心臟病 (Heartbreak) - 多人卡牌遊戲

## 專案概述
這是一個支援 iPhone 和 iPad 的 2-6 人對戰心臟病卡牌遊戲。玩家各站一側，透過觸控紅色按鈕進行遊戲。

- **Repository**: `popcornylu/heartbreak`
- **URL**: https://popcornylu.github.io/heartbreak/
- **技術**: 純 HTML/CSS/JavaScript，單一檔案 + mp3 音效檔

## 遊戲規則

1. 系統自動喊數 1→2→3→...→K→1（循環）
2. 當翻出的牌 = 喊的數字時，要拍按鈕
3. 🃏 Joker 出現時，不管喊什麼都要拍
4. 拍慢或拍錯的人收走牌堆中的所有牌到棄牌堆
5. 收牌後，喊數重新從 1 開始
6. 54 張牌發完時，棄牌最少的玩家獲勝

## 技術架構

### 單一檔案設計
整個遊戲在 `index.html` 一個檔案中，包含 HTML、CSS、JavaScript，方便部署到 GitHub Pages。

### 狀態管理
```javascript
state = {
    deck: [],              // 系統牌堆（52張 + 2張Joker = 54張）
    player1Discard: [],    // 玩家1棄牌堆（收到的牌，不會再出）
    player2Discard: [],    // 玩家2棄牌堆
    centerPile: [],        // 中央牌堆（發出的牌）
    currentNumber: 0,      // 當前喊的數字 (1-13)
    currentCard: null,     // 當前翻出的牌
    isPlaying: false,      // 遊戲進行中
    isPaused: false,       // 遊戲暫停中
    canSlap: false,        // 可以拍牌
    flipTimeout: null,     // 計時器 ID
    slapRecorded: { player1: null, player2: null }  // 拍牌記錄 {time, correct}
}
```

### 音效系統
使用 Web Audio API 的 `AudioBuffer` 預載 mp3 檔案，確保快速、可靠的音效播放：

#### 音效檔案
| 檔案 | 用途 | 觸發時機 |
|------|------|---------|
| `correct.mp3` | 拍對音效 | 該拍時第一個拍的人 |
| `incorrect.mp3` | 拍錯音效 | 不該拍時第一個拍的人 |
| `tick.mp3` | 轉盤滴答聲 | 轉盤旋轉時每過一個色塊邊界 |
| `result.mp3` | 轉盤結果音效 | 轉盤停止揭曉結果時 |

#### 技術細節
- 開始遊戲時透過 `loadAudioBuffers()` 將所有 mp3 預載為 `AudioBuffer`
- 播放時建立新的 `BufferSource`，支援同時播放多個音效（不會互相蓋掉）
- 每次播放前檢查 `AudioContext` 狀態，自動 `resume()` 避免 iOS 暫停問題
- mp3 URL 加上 `?v=timestamp` cache buster 確保載入最新版本
- `playCardSound()` - 發牌音效（Web Audio API 合成，咻聲）

#### 拍牌音效規則
- 每次可以拍牌時，**只有第一個拍的人**會觸發音效
- 拍對播放 `correct.mp3`，拍錯播放 `incorrect.mp3`

#### 轉盤音效
- 使用 `requestAnimationFrame` 追蹤轉盤實際旋轉角度
- 每跨過一個色塊邊界播放一次 `tick.mp3`，與視覺同步
- 轉盤停止時播放 `result.mp3`，同時高亮中選區塊、暗化其他區塊

## UI 設計原則

### 玩家區域
- iPad：左右各一個玩家區域
- iPhone：上下各一個玩家區域
- 玩家1區域旋轉180度（面對面遊玩）
- 黑色背景作為初始狀態
- 紅色實體按鈕（3D 立體效果，按下有下沉動畫）
- 棄牌堆顯示亂亂散落的牌（隨機角度和位置）
- **只有按到按鈕才能觸發**，按按鈕外面無效

### 反饋系統
- 按對：顯示綠色 ⭕
- 按錯：顯示紅色 ❌
- 背景顏色：
  - 按對且最快：綠色（贏家）
  - 按慢或按錯：紅色（輸家）
  - 兩人都按錯：先按紅色，後按黃色
- 反饋圖示只在各自玩家區域內顯示

### 中央區域
- 顯示當前喊的數字
- 牌堆（每張牌有隨機偏移和旋轉，看起來亂亂的）
- 剩餘牌數
- 暫停按鈕 ⏸

### 暫停功能
按下暫停按鈕顯示選單：
- **繼續遊戲**（綠色）- 回到遊戲
- **重新開始**（黃色）- 重新開始新遊戲
- **離開遊戲**（紅色）- 回到開始畫面

## 響應式設計

### 斷點
| 螢幕寬度 | 裝置 | 排列方式 |
|---------|------|---------|
| < 600px | iPhone 直向 | 垂直（玩家1上、玩家2下） |
| 600-900px | iPhone 橫向/小平板 | 橫向（縮小尺寸） |
| > 900px | iPad | 橫向（完整尺寸） |

### 調整項目
- 遊戲容器方向 (row/column)
- 中央區域方向和尺寸
- 按鈕、牌堆、文字大小
- 暫停按鈕位置（中央區域右上角）

## 部署

### GitHub Pages
- 推送到 main branch 自動部署
- `.nojekyll` 檔案跳過 Jekyll 建置，加速部署
- 約 30 秒內自動更新

### 部署流程
```bash
# 1. 更新版本號（在 index.html 中）
# 2. 提交並推送
git add .
git commit -m "描述

Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>"
git push
```

## 版本號規則
- 右上角顯示版本號
- 格式：`vX.Y.Z`
  - **X**: 大改版 / 破壞性更新
  - **Y**: 新功能
  - **Z**: Bug 修復 / 小調整
- 每次部署前更新版本號

## 注意事項

### 觸控事件
- 使用 `touchstart` / `touchend` 處理觸控
- 按鈕同時支援 click 和 touch（用於桌面測試）
- `preventDefault()` 防止預設行為和縮放
- `stopPropagation()` 防止事件冒泡

### 計時器管理（重要！）
- **所有** `setTimeout` 都存到 `state.flipTimeout`
- 暫停時用 `clearTimeout(state.flipTimeout)` 清除
- 所有 callback 中檢查 `state.isPaused` 狀態
- 避免暫停/繼續時重複觸發翻牌

### 棄牌堆視覺
- 只在收牌時更新，不是每次發牌都重繪
- 使用追加模式，只加入新收到的牌
- 現有的牌位置保持不變，不會跳動

### 牌堆視覺
- 中央牌堆每張牌有隨機偏移（-10 到 10px）和旋轉（-15 到 15度）
- 棄牌堆每張牌有較大的隨機偏移和旋轉，呈現散落效果
