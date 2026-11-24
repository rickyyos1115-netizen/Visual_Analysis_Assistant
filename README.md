# Ollama-Groq 視覺分析助手

使用 Streamlit 建立的 AI 圖片分析應用程式，可上傳 JPG、JPEG、PNG 圖片，輸入提示詞，並選擇不同的視覺語言模型進行分析。

## 功能特色

- 支援 JPG、JPEG、PNG 圖片上傳。
- 可自訂圖片分析提示詞。
- 使用 Pillow 修正 EXIF 方向，並將圖片轉換為 RGB JPEG，再編碼為 Base64。
- 提供三種分析服務：
  - **Ollama**：透過本機 Ollama Python SDK 呼叫 `gemma4:12b`。
  - **Ollama Cloud**：透過 Ollama Cloud API 呼叫 `gemma4:31b-cloud`。
  - **Groq**：透過 Groq API 呼叫 `qwen/qwen3.8-27b`。
- 以雙欄方式顯示上傳圖片與分析結果。

## 專案檔案

```text
your-repository/
├── ch16-4-3.py
├── README.md
└── requirements.txt
```

## 環境需求

- Python 3.10 或更新版本（建議本機與 Streamlit Community Cloud 使用相同 Python 版本）。
- Ollama 本機服務（只有選擇「Ollama」模式時需要）。
- 使用 Ollama Cloud 或 Groq 模式時，需有對應服務的 API Key。

## 本機安裝與執行

### 1. 下載專案

```bash
git clone https://github.com/<你的GitHub帳號>/<你的Repository名稱>.git
cd <你的Repository名稱>
```

也可以直接下載 GitHub Repository ZIP 並解壓縮。

### 2. 建立虛擬環境（建議）

```bash
python -m venv .venv
```

Windows：

```bash
.venv\Scripts\activate
```

macOS / Linux：

```bash
source .venv/bin/activate
```

### 3. 安裝套件

```bash
pip install -r requirements.txt
```

### 4. 設定 API Key

Ollama Cloud 與 Groq 模式需要各自的 API Key。建議透過環境變數或 Streamlit Secrets 管理，不要把 Key 寫死在程式碼中，也不要上傳到 GitHub。

Windows PowerShell 範例：

```powershell
$env:OLLAMA_API_KEY="你的_Ollama_API_Key"
$env:GROQ_API_KEY="你的_Groq_API_Key"
```

macOS / Linux 範例：

```bash
export OLLAMA_API_KEY="你的_Ollama_API_Key"
export GROQ_API_KEY="你的_Groq_API_Key"
```

> **重要：目前程式碼的金鑰讀取方式需要配合部署環境調整。** 程式使用 `os.environ.get("OLLAMA_API_KEY")`，並由 `Groq()` 讀取 `GROQ_API_KEY`。Streamlit Community Cloud 的 Secrets 應以 `st.secrets` 讀取；僅在 Cloud 後台新增 Secrets，並不代表目前程式中的 `os.environ` 或 `Groq()` 一定能直接讀到它們。部署前請依本文「Streamlit Community Cloud 部署前注意事項」調整程式碼。

### 5. 啟動應用程式

```bash
streamlit run ch16-4-3.py
```

啟動後，依照終端機顯示的網址開啟應用程式，通常是：

```text
http://localhost:8501
```

## Streamlit Community Cloud 部署

### 1. 推送專案至 GitHub

確認 Repository 至少包含：

- `ch16-4-3.py`
- `requirements.txt`
- `README.md`

將專案推送到 GitHub。請勿提交 API Key、`.env`、`secrets.toml` 或其他包含機密資訊的檔案。

### 2. 開啟 Streamlit Community Cloud

