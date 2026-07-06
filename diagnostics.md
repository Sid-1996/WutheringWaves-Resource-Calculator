# 🎨 美術 UI 設計診斷報告 — Sid 的鳴潮智慧素材需求計算器

> 本文件為「美術 UI 設計診斷」之彙整產出，作為後續分階段重構的施工藍圖。
> **當前狀態**：保留背景圖改動 + 文件化（方案 B）。
> **後續執行準則**：沿用單一 `index.html`、不拆檔、不引入新框架。

---

## 0. 文件目的 & 決策紀錄

### 0.1 使用者決策序列
| 階段 | 決策內容 |
|------|---------|
| 初始 | 診斷專案美術 UI 設計（唯讀） |
| Q1 | 「先把所有發現彙整成 `diagnostics.md`，不改 `index.html`」 |
| Q2 | 「換成電玩賽博龐克（霓虹紫 + 螢青）」 |
| Q3 | 「沿用單一 `index.html`（推薦）」 |
| 背景圖需求 | 新增本地圖 `image/4565878123.jpg` 並將之調暗 |
| 最終範圍確認 | **方案 B**：保留背景改動 + 文件化 |

### 0.2 文件邊界（Scope Guard）
- ❌ 不再編輯 `index.html`（已完成部分視為「已套用項」，於 §8 記錄）
- ❌ 不建立其他檔案
- ❌ 不引入第三方相依或新框架
- ✅ 提供 Cyberpunk Neon 設計令牌對照表，供未來落地使用

---

## 1. 專案速寫

| 項目 | 內容 |
|------|------|
| 主檔 | `index.html`（2716 行，132 KB） |
| 架構 | 單檔 SPA：HTML + 內嵌 `<style>`（655 行）+ 內嵌 `<script>`（1600+ 行） |
| 框架 | TailwindCSS（CDN：`https://cdn.tailwindcss.com`） |
| 字型 | Noto Sans TC（Google Fonts，權重 300 / 400 / 500 / 600 / 700） |
| 主視覺來源 | 5 張遠端 IGDB 圖床（`images.igdb.com/...`，已替換其中 1 張為本地圖） |
| 主題色 | 深空藍 `#0b1020` + 青綠 `#5ad7d8` + 琥珀 `#d6b35f` |
| 截圖 | `images.png`（社群示意）、`screenshot.png`（功能示意，僅覆蓋上半段） |
| 目標設計 | Cyberpunk Neon（霓虹紫 + 螢青），令牌見 §7 |

---

## 2. 規範比對摘要

| 狀態 | 數量 | 條目摘要 |
|------|------|---------|
| ✅ 符合 | 6 | 多層漸層背景、卡片圓角與陰影、backdrop-filter、響應式斷點、i18n 繫結、素材顏色錨點 |
| ⚠️ 違規 | 14 | 詳見 §3 程式碼層級與 §4 視覺設計層級問題 |
| 🔵 觀察 | 5 | 外部資源風險、字型一致性、動畫可訪問性、Lighthouse 指標、截圖對齊版本 |

---

## 3. 程式碼層級問題

### 3.1 `:root` 重複宣告（重複定義）
- `index.html:41-52`：定義 `--wuwa-ink/night/mist/cloud/cyan/teal/emerald/amber/rose/shadow`
- `index.html:652-658`：定義 `--wuwa-hero/keyart/scene-1/scene-2/artwork` 圖片 URL
- **影響**：兩段互相覆蓋風險；若有人重排 CSS 易破圖
- **建議**：合併為單一 `:root`，刪除未引用的死變數

### 3.2 未被引用的死變數
下列變數在 CSS 中**完全無選擇器使用**（搜尋結果 0）：
- `--wuwa-mist`、`--wuwa-cloud`、`--wuwa-rose`、`--wuwa-shadow`、`--wuwa-ink`、`--wuwa-night`、`--wuwa-emerald`（在 41 行 root 中定義但未被引用）
- `--wuwa-keyart`、`--wuwa-scene-2`、`--wuwa-artwork`（部分僅在 `.card.hero/theme-2/4::before` 引用，但 `theme-2` 僅 1 處、`theme-4` 僅 2 處，密度低）

