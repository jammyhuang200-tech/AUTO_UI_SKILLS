---
name: adv_autoui-workflow
description: 依使用者目標、產品文件與已發布元件能力，規劃 AutoUI V4.1 批次方法呼叫，透過既有 Validator 與 Runner 執行並分析結果。適用於桌面 UI 測試架構、協作與產品接入。
---

# AutoUI：Agentic UI Testing 架構與協作規範 (V4.1)

本文件定義 AutoUI 的核心架構、元件契約、Agent 協作、執行與發布邊界，供框架維護者、產品接入者、測試作者與 Agent 共同使用。

適用基線為 32 位元 .NET WinForms、標準控制項、Legacy／自繪（Owner-drawn）控制項與跨桌面 Broker。其他桌面技術需另行提供並驗證 Adapter。目標程式為 32 位元，不代表所有 Python／Bridge 程序都必須採相同位元數；相容組合應由部署版本明確列出。

**文件定位**：本文件提供批次方法呼叫架構的協作規範與驗收基準。實際可用能力以部署環境的 Schema、Catalog、Registry 與測試證據為準；文件本身不構成實作驗證或回歸測試證明。

---

## 1. 核心定義

AutoUI 測試框架接受結構化的 **V4 JSON Execution Plan**，呼叫已發布的元件方法，回傳可追蹤的結構化執行事實。

| 核心層 | 定義 |
|---|---|
| **Capability** | 元件發布可呼叫的方法、輸入、輸出、前置條件與副作用契約 |
| **Execution Plan** | 外部 Agent 指定 target、method、args、順序與必要結果綁定的唯一 JSON IR |
| **Runner** | Python 依計畫分派方法，保存結構化結果與證據，並結束批次 |

```text
Agent 規劃 API (JSON Plan) → Python 呼叫 API (Runner) → Components 封裝 UI/硬體操作
                                                                ↓
Agent 分析結果 ← 結構化執行事實 (Artifacts + SHA-256) ←──────────┘
```

* **規劃哲學**：採 Goal-driven + Document-constrained。使用者決定目標；產品文件 (SOP/Policy) 提供流程與安全約束；目前 UI 狀態決定起始上下文；已發布的 Method Catalog 限制可呼叫的方法。
* **Plan 本質**：Execution Plan 是批次 API 呼叫資料（中間表示式 IR），不承載底層 UI 操作實作（如 UIA 指令、座標、等待重試）。即使沒有 Agent 介入，預先驗證的 Plan 也能透過相同框架在 CI/CD 環境確定性重播。
* **完整系統邊界**：`完整 AutoUI 系統 = Agent 外部規劃與分析 + 共享框架 SDK (uia-agent-framework) + 產品適配資料 (Page Views, Typed Components, Policies)`。安裝本規範文件本身不等於安裝執行期環境。

---

## 2. 整體流程與雙軌交付

```text
User Intent + Mandatory Policy YAML + Current UI State
                         +
Page Object Views + Method Catalog + Component Inventory
                         ↓
             Agent Planner (外部 Agent)
                         ↓
              V4 JSON Execution Plan
                         ↓
                  V4PlanValidator
                         ↓
             Validated Execution Plan
                 ├─ 立即執行 (Execute Now) ──────────┐
                 └─ 發布 Managed Package             │
                          ↓                          │
                   Standalone／CI 執行               │
                          ↓                          │
                    相同 Validator                   │
                          └──────────────────────────┤
                                                     ↓
                                       V4Runner + Dispatcher
                                                     ↓
                                     Component Registry + Runtime
                                                     ↓
                                             Typed Components
                                                     ↓
                                            Locator／Smart Wait
                                                     ↓
                                         Desktop Bridge (input/remote)
                                                     ↓
                                        WinForms / Legacy UI
                                                     ↓
                                  Execution Results + Artifact Manifest
                                                     ↓
                                      Agent Analyzer／Quality Gate
                                                     ↓
                                    完成、阻擋，或產生新 Plan 再驗證
```

