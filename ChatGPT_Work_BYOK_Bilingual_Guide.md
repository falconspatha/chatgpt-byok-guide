Credit by falconspatha
# ChatGPT Desktop "ChatGPT Work" Mode + BYOK (Third-Party API) Ultimate Guide
# ChatGPT 桌面端「ChatGPT Work」模式搭配 BYOK（第三方 API）終極教學指南

---

## 🌐 Language Navigation / 語言導覽
* [Part 1: 繁體中文完整教學指南 (Traditional Chinese Guide)](#part-1-繁體中文完整教學指南)
* [Part 2: Full English Guide (英文完整教學指南)](#part-2-full-english-guide)

---

# Part 1: 繁體中文完整教學指南

## 📌 適用情境與解決的三大錯誤
當你在新版 ChatGPT / Codex 桌面端（`26.1013` / `26.1016` 及以上版本）想使用 **ChatGPT Work 介面與內建瀏覽器**，同時連接自己的第三方中轉站 / BYOK API Key（例如 Techikey 等）時，這套雙軌認證方法能完美解決以下三大常見報錯：

1. `Error submitting message: Work browser access requires an authenticated account`
2. `ChatGPT configuration could not be loaded: model_providers contains reserved built-in provider IDs: 'openai'`
3. `unexpected status 401 Unauthorized: {"code":"INVALID_API_KEY","message":"Invalid API key"}, url: wss://.../v1/responses`

---

## 🚀 完整設定 4 步驟

### 步驟 1：先登入一次免費官方帳號（產生合法 `auth.json`）
为了让桌面端的 `ChatGPT Work` 模式與內建瀏覽器通過身分檢查，本地的 `~/.codex/auth.json` 必須保有官方帳號的登入憑證。

1. 打開 `~/.codex/` 資料夾：
   * **macOS：** 打開 Finder，按快捷鍵 `Cmd + Shift + G`，輸入 `~/.codex/` 並按 Enter。
   * **Windows：** 打開檔案總管，前往 `C:\Users\<你的使用者名稱>\.codex\`。
2. 如果裡面已經有改過的 `config.toml`，請先將 `config.toml` 暫時移到桌面備份，並刪除舊的 `auth.json`。
3. 啟動 `ChatGPT.app`，使用**任何一個官方 ChatGPT 帳號（免費版 Free 帳號即可）**完成登入。
4. 登入成功後，完全退出 App（macOS 按 `Cmd + Q`）。此時系統會在 `~/.codex/auth.json` 自動生成合法的官方認證 Token。

#### 📄 `auth.json` 標準範本與檢查方式
請打開 `~/.codex/auth.json` 確認格式如下（**請保持官方自動生成的內容不變，`OPENAI_API_KEY` 維持 `null`，不要填入第三方 Key**）：

```json
{
  "auth_mode": "chatgpt",
  "OPENAI_API_KEY": null,
  "tokens": {
    "id_token": "eyJhbG...(保持官方自動生成的 id_token，請勿修改)...",
    "access_token": "eyJhbG...(保持官方自動生成的 access_token，請勿修改)...",
    "refresh_token": "rt.1...(保持官方自動生成的 refresh_token，請勿修改)...",
    "account_id": "你的官方 Account_ID (保持不變)"
  },
  "last_refresh": "2026-10-22T11:40:02.879709Z"
}
```

---

### 步驟 2：編寫 `~/.codex/config.toml`（實現雙軌認證分離）
打開 `~/.codex/config.toml`，將檔案內容替換為以下標準範本（請將範例中的 `https://techikey.com/v1`、`gpt-6.1-sol` 與 `sk-你的BYOK_API_KEY` 替換為你自己的資訊）：

#### 📄 `config.toml` 完整標準範本
```toml
# ============================================================================
# 1. 頂層全域設定（必須放在所有 [xxx] 區塊標籤的最上方！）
# ============================================================================
# 注意：model_provider 必須寫成首字母大寫的 "OpenAI"！
# 小寫 "openai" 為系統保留字會報錯，大寫 "OpenAI" 則可合法建立自訂區塊。
model_provider = "OpenAI"
openai_base_url = "https://techikey.com/v1"
model = "gpt-6.1-sol"
model_reasoning_effort = "medium"

# ============================================================================
# 2. 桌面端介面模式設定
# ============================================================================
[desktop]
followUpQueueMode = "queue"
conversationDetailMode = "STEPS_PROSE" # 設為 STEPS_PROSE 以保持在 ChatGPT Work 模式介面

# ============================================================================
# 3. 自訂供應商與雙軌鑑權設定（名稱必須與頂層 model_provider 完全一致）
# ============================================================================
[model_providers.OpenAI]
name = "techikey"
base_url = "https://techikey.com/v1"
wire_api = "responses"
experimental_bearer_token = "sk-你的BYOK_API_KEY"
requires_openai_auth = true   # 關鍵 1：讓 Work Browser 讀取 auth.json 通過官方身分驗證
supports_websockets = false   # 關鍵 2：關閉 wss:// WebSocket 改走 HTTPS，避免 401 報錯

[model_providers.OpenAI.http_headers]
Authorization = "Bearer sk-你的BYOK_API_KEY"
Content-Type = "application/json"

# ============================================================================
# 4. 官方內建插件設定（以下保持開啟即可在 Work 模式使用完整功能）
# ============================================================================
[marketplaces.openai-bundled]
source_type = "local"
source = "/Users/你的Mac使用者名稱/.codex/.tmp/bundled-marketplaces/openai-bundled"

[marketplaces.openai-primary-runtime]
source_type = "local"
source = "/Users/你的Mac使用者名稱/.cache/codex-runtimes/codex-primary-runtime/plugins/openai-primary-runtime"

[plugins."browser@openai-bundled"]
enabled = true

[plugins."unified-computer-use@openai-bundled"]
enabled = true

[plugins."chrome@openai-bundled"]
enabled = true

[plugins."visualize@openai-bundled"]
enabled = true

[plugins."code-review@openai-bundled"]
enabled = true

[plugins."documents@openai-primary-runtime"]
enabled = true

[plugins."pdf@openai-primary-runtime"]
enabled = true

[plugins."spreadsheets@openai-primary-runtime"]
enabled = true

[plugins."presentations@openai-primary-runtime"]
enabled = true

[plugins."template-creator@openai-primary-runtime"]
enabled = true
```
*(註：如果你原本的 `config.toml` 最上方有設定區塊，只需修改前 3 個區塊，並保留你自己檔案下半部自動產生的 `[marketplaces]`、`[plugins]` 與 `[hooks]` 即可。)*

---

### 步驟 3：徹底清除背景進程並重啟 App
因為桌面端的 `codex` 與 `node_repl` 會在背景常駐，只按視窗上的 `Retry` 不會完整重新載入新憑證。請打開 **終端機（Terminal）** 執行以下指令徹底關閉所有背景進程：

* **macOS 終端機（Terminal）指令：**
  ```bash
  killall -9 ChatGPT codex node_repl SkyComputerUseClient 2>/dev/null
  ```
* **Windows 命令提示字元（CMD / PowerShell）指令：**
  ```cmd
  taskkill /F /IM ChatGPT.exe /IM codex.exe /IM node_repl.exe
  ```

---

### 步驟 4：重新啟動並開啟「全新對話（New Chat）」
1. 重新打開 `ChatGPT.app`。
2. **切勿在舊的報錯對話中按重試**（舊對話的 Session 已鎖死在 `Could not start this chat`）。
3. 點擊左上角的 **`New chat`** 開啟一個新對話，確認左上角為 **`ChatGPT Work`**、右下角顯示你的模型名稱（如 `6.1 Sol Medium`），即可順暢對話！

---

## 💡 核心原理解析與常見錯誤排解表

| 遇到的錯誤訊息 / 問題 | 原因解析 | 解決參數與方法 |
|:---|:---|:---|
| `Work browser access requires an authenticated account` | 客戶端檢測到未綁定官方認證，拒絕啟動內建瀏覽器 | 1. 確保 `auth.json` 內有官方登入的 `tokens`<br>2. 在 `[model_providers.OpenAI]` 加上 `requires_openai_auth = true` |
| `model_providers contains reserved built-in provider IDs: 'openai'` | 新版禁止覆寫全小寫的內建 `"openai"` ID | 將 `model_provider` 與區塊名稱改為大小寫區分的 **`"OpenAI"`**（即 `[model_providers.OpenAI]`） |
| `401 Unauthorized: {"code":"INVALID_API_KEY"} (wss://.../v1/responses)` | 開啟官方認證後，Codex 預設將官方 JWT Token 透過 WebSocket 發給中轉站 | 在 `[model_providers.OpenAI]` 中同時設定：<br>1. `experimental_bearer_token = "sk-..."`<br>2. `supports_websockets = false` |
| 右下角模型名稱沒有變（仍顯示舊模型如 `6-luna`） | 把 `model = "..."` 寫在了 `[desktop]` 等區塊下方，變成子屬性 | 確保 `model_provider`、`openai_base_url`、`model` 寫在檔案**第 1～4 行（所有 `[xxx]` 標籤之前）** |

---
---

# Part 2: Full English Guide

## 📌 Overview & Errors Resolved
When using the latest ChatGPT / Codex Desktop App (`v26.1013` / `v26.1016` and above) in **"ChatGPT Work" mode** with a third-party API / BYOK provider (such as Techikey), this dual-authentication configuration resolves the three major errors below:

1. `Error submitting message: Work browser access requires an authenticated account`
2. `ChatGPT configuration could not be loaded: model_providers contains reserved built-in provider IDs: 'openai'`
3. `unexpected status 401 Unauthorized: {"code":"INVALID_API_KEY","message":"Invalid API key"}, url: wss://.../v1/responses`

---

## 🚀 Step-by-Step Configuration (4 Steps)

### Step 1: Log in with a Free Official ChatGPT Account (Generate `auth.json`)
To pass the client's `requireWorkBrowserIdentity` check in **ChatGPT Work** mode, `~/.codex/auth.json` must contain valid official ChatGPT login tokens.

1. Open your `~/.codex/` directory:
   * **macOS:** Open Finder, press `Cmd + Shift + G`, type `~/.codex/`, and press Enter.
   * **Windows:** Open File Explorer and go to `C:\Users\<YourUsername>\.codex\`.
2. If you already modified `config.toml`, temporarily move `config.toml` to your Desktop as a backup and delete any existing `auth.json`.
3. Launch `ChatGPT.app` and log in with **any official ChatGPT account (a Free tier account works fine)**.
4. Once logged in, quit the app completely (`Cmd + Q` on macOS). This generates a valid `~/.codex/auth.json` file.

#### 📄 `auth.json` Template & Verification
Open `~/.codex/auth.json` and verify that it matches the structure below (**keep the official auto-generated tokens intact, and leave `OPENAI_API_KEY` as `null`**):

```json
{
  "auth_mode": "chatgpt",
  "OPENAI_API_KEY": null,
  "tokens": {
    "id_token": "eyJhbG...(keep official auto-generated id_token untouched)...",
    "access_token": "eyJhbG...(keep official auto-generated access_token untouched)...",
    "refresh_token": "rt.1...(keep official auto-generated refresh_token untouched)...",
    "account_id": "your-official-account-id"
  },
  "last_refresh": "2026-10-22T11:40:02.879709Z"
}
```

---

### Step 2: Configure `~/.codex/config.toml` (Dual-Track Auth Separation)
Open `~/.codex/config.toml` and update the top sections using the template below (replace `https://techikey.com/v1`, `gpt-6.1-sol`, and `sk-YOUR_BYOK_API_KEY` with your actual provider details):

#### 📄 `config.toml` Full Template
```toml
# ============================================================================
# 1. Top-Level Global Settings (MUST be placed at the very top before any [section]!)
# ============================================================================
# IMPORTANT: Use capitalized "OpenAI" to bypass the reserved lowercase "openai" ID restriction.
model_provider = "OpenAI"
openai_base_url = "https://techikey.com/v1"
model = "gpt-6.1-sol"
model_reasoning_effort = "medium"

# ============================================================================
# 2. Desktop UI Mode Settings
# ============================================================================
[desktop]
followUpQueueMode = "queue"
conversationDetailMode = "STEPS_PROSE" # Keeps the UI in "ChatGPT Work" mode

# ============================================================================
# 3. Custom Provider & Dual-Auth Settings (ID must match top-level model_provider)
# ============================================================================
[model_providers.OpenAI]
name = "techikey"
base_url = "https://techikey.com/v1"
wire_api = "responses"
experimental_bearer_token = "sk-YOUR_BYOK_API_KEY"
requires_openai_auth = true   # Key 1: Passes Work Browser identity check using auth.json
supports_websockets = false   # Key 2: Disables wss:// WebSockets & forces HTTPS to prevent 401 errors

[model_providers.OpenAI.http_headers]
Authorization = "Bearer sk-YOUR_BYOK_API_KEY"
Content-Type = "application/json"

# ============================================================================
# 4. Bundled Plugins (Keep your existing [marketplaces], [plugins], and [hooks] below)
# ============================================================================
[plugins."browser@openai-bundled"]
enabled = true

[plugins."unified-computer-use@openai-bundled"]
enabled = true

[plugins."chrome@openai-bundled"]
enabled = true

[plugins."visualize@openai-bundled"]
enabled = true

[plugins."code-review@openai-bundled"]
enabled = true

[plugins."documents@openai-primary-runtime"]
enabled = true

[plugins."pdf@openai-primary-runtime"]
enabled = true

[plugins."spreadsheets@openai-primary-runtime"]
enabled = true

[plugins."presentations@openai-primary-runtime"]
enabled = true

[plugins."template-creator@openai-primary-runtime"]
enabled = true
```

---

### Step 3: Kill Background Processes & Restart the App
Because `codex` and `node_repl` run persistently in the background, clicking `Retry` in the UI is not enough. Open your **Terminal** and run the command below to terminate all background processes:

* **macOS (Terminal):**
  ```bash
  killall -9 ChatGPT codex node_repl SkyComputerUseClient 2>/dev/null
  ```
* **Windows (CMD / PowerShell):**
  ```cmd
  taskkill /F /IM ChatGPT.exe /IM codex.exe /IM node_repl.exe
  ```

---

### Step 4: Reopen the App & Start a "New Chat"
1. Launch `ChatGPT.app`.
2. **Do not retry inside an old failed chat thread** (failed threads remain locked in the `Could not start this chat` state).
3. Click **`New chat`** in the top-left corner, verify that the top-left mode says **`ChatGPT Work`** and the bottom-right selector shows your custom model (e.g., `6.1 Sol Medium`), and send your message!

---

## 💡 Troubleshooting & Key Parameter Reference

| Error Message / Symptom | Root Cause | Solution |
|:---|:---|:---|
| `Work browser access requires an authenticated account` | Client blocks the built-in Work Browser when the provider is not linked to OpenAI auth | 1. Keep official `tokens` inside `auth.json`<br>2. Set `requires_openai_auth = true` under `[model_providers.OpenAI]` |
| `model_providers contains reserved built-in provider IDs: 'openai'` | Newer versions forbid overriding the lowercase `'openai'` provider ID | Use case-sensitive **`"OpenAI"`** for `model_provider` and `[model_providers.OpenAI]` |
| `401 Unauthorized: {"code":"INVALID_API_KEY"} (wss://.../v1/responses)` | With OpenAI auth enabled, Codex sends the official JWT token over WebSockets instead of your BYOK key | Under `[model_providers.OpenAI]`, set both:<br>1. `experimental_bearer_token = "sk-..."`<br>2. `supports_websockets = false` |
| Model selector still shows old model (e.g., `gpt-6-luna`) | `model` and `model_provider` were placed underneath `[desktop]` instead of the root level | Move `model_provider`, `openai_base_url`, and `model` to the **very top of `config.toml`** before any `[section]` header |