### 3.3 死樣式 / 未引用的 class
| Class | 位置 | 狀態 |
|-------|------|------|
| `.accent-divider` | `index.html:521-530` | HTML 中**無任何元素使用**此 class |
| `.slider-track` | `index.html:326-330` | 被 `input[type=range]` 套用但視覺被 Tailwind utility 蓋掉 |
| `.pulse-animation` | `index.html:357` | 僅由 JS 動態加入，初始無引用 |

### 3.4 未引用或低密度的 @keyframes
| 動畫 | 位置 | 引用密度 |
|------|------|---------|
| `pulse` | `:357` | ✅ 有使用（`animate-pulse` Tailwind 且 `.pulse-animation` JS 套用） |
| `heartbeat` | `:403` | ⚠️ 僅 JS 動態套用於 `skill-level-*` 按鈕 |
| `breathe` | `:413` | ⚠️ 僅 JS 套用於 `save-record-btn` |
| `wiggle` | `:429` | ⚠️ 僅 JS 套用於 `.btn-secondary` 與語言按鈕 |
| `bounce` | `:440` | ⚠️ JS 套用於 primary/success 按鈕點擊 |
| `ripple` | `:367` | ✅ CSS `.btn-ripple::before/active` 有引用 |
| `spin` | `:460` | ✅ `.btn-loading::after` 引用 |
| `checkmark` | `:486` | ⚠️ `.btn-success-check::after` 但 JS 中未見該 class 被加入 |

### 3.5 `.card::after` 對未初始化變數的依賴
- `index.html:110`：`background: var(--wuwa-artwork)`
- `--wuwa-artwork` 在 `:root` 中**第二段（652 行）才宣告**；若 CSS 被重排或第一段 root 移除會破圖

### 3.6 缺失的 ARIA 語意
| 元素 | 現況 | 建議 |
|------|------|------|
| `.language-btn` | `<button>` 無 `aria-pressed` | 加 `aria-pressed="true/false"` |
| 策略徽章 `<span class="…animate-pulse">` | 無 `role` | 加 `role="status" aria-live="polite"` |
| 卡片標題 `<h2>` | 無 id 供 label 連結 | 加 `id` 並讓卡片容器 `aria-labelledby` |
| emoji-only 標題 | `🎮`/`🟢` 等會被螢幕閱讀器朗讀為「火箭符號」 | 加 `aria-hidden="true"` 至 emoji span；文字部分另標 |

---

## 4. 視覺設計層級問題

### 4.1 色彩 emoji 作為主要語意區隔
- **位置**：`target-*`、`current-*`、`preview-*`、`status-table`（11 處 🟢🔵🟣🟡）
- **問題**：
  1. 色弱玩家（紅綠色盲約 8% 男性）無法可靠分辨綠/藍/紫/金
  2. 不同 OS 渲染差異大（Windows Segoe UI Emoji vs macOS Apple Color Emoji）
- **建議**：加第二視覺通道 — emoji 之外補「T1 / T2 / T3 / T4」文字標籤 + 不同形狀徽章

### 4.2 金色素材顏色被誤用為 yellow
- **位置**：`index.html:843 / 884`、`.material-gold: #b88d33`（`:171`）
- **問題**：Tailwind `bg-yellow-50` 帶輕微綠調，與鳴潮官方金素材偏橙金的視覺不符
- **建議**：換成 `bg-amber-50` + material-gold `#c08a2f`

