# Work browser access requires an authenticated account: ChatGPT Work + BYOK Fix

Seeing `Error submitting message: Work browser access requires an authenticated account` in ChatGPT or Codex Desktop? This guide explains a dual-authentication setup for using a third-party BYOK API provider with **ChatGPT Work mode**, the built-in browser, or computer-control tools.

## When to use this method

Use the dual-authentication setup in the [full bilingual guide](ChatGPT_Work_BYOK_Bilingual_Guide.md) if all of the following apply:

- Your ChatGPT / Codex Desktop version requires an authenticated account for Work mode or its built-in tools.
- Your model requests use a third-party BYOK API key.
- You want to use ChatGPT Work mode, the built-in browser, or computer-control tools.
- Sending a message is blocked with this error:

  > **Error submitting message**
  > **Work browser access requires an authenticated account**

In this configuration, the desktop client needs an official ChatGPT sign-in for Work/browser identity checks, while your custom provider uses your BYOK key for model requests. A BYOK key alone does not provide the official account identity required by those features. The method in the full guide keeps those credentials in their respective roles.

If you do not use Work mode or the built-in tools, or you are not seeing this error, this method may not apply to your issue.

## 使用時機

如果你同時符合以下條件，請參考[完整中英雙語指南](ChatGPT_Work_BYOK_Bilingual_Guide.md)中的雙軌認證設定：

- 使用要求 Work 模式或內建工具必須登入官方帳號的 ChatGPT / Codex 桌面端版本。
- 使用第三方 BYOK API Key 呼叫模型。
- 想使用 ChatGPT Work 模式、內建瀏覽器或電腦控制工具。
- 傳送訊息時遇到以下錯誤：

  > **Error submitting message**
  > **Work browser access requires an authenticated account**

此設定情境下，桌面端需要官方 ChatGPT 帳號登入狀態，才能通過 Work／瀏覽器功能的身分檢查；自訂模型供應商則使用你的 BYOK Key 傳送模型請求。單獨設定 BYOK Key 並不會提供這些功能要求的官方帳號身分。完整指南中的方法會讓兩種憑證各自用於對應用途。

如果你沒有使用 Work 模式或內建工具，或沒有遇到上述錯誤，這個方法可能不適用於你的問題。

## Credential safety / 憑證安全

Keep `auth.json` and your BYOK API key private. Do not publish, commit, or send files containing access tokens, refresh tokens, account IDs, or API keys. / 請妥善保管 `auth.json` 與 BYOK API Key。不要公開、提交至版本庫，或傳送含有存取權杖、更新權杖、帳號 ID 或 API Key 的檔案。
