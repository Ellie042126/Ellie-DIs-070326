# 智慧型醫療器材通路與採購分析系統 (DHA-Hub)

## 系統架構與技術規格說明書 (Technical Specification)

本規格書旨在定義一套專為醫療器材（特別是高風險、具追溯性之植入式醫材，如心臟節律器）設計的「雙層型自主 Agent 決報系統 (DHA-Hub)」。本系統不採用向量資料庫（No Vector DB），完全基於「Code-as-the-Agent（程式即代理）」與「動態上下文明細注入」架構，以每月定期執行的通路與採購大數據分析為核心目標。

---

## 1. 系統架構與設計哲學

### 1.1 核心設計原則

1. **極致 Token 節約 (Token Frugality)**：不將百萬行的原始 CSV/JSON 數據直接塞入大型語言模型 (LLM) 的上下文視窗 (Context Window)。
2. **計算流與推理流分離 (Decoupling Computation & Inference)**：
* **計算與過濾（程式碼執行）**：由系統在本地安全沙箱環境中，透過 LLM 生成的 Python 程式碼或硬編碼核心模組（Pandas/SciPy/NetworkX）本地執行。
* **推理與策略生成（LLM 運算）**：LLM 僅讀取程式碼執行後輸出的「高度聚合統計矩陣」、「異常代碼列表」與「網路圖拓撲特徵」。


3. **無向量資料庫架構 (No Vector DB Architecture)**：放棄 RAG (檢索增強生成) 的向量嵌入與檢索機制。每月一次的分析具有高度的時間序列與結構化特性，系統採用「關聯式記憶緩存 (State-DB)」與「動態 Schema 腳本映射」來完成上下文對齊。
4. **雙層雙模組設計 (Dual-Layer Model Gateway)**：
* **本地核心應用 (Local App)**：預設使用 `gemini-3.1-flash-lite`（可切換其他模型），負責高頻次、低成本的代碼生成、數據清洗、欄位映射校對與基礎異常篩選。
* **雲端高階應用 (Claude App / SUPER LLM)**：預設使用 `gemini-3.5-flash`（可切換其他高級模型，如 Claude 3.5 Sonnet / GPT-4o），專門負責跨月度趨勢綜合推理、供應鏈斷鏈風險預測、地緣物流瓶頸診斷及最終高階策略報告撰寫。



### 1.2 雙層模型系統工作流

```
[每月原始數據: 採購/通路/地理資訊]
         │
         ▼
 ┌──────────────────────────────────────────────┐
 │             Local App (低成本/高效率)         │
 │  預設模型: gemini-3.1-flash-lite             │
 ├──────────────────────────────────────────────┤
 │  1. 欄位 Schema 校正與動態代碼生成            │
 │  2. 本地執行 Pandas/NetworkX 計算             │
 │  3. 產出統計矩陣、批號追溯表與異常指標          │
 └──────────────────────┬───────────────────────┘
                        │
                        ▼ [僅傳輸高度壓縮的分析 JSON 與摘要]
 ┌──────────────────────────────────────────────┐
 │         Claude App / SUPER LLM (高階推理)    │
 │  預設模型: gemini-3.5-flash                  │
 ├──────────────────────────────────────────────┤
 │  1. 跨月度供應鏈斷鏈風險推理                  │
 │  2. 地理物流網絡 (Hub-and-Spoke) 瓶頸診斷      │
 │  3. 撰寫符合醫療法規之月度決策策略報告          │
 └──────────────────────────────────────────────┘

```

---

## 2. 數據源架構與語意分析 (Data Schema & Semantics)

系統每月輸入三個核心數據集。以下基於提供的範例數據進行結構化定義與欄位語意解析。

### 2.1 採購數據集 (Purchase Dataset Schema)

記錄各醫療機構或申報業者向供應商採購醫療器材的進貨明細。具有高度的品項追溯欄位。