### 4.3 `.hero` 漸層與下方主色不一致
- **位置**：`index.html:665`
- **現況**：`linear-gradient(135deg, rgba(79,70,229,0.72), rgba(124,58,237,0.72), rgba(236,72,153,0.72))` — 紫→深紫→粉
- **問題**：下方卡片主色是「青綠 + 琥珀」，hero 紫粉與之斷裂
- **建議**：依 §7 對照表換成 Cyberpunk Neon 漸層（螢青→霓虹紫→紫電）

### 4.4 `mix-blend-mode: screen` 色相漂移
- **位置**：`index.html:74`（舊版，現已移除於 `.page-bg`）
- **問題**：舊版 `opacity: 0.2 + mix-blend-mode: screen` 讓背景被白淡化、變亮；不同瀏覽器色相漂移
- **現況**：已替換為 `linear-gradient(180deg, rgba(6,12,26,0.75)→0.85)` 暗化層（見 §8）
- **殘留**：`.card.theme-1/2/3/4::before` 仍用 `linear-gradient + 主題圖`，無 blend mode；建議同步檢查

### 4.5 按鈕動畫堆疊衝突
- **`save-record-btn`**：同時掛 `btn-breathe`（無限呼吸動畫）+ `btn-ripple`（點擊擴散）+ `:active translateY` + `:hover translateY(-2px)`
- **影響**：4 種動畫同時觸發時視覺抖動
- **建議**：移除 `btn-breathe`，只保留 `ripple` + focus ring（Cyberpunk Neon 改用 `glow-pulse`）

### 4.6 `.card::after` 旋轉圖壓住 H2
- **位置**：`index.html:103-114`，右上 220x220、旋轉 8°
- **問題**：當 H2 標題較長時，右上角裝飾圖剛好覆蓋到標題
- **建議**：加 `mix-blend-mode: multiply` 或降低 z-index、或直接以 `--wuwa-artwork-glow` 邊光暈替代裝飾圖

### 4.7 `backdrop-filter` 無降級
- **位置**：`index.html:91, 507`
- **問題**：Firefox <103、舊 Edge 不支援 `backdrop-filter`
- **建議**：加 `@supports not (backdrop-filter: blur(10px)) { .card { background: rgba(245,247,251,1); } }`

### 4.8 字體 fallback 一致性
- **位置**：`@import` 已含 300–700
- **問題**：報告區 `#report-content` 用 `font-mono`，但 Noto Sans TC 不含 mono 字重，會 fallback 為系統 mono（Windows: Consolas、macOS: SF Mono）
- **建議**：Cyberpunk Neon 升級時引入 `JetBrains Mono` 或 `Share Tech Mono` 作 mono

### 4.9 缺失 `tailwind.config`
- **現況**：`<script src="cdn.tailwindcss.com">` 後無 `tailwind.config = {...}`
- **影響**：無法擴充品牌色 token，所有色彩靠裸 utility class
- **建議**：Cyberpunk 重構時加 inline `tailwind.config = { theme: { extend: { colors: { 'neon-cyan': '#00f0ff', ... } } } }`

---

## 5. 圖片策略問題

| 問題 | 位置 / 影響 |
|------|------------|
| **5 張主視覺全部來自第三方 IGDB CDN** | 版權風險（IGDB 非內容發行方）、中國大陸存取可能失敗、hotlink 隨時失效 |
| **`mix-blend-mode: screen` 色相漂移** | Safari / Firefox 顯示略偏綠（已在 `.page-bg` 修正） |
| **截圖不對應當前版本** | `screenshot.png` 只覆蓋技能設定區，下方報告/預覽主要功能無視覺示意 |
| **`background-attachment: fixed` 在 iOS Safari** | 已知 iOS 對 `fixed` 支援有限；行動版可能表現為 `scroll`，需驗證 |

---

## 6. 🎨 未來設計風格：Cyberpunk Neon（霓虹紫 + 螢青）

