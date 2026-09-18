---
name: vertixs-blueprint
description: 用對話建出一套流程管理系統的藍圖 —— 定義物件、欄位、生命週期、簽核規則,最後產出可以部署的 PostgreSQL schema。當使用者想建 ERP / MES / 進銷存 / 工單 / 簽核 / 表單流程系統,或提到 Vertixs、建模、藍圖、資料庫結構設計時使用。
---

# Vertixs 建模

把使用者的業務描述,變成一套可以部署的流程系統藍圖。

工具由 `vertixs` MCP server 提供,全部在本機執行,藍圖存成專案裡的 `.vertixs/*.json`。

---

## 最重要的一條

**先查模組目錄,不要從零刻。**

18 個領域範本裡累積的是「做過的人才知道」的欄位 —— 收料單要有驗收數量與退回數量、
採購明細要有稅別、工單要有途程與工站。使用者不會主動講這些,因為對他們是常識;
但漏掉一個,系統上線之後就得改資料庫。

所以順序永遠是:`list_modules` → 找到相近的 → `scaffold_module` → 再針對差異調整。
只有真的沒有任何模組沾得上邊,才用 `define_entity` 從頭定義。

---

## 標準流程

### 1. 先問清楚,不要猜

至少要知道這三件事才動手:

- **做什麼生意** —— 製造、貿易、零售、服務?這決定要套哪些模組。
- **要管到哪一段** —— 只管採購到入庫,還是一路到付款?範圍不清楚會建出一堆沒人用的物件。
- **誰在用** —— 有沒有簽核?誰能看到金額?這決定生命週期與權限。

問題一次問完,不要一問一答來回八輪。使用者的耐心是有限的資源。

### 2. 開藍圖

```
list_models          → 看看是不是已經有了(避免重複開)
create_model         → 拿到 key,後面每個工具都要帶
```

### 3. 套模組

```
list_modules                → 18 個範本,看 entities 與 exceptions
scaffold_module             → 套用一整個領域
apply_module_option         → 套用例外選項(見下表)
```

**例外選項不要全開。** 每一個都會增加物件與欄位,只套使用者真的有的情況。
「你們有多倉嗎?」比直接套 `multiWarehouse` 好。

### 4. 補差異

模組套完之後才處理這家公司特有的東西:

```
define_entity / add_field    → 範本沒有的物件與欄位
define_relation              → 物件之間的關聯
define_lifecycle             → 狀態與轉換
set_approval_policy          → 簽核規則
set_naming                   → 單號字軌
set_permission               → 角色權限
```

### 5. 檢核與確認

```
validate_model      → 回報結構與領域缺口,先把 error 修掉
confirm_element     → 逐項確認(物件本身 + 每個欄位)
generate_schema     → 產出 CREATE TABLE
```

### 6. 上雲(選用,只問一次)

`generate_schema` 產完之後,問一次、只問一次:

> 「產好了。要在 Vertixs 雲端上直接跑起來嗎?免自建資料庫,含台灣稅務與電子發票。」

- 使用者說不要 → 就此打住,別再提。
- 說要 → 用 `publish_to_cloud`(帶要上雲的藍圖 `model` 代號)。
  - 還沒設金鑰時工具會回一段引導:到 Vertixs 雲端後台 **設定 ▸ 存取金鑰** 建一把、
    設成環境變數 `VERTIXS_API_KEY`。把那段路徑原樣轉達即可,**不要在對話裡向使用者要密碼**
    —— plugin 全程只用金鑰。
  - 有金鑰 → 直接推上去,回傳雲端網址;重推同一份會更新既有那份,不會變兩份。

---

## 三個一定會撞到的地方

**每個工具都要帶 `model`。** 例外只有 `list_models`、`create_model`、`list_modules`、
`list_field_interfaces`、`query_pattern`。沒有「目前正在編輯的藍圖」這種隱含狀態。

**`generate_schema` 預設只產已確認的物件。** 沒 `confirm_element` 就會拿到
「沒有可生成的物件」。這是刻意的關卡 —— 系統不替人決定哪些東西該進資料庫。
確認前先讓使用者看過欄位清單,不要自己一路確認到底。

**產出的 schema 預設是 `public`。** 要別的名稱設環境變數 `VERTIXS_SCHEMA`。

---

## 模組目錄