驗證完成後具備兩條標準交付路徑：
1. **立即執行 (Execute Now)**：直接在具備授權的連線環境下調度執行。
2. **發布 Managed Script**：凍結語意計畫與版本依賴，封裝為獨立的 Standalone / CI/CD 可重播套件。

---

## 3. 分工與依賴架構

| 模組／資料 | 責任與邊界 |
|---|---|
| **User Intent** | 目標、起始條件、輸入、授權範圍、成功條件 |
| **Mandatory Policy YAML** | 產品與硬體安全規則、寫入防護、前置探測與復原要求 |
| **Current UI State** | 帶有觀察時間與來源的目前頁面、值與可操作狀態 |
| **Page Object View YAML** | 按頁面組織已發布的元件能力與契約索引（非可執行計畫） |
| **Method Catalog** | 型別可用方法、參數、回傳型別、適用條件與副作用契約 |
| **Component Inventory** | 語意 target 與元件型別的清單 |
| **Agent Planner** | 外部 Agent 職責；選擇合法能力，產生完整批次 JSON Plan |
| **Method Retriever** | 專案內建確定性詞彙檢索工具（如 `--goal`），非推理引擎 |
| **Plan Validator** | 靜態檢查呼叫、型別、參照、變數作用域與結構限制 |
| **Runner Context** | 負責批次調度、遍歷 calls、解決結果參照與記錄批次事實 |
| **ComponentRuntime** | 負責管理 Bridge 連線、實例生命週期與驅動環境 |
| **Dispatcher** | 從明確公開方法表分派呼叫，拒絕未知方法 |
| **Component Registry** | 將唯一語意 target 對應到 Typed Component 實例 |
| **Typed Components** | 實現方法契約，封裝定位、同步、訊息框、驗證與復原 |
| **Locator／Adapter** | 私有定位與底層 UIA／Desktop Bridge 實體操作 |
| **Artifact Store** | 建立隔離 Run 目錄、`manifest.json`、`result.json`、SHA-256 與截圖 |
| **Agent Analyzer** | 依結構化事實評估目標；不篡改結果或暗中執行補救操作 |
| **Quality Gate** | 依事先定義的驗收規則判定通過或阻擋 |

* **依賴方向**：`Planner → Capability`；`Plan → 公開契約`；`Runner → Validator／Dispatcher／Registry`；`Components → Locator／Adapter`；`專案 → 共享 uia-agent-framework SDK`。
* **禁止反向耦合**：Runner 不依賴使用者目標或 Agent 推理；Plan 不依賴定位器；Agent 不依賴 Python 類別路徑；Component 不依賴特定測試案例；Adapter 不依賴 Plan 語法。

---

## 4. 檔案與三態資料契約 (Capability 與方法契約)

專案中嚴格區分三種不同角色的檔案：

1. **Mandatory Policy YAML**（如 `amax-5070-io-module-utility.yaml`）：定義安全限制、唯讀規範、寫入前置探測條件與還原要求。
2. **Page Object View YAML**（如 `page_views/*.yaml`）：保存頁面語意結構、元件型別與狀態標記（`runtime_published`, `pending_v4_binding`, `proposed_contract`）。**強調 View 不是可執行計畫**。
3. **V4 JSON Execution Plan**（`execution_plan/*.json`）：唯一合法交由 Runner 執行的結構化呼叫序列。

### 4.1 Method Catalog

每個公開方法應在 Catalog 中說明：
* 名稱、用途、適用元件型別。
* 必要與選用參數、型別及值限制。
* 回傳型別、可觀察結果（Observation）及錯誤契約。
* **Precondition**：方法適用的狀態與輸入條件。
* **Observation**：方法提供的結果與狀態來源。
* **Side-effects**：副作用、是否可安全重複呼叫、還原責任與失敗邊界。

Catalog 嚴格禁止暴露 AutomationId、選擇器、座標、UIA patterns、Win32 Handle 或等待重試實作。

### 4.2 Page Object View 與狀態標記