### 6.1 設計調性
- **核心關鍵字**：霓虹、螢光、賽博龐克、未來廢土、電路光暈
- **情緒座標**：科技 + 神秘 + 危險感 + 高對比
- **視覺語彙**：暗底 + 銳利邊光 + 漸層光暈 + 微噪點

### 6.2 字型策略
| 用途 | 現況 | Cyberpunk Neon |
|------|------|----------------|
| 內文 | Noto Sans TC（300-700） | Noto Sans TC **保留** |
| 標題 | Noto Sans TC 600 | **Noto Sans TC + Orbitron**（標題； sci-fi 字型） |
| 數據 / 報告 | `font-mono`（fallback 系統） | **JetBrains Mono** 或 **Share Tech Mono** |

### 6.3 圓角策略
| 元素 | 現況 | Cyberpunk Neon |
|------|------|----------------|
| `.card` | `border-radius: 20px` | `16px`（更銳利） |
| `.btn-*` | `12px` | `10px` |
| 小元素 / 徽章 | `10px` | `8px` |
| 加：邊角斜切（cyber-corner） | ❌ 無 | ✅ 加 `clip-path` 切角 |

### 6.4 陰影 / 光暈策略
| 元素 | 現況 | Cyberpunk Neon |
|------|------|----------------|
| `.card` 陰影 | `0 26px 60px rgba(8,16,32,0.32)` | 同 + 內陰影光暈 `inset 0 0 0 1px rgba(0,240,255,0.25)` |
| 按鈕盒陰影 | `0 8px 25px rgba(14,147,153,0.35)` | `var(--wuwa-glow-cyan)` |
| 邊框 | `1px solid rgba(255,255,255,0.2)` | `1px solid rgba(0,240,255,0.35)` + 內陰影光暈 |

### 6.5 動畫收斂策略
|動畫| 現況 | Cyberpunk Neon |
|------|------|----------------|
| `breathe` | 無限呼吸 | ❌ 移除 |
| `heartbeat` | 點擊心跳 | ❌ 移除 |
| `wiggle` | hover 搖擺 | ❌ 移除 |
| `bounce` | 點擊彈跳 | ❌ 移除 |
| `ripple` | 點擊擴散 | ✅ 保留 |
| `pulse` | opacity 呼吸 | ✅ 保留（強調用） |
| 新增 `glow-pulse` | ❌ | ✅ 按鈕 hover 時 box-shadow 光暈擴張 |
| 新增 `scan-line` | ❌ | ✅ 卡片表面橫向掃描線（可選） |

---

## 7. 設計令牌對照（Before → After）

### 7.1 CSS 變數：`--wuwa-*`

| Token | Before（現況） | After（Cyberpunk Neon） | 用途 |
|-------|---------------|------------------------|------|
| `--wuwa-ink` | `#0b1020` | `#06020f` | 主背景深處 |
| `--wuwa-night` | `#111827` | `#0d0820` | 卡片基底 |
| `--wuwa-mist` | `#e6eef7`（未用） | ❌ 移除 | — |
| `--wuwa-cloud` | `#f5f7fb`（未用） | ❌ 移除 | — |
| `--wuwa-cyan` | `#5ad7d8` | `#00f0ff` | 螢青（主強調） |
| `--wuwa-teal` | `#18b7b3` | `#08d6c8` | 青綠輔色 |
| `--wuwa-emerald` | `#22c39a` | `#00ffa3` | 成功 / 綠素材 |
| `--wuwa-amber` | `#d6b35f` | `#ffb347` | 琥珀（警示 / 金素材） |
| `--wuwa-rose` | `#ef7d8c`（未用） | `#ff4f7b` | 錯誤 / 危險 |
| `--wuwa-shadow` | `rgba(6,12,26,0.45)`（未用） | ❌ 移除 | — |
| `--wuwa-magenta` | ❌ 無 | `#ff2bd6` | **新增** 霓虹紫（主色） |
| `--wuwa-violet` | ❌ 無 | `#7a3bff` | **新增** 紫電（輔色） |
| `--wuwa-glow-cyan` | ❌ 無 | `0 0 12px #00f0ff, 0 0 32px rgba(0,240,255,0.4)` | 螢青光暈 |
| `--wuwa-glow-magenta` | ❌ 無 | `0 0 12px #ff2bd6, 0 0 32px rgba(255,43,214,0.4)` | 霓虹光暈 |
| `--wuwa-glow-violet` | ❌ 無 | `0 0 12px #7a3bff, 0 0 32px rgba(122,59,255,0.4)` | 紫電光暈 |