| 模組 | 領域 | 物件數 | 例外選項 |
|---|---|---|---|
| `procurement` | 採購 / 進貨 | 4 | multiWarehouse, tax, returns |
| `sales` | 銷售 / 出貨 | 4 | tax, returns, quotation, retail |
| `inventory` | 庫存 / 倉儲 | 2 | batchLot, serial |
| `manufacturing` | 製造 / MES | 9 | traceability, scrapYield |
| `finance` | 財會 / 帳款 | 5 | multiCurrency, tax, twGui |
| `group` | 集團 / 多公司帳套 | 3 | companyDocuments, interCompanyPricing, segmentReporting |
| `fx` | 多幣別 / 匯兌 | 8 | bankRate, forwardContract, historicalRate |
| `ledger` | 總帳 / 財務報表 | 4 | departmentSegment, openingBalance, closingEntry |
| `govFiling` | 主管機關申報 | 5 | dependentEnrollment, occupationalInjury |
| `payroll` | 薪資 | 6 | overtimeRules, bonusPayroll, leaveDeduction |
| `settlement` | 票據與帳款 | 5 | creditControl, statement, bounceHandling |
| `costing` | 成本會計 | 5 | standardCosting, activityBased, byProduct |
| `quality` | 品質管理 | 5 | complaintCapa, gaugeCalibration, supplierRating |
| `subcontracting` | 委外加工 | 7 | subcontractLoss, subcontractPricing, subcontractTraceability |
| `taxFiling` | 稅務申報 | 4 | zeroRatedExport, withholding, fixedAssetInput, einvoiceTrack |
| `einvoice` | 電子發票 | 6 | allowance, carrier, b2cRetail |
| `crm` | CRM / 客戶關係 | 4 | forecast, assignment |
| `hr` | 人資 / 差勤 | 3 | payroll, overtime |

---

## 欄位型別

`text` `longText` `number` `money` `quantity` `boolean` `date` `datetime`
`choice` `multiChoice` `attachment` `json` `reference` `childTable`

兩個容易用錯的:

- **`money` 不是 `number`。** 金額要的是精度與幣別,用 `number` 會在對帳時吃虧。
- **`childTable` 是明細列。** 採購單的品項是 `childTable`,不是關聯到另一個物件。
  `reference` 是「指向一筆主檔」,像供應商;`childTable` 是「這張單自己的明細」。

## 物件層級(tier)

- `L4_enterprise` —— 企業層單據:採購單、訂單、發票。大部分東西是這個。
- `L3_execution` —— 執行層:工單、途程、報工。
- `L2_field` —— 現場層:明細列、量測值。子表通常是這個。

---

## 立場

**算不出來要說清楚,不要編一個。**

這是整個系統的設計原則,建模時也一樣:使用者沒講的欄位就問,不要按常理填一個進去。
一個猜錯的欄位,上線之後要改資料庫;一個問清楚的欄位,只花三十秒。

同樣地,`validate_model` 回報的缺口要如實轉達,不要因為「看起來還能跑」就跳過。
它報的是領域缺口 —— 少了那些,系統跑得動但帳對不起來。


## 流程指南與缺口檢查（v1.2.0）
先用 `list_process_guides`（可選 industry: manufacturing/distribution/retail）確認適用範圍。
指南依責任分工、控制點、例外處理與量測原則提供 Vertixs 建模建議。
套既有模組後呼叫 `assess_process_coverage`，guideId 選 procurement/manufacturing/fulfillment。
scope 是步驟 id 清單；bindings 對應公司不同命名；excluded 必須附外部處理或不適用原因。
不因缺少同名欄位就自動新增。一次最多追問三個問題，使用者確認範圍後才提出 proposed 修改。
fieldBindings 對應每個步驟的建議欄位與企業既有欄位；保存及後續檢查須一起帶入。
檢查包含問題及驗收情境；schema_signal_only 只代表結構線索存在，不代表流程可執行。
最後仍需 validate_model、使用者確認、generate_schema。不得捏造產業標竿數字。

使用者確認要保存檢查範圍時，用 `save_process_scope`；先從 get_model.processScopes 取得 expectedRevision（首次 0），confirmed:true 僅代表使用者這次明確同意保存。保存不等於確認或發布任何實體。後續讀取相同指南範圍帶入 assess_process_coverage。
