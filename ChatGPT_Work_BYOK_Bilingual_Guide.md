# ChatGPT Desktop "ChatGPT Work" Mode + BYOK Guide

# ChatGPT 桌面端「ChatGPT Work」模式搭配 BYOK 指南

---

## 🌐 Language Navigation / 語言導覽

- [Part 1: 繁體中文完整教學指南 (Traditional Chinese Guide)](#part-1-繁體中文完整教學指南)
- [Part 2: Full English Guide (英文完整教學指南)](#part-2-full-english-guide)

---

# Part 1: 繁體中文完整教學指南

## 📌 適用情境與解決的三大錯誤

當 ChatGPT / Codex 桌面端的特定版本要求 Work 模式或內建工具必須有官方帳號登入，而你又使用第三方 BYOK API 供應商時，可參考以下雙軌認證方式。版本行為、設定欄位及工具支援可能變動；請先確認你的客戶端與供應商支援相關設定。本指南不保證適用於所有版本或所有錯誤。

1. `Error submitting message: Work browser access requires an authenticated account`
2. `ChatGPT configuration could not be loaded: model_providers contains reserved built-in provider IDs: 'openai'`
3. `unexpected status 401 Unauthorized: {"code":"INVALID_API_KEY","message":"Invalid API key"}, url: wss://.../v1/responses`

---

此設定主要針對第一項錯誤；其餘錯誤僅在原因與下方說明相符時才適用。

## 🚀 完整設定 4 步驟

### 步驟 1：使用客戶端支援的官方帳號登入

若客戶端要求 Work 模式或內建工具驗證官方帳號身分，請使用該客戶端支援的官方帳號登入。帳號資格與功能可用性依服務條款、帳號類型及客戶端版本而異。

1. 在應用程式的帳號設定中登入，或依該客戶端的官方說明完成登入。
2. 若需要檢查本機憑證檔案，請先確認該版本的官方文件所述位置；不同作業系統與安裝方式可能不同。
3. 不要手動建立、編輯、刪除或分享認證檔案。登入憑證應由客戶端管理。

#### 📄 `auth.json` 結構示意

認證檔案格式可能因版本而異。以下僅為結構示意，**不可直接複製或用來建立認證檔案**；請保留客戶端自動產生的內容，不要在文件、截圖或問題回報中貼出任何憑證：

```json
{
  "auth_mode": "chatgpt",
  "OPENAI_API_KEY": null,
  "tokens": {
    "id_token": "REDACTED",
    "access_token": "REDACTED",
    "refresh_token": "REDACTED",
    "account_id": "REDACTED"
  },
  "last_refresh": "REDACTED"
}
```

---

### 步驟 2：依客戶端說明設定 BYOK 供應商

依照客戶端文件找到設定檔，並備份現有設定。以下是欄位示意，並非適用於所有版本或供應商的通用設定檔。請將 `https://api.example.com/v1`、`YOUR_MODEL_ID` 和 `YOUR_API_KEY` 換成供應商提供的值，並確認該供應商支援所選 API 格式及認證方式：

#### 📄 `config.toml` 欄位示例

```toml
# ============================================================================
# 1. 頂層全域設定（必須放在所有 [xxx] 區塊標籤的最上方！）
# ============================================================================
# 使用未被保留的自訂供應商 ID，並在下方保持名稱一致。
model_provider = "CustomProvider"
openai_base_url = "https://api.example.com/v1"
model = "YOUR_MODEL_ID"
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
[model_providers.CustomProvider]
name = "Your BYOK provider"
base_url = "https://api.example.com/v1"
wire_api = "responses"
experimental_bearer_token = "YOUR_API_KEY"
requires_openai_auth = true   # 只有客戶端設定文件支援時才使用
supports_websockets = false   # 若供應商不支援 WebSocket，可依文件停用

[model_providers.CustomProvider.http_headers]
Authorization = "Bearer YOUR_API_KEY"
Content-Type = "application/json"

```

_(註：設定欄位會因客戶端版本和供應商而異。只修改官方文件要求的欄位，並保留客戶端自動產生的 `[marketplaces]`、`[plugins]` 與 `[hooks]`。不要直接用範例覆蓋整份設定檔。)_

---

### 步驟 3：正常重新啟動應用程式

儲存設定後，使用應用程式的正常退出功能完全關閉並重新開啟。避免使用強制終止程序的指令；若應用程式無法正常關閉，請依你的作業系統或客戶端官方指示處理。

---

### 步驟 4：重新啟動並開啟「全新對話（New Chat）」

1. 重新開啟應用程式並確認登入狀態。
2. 建立新對話，選擇所需模式及自訂模型。
3. 先傳送一則不含敏感資料的測試訊息；若仍失敗，記錄錯誤文字並依客戶端及供應商文件排查。

---

## 💡 核心原理解析與常見錯誤排解表

| 遇到的錯誤訊息 / 問題                                     | 原因解析                                               | 解決參數與方法                                                                           |
| :-------------------------------------------------------- | :----------------------------------------------------- | :--------------------------------------------------------------------------------------- |
| `Work browser access requires an authenticated account`   | 客戶端要求官方帳號身分，但目前登入狀態或設定未滿足要求 | 確認客戶端登入狀態與該版本文件；只有文件支援時才設定 `requires_openai_auth = true`       |
| `model_providers contains reserved built-in provider IDs` | 自訂供應商 ID 與客戶端保留 ID 衝突                     | 選擇非保留的自訂 ID，並在頂層設定與供應商區塊中保持一致                                  |
| `401 Unauthorized` 或 API Key 錯誤                        | 端點、金鑰、認證格式或 API 相容性不正確                | 依供應商文件核對 base URL、模型 ID、API 格式和金鑰；如使用 WebSocket，確認供應商支援情況 |
| 模型選擇器仍顯示舊模型                                    | 設定值錯誤、位置不符或客戶端尚未重新載入               | 確認頂層欄位位置與客戶端設定格式，然後正常重新啟動                                       |

---

---

# Part 2: Full English Guide

## 📌 Overview & Errors Resolved

When a ChatGPT / Codex Desktop version requires an official account identity for Work mode or built-in tools, and you also use a third-party BYOK provider, the dual-authentication approach below may apply. Client behavior, configuration fields, and provider support can change; verify them against the documentation for your versions. This guide is not guaranteed to apply to every version or error.

1. `Error submitting message: Work browser access requires an authenticated account`
2. `ChatGPT configuration could not be loaded: model_providers contains reserved built-in provider IDs: 'openai'`
3. `unexpected status 401 Unauthorized: {"code":"INVALID_API_KEY","message":"Invalid API key"}, url: wss://.../v1/responses`

---

This setup primarily addresses the first error below. The other errors apply only when their stated causes match your configuration.

## 🚀 Step-by-Step Configuration (4 Steps)

### Step 1: Sign in with an account supported by the client

If the client requires official account identity for Work mode or built-in tools, sign in using an account supported by that client. Eligibility and feature availability depend on the service, account type, and client version.

1. Sign in through the app's account settings or follow the client's official sign-in instructions.
2. If you need to locate local credential files, check the official instructions for your specific version and operating system.
3. Do not manually create, edit, delete, or share authentication files. Let the client manage sign-in credentials.

#### 📄 `auth.json` Structure Example

Credential-file formats vary by version. The following is only a structural illustration; **do not copy it to create an authentication file**. Keep client-generated content intact, and never paste credentials into documentation, screenshots, or support requests:

```json
{
  "auth_mode": "chatgpt",
  "OPENAI_API_KEY": null,
  "tokens": {
    "id_token": "REDACTED",
    "access_token": "REDACTED",
    "refresh_token": "REDACTED",
    "account_id": "REDACTED"
  },
  "last_refresh": "REDACTED"
}
```

---

### Step 2: Configure the BYOK provider using client documentation

Find the configuration file using your client's documentation and back up your existing settings. This is a field example, not a universal configuration for every client or provider. Replace `https://api.example.com/v1`, `YOUR_MODEL_ID`, and `YOUR_API_KEY` with values from your provider, and verify that it supports the selected API format and authentication method:

#### 📄 `config.toml` Field Example

```toml
# ============================================================================
# 1. Top-Level Global Settings (MUST be placed at the very top before any [section]!)
# ============================================================================
# Use a custom provider ID that is not reserved, and keep it consistent below.
model_provider = "CustomProvider"
openai_base_url = "https://api.example.com/v1"
model = "YOUR_MODEL_ID"
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
[model_providers.CustomProvider]
name = "Your BYOK provider"
base_url = "https://api.example.com/v1"
wire_api = "responses"
experimental_bearer_token = "YOUR_API_KEY"
requires_openai_auth = true   # Use only if supported by your client configuration
supports_websockets = false   # If the provider lacks WebSocket support, follow its documentation

[model_providers.CustomProvider.http_headers]
Authorization = "Bearer YOUR_API_KEY"
Content-Type = "application/json"

```

---

### Step 3: Restart the app normally

After saving settings, fully quit and reopen the app using its normal exit controls. Avoid force-kill commands; if the app cannot close normally, follow the official instructions for your operating system and client.

---

### Step 4: Reopen the App & Start a "New Chat"

1. Reopen the app and confirm that you are signed in.
2. Start a new conversation and select the desired mode and custom model.
3. Send a test message that contains no sensitive information. If it still fails, record the error text and consult the client and provider documentation.

---

## 💡 Troubleshooting & Key Parameter Reference

| Error Message / Symptom                                   | Root Cause                                                                                                             | Solution                                                                                                               |
| :-------------------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------- |
| `Work browser access requires an authenticated account`   | The client requires official account identity, but the current sign-in or configuration does not meet that requirement | Verify sign-in and the documentation for your client version; set `requires_openai_auth = true` only if supported      |
| `model_providers contains reserved built-in provider IDs` | A custom provider ID conflicts with a reserved ID                                                                      | Choose a non-reserved custom ID and use it consistently in the top-level setting and provider section                  |
| `401 Unauthorized` or API key error                       | The endpoint, key, authentication format, or API compatibility is incorrect                                            | Verify the base URL, model ID, API format, and key against provider documentation; check WebSocket support if relevant |
| Model selector still shows an old model                   | A setting is invalid, misplaced, or not reloaded                                                                       | Verify the required top-level fields and configuration format, then restart the app normally                           |