| 欄位名稱 | 範例數據 | 資料型態 | 語意說明與分析維度 |
| --- | --- | --- | --- |
| **序號** | 150 | Integer | 唯一流水號。 |
| **申報業者** | A00013 | String (Key) | 進貨之法人主體編碼（通常為大型醫院群或一級經銷商）。 |
| **收貨日期** | 20260429 | Date (YYYYMMDD) | 計算庫存前置時間 (Lead Time) 與採購週期的核心時間戳。 |
| **供應商** | C00306 | String (Key) | 上游製造商或總代理商編碼。可用於分析集中度風險。 |
| **許可證號** | 衛部醫器輸字第030747號 | String | 官方核發之醫療器材許可證字號。用於合規性與單一品項聚合。 |
| **中文品名** | “美敦力” 亞士爾磁振造影植入式心臟節律器 | String | 產品官方中文名稱。 |
| **UDI_DI** | 00763000956004 | String (GTIN) | 醫療器材單一識別系統-識別碼 (Unique Device Identifier - Device Identifier)。用於精準識別全球動態品項。 |
| **醫療器材次類別** | E.3610植入式心律器之脈搏產生器 | String | 法規分類碼，用於次類別市佔率與品項大類分析。 |
| **產品批號** | *空值或特定代碼* | String | 生產批次。對於醫材回收 (Recall) 判定至關重要。 |
| **產品序號** | RNJ146480G2001 | String (Unique) | 單一植入物之全球唯一身分證號。本系統之追溯核心（Serial Number）。 |
| **產品型號** | W3DR01 | String | 製造商產品型號，用於規格細分分析。 |
| **數量** | 1 | Integer | 採購數量。 |
| **單位** | 個 / 組 | String | 計量單位。需要執行標準化轉換（如：1組 = 1個，需由 Local App 自動對齊）。 |
| **製造日期** | *空值* | Date | 用於計算產品貨架壽命。 |
| **有效期間** | 20270628 | Date (YYYYMMDD) | 防範過期醫材流入市場之關鍵指標。 |
| **保存期限** | 0 | Integer | 剩餘保存月數或天數。 |
| **退貨資訊** | 1 | Integer | 是否退貨之旗標（0: 正常, 1: 退貨/異常）。 |
| **建立日期** | 2026/05/06 | DateTime | 系統寫入時間。可用於評估申報時效延遲（建立日期減去收貨日期）。 |

### 2.2 通路數據集 (Distribution Dataset Schema)

記錄中游經銷商（申報業者）將產品交付給下游終端對象（醫療機構或次級管道）的流向明細。

| 欄位名稱 | 範例數據 | 資料型態 | 語意說明與分析維度 |
| --- | --- | --- | --- |
| **序號** | 521 | Integer | 唯一流水號。 |
| **申報業者** | B00047 | String (Key) | 出貨之法人主體編碼（通常為總代理或大型中游經銷商）。 |
| **交貨日期** | 20260331 | Date (YYYYMMDD) | 通路流轉時間戳。 |
| **供應對象** | C05816 | String (Key) | 下游接收主體編碼（如：台中榮民總醫院）。 |
| **許可證號** | 衛部醫器輸字第030747號 | String | 用於與採購數據集進行多對多關聯校對。 |
| **醫療器材次類別** | E.3610植入式心律器之脈搏產生器 | String | 品類聚合。 |
| **UDID** | 00763000955953 | String (GTIN) | 產品識別碼（註：欄位名稱在採購數據中為 `UDI_DI`，在通路數據中為 `UDID`，此差異需由 Local App 的程式自動對齊）。 |
| **中文品名** | “美敦力” 亞士爾磁振造影植入式心臟節律器 | String | 品名。 |
| **產品批號** | *空值* | String | 生產批次。 |
| **產品序號** | RNE644378S | String (Unique) | 下游流轉之唯一醫材序號。**用於與採購數據進行生產鏈與銷貨鏈的生命週期碰撞分析。** |
| **產品型號** | W2SR01 | String | 型號。 |
| **數量** | 1 | Integer | 出貨數量。 |
| **單位** | 組 / 個 | String | 單位。 |
| **製造日期** | *空值* | Date | 製造日期。 |
| **有效期間** | 20261214 | Date (YYYYMMDD) | 下游產品之失效日期。 |
| **保存期限** | *空值* | Integer | 保存期限。 |

### 2.3 地理網絡站點數據集 (DHA Hub Geolocation Stations Dataset)

提供實體節點的空間地理資訊與主體屬性，用於構建 Hub-and-Spoke 供應鏈網絡圖。