### 7.2 素材階級 T1~T4（綠 / 藍 / 紫 / 金）

| Tier | Before（Tailwind + hex） | After（Cyberpunk Neon） |
|------|--------------------------|------------------------|
| T1 綠 | `bg-green-50` + `border-green-200` / `#169a85` | `bg-emerald-50` / `#00ffa3` + 細亮 border + glow |
| T2 藍 | `bg-blue-50` + `border-blue-200` / `#3aa0b3` | `bg-cyan-50` / `#00f0ff` + 細亮 border + glow |
| T3 紫 | `bg-purple-50` + `border-purple-200` / `#6b7bd6` | `bg-violet-50` / `#7a3bff` + 細亮 border + glow |
| T4 金 | `bg-yellow-50` + `border-yellow-200` / `#b88d33` | `bg-amber-50` / `#ffb347` + 細亮 border + glow |

### 7.3 漸層

| 區塊 | Before | After |
|------|--------|-------|
| `body` | `radial 青綠/琥珀 + linear #0b1020→#1b2b3f` | `radial 螢青/霓虹紫 + linear #06020f→#1a0a3d` |
| `.hero` | `linear 135deg, rgba(79,70,229,0.72), rgba(124,58,237,0.72), rgba(236,72,153,0.72)` | `linear 135deg, rgba(0,240,255,0.55), rgba(255,43,214,0.55), rgba(122,59,255,0.55)` |
| `.btn-primary` | `linear 135deg, #0e9399, #18b7b3` | `linear 135deg, #00f0ff, #08d6c8` + `var(--wuwa-glow-cyan)` |
| `.btn-success` | `linear 135deg, #22c39a, #11998e` | `linear 135deg, #00ffa3, #08d6c8` + `var(--wuwa-glow-cyan)` |
| `.btn-warning` | `linear 135deg, #d6b35f, #c49334` | `linear 135deg, #ffb347, #ff4f7b` + `var(--wuwa-glow-magenta)` |
| `.btn-info` | `linear 135deg, #5ad7d8, #2fbfd1` | `linear 135deg, #00f0ff, #7a3bff` + `var(--wuwa-glow-violet)` |
| `.btn-secondary` | `linear 135deg, #1f2937, #334155` | `linear 135deg, #0d0820, #1a0a3d` + 細亮邊框 |

### 7.4 字體 / 圓角 / 動畫彙整

| Token 類別 | Before | After |
|-----------|--------|-------|
| 主字 | Noto Sans TC | Noto Sans TC + **Orbitron**（標題） |
| Mono | 系統 fallback | **JetBrains Mono** 或 **Share Tech Mono** |
| 圓角 | `20px / 12px / 10px` | `16px / 10px / 8px` |
| 卡片邊框 | `rgba(255,255,255,0.2)` | `rgba(0,240,255,0.35)` + 內陰影光暈 |
| 按鈕動畫 | breathe / heartbeat / wiggle / bounce 混用 | 收斂成：`glow-pulse`（hover 强調）+ `ripple`（點擊）+ `lift`（hover translateY） |

### 7.5 主視覺策略

| 方案 | 內容 | 推薦度 |
|------|------|--------|
| **A** | 用 SVG/CSS 抽象電路／極光格柵當底圖，不依賴第三方 | ⭐⭐⭐ |
| **B** | 保留現有 `image/4565878123.jpg` + IGDB 圖，加 `filter: hue-rotate(-15deg) saturate(1.25)` 推向霓虹 | ⭐⭐ |
| **C** | 全部換成本地圖，建立 `assets/bg-*.webp` | ⭐⭐⭐ |

