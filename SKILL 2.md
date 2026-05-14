---
name: lianxing-weekly-report-editor
description: >
  製作良興週報互動版 HTML 編輯器。當使用者說「週報編輯器」、「週報 HTML 編輯」、
  「可以拖曳的週報」、「週報版面調整」時啟動。
  輸出一個可在瀏覽器開啟的互動版 HTML 檔，使用者可以直接拖曳調整版面後截圖使用。
  與 weekly-report-image（AI 生圖版）和 lianxing-weekly-report（靜態 HTML 版）為不同 skill，請勿混用。
---

# 良興週報 HTML 互動編輯器 Skill

每週輸出可在瀏覽器操作的週報 HTML 編輯器，使用者可自行調整版面後截圖發布。

適用情境：需要精確排版控制、希望自己微調位置與比例的週報製作流程。

---

## 觸發方式

使用者說以下任一詞時啟動：
- 「週報編輯器」
- 「可以拖曳的週報」
- 「週報版面調整」
- 「週報 HTML 編輯」
- 「幫我做可以調整的週報」

---

## 輸出檔案規格

| 項目 | 說明 |
|---|---|
| 格式 | 單一 `.html` 檔（所有資源含背景圖皆 base64 內嵌） |
| 尺寸 | 540 × 962px（對應背景圖比例 650×1158） |
| 背景圖 | `/mnt/user-data/uploads/week.jpg`（良興科技感深藍背景） |
| 字型 | Noto Sans TC（Google Fonts CDN） |
| 色系 | 深藍科技感，主色 `#00BFFF`，強調色保留品牌橘 |

---

## 版面結構

### Header（固定，不可拖曳）
- 高度：250px
- 背景：透明（露出背景圖）
- 內容由上至下：`2026 · 第N週` → `良興週報`（72px）→ `副標語`
- `2026 · 第N週` 置頂，`良興週報` 居中放大，副標語靠底

### 內容區（4個可調節區塊）
內容區從 Header 底部延伸至距底部 48px（保留背景圖的「良興管理部 · 2026」文字）。

4 個區塊使用 `flex` 比例分配，**撐滿整頁，無留白**：

| 區塊 ID | 名稱 | 預設 flex |
|---|---|---|
| `blk-kpi` | KPI 數字（3欄） | 1.4 |
| `blk-hero` | 主數字（大字） | 1.0 |
| `blk-slist` | 本週重點（條列） | 1.8 |
| `blk-bottom` | 公告 / 良興哲學（左右分欄） | 2.2 |

---

## 互動功能說明

### 工具列按鈕

| 按鈕 | 功能 |
|---|---|
| `↺ 重置` | 恢復所有區塊的預設 flex 比例 |
| `🔓 解鎖 / 🔒 鎖定` | 切換編輯模式；鎖定後出現下載按鈕 |
| `⬇ 下載鎖定版`（鎖定後出現） | 輸出不含工具列與拖曳功能的純淨 HTML |

### 解鎖後可操作

1. **拖曳排序**：上下拖曳區塊超過 40px 即換位，水平固定置中
2. **調整高度**：滑鼠懸停區塊底部出現藍色把手，上下拖曳改變 flex 比例，其他區塊自動補齊空間
3. **邊界限制**：區塊不可拖出 Header 上方或畫布底部

### 鎖定後下載

點擊 `⬇ 下載鎖定版` 後：
- 序列化當前完整頁面 HTML（含所有 base64 背景圖）
- 移除工具列、拖曳把手、resize 把手、JavaScript
- 輸出檔名：`indexV2.html`
- 用瀏覽器開啟後截圖即為成品

---

## Step 0：確認必要資訊

執行前確認以下欄位已提供，缺少任何一項則主動詢問：

| 欄位 | 說明 |
|---|---|
| 週次 | 第幾週（e.g. 第4週） |
| 副標語 | Header 的一行主題句（e.g. 業績創新高 團隊再突破） |
| KPI 數字 | 3 組，每組含數字與標籤（e.g. 528M / 業績總額） |
| 主數字 | 大字顯示的核心數字（e.g. 528M，年增 38.2%） |
| 本週重點 | 2–3 項，每項含標題與說明 |
| 公告事項 | 1–2 條 |
| 良興哲學 | 條次與金句全文 |

---

## Step 1：生成 HTML 編輯器

將使用者提供的資料填入以下模板，生成完整 HTML 檔案輸出至 `/mnt/user-data/outputs/良興週報_編輯器_第N週.html`。

### 技術規格