* **JSON 結構語意**：
* `entity_id`: 映射至採購/通路數據中的「申報業者」、「供應商」或「供應對象」。
* `official_name`: 實體官方名稱（如：國立臺灣大學醫學院附設醫院）。
* `entity_type`: 節點角色定義（`Distributor` 經銷商、`Hospital_Group` 醫療機構群）。
* `postal_code` / `street_address`: 實體地址與郵遞區號。
* `latitude` / `longitude`: 經緯度坐標，用於計算歐幾里得距離或路網流向距離。



---

## 3. 功能模組與 Agent 運作架構

本系統完全基於 **Code-as-the-Agent** 模式，將整體分析拆分為四個階段。前三個階段由 **Local App** 執行，最後一階段由 **Claude App / SUPER LLM** 進行戰略推理。

### 3.1 階段一：Schema 自動對齊與數據清洗（Local App 驅動）

* **痛點**：採購數據與通路數據的欄位命名不一致（例如 `UDI_DI` 與 `UDID`；單位 `個` 與 `組`）。
* **Agent 策略**：
1. 系統載入動態 Prompt，將採購數據、通路數據之前 3 行樣本以及 Geolocation JSON 的 Schema 傳送給 `gemini-3.1-flash-lite`。
2. Local LLM 辨識欄位語意關聯性，生成一個名為 `clean_and_align.py` 的 Python 腳本。
3. 系統自動在本地環境調用該腳本，執行標準化對齊：
* 將所有日期格式統一轉換為 `datetime64[ns]`。
* 將 `UDID` 與 `UDI_DI` 欄位名對齊為標準鍵值。
* 清理產品型號中的雜訊字元（例如將 `# 美敦力亞士爾... W2SR01` 提取出乾淨的 `W2SR01`）。





### 3.2 階段二：串聯追溯與黑洞追蹤（Local App 驅動）

* **核心演算法**：產品序號生命週期碰撞檢驗（Lifecycle Collision Matching）。
* **運算逻辑**：
* 對於每一筆高風險植入式醫材（透過唯一「產品序號」識別），比對其在採購數據集中的「收貨日期」與通路數據集中的「交貨日期」。
* **追溯黑洞 (Traceability Black Hole)**：若某「產品序號」出現在通路出貨明細中，但在過去三個月的採購進貨明細中完全找不到對應的進貨記錄，或者交貨日期早於進貨日期，系統自動將其標記為「來源不明異常」或「時序逆轉異常」。
* Local App 將此邏輯自動轉化為 Pandas 矩陣運算（`pd.merge(..., on='產品序號', how='outer')`），產出精簡的黑洞異常清單 JSON。



### 3.3 階段三：Hub-and-Spoke 物流網絡拓撲分析（Local App 驅動）

* **核心演算法**：基於 `NetworkX` 的有向權重圖分析（Directed Weighted Graph）。
* **運算逻辑**：
* 以 `申報業者`、`供應商`、`供應對象` 為頂點 (Nodes)。
* 以交貨/收貨事件為有向邊 (Edges)，交易數量或頻次為邊的權重 (Weights)。
* 結合 Geolocation 坐標，計算節點間的平均轉運距離。
* **指標計算**：計算各流通樞紐的「度中心性 (Degree Centrality)」與「介數中心性 (Betweenness Centrality)」，藉此識別出誰是整個臺灣醫材供應鏈的核心調度 Hub（例如範例中的 `B00047` 與 `B00446`）。
* 產出壓縮後的網絡特徵摘要（包含 Top 3 樞紐節點、平均流轉度數、異常孤立節點）。



### 3.4 階段四：月度戰略決策推理與預警（Claude App / SUPER LLM 驅動）

* **輸入資料**：前三階段由 Local App 處理完畢後輸出的「高度濃縮統計 JSON」（體積一般 < 5KB，極度節約 Token）。
* **執行任務**：
1. 執行跨維度深度推理：結合當月醫療器材法規、有效期限分佈（例如發現某些特定型號如 `W2SR01` 集中在 2026 年底過期），評估潛在的庫存呆滯或過期流向風險。
2. 撰寫最終版「DHA-Hub 月度通路與採購策略分析報告」，包含：斷鏈風險評級、通路異常審計建議、安全庫存調配決策。



---

## 4. Prompt 工程設計與狀態記憶管理

為了確保在**不使用向量資料庫**的前提下，系統依然能保有精準的上下文記憶，我們設計了一套基於本地關聯資料庫（SQLite / JSON 檔案）的 **State-DB（狀態數據庫）機制**。

### 4.1 狀態記憶數據結構 (State-DB Schema)