---

## 8. 已套用變更紀錄（背景圖改動）

> 本批次改動屬「視覺微調落地」，保留為基線；不違反後續「換 Cyberpunk Neon 主色調」目標。

| # | 位置 | Before | After | 原因 |
|---|------|--------|-------|------|
| 1 | `index.html:656`（`--wuwa-scene-1`） | `url("https://images.igdb.com/igdb/image/upload/t_720p/scuir7.png")` | `url("image/4565878123.jpg")` | 使用者加入本地圖，移除第三方熱連結風險 |
| 2 | `index.html:66-77`（`.page-bg`） | 兩道 radial 漸層 + 原圖，`opacity: 0.2` + `mix-blend-mode: screen` | 新增頂層 `linear-gradient(180deg, rgba(6,12,26,0.75)→0.85)` 暗化層、移除 opacity/blend、加 `background-attachment: fixed` | 原設定讓背景被白淡化、變亮；需壓暗讓 `.card` 白底更凸顯 |
| 3 | `index.html:533-538`（媒體查詢 `.page-bg`） | 同舊版（無暗化層、有 mix-blend） | 同步加入暗化層 `linear-gradient(180deg, rgba(6,12,26,0.75)→0.85)` | 行動裝置視口效果一致 |

### 8.1 後續調整整數指南
- 覺得還不夠暗 → 把 `0.75` / `0.85` 往上調（如 `0.82` / `0.90`）
- 想更亮 → 把這兩個值往下調（如 `0.65` / `0.75`）
- 想 Cyberpunk 偏色調 → 把 `rgba(6,12,26,...)` 改成 `rgba(13,8,32,...)`（紫黑）
- 想解決 iOS Safari `background-attachment: fixed` 問題 → 在 `@supports` 中 fallback 為 `scroll`

---

## 9. 優先順序 P0~P2

### 🔴 P0（必修，第一眼印象）
1. 合併 `:root`：兩段合一，刪除未引用的死變數
2. 刪除死樣式：`.accent-divider`、`.slider-track`（或遷移使用）
3. 刪除未引用 `@keyframes`：`checkmark`、`breathe`（後者可保留 CSS 但移除 JS 套用）
4. 金色素材修正：`bg-yellow-50` → `bg-amber-50`、`material-gold` → `#c08a2f`
5. Tailwind `tailwind.config` inline 設置

### 🟠 P1（強烈建議）
6. 色盲友善：emoji 外加「T1/T2/T3/T4」文字標籤 + 狀態徽章加 ✓/✗ icon
7. `.hero` 漸層換 Cyberpunk Neon（§7.3）
8. `backdrop-filter` 加 `@supports` fallback
9. 字體引入 Orbitron / JetBrains Mono 並設 `font-display: swap`
10. ARIA 補強：`aria-pressed` / `role="status"` / `aria-labelledby` / emoji `aria-hidden`

### 🟢 P2（優化，可累積）
11. 主視覺本地化（方案 C）+ 建立本地 `assets/`
12. Light/Dark 主題切換（用 `body[data-theme="light"]`）
13. `.card::after` 與 H2 重疊修正
14. 動畫去抖：`save-record-btn` 移除 `btn-breathe`
15. 報告區表格化 / chip 化（強化 data report 視覺）
16. 加 `scan-line` / `cyber-corner` 風格元素

---

## 10. 驗證與回歸測試建議

### 10.1 視覺驗證
| 項目 | 方法 |
|------|------|
| 重製截圖 | 拍 3 張：技能區 / 報告區 / 中英文切換 |
| 色盲模擬 | Stark / Polypane（Protanopia / Deuteranopia / Tritanopia）測四階素材 |
| 響應式 | 320px / 375px / 768px / 1280px / 1920px 五段截圖比對 |
| `background-attachment: fixed` | iOS Safari 實機驗證 |