前往 [https://share.streamlit.io/](https://share.streamlit.io/)，使用 GitHub 帳號登入，並授權 Streamlit 存取要部署的 Repository。

### 3. 建立 App

在 Streamlit Community Cloud 中：

1. 點選 **Create app**。
2. 選擇 **Yup, I have an app**。
3. 選擇 GitHub Repository。
4. Branch 選擇 `main`（或實際存放程式碼的分支）。
5. Main file path 填入 `ch16-4-3.py`。
6. 在 Advanced settings 中確認 Python 版本，並設定 Secrets。
7. 點選 Deploy。

### 4. 設定 Secrets

在部署設定的 **Secrets** 欄位填入 TOML 格式，例如：

```toml
OLLAMA_API_KEY = "你的_Ollama_API_Key"
GROQ_API_KEY = "你的_Groq_API_Key"
```

請使用自己的有效 API Key。不要將真正的 Key 放在 README、Python 原始碼或公開 GitHub Repository。

### 5. 部署完成

部署成功後，Streamlit Community Cloud 會提供一個 `*.streamlit.app` 網址。之後推送新的程式碼至所設定的 GitHub 分支，Cloud 通常會自動更新應用程式。

## Streamlit Community Cloud 部署前注意事項

### 1. 修正 Secrets 讀取方式

目前程式有以下寫法：

```python
client = Client(
    host="https://ollama.com",
    headers={'Authorization': 'Bearer ' + os.environ.get('OLLAMA_API_KEY')}
)
```

如果 `OLLAMA_API_KEY` 不存在，`os.environ.get()` 會回傳 `None`，字串串接時會出錯。此外，Streamlit Cloud 的 Secrets 應從 `st.secrets` 讀取。

建議改成可同時支援本機環境變數與 Streamlit Secrets 的方式：

```python
def get_secret(key):
    value = os.environ.get(key)
    if value:
        return value

    try:
        return st.secrets[key]
    except (KeyError, FileNotFoundError):
        return None


ollama_api_key = get_secret("OLLAMA_API_KEY")
groq_api_key = get_secret("GROQ_API_KEY")

client = None
if ollama_api_key:
    client = Client(
        host="https://ollama.com",
        headers={"Authorization": f"Bearer {ollama_api_key}"}
    )
```

接著，Ollama Cloud 呼叫前應確認 `client` 不為 `None`；Groq 建議明確傳入 API Key：

```python
if not groq_api_key:
    return "Groq API 錯誤：尚未設定 GROQ_API_KEY。"

client = Groq(api_key=groq_api_key)
```

### 2. 本機 Ollama 模式無法直接沿用到 Streamlit Cloud

目前「Ollama」模式呼叫：

```python
ollama.chat(model="gemma4:12b", ...)
```

這通常需要執行中的 Ollama 本機服務，以及已下載的 `gemma4:12b` 模型。Streamlit Community Cloud 不會自動提供你的個人電腦上的 Ollama 服務或模型。

因此，部署到 Streamlit Community Cloud 時：
- **Ollama Cloud**：可透過雲端 API 使用，前提是 API Key 與程式設定正確。
- **Groq**：可透過 Groq API 使用，前提是 API Key 與程式設定正確。
- **Ollama（本機）**：在 Community Cloud 上通常不可直接使用；除非另外提供 Cloud 可連線的 Ollama 服務，並修改程式連線設定。

建議部署版本先隱藏或停用本機 Ollama 選項，避免使用者選取後才發現服務無法連線。

### 3. API Key 安全性

- 不要將 API Key 寫在公開程式碼中。
- 不要把 `.env` 或 `secrets.toml` 推送到 GitHub。
- 使用 Streamlit Community Cloud 的 Secrets 管理雲端金鑰。
- 若金鑰曾經被推送到公開 Repository，請立即到服務提供者平台撤銷並重新產生。

## 常見問題

**Q：為什麼部署後出現 `ModuleNotFoundError`？**

請確認 `requirements.txt` 位於 Repository 根目錄，並且列出程式所需的第三方套件。修改後重新推送，Community Cloud 會重新安裝依賴套件。

**Q：為什麼 Ollama Cloud 或 Groq 回報 API Key 錯誤？**

請檢查 Streamlit Cloud 的 Secrets 名稱是否與程式使用的名稱完全一致，並確認程式已改為讀取 `st.secrets` 或相容的金鑰讀取函式。

**Q：為什麼本機 Ollama 模式在 Cloud 無法使用？**

因為 Community Cloud 不會自動執行你個人電腦上的 Ollama Server。請改用 Ollama Cloud / Groq，或另外部署可從網路連線的 Ollama 服務。

## 參考資料

- [Streamlit Community Cloud 部署文件](https://docs.streamlit.io/deploy/streamlit-community-cloud/deploy-your-app/deploy)
- [Streamlit Community Cloud Secrets 管理](https://docs.streamlit.io/deploy/streamlit-community-cloud/deploy-your-app/secrets-management)
- [Streamlit App Dependencies](https://docs.streamlit.io/deploy/streamlit-community-cloud/deploy-your-app/app-dependencies)