每次分析完畢後，系統將核心摘要寫入本地 State-DB，以便下個月分析時，LLM 能透過動態注入「歷史快照」來理解跨月趨勢。

```json
{
  "analysis_month": "2026-04",
  "historical_snapshot": {
    "total_purchase_volume": 25,
    "total_distribution_volume": 22,
    "active_hub_nodes": ["B00047", "B00446"],
    "critical_expired_risk_models": ["W2SR01"],
    "unresolved_black_hole_count": 3
  }
}

```

### 4.2 Local App 系統 Prompt (預設：gemini-3.1-flash-lite)

```markdown
# Role
你是一個精通高階 Pandas 與 NetworkX 的「醫材物流數據科學 Agent」。

# Goal
你的任務是根據用戶提供的資料 Schema 樣本，生成精準、無 Bug、且包含完整異常捕獲機制的 Python 程式碼。

# Rules
1. 只能輸出純 Python 程式碼，包裹在 ```python ... ``` 中。不要包含任何額外的解釋性文字。
2. 必須處理採購數據集的 'UDI_DI' 與通路數據集的 'UDID' 欄位名稱不一致問題，將其對齊為 'standard_udi'。
3. 必須清理產品型號欄位，移除 '#' 及其後的中文雜訊，僅保留代碼。
4. 計算「產品序號」的生命週期匹配率，找出有出貨卻無進貨紀錄的「追溯黑洞」，並輸出為 JSON 格式。
5. 使用 NetworkX 計算有向圖的度中心性，找出 Top 2 核心 Hub。
6. 所有的輸出必須透過 standard output (print) 列印出最終的 JSON 摘要字串，以便主程序捕獲。

```

### 4.3 Claude App / SUPER LLM 系統 Prompt (預設：gemini-3.5-flash)

```markdown
# Role
你是一家頂級醫療器材供應鏈的「資深戰略營運總監兼法規合規官」。

# Task
請根據 Local App 預處理完成的供應鏈拓撲與異常 JSON 矩陣，結合提供的歷史快照資訊，撰寫一份具備深度商業洞察力與法規合規性的【月度通路與採購策略綜合報告】。

# Rules
1. 禁止重複原始的統計數字，必須直接指出隱藏在數據背後的核心風險（如：特定Hub節點集中度過高、過期倒數風險、法規追溯黑洞）。
2. 在報告中提出 3 個具體可執行的優化行動方案。
3. 報告結構必須包含：一、執行摘要；二、通路由徑拓撲健康度分析；三、產品生命週期與合規黑洞審計；四、次月營運優化與斷鏈防範策略。
4. 本次推理必須參考注入的【歷史快照歷史數據】，評估異常指標是惡化還是改善。

```

---

## 5. 原始數據解析與程式碼生成實例

以下展示 Local App 在接收到用戶給出的範例數據後，自動生成的本地數據分析核心代碼 `dha_core_analyser.py` 的結構。主程序會自動執行此段代碼，將輸出結果送給 Claude App。