### 10.2 瀏覽器覆蓋率
| 瀏覽器 | 版本 | 重點 |
|--------|------|------|
| Chrome | 最新 | 主流基線 |
| Edge | 最新 | `backdrop-filter` 支援 |
| Firefox | 103+ | `backdrop-filter` 首次支援 |
| Safari | 16+ | `mix-blend-mode` 色相漂移測試 |
| iOS Safari | 16+ | `background-attachment: fixed` 驗證 |

### 10.3 Lighthouse 指標目標
| 指標 | 現況 | 目標 |
|------|------|------|
| Accessibility | 未測 | ≥ 95 |
| Performance | 未測 | ≥ 90 |
| Best Practices | 未測 | ≥ 95 |
| SEO | 未測 | ≥ 90 |

---

## 11. 附錄 A：程式碼引用清單

| 章節 | 檔案 | 行號 | 主題 |
|------|------|------|------|
| §3.1 | `index.html` | 41-52, 652-658 | `:root` 重複宣告 |
| §3.2 | `index.html` | 41-52 | 未引用死變數 |
| §3.3 | `index.html` | 326-330, 521-530, 357 | `.accent-divider` / `.slider-track` / `.pulse-animation` |
| §3.4 | `index.html` | 357, 403, 413, 429, 440, 486 | 各 `@keyframes` |
| §3.5 | `index.html` | 110 | `.card::after` 依賴 `--wuwa-artwork` |
| §3.6 | `index.html` | 701-703, 963, 705-706 | ARIA 缺失 |
| §4.1 | `index.html` | 829, 834, 839, 844（及其他 8 處） | emoji 語意區隔 |
| §4.2 | `index.html` | 171, 843, 884 | 金色誤用 yellow |
| §4.3 | `index.html` | 665 | `.hero` 漸層 |
| §4.5 | `index.html` | 424, 2554-2603 | 按鈕動畫堆疊 |
| §4.6 | `index.html` | 103-114 | `.card::after` 與 H2 重疊 |
| §4.7 | `index.html` | 91, 507 | `backdrop-filter` |
| §4.9 | `index.html` | 7 | Tailwind CDN 無 config |
| §8 | `index.html` | 66-77, 533-538, 656 | 背景圖改動三處 |

---

## 12. 附錄 B：外部資源清單

| 資源 | URL | 用途 | 風險 |
|------|-----|------|------|
| TailwindCSS CDN | `https://cdn.tailwindcss.com` | 樣式框架 | 中（版本變動、CORS） |
| Google Fonts - Noto Sans TC | `https://fonts.googleapis.com/css2?family=Noto+Sans+TC...` | 內文字型 | 低 |
| Google Analytics | `https://www.googletagmanager.com/gtag/js?id=G-R53CJ6QEBG`（已加密） | 流量分析 | 中（已加密隱藏） |
| IGDB 圖床（4 張剩餘） | `https://images.igdb.com/igdb/image/upload/t_720p/...` | 主視覺 | **高（hotlink 失效 + 版權）** |
| Orbitron（建議新增） | `https://fonts.googleapis.com/css2?family=Orbitron:wght@500;700&display=swap` | Cyberpunk 標題 | 低 |
| JetBrains Mono（建議新增） | `https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;500&display=swap` | 報告數據 | 低 |

---

## 13. 文件版資訊

| 項目 | 內容 |
|------|------|
| 建立 | 2026-07-06 |
| 作者 | 診斷工作階段產出（GLM-5.2） |
| 範圍 | 唯讀 + 唯寫 `diagnostics.md`（保留背景改動） |
| 下一步 | 等使用者簽收後，可進入 P0 / P1 執行階段（依舊單檔 `index.html`） |