```python
import base64

with open('/mnt/user-data/uploads/week.jpg', 'rb') as f:
    bg_b64 = base64.b64encode(f.read()).decode()

min_h = round(540 * 1158 / 650)  # = 962px
```

### 關鍵 CSS 結構

```css
/* 畫布固定尺寸 */
.wrap { width: 540px; height: 962px; position: relative; }

/* 背景圖鋪滿 */
.bg { position: absolute; inset: 0; width: 100%; height: 100%; object-fit: cover; }

/* Header 透明，固定 250px */
.hdr { position: absolute; top: 0; left: 0; right: 0; height: 250px; background: transparent; }

/* 內容區：flex column 撐滿 */
.content-area {
  position: absolute;
  top: 250px; left: 0; right: 0; bottom: 0;
  display: flex; flex-direction: column;
  padding: 8px 20px 48px; /* 底部 48px 保留背景 footer */
  gap: 8px;
}

/* 每個區塊 flex 比例分配 */
.block { flex: 1; border-radius: 12px; display: flex; flex-direction: column; }
```

### JavaScript 核心邏輯

```javascript
// 高度調整：改變 flex 比例，其他自動補齊
handle.addEventListener('mousedown', e => {
  const startFlex = parseFloat(el.style.flex) || 1;
  const onMove = e => {
    const newFlex = Math.max(0.3, startFlex + (e.clientY - startY) / 80);
    el.style.flex = newFlex.toFixed(2);
  };
});

// 拖曳排序：超過 40px 換位
const onMove = e => {
  const dy = e.clientY - startY;
  if (dy > 40 && currentIdx < blocks.length - 1) {
    contentArea.insertBefore(el, blocks[currentIdx + 1].nextSibling);
    startY = e.clientY;
  } else if (dy < -40 && currentIdx > 0) {
    contentArea.insertBefore(el, blocks[currentIdx - 1]);
    startY = e.clientY;
  }
};

// 下載：序列化整頁 HTML 並清理互動元素
function downloadLocked() {
  let html = document.documentElement.outerHTML;
  html = html.replace(/<div class="toolbar"[\s\S]*?<\/div>\s*(?=<div class="wrap")/, '');
  html = html.replace(/<div class="drag-handle"[^>]*>[\s\S]*?<\/div>/g, '');
  html = html.replace(/<div class="resize-handle"[^>]*><\/div>/g, '');
  html = html.replace(/<script>[\s\S]*?<\/script>/, '');
  // 輸出為 blob 下載
}
```

---

## Step 2：交付與說明

輸出 HTML 後告知使用者操作方式：

1. 下載 HTML 檔，用瀏覽器開啟
2. 點 **🔓 解鎖** 進入編輯模式
3. **拖曳**區塊上下換位；**拉底部把手**調整高度比例
4. 調整完畢點 **🔒 鎖定**
5. 點 **⬇ 下載鎖定版** 輸出最終版
6. 瀏覽器截圖或列印成圖片

---

## 踩過的坑

| 問題 | 解法 |
|---|---|
| 背景圖被截斷 | 畫布用固定 `height: 962px`，而非 `min-height` |
| 下載後畫面空白 | 用 `document.documentElement.outerHTML` 序列化，不要 clone canvas |
| 區塊中間有大空白 | 用 `flex` 比例取代固定 `height`，所有區塊自動填滿 |
| 拖曳時其他區塊跟著跑 | 拖曳時不呼叫 `resolveCollisions`，只移動當前區塊 |
| 最下方區塊蓋住背景 footer | `content-area` 設定 `padding-bottom: 48px` |
| 水平偏移 | 拖曳時鎖定 `left = (canvas.width - el.width) / 2`，只允許垂直移動 |
| 高度調整把手不見 | 把手改為 `position: absolute; bottom: 0` 在區塊內，不放在 flex 流中 |

---

## 版本歷程

| 版本 | 主要改動 |
|---|---|
| v1–v5 | 基礎版面建立，背景圖套入，藍色光暈色系 |
| v6–v10 | Header 重構：三欄 → 豎線靠左 → 全置中 → 標題放大 |
| v11–v14 | Header 背景透明、高度調整至 250px |
| v15–v17 | 邊界限制、水平鎖定置中、移除碰撞推移 |
| v18–v19 | 加入鎖定後下載功能，修正下載空白問題 |
| v20–v25 | 週次移至標題上方，細部字級與間距調整 |
| v26–v27 | 全面改為 flex 比例分配，消除空白，底部留 48px 保留 footer |