```python
import pandas as pd
import networkx as nx
import json
import re
from io import StringIO

# 1. 載入原始範例數據 (在實際環境中這將讀取每月上傳的 CSV 檔案)
purchase_raw = """序號,申報業者,收貨日期,供應商,許可證號,中文品名,UDI_DI,醫療器材次類別,產品批號,產品序號,產品型號,數量,單位,製造日期,有效期間,保存期限,退貨資訊,剩餘數量,建立日期
150,A00013,20260429,C00306,衛部醫器輸字第030747號,“美敦力” 亞士爾磁振造影植入式心臟節律器,00763000956004,E.3610植入式心律器之脈搏產生器,,RNJ146480G2001,W3DR01,1,個,,,20270628,0,1,2026/05/06
152,A00013,20260429,C00306,衛部醫器輸字第030747號,“美敦力” 亞士爾磁振造影植入式心臟節律器,00763000956004,E.3610植入式心律器之脈搏產生器,,RNJ146525G2001,W3DR01,1,個,,,20270628,0,1,2026/05/06
178,A00013,20260427,C00306,衛部醫器輸字第030747號,“美敦力” 亞士爾磁振造影植入式心臟節律器,00763000955953,E.3610植入式心律器之脈搏產生器,,RNE644291S2001,W2SR01,1,個,,,20261214,0,1,2026/05/06
362,A00002,20260409,C04961,衛部醫器輸字第030747號,“美敦力” 亞士爾磁振造影植入式心臟節律器,00763000955960,E.3610植入式心律器之脈搏產生器,RNJ769542S,,W3DR01,1,組,,,20260714,0,1,2026/04/13
530,A00032,20260331,C04961,衛部醫器輸字第030747號,“美敦力” 亞士爾磁振造影植入式心臟節律器,00763000955953,E.3610植入式心律器之脈搏產生器,,RNE644338S,# 美敦力亞士爾磁振造影植入式心臟節律器-單腔 W2SR01,1,個,,,20261214,0,1,2026/04/13"""

distribution_raw = """序號,申報業者,交貨日期,供應對象,許可證號,醫療器材次類別,UDID,中文品名,產品批號,產品序號,產品型號,數量,單位,製造日期,有效期間,保存期限
521,B00047,20260331,C05816,衛部醫器輸字第030747號,E.3610植入式心律器之脈搏產生器,00763000955953,“美敦力” 亞士爾磁振造影植入式心臟節律器,,RNE644378S,W2SR01,1,組,,,20261214
523,B00446,20260331,C07359,衛部醫器輸字第030747號,E.3610植入式心律器之脈搏產生器,00763000956004,“美敦力” 亞士爾磁振造影植入式心臟節律器,,RNJ135962G,W3DR01,1,個,,,20270114
1026,B00446,20260326,C05965,衛部醫器輸字第030747號,E.3610植入式心律器之脈搏產生器,00763000955960,“美敦力” 亞士爾磁振造影植入式心臟節律器,,RNJ769542S,W3DR01,1,個,,,20260714"""

# 2. 數據轉換與清洗
df_pur = pd.read_csv(StringIO(purchase_raw))
df_dist = pd.read_csv(StringIO(distribution_raw))

# 欄位對齊
df_pur.rename(columns={'UDI_DI': 'standard_udi'}, inplace=True)
df_dist.rename(columns={'UDID': 'standard_udi'}, inplace=True)

# 產品型號清洗常規化
def clean_model(val):
    if pd.isna(val): return val
    match = re.search(r'[A-Za-z0-9]+$', str(val).strip())
    return match.group(0) if match else val

df_pur['產品型號'] = df_pur['產品型號'].apply(clean_model)
df_dist['產品型號'] = df_dist['產品型號'].apply(clean_model)

# 3. 追溯黑洞碰撞分析
# 提取各自的序號集
pur_serials = set(df_pur['產品序號'].dropna().unique())
dist_serials = set(df_dist['產品序號'].dropna().unique())

# 在通路中出現，但採購明細中未包含的序號 = 黑洞異常
black_hole_serials = dist_serials - pur_serials

# 4. 建立地理供應鏈網絡圖
G = nx.DiGraph()
for _, row in df_dist.iterrows():
    G.add_edge(row['申報業者'], row['供應對象'], weight=int(row['數量']))

# 計算度中心性
centrality = nx.degree_centrality(G)
sorted_hubs = sorted(centrality.items(), key=lambda x: x[1], reverse=True)

# 5. 生成高度壓縮的統計結果 JSON 字串
analysis_output = {
    "metrics": {
        "processed_purchase_records": len(df_pur),
        "processed_distribution_records": len(df_dist),
        "detected_black_hole_count": len(black_hole_serials),
        "black_hole_list": list(black_hole_serials)
    },
    "network_topology": {
        "identified_hubs": [node for node, score in sorted_hubs[:2]],
        "edge_count": G.number_of_edges()
    },
    "risk_alerts": {
        "near_expired_items_count": len(df_dist[df_dist['有效期間'].astype(str).str.startswith('2026')])
    }
}

print(json.dumps(analysis_output, indent=2, ensure_ascii=False))

```

---

## 6. 系統整合、API 管理與例外處理機制

### 6.1 本地與雲端雙模組網關切換 (Dual-App Router Engine)

主程式採用非同步事件驅動設計，提供完全解耦的模型介接介面。用戶可在 GUI 或設定檔中指定當前分析應調用的主/副模型。

