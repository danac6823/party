# 命中靶秒 — 部署說明

一款給 40–50 人現場玩的手機派對遊戲：主持人設定目標秒數，玩家盲挑按停，最接近者得分。
純靜態網頁（HTML/CSS/vanilla JS），多人連線用 Firebase Realtime Database。
玩家掃 QR 或開網址即可加入，不需安裝。

---

## 檔案清單

| 檔案 | 說明 | 需要 Firebase？ |
|---|---|---|
| `index.html` | 進入頁（自動跳轉到主活動） | 否 |
| `target-second-game.html` | **主活動**：指定秒數／猜秒數。多人連線。含神準、幸運數字、十二之神、自由挑戰、頒獎 | **是** |
| `database.rules.json` | Firebase RTDB 安全規則 | — |

> 沒填 Firebase 設定時，會自動進入「本地試玩模式」（單機、不連線），
> 方便先驗證流程；正式多人一定要完成下面的 Firebase 設定。

---

## 部署步驟

### 1) 建立 Firebase 專案 + Realtime Database
1. 到 <https://console.firebase.google.com> 建一個專案。
2. 左側 **Build → Realtime Database → Create Database**（地區選離台灣近的，如 asia-southeast1）。
3. 建好後到 **Rules** 分頁，貼上 `database.rules.json` 的內容並發布。
   - 這組規則對 `rooms/*` 開放讀寫，適合「一次性活動」的暫時房間。
   - 若要更保險，可在活動後刪除資料，或之後改用帶到期時間／驗證的規則。
4. 到 **專案設定 (⚙️) → 一般 → 你的應用程式 → Web (</>)** 註冊一個 Web App，
   複製那段 `firebaseConfig`（含 `apiKey`、`databaseURL` 等）。

### 2) 把設定填進主活動檔案
在 `target-second-game.html` 檔案「最上方 `<script>` 開頭」有一段：

```js
const FIREBASE_CONFIG = { apiKey:"", authDomain:"", databaseURL:"", projectId:"", appId:"" };
const FIXED_ROOM = "";
```

- 把 `FIREBASE_CONFIG` 換成 Firebase 給的那組（**`databaseURL` 一定要有**，格式像
  `https://<專案>-default-rtdb.asia-southeast1.firebasedatabase.app`）。
- `FIXED_ROOM`：想**事前印好 QR**就填一個固定房號（例：`"EMBA"`）；留空則由主持人當場自動產生。

### 3) 上線（擇一）

**A. GitHub Pages（推薦，跟現有 danac6823.github.io 一致）**
1. 把整個資料夾推到一個 repo（或現有 repo 的子資料夾）。
2. Repo **Settings → Pages**，Source 選 `main` 分支、根目錄，儲存。
3. 幾分鐘後就有網址（例 `https://<帳號>.github.io/<repo>/`）。
4. 首頁 `index.html` 會自動跳進主活動。

**B. Firebase Hosting**
```bash
npm i -g firebase-tools
firebase login
firebase init hosting        # public 目錄選這個資料夾；不要覆蓋 index.html
firebase deploy
```

### 4) 驗收
- 手機開 `target-second-game.html`，右上狀態應顯示「已連線」（不是「本地試玩模式」）。
- 用兩支手機：一支當主持人建房、一支掃 QR 加入，確認人數會即時增加、能開始一回合。

---

## 現場流程（主持人）
1. 開 `target-second-game.html` → 設定目標秒數（預設 12.00）、神準容差（0.12）、幸運數字、頒獎名額（12）。
2. 建房 → 投出／給玩家掃 QR 加入。
3. 每回合：▶ 開始倒數 → 玩家盲挑按停 → 看「已完成 X/N」→ 結算·揭曉神準。建議打 4–5 回合。
4. 想加碼：🎯 進入自由挑戰（各自 10 次挑戰 12.00，達標會通知主持人）。
5. 結束活動·頒獎：一頁顯示十二之神、總冠軍/神準王/積分王、最準前 12 名、幸運數字加碼獎（可列印）。

## 計分邏輯（重點）
- 主排名＝**名次反向計分**：每回合第 1 名得（當回合人數）分，最後 1 分。
- 得獎名單前 12 名＝**平均誤差最小**（最準）排序，顯示平均誤差。
- 三個特別獎**不重複得獎**：總冠軍(最準)／神準王(神準最多)／積分王(積分最高)。
- **神準** = 誤差 ≤ 容差(預設 0.12s)；**十二之神** = 誤差 < 0.01s（更高一階、獨立獎）。
- **幸運數字** = 停在指定吉祥數字（主持人可自訂），最後頒獎才驚喜揭曉。

## 技術備註
- 計時全部在**玩家本機**用 `performance.now()` 量，網路延遲不影響公平性；只有結果／彙總會同步。
- 玩家身分存在瀏覽器 `localStorage`，重整不會重複計分。
- Firebase 免費方案同時連線上限約 100，50 人綽綽有餘。
- 無後端、無建置流程；直接丟靜態主機即可。