View 屬於 Capability 層，按頁面整理型別、方法與觀察來源：
* `runtime_published`：已在 Registry 登錄並實作，可直接由 Planner 引用。
* `pending_v4_binding`：已定義但尚未完成底層適配，不可執行。
* `proposed_contract`：設計草案，不能規劃執行。

---

## 5. Agent Planner 協作流程

Agent Planner 遵循 6 大核心步驟進行規劃：

1. **理解目標與限制**：確認使用者目標、輸入、寫入授權、安全限制與可觀察成功條件。
2. **查閱 Page Object View**：只載入與目標相關的頁面，且僅使用標記為 `runtime_published` 的能力。
3. **檢索 Method Catalog**：透過本機 Method Retriever 或 Catalog 檢索候選方法，確認參數、回傳型別與前置條件。
4. **組合完整 Execution Plan**：產生合規的 JSON Plan，定義 `target`、`method`、`args`、`save_as` 與受限控制流。
5. **交由 Plan Validator 檢查**：在任何 UI 操作前執行靜態檢驗；失敗時依錯誤代碼重新規劃。
6. **輸出驗證結果**：通過後依任務授權選擇「立即執行」或「發布 Managed Script」。

### 跨操作不變條件 (Invariants)
若操作包含「選值 $\to$ Apply $\to$ 處理彈窗 $\to$ 驗證 $\to$ 還原」的整體流程，**必須選擇單一原子 Typed Component 方法（如 `AppliedComboBox.apply_each_option()`）**。禁止 Planner 將其拆解為鬆散的 click/fill 呼叫，所有狀態保護由 Component 內部封裝。

---

## 6. Execution Plan 規範

可執行計畫唯一採用 `schema_version: "4.0"` 的 JSON 格式（非 TC YAML）：

```json
{
  "schema_version": "4.0",
  "plan_id": "ao-slew-rate-test",
  "calls": [
    {
      "id": "step_1",
      "target": "amax5024.ao.slew_rate",
      "method": "apply_each_option",
      "args": {},
      "save_as": "slew_rate_result"
    }
  ]
}
```

* **呼叫欄位**：僅允許 `id`, `target`, `method`, `args`, `save_as`。
* **受限控制流**：允許 bounded `foreach`（遍歷已取得資料）與 `if`（依回傳值分支）。預設最大巢狀深度為 4、呼叫數上限為 500、單一 `foreach` 迭代上限為 100。
* **無獨立 Assertion DSL**：目前 V4 Plan **不包含通用 step-assertion 或 top-level assertion DSL**。前置 guard、訊息框處理、回讀比對與 restoration 完全由已發布的原子 Component 方法負責，並透過方法 result / observation 回報。
* **禁止項目**：嚴禁包含定位器、AutomationId、座標、UIA 指令、任意 Python、eval、lambda、goto 或執行中要求 Agent 推理的指令。

---

## 7. Plan Validator 規範

Validator 使用與 Planner 相同版本的 Catalog、Inventory 與 Schema，在產生任何 UI 副作用前執行靜態檢查：

1. **Target & Method 存在性**：Target 已在 Registry 登錄且唯一，Method 對該型別公開。
2. **參數契約**：必填參數、名稱、值型別及宣告限制完全符合契約。
3. **變數作用域**：結果參照 (`save_as`) 與迴圈變數先定義後使用，型別相容。
4. **控制流約束**：`foreach` 接收可迭代資料，`if` 比較合法型別，深度與呼叫數未超限。
5. **程式碼清白性**：不含未允許的實作欄位、選擇器或可執行程式碼。

Validator 驗證失敗時回傳結構化錯誤代碼、路徑與原因，不自行猜測改寫 Plan。

---

## 8. Runner、Dispatcher、Registry 與共享 SDK

執行體系由共享核心 `uia-agent-framework` SDK 與產品專案分層組成：

* **Runner Context**：負責按序走訪 calls、解析結果參照、執行受限控制流、記錄呼叫狀態與回傳 partial results。遇到未處理例外立即停止，不盲目重跑。
* **Dispatcher**：維護公開方法白名單，只分派已註冊的方法，禁止動態 `getattr` 或呼叫私有方法。
* **Component Registry**：將語意 target 精確映射至 Typed Component 實例。
* **ComponentRuntime**：管理底層 Driver 與 Bridge 連線，維護桌面生命週期。

