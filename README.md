# Vertixs 建模

用對話建出流程管理系統的藍圖,產出 PostgreSQL 結構。在您自己的電腦執行,不需要帳號或連線。

---

## 1. 事前準備

| 項目 | 說明 |
|---|---|
| Claude 訂閱 | Pro、Max、Team 或 Enterprise 方案,或 [Claude Console](https://console.anthropic.com/) 帳號 |
| 終端機 | macOS 的「終端機」、Windows 的 PowerShell,或 Linux 的 shell |
| Node 20 以上 | 執行 `node -v` 確認;沒有的話從 [nodejs.org](https://nodejs.org) 安裝 |

---

## 2. 安裝 Claude Code

已經安裝過的話跳過這一節。

**macOS / Linux / WSL**

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

**Windows PowerShell**

```powershell
irm https://claude.ai/install.ps1 | iex
```

確認安裝成功:

```bash
claude --version
```

接著登入。執行 `claude`,依畫面指示在瀏覽器完成登入,登入一次即可。

```bash
claude
```

---

## 3. 安裝 Vertixs 外掛

在 Claude Code 裡依序輸入:

```
/plugin marketplace add chunchinsu/vertixs-blueprint
```

```
/plugin install vertixs-blueprint
```

輸入 `/exit` 離開,再執行 `claude` 重新進入。

---

## 4. 建立第一個藍圖

先建一個資料夾放這個專案,在裡面開啟 Claude Code:

```bash
mkdir 我的ERP && cd 我的ERP
claude
```

然後把需求講出來。以下是一段實際的流程:

> **您**|幫我建一個精機廠的採購流程。採購單要有單號、供應商、總金額,
> 走草稿 → 已核准 → 結案三個狀態,超過五十萬要二級簽核。

> **代理人**|開始之前想確認三件事:要管到哪一段(只到採購入庫,還是含付款)?
> 有沒有多個倉庫?需要管批號或序號嗎?

> **您**|到入庫就好。有兩個倉。不用管批號。

> **代理人**|已套用採購模組並加上多倉選項,建立了 4 個物件:採購單、採購明細、
> 收料單、收料明細,並設定生命週期與五十萬的二級簽核。請確認這些欄位是否符合實際作業。

> **您**|採購單再加一個預計交期。確認後產出資料庫結構。

> **代理人**|已加入預計交期並完成確認,產出的結構已存成 schema.sql。

**注意:** 產出資料庫結構前需要先逐項確認欄位。代理人會列出清單讓您過目,
確認過的才會生成。若出現「沒有可生成的物件」,回覆「請逐項確認後再產出」即可。

---

## 5. 常用的指示

不需要記指令,用平常說話的方式即可。

| 您想做的事 | 這樣說 |
|---|---|
| 看有哪些領域範本 | 列出可用的模組 |
| 加一個領域 | 加上庫存管理 |
| 加例外情況 | 我們有多倉 / 要管批號 |
| 加欄位 | 採購單加一個交期欄位 |
| 改流程 | 草稿跟已核准之間加一個待審 |
| 設簽核 | 超過一百萬要三級簽核 |
| 檢查有沒有漏 | 檢核這份藍圖 |
| 產出結構 | 產出資料庫結構 |
| 看目前狀態 | 列出現有藍圖 |
| 接續之前的工作 | 繼續編輯我的採購藍圖 |

---

## 6. 涵蓋的領域

採購進貨、銷售出貨、庫存倉儲、製造 MES、財會帳款、集團多公司帳套、
多幣別匯兌、總帳財報、主管機關申報、薪資、票據帳款、成本會計、
品質管理、委外加工、稅務申報、電子發票、CRM、人資差勤。

每個領域另有例外選項(多倉、批號、序號、追溯、零稅率出口、代扣等),需要時再加。

**範圍到建模與產出資料庫結構為止。** 執行期(開單、過帳、營業稅申報、電子發票、
勞健保申報)不在這個外掛裡。

---

## 7. 部署到資料庫

完成後專案資料夾裡會有兩樣東西:

| 檔案 | 說明 |
|---|---|
| `.vertixs/*.json` | 藍圖。建議加入版控,之後要修改流程時代理人會讀它 |
| `schema.sql` | PostgreSQL 結構,預設 schema 為 `public` |

套用到您的資料庫:

```bash
psql "postgresql://使用者:密碼@主機:5432/資料庫" -f schema.sql
```

或用 pgAdmin、DBeaver 等工具開啟 `schema.sql` 執行。

**請先在測試資料庫執行確認無誤,再套用到正式環境。**

---

## 8. 把它在 Vertixs 上跑起來(免自建資料庫)

不想自己架資料庫、寫前端?把藍圖推到 Vertixs 雲端,直接就有可操作的營運台
(開單、簽核、過帳、報表,含台灣稅務與電子發票)。三步:

1. **建一把存取金鑰**：到 Vertixs 雲端後台登入 → **設定 ▸ 存取金鑰** → 建一把
   (勾 `blueprint:write`)。完整金鑰只顯示一次,複製起來。
2. **設成環境變數**：把金鑰設為 `VERTIXS_API_KEY`(外掛全程只用這把金鑰,**不碰你的密碼**)。

   ```bash
   export VERTIXS_API_KEY=vtx_live_你的金鑰
   ```
3. **說「推上雲端」**：在 Claude Code 裡對代理人說「把這個藍圖推上雲端」。推完會給你
   雲端網址,點進去就能開始用。之後改了本機藍圖再推一次,雲端會更新同一份、不會變兩份。

> 反向也行:雲端後台可把藍圖下載成 plugin 格式(DomainModel JSON),本機改完再推回來。

---

## 9. 設定

預設值可直接使用,需要調整時設定環境變數:

| 環境變數 | 預設 | 用途 |
|---|---|---|
| `VERTIXS_BLUEPRINT_DIR` | `<專案>/.vertixs` | 藍圖存放目錄 |
| `VERTIXS_SCHEMA` | `public` | 產出 DDL 的 schema 名稱 |
| `VERTIXS_API_KEY` | (無) | 上雲用的存取金鑰;設了才出現「推上雲端」能力 |
| `VERTIXS_CLOUD_URL` | `https://augmented-erp.vercel.app` | 雲端位址(自架時才需改) |

---

## 10. 疑難排解

| 症狀 | 處理 |
|---|---|
| 裝完看不到工具 | 重新開啟 Claude Code;仍無效則執行 `claude plugin list` 確認狀態為 enabled |
| 「沒有可生成的物件」 | 尚未確認欄位,回覆「請逐項確認後再產出」 |
| 「查無藍圖」 | 藍圖代號不對,回覆「先列出現有藍圖」 |
| 外掛啟動失敗 | 執行 `node -v`,確認為 20 以上 |
| `claude` 找不到指令 | 安裝後需重新開啟終端機 |
| 換一台電腦繼續 | 把專案資料夾(含 `.vertixs`)複製過去即可 |
| 「推上雲端」說要先設金鑰 | 到雲端後台 設定 ▸ 存取金鑰 建一把,設為 `VERTIXS_API_KEY` 後再說一次 |
| 上雲回 401 | 金鑰無效或已撤銷,重發一把 |

---

## 11. 其他 MCP 客戶端

底層是標準的 stdio MCP server,Codex 等客戶端也可以使用:

```bash
git clone https://github.com/chunchinsu/vertixs-blueprint.git
```

`~/.codex/config.toml`:

```toml
[mcp_servers.vertixs]
command = "node"
args = ["/clone 的路徑/vertixs-blueprint/server/plugin-mcp.js"]
env = { VERTIXS_BLUEPRINT_DIR = "/您的專案/.vertixs", VERTIXS_LOG_STDERR = "1" }
```

---

問題回報:https://github.com/chunchinsu/vertixs-blueprint/issues