```python
import os
import google.generativeai as genai

class DHAModelGateway:
    def __init__(self, local_model_name="gemini-3.1-flash-lite", cloud_model_name="gemini-3.5-flash"):
        self.local_model = local_model_name
        self.cloud_model = cloud_model_name
        # 初始 API 金鑰配置
        genai.configure(api_key=os.getenv("GEMINI_API_KEY"))

    def call_local_agent(self, prompt, schema_context):
        """用於執行高頻數據清洗與代碼生成"""
        # 模型調用邏輯與安全過濾器設定
        model = genai.GenerativeModel(self.local_model)
        response = model.generate_content(f"{prompt}\n\nSchema Context:\n{schema_context}")
        return response.text

    def call_cloud_super_llm(self, aggregated_json, historical_snapshot):
        """用於高階綜合策略推理（極省 Token）"""
        model = genai.GenerativeModel(self.cloud_model)
        combined_prompt = f"Data Summary: {aggregated_json}\nHistory Snap: {historical_snapshot}"
        response = model.generate_content(combined_prompt)
        return response.text

```

### 6.2 例外與回退處理機制 (Exception & Fallback Mechanism)

1. **程式碼執行沙箱異常**：若 Local App 生成的 Python 腳本在執行時崩潰（例如引發 `KeyError` 或 `ValueError`），主控程序將捕獲 `sys.stderr` 錯誤訊息，連同前次代碼再次餵回給 Local App 進行 **Self-Correction（自我修正機制）**。最多重試 3 次，若皆失敗則切換為硬編碼的備用清洗模組（Fallback Baseline Code）。
2. **Token 超載與速率限制 (Rate Limit - 429)**：雖然本架構單次調用僅消耗幾千個 Token，但若因併發任務觸發 Rate Limit，Gateway 會自動啟用**指數型後退重試算法 (Exponential Backoff)**，在重試間隔中加入動態隨機抖動 (Jitter)，確保系統的高可用性。
3. **無損法規追溯例外中斷**：一旦 `detected_black_hole_count` 比例超過當月出貨數據的 15%，系統會觸發硬性中斷，發出高危合規性警報（Critical Compliance Alert），不進入雲端策略推理，直接要求人工審查數據源頭。

---

## 7. 20 個深度全面性後續追蹤問題 (20 Comprehensive Follow-up Questions)

為了進一步優化 DHA-Hub 系統在實際生產環境中的部署，請針對以下 20 個關鍵技術、商業與法規層面的問題進行評估與調整：

### 系統架構與 Token 優化層面

1. **模型降級可行性**：當 10M 高級 Token 剩餘量低於臨界值時，Local App 是否能完全在本地部署開源的 `Llama-3-8B-Instruct` 進行程式碼生成，將所有的 API 額度百分之百保留給 Claude App 雲端推理？
2. **Prompt 緩存策略**：您所選用的模型提供商（如 Google Vertex AI 或 Google AI Studio）是否已開通自動 Prompt 緩存 (Prompt Caching) 功能？如果是，如何優化我們的系統 Prompt 結構以最大化快取命中率、進一步將 Token 費用降低 50% 以上？
3. **代碼沙箱安全性**：Local App 生成的 Python 腳本將在何種環境下執行？是否需要配置 Docker 輕量化隔離沙箱，以防止 LLM 意外生成具破壞性的 I/O 操作代碼（例如刪除本地原始數據檔）？

### 數據結構與品質控制層面

4. **缺失值與冷啟動處理**：在範例數據中，多筆進貨與出貨記錄的「產品批號」欄位為空字串，且許多醫院的「製造日期」未填寫。當系統計算貨架壽命時，應採用何種推估邏輯（例如：依據有效期間往前推算固定年限）作為預設程式邏輯？
5. **多重計量單位轉換**：採購數據中同時出現「個」與「組」兩種單位（例如序號 203 為 1組，序號 202 為 1個），系統該如何建立動態對照表，以防程式碼在聚合數量（Quantity）時產生因單位不同而导致的計算偏誤？
6. **非結構化備註解析**：採購數據第 530 行的產品型號包含額外的中文備註（`# 美敦力亞士爾磁振造影植入式心臟節律器-單腔 W2SR01`），除了使用正則表達式，如何讓 Local App 在代碼生成時，能自動識別此類異質數據模式並予以清洗？

### 供應鏈演算法與業務邏輯層面