---

## 9. Typed Components、實體操作與自繪控制項

Component 對方法契約負全責，封裝所有底層實作細節：

### 定位與自繪控制項適配
* **定位優先序**：`AutomationId → Name + ControlType → ClassName + 祖先範圍 → Semantic → Vision → 當前驗證座標`。
* **Legacy / Owner-drawn 適配**：針對 WinForms 自繪控制項（如 UIA 遺漏 Row/Cell 的 `ColumnTreeView`），底層由 pywinauto 截圖比對 `+`/`-` 符號與座標雙擊作為健全 Fallback，不將 Provider 缺陷誤報為控制項不存在。
* **跨桌面 Broker**：在 `WinSta0/Default` 互動桌面優先使用 Desktop Bridge 的 `input` 模式；跨 Session 時透過 `desktop-bridge.exe serve` 遠端代理。

### 寫入安全與復原契約
已授權的寫入方法必須在 Component 內部維護嚴格的不變條件：
$$\text{讀取原值與 Guard} \to \text{單次寫入} \to \text{即時回讀} \to \text{記錄並確認 Modal} \to \text{還原原值} \to \text{回讀確認還原} \to \text{回報 restoration\_verified}$$

---

## 10. Live 執行與實體安全閘道

為防止自動化測試誤操作實體設備或干擾非目標程式，Live 執行必須同時通過以下安全閘門：

* **`--profile <profile>`**：明確指定模組設定檔（如 `ao`, `ai`）。
* **`--plan <plan.json>`**：指定驗證通過的 JSON Plan。
* **`--execute`**：使用者對本次 Live 執行的明確授權標記（未加此參數僅作離線驗證）。
* **`--pid <PID>`**：強制綁定唯一的目標程式 Process ID。
* **`--authorized`**：解鎖可寫 Registry 的安全旗標。
* **`Discovery Gate`**：AI 寫入前必須先具備符合目前政策 Hash 的成功 read-only discovery 證據。

```powershell
# Live 執行標準範例
python run_v4.py --profile ao --authorized --plan execution_plan\plan.json --execute --pid 12345 --bridge-mode input
```

---

## 11. Results、Artifact 證據鏈與 Quality Gate

每次 Live 執行自動建立隔離的 Run 目錄：

```text
artifacts\runs\<script-id>\<timestamp>-<run-id>\
├── plan.json
├── manifest.json       (記錄環境、CLI 參數、時間與依賴版本)
├── result.json         (完整結構化執行事實與 observation)
├── screenshots/        (各步驟截圖)
└── hashes.sha256       (所有產出物的 SHA-256 雜湊)
```

* **`latest.json`**：動態指標，永遠指向最新完成的 Run。
* **Compact Result**：CLI 預設輸出精簡摘要給 Agent，詳細 observation 存於 artifact，避免 context 爆炸。
* **三層判定體系**：區分 `Invocation status`（方法是否成功）、`Goal evaluation`（是否滿足目標）與 `Quality Gate`（是否符合驗收標準）。
* **Retention Policy**：提供 `manage_artifacts.py` 管理工具，依保留策略安全清理（丟入資源回收筒），禁止刪除 Managed Script 依賴的 provenance。

---

## 12. Managed Scripts 與無 Agent 執行

已驗證的 Plan 可透過發布工具封裝為不可變版本化套件：

```powershell
python publish_v4_script.py --profile ao --authorized --plan execution_plan\plan.json --script-id amax5024-ao-test --version 1.0.0
```

* **套件內容**：包含凍結的 Plan、`manifest.json`、SHA-256、`run_test.py`、`run.ps1` 與 `run.cmd`。
* **CI/CD 重播**：在無 Agent 介入的環境下，入口腳本直接調用既有 Validator 與 Runner，以 100% 確定性執行。