7. **黑洞時間窗口定義**：判定「追溯黑洞」時，是否需要加入時間緩衝期？例如，若某醫材的通路交貨日期為 3 月 31 日，但採購申報日期因行政延遲到了 4 月 5 日才寫入，系統是否應該允許一個「跨月追溯容忍視窗」（如正負 14 天）？
8. **網絡中心性之業務對應**：透過 NetworkX 計算出的「介數中心性 (Betweenness Centrality)」在醫材通路業務中代表何種實質含意？若某中游業者（如 `B00047`）的介數中心性極高，這代表它是效率樞紐，還是單點脆弱性風險（Single Point of Failure）？
9. **退貨指標整合**：採購數據中包含 `退貨資訊` 欄位（例如範例均標示為 1）。在生命週期碰撞分析中，已被標記為退貨的產品序號是否應自動從「可用庫存網絡」中扣除？其物流轉運圖的方向應如何反轉計算？

### 系統狀態與跨月记忆管理層面

10. **State-DB 的擴展性**：在完全不使用向量資料庫的前提下，隨著累積的月份變多（如運行 3 年共 36 個月），本地 SQLite 的快照摘要大小將逐漸增加。我們應該採取何種滑動窗口策略（例如：僅動態注向前 3 個月詳細快照 + 去年同期的年同比快照）來避免 Context 膨脹？
11. **歷史趨勢指標定義**：為了讓 Claude App 能精準識別趨勢，State-DB 應該額外儲存哪些跨月衍生指標？（例如：黑洞序號的月增長率、樞紐節點的權重飄移係數等）。
12. **數據版本控制與重算機制**：若某醫療機構在 6 月份修正了其 4 月份的申報數據，本系統的無向量架構如何自動觸發歷史月份的程式碼重算，並同步更新 State-DB 中的歷史快照？

### 地理空間與物流優化層面

13. **真實路網與歐幾里得距離**：Geolocation 站點數據集提供了精確的經緯度。Local App 在計算經銷商到醫院的配送半徑時，僅計算直線距離是否足夠？是否需要引入外部開源 API（如 OSRM - Open Source Routing Machine）來獲取真實的臺灣公路車程時間？
14. **冷鏈與特殊物流限制**：特定心臟節律器或高階醫材在運送中可能有嚴格的溫濕度控管要求。我們的地理網絡圖是否需要針對「具備冷鏈資質的 Hub 節點」加入條件約束，並在拓撲分析中將不合規的轉運路徑篩選出來？
15. **區域集中度預警機制**：如何利用地址中的 `postal_code`（郵遞區號）前三碼，讓系統自動對特定區域（例如台北市中山區、台中市西屯區）進行醫材集中度熱點分析，以防範天災或局部突發事件導致的區域性醫療斷鏈？

### 法規合規與人機協作 (HITL) 層面

16. **UDI 全球法規對齊**：臺灣衛生福利部食品藥物管理署 (TFDA) 對於醫療器材單一識別系統 (UDI) 有嚴格的申報時效規定。系統在判定「建立日期」減去「收貨日期」的時差時，能否自動關聯當前的法規條款，並在 Claude App 的報告中直接指出不合規的申報法人？
17. **人工介入與異常標籤覆寫**：當 Claude App 在月度報告中指出某幾筆序號為「非法合規黑洞」時，系統應提供何種 UI 介面，讓人工審查員在核實「實體發票」後，能在系統中勾選「誤報/法規豁免」，並防止該異常再次干擾下個月的分析？
18. **自動化警報分發機制**：當系統完成分析後，除了生成 Markdown 報告，是否需要設計一個自動化下游模組，根據 Claude App 評定出的風險等級，自動將不同層級的代辦事項 (Action Items) 透過 Webhook 發送到對應部門的 Slack、Microsoft Teams 或 Email？

### 部署、維護與擴展層面

19. **每月自動化觸發架構**：本系統為「每月定期分析」，最佳的雲端/本地自動化調度架構是什麼？（例如使用 Linux Cron Job、Windows Task Scheduler，亦或是雲端託管的 AWS Lambda / GCP Cloud Run 配合 Cloud Scheduler？）
20. **系統效能瓶頸評估**：當全臺灣的醫材申報數據月交易量從範例的數十條暴增至數十萬條時，雖然 LLM 由於不讀取明細而不會遇到 Context Limit，但本地 Pandas 與 NetworkX 的內存與 CPU 消耗將成為新瓶頸。對此，系統架構是否保留了向 `PySpark` 或 `Dask` 進行橫向擴展的程式碼重構彈性？