---

## 13. 工作模式與維護原則

| 模式 | 說明 |
|---|---|
| **架構討論** | 閱讀與整理契約，不操作 UI、不改程式 |
| **計畫產生** | 外部 Agent 建立並靜態驗證 Plan，不自動執行 |
| **測試執行** | 通過安全閘道，透過 Runner 執行已授權批次 |
| **盤點／診斷** | 依產品政策取得觀察，直接工具操作獨立記錄 |
| **框架維護** | 執行 Runtime Scan $\to$ Diff $\to$ CodeGen 更新 Typed Components |
| **套件發布** | 封裝已驗證 Plan 為 Managed Script |

正常執行嚴禁修改 Runner Core、Driver Core 或共享 SDK，亦不得將產品特定的業務邏輯或 IP 寫入通用框架。

---

## 14. 開源交付與目錄結構標準

```text
uia-agent-framework/ (共享 SDK 核心)
├── validation/         (V4PlanValidator)
├── runner/             (V4Runner, Dispatcher)
├── runtime/            (ComponentRuntime)
└── interfaces/         (MethodSpec, ComponentBase)

festo_ui_automation/ (產品專案庫)
├── planner/            (agent_planner_contract.md)
├── capabilities/       (method_catalog.json, metadata)
├── page_views/         (Page View YAMLs, runtime_bindings.json)
├── execution_plan/     (V4 JSON Plans)
├── components/         (AMAX Typed Components, Registry)
├── policies/           (Mandatory Policy YAMLs)
├── managed-scripts/    (發布之版本化套件庫)
├── artifacts/          (隔離之執行證據與紀錄庫)
└── run_v4.py           (CLI 整合入口與檢索工具)
```

---

## 15. 通用架構規範標準 (10 大核心準則)

1. **統一中間表示式 (Unified IR)**：全面採用符合 Schema 4.0 的結構化 JSON Execution Plan 作為唯一合法的執行輸入，淘汰傳統腳本式 TC YAML。
2. **三態資料角色分離 (Tripartite Data Separation)**：嚴格劃分 Policy YAML（安全策略）、Page View YAML（介面靜態契約）與 JSON Plan（動態執行序列）。
3. **原子能力封裝斷言 (Atomic Component-Encapsulated Assertions)**：取消獨立 Assertion DSL，狀態 guard、回讀比對與復原驗證完全由原子 Component 方法負責。
4. **調度與驅動雙層上下文解耦 (Dual-Layer Context Decoupling)**：執行期拆分為調度層的 Runner Context 與驅動層的 ComponentRuntime。
5. **外部規劃與本機確定性邊界 (External Planner & Deterministic Boundary)**：Agent Planner 定位為外部智慧體，本機專案為 100% 確定性執行引擎，僅提供規劃契約與檢索工具。
6. **核心 SDK 與產品模組分層 (Core SDK & Product Specialization)**：通用機制沉澱於 `uia-agent-framework` SDK，應用專案僅專注於產品專屬的 Components 與 Page Views。
7. **交付雙軌制 (Dual-Track Execution & Packaging)**：合法 Plan 兼具「即時測試執行」與「發布為不可變 Managed Script 套件」雙重能力。
8. **多層級實體安全閘道 (Multi-Tiered Safety Gates)**：透過 `--execute`、`--pid`、`--authorized` 與 Discovery Gate 構建嚴密的防護網。
9. **結構化證據鏈與生命週期管理 (Evidence Engineering & Retention)**：每次執行生成含 SHA-256 的隔離證據鏈，CLI 對外暴露 Compact Result，並支援 Retention 管理。
10. **混合控制項適配與跨桌面穿透 (Hybrid Component & Desktop Broker)**：驅動層全面支援標準 UIA、WinForms 自繪（Owner-drawn）控制項與跨 Session/桌面 Desktop Bridge Broker。

> **Method Catalog 說明能呼叫什麼；Execution Plan 指定要呼叫什麼；Runner 執行並回傳事實；Agent Analyzer 根據事實判定目標是否達成。**
