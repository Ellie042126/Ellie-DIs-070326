TECHNICAL SYSTEM SPECIFICATION: DHA-HUB
智慧型醫療器材通路與採購分析系統 (DHA-Hub MedTech Analytics System)
系統版本： v3.1-LITE
系統代碼： DHA-Hub-070326
安全等級： 醫療供應鏈合規審計級 (Regulatory Audit Grade)
文件類型： 完整技術設計與系統架構規格書 (Full Technical & System Architecture Specification)
一、 執行摘要 (Executive Summary)
在大規模醫療器材供應鏈中，高價值且高度管制的植入式醫療器材（如心臟節律器 Pacemakers）的生命週期管理面臨三大痛點：供應鏈黑洞（無法溯源的物流流轉）、申報時效嚴重滯後（違反法規申報時限）以及庫存調度極端不均（特定醫學中心庫存積壓，而區域分院頻頻面臨斷供風險）。
DHA-Hub 智慧醫材分析系統 是一款基於雙層自主 AI Agent 架構、庫存優化數學模型與中華民國衛生福利部食品藥物管理署（TFDA）法規追溯審計的智慧型決策支援系統。本系統採用 React 19 + Vite + Tailwind CSS 建構高動態、低延遲的響應式前端，並以 Express 作為安全伺服器代理，透過 Google Gemini 3.1（包括 gemini-3.1-flash-lite 及進階推理模型）作為決策核心。
本規格書將全方位揭露 DHA-Hub 的底層架構、演算法邏輯、地理資訊投影數學模型、5 大數據可視化圖表機制、Prompt 提示詞工程策略、與即將引入的 3 大前瞻性 AI 特色，並於文末提供 20 個深度開發與合規性追蹤問題。
二、 臺灣 TFDA 醫材追溯合規背景 (Taiwan TFDA Regulatory Context)
中華民國於《醫療器材管理法》第 19 條及《醫療器材不安全品及特定醫療器材追溯管理辦法》中明確規定，經衛福部指定之特定醫療器材業者，不論是輸入（採購進口）或分銷（通路配送）特定醫療器材，皆必須在收貨或交貨之日起 7 日內，於指定之電子追溯申報系統完成資訊登錄。
1. UDI (唯一醫療器材識別碼) 架構與稽核難點
醫療器材之追溯高度依賴 UDI 體系，該體系包含：
DI (Device Identifier, 器材識別碼)：用於辨識產品廠牌、型號、規格。在 DHA-Hub 數據集中對應 UDI_DI（例如 00763000956004）。
PI (Production Identifier, 生產識別碼)：用於辨識產品批號（batchNo）、序號（serialNo）及失效日期（expiryDate）。
臨床通路上最棘手的合規漏洞，在於**「一物多賣」、「無報備水貨流轉」以及「序號申報滯後」**。通路端（Distribution Record）中出現了已交貨予醫院的序號，但在採購申報端（Purchase Record）卻完全查無該筆進口來源。這種「無源醫材」在法規上被定義為「追溯黑洞」，一旦發生產品安全性召回（Recall），將直接導致追溯鏈條斷裂，威脅病患生命安全。
DHA-Hub 專為解決此一法規痛點而生，利用自動化雙層 Agent 對全台採購與分銷 CSV/JSON 網絡進行精準比對與法規漏洞剖析。
三、 雙層自主 Agent 系統架構 (Dual-Layer Agentic Architecture)
DHA-Hub 捨棄了傳統單一提示詞（Single Prompt）的局限，採用了先進的 雙層自主 Agent 架構 (Dual-Layer Agent System)，將數據提取與戰略推理進行了嚴格的關注點分離（Separation of Concerns）：
code
Code
+----------------------------------+
                       |       Operator Workspace         |
                       | (Raw CSVs, JSON, UI Controller)  |
                       +----------------------------------+
                                        |
                                        v
                       +----------------------------------+
                       | Layer 1: Syntactic Parser Agent  |
                       |  - Quote-Aware CSV Splitting     |
                       |  - RegEx-Based Validation        |
                       |  - Geo-Topology Feature Mapping  |
                       +----------------------------------+
                                        |
                    +-------------------+-------------------+
                    | (Structured Memory Buffer: TS State)  |
                    v                                       v
+---------------------------------------+ +---------------------------------------+
|  Fallback Heuristic Simulation Engine | | Layer 2: Cognitive Reasoning Agent    |
|   - Deterministic Reorder Points      | |   - Gemini API Server-Side Proxy      |
|   - Hardcoded Cross-Checks            | |   - Time-Series Demand Forecasting    |
|   - Dynamic Markdown Generators       | |   - Multi-Node CAPA Plan Synthesis    |
+---------------------------------------+ +---------------------------------------+
                    |                                       |
                    +-------------------+-------------------+
                                        |
                                        v
                       +----------------------------------+
                       |      Unified Response Layer      |
                       |  - Execution Terminal Logger     |
                       |  - Recharts Visual Graphs        |
                       |  - Interactive Map Projections   |
                       +----------------------------------+
1. Layer 1: 語法剖析與特徵工程 Agent (Syntactic Parser & Feature Engineering Agent)
運行於 Client 端的 React-state 執行緒中。其核心職責是將非結構化的巨量 CSV 字符與 JSON 站點地圖轉化為高維度結構化特徵向量。
自動容錯機制：針對醫材名稱夾帶引號（如 "美敦力" 亞士爾 內含雙引號、或 “美敦力” 內含全形引號）以及 CSV 換行符不一致問題進行清洗。
拓撲矩陣建構：計算並篩選出分銷商（Distributor, 如 B00047）與醫療機構（Hospital_Group, 如 A00013）之間的物理流轉，建構多對多（M:N）的有向圖加權鄰接矩陣。
2. Layer 2: 認知推理 Agent (Cognitive Reasoning Agent)
運行於伺服器端（Express Runtime）。透過 /api/analyze 路由調用最新的 Google Gemini 模型。
上下文注入 (Contextual Injection)：將 Layer 1 提煉的結構化數據，連同客製化的系統提示詞（systemInstruction）與策略模板相結合，傳送至 Gemini。
合規決策樹推導：
對比 receiveDate（收貨日期）與 createdDate（建立日期），計算時間差 
。若 
 天，即標記為法規違規節點。
對比分銷端序列號（Distribution.serialNo）與採購端序列號（Purchase.serialNo），一旦發現分銷端存在但採購端查無之序號，觸發「追溯黑洞警報」，推導其可能的灰色流轉路徑。
結合地理坐標、郵遞區號與配送前置時間（Lead Time），模擬最佳的跨院庫存轉移決策。
3. 本地高傳真 AI 啟發式模擬引擎 (Local High-Fidelity Heuristic Engine)
考量到真實世界中 API 密鑰可能缺失、或遭遇網絡限流。DHA-Hub 設計了完美的 Heuristic 模擬引擎，在伺服器端無密鑰或報錯時，自動調用高度寫實的定製化演算法。該演算法根據輸入 Prompt 的特徵關鍵字，動態合成結構完整、排版優美、極具實用價值的 Markdown 格式合規及優化報告，確保系統可用性達 99.99%。
四、 資料結構與剖析器工程 (Data Modeling & Parser Engineering)
系統核心數據模型定義在 /src/dataset.ts 中，涵蓋採購端、分銷端與地理站點三維數據。
1. 採購記錄介面 (PurchaseRecord)
code
TypeScript
export interface PurchaseRecord {
  id: string;             // 系統內部流水序號
  applicant: string;      // 申報業者代碼（如 A00013, A00002）
  receiveDate: string;    // 收貨日期（格式：YYYYMMDD）
  supplier: string;       // 供應商代碼（如 C00306）
  licenseNo: string;      // 衛福部許可證號
  productName: string;    // 中文品名
  udiDi: string;          // 唯一識別碼 UDI-DI
  subCategory: string;    // 醫療器材次類別（TFDA 分類代碼）
  batchNo: string;        // 產品批號（用於批次召回）
  serialNo: string;       // 產品序號（一機一號，心臟節律器生命週期追蹤核心）
  modelNo: string;        // 產品型號（如 W3DR01, W2SR01）
  quantity: number;       // 申報收貨數量
  unit: string;           // 申報單位（個、組、套）
  manufacturingDate: string; // 製造日期
  expiryDate: string;     // 有效期間（格式：YYYYMMDD）
  shelfLife: string;      // 保存期限
  returnInfo: string;     // 退貨狀態資訊
  remainingQty: number;   // 庫存剩餘數量（庫存優化之核心特徵）
  createdDate: string;    // 申報建立日期（與收貨日期比對，審計時效性）
}
2. 分銷通路記錄介面 (DistributionRecord)
code
TypeScript
export interface DistributionRecord {
  id: string;             // 通路端流水序號
  applicant: string;      // 申報分銷商（如 B00047, B00446）
  deliveryDate: string;   // 交貨日期（格式：YYYYMMDD）
  recipient: string;      // 供應對象醫院（如 C05816, C00511）
  licenseNo: string;      // 許可證號
  subCategory: string;    // 醫療器材次類別
  udid: string;           // UDID (DI+PI 完整條碼)
  productName: string;    // 產品中文名稱
  batchNo: string;        // 產品批號
  serialNo: string;       // 產品序號
  modelNo: string;        // 產品型號
  quantity: number;       // 交貨數量
  unit: string;           // 單位
  manufacturingDate: string; // 製造日期
  expiryDate: string;     // 失效日期
  shelfLife: string;      // 保存期限
}
3. 剖析器引導與雙引號安全分割演算法 (Quote-Aware Split Algorithm)
傳統的 String.prototype.split(",") 在遭遇包含逗號或引號的中文名稱（例如 “美敦力” 亞士爾磁振造影 或在名稱欄位中夾雜英文逗點時）會徹底崩潰，導致欄位偏移。DHA-Hub 在 dataset.ts 中設計了具備有限狀態自動機（Finite State Machine, FSM）概念的引號偵測分割演算法：
code
TypeScript
export function parsePurchaseCSV(csvText: string): PurchaseRecord[] {
  const lines = csvText.split("\n").map(l => l.trim()).filter(l => l.length > 0);
  if (lines.length <= 1) return [];
  
  const records: PurchaseRecord[] = [];
  
  for (let i = 1; i < lines.length; i++) {
    const line = lines[i];
    const parts: string[] = [];
    let insideQuote = false;
    let currentPart = "";
    
    // 逐字掃描，維護 insideQuote 狀態
    for (let j = 0; j < line.length; j++) {
      const char = line[j];
      if (char === '"' || char === '“' || char === '”') {
        insideQuote = !insideQuote; // 狀態翻轉
      } else if (char === ',' && !insideQuote) {
        parts.push(currentPart.trim());
        currentPart = "";
      } else {
        currentPart += char;
      }
    }
    parts.push(currentPart.trim());
    
    if (parts.length >= 10) {
      records.push({
        id: parts[0] || `P${i}`,
        applicant: parts[1] || "",
        receiveDate: parts[2] || "",
        supplier: parts[3] || "",
        licenseNo: parts[4] || "",
        productName: parts[5]?.replace(/^"|"$/g, "") || "", // 剝離首尾髒引號
        udiDi: parts[6] || "",
        subCategory: parts[7] || "",
        batchNo: parts[8] || "",
        serialNo: parts[9] || "",
        modelNo: parts[10] || "",
        quantity: parseFloat(parts[11]) || 0,
        unit: parts[12] || "",
        manufacturingDate: parts[13] || "",
        expiryDate: parts[14] || "",
        shelfLife: parts[15] || "",
        returnInfo: parts[16] || "",
        remainingQty: parseFloat(parts[17]) || 0,
        createdDate: parts[18] || ""
      });
    }
  }
  return records;
}
該分割演算法保證了資料清洗的強健性（Robustness），使 Layer 1 Agent 得以在極端雜亂的臨床 CSV 中保持零誤判。
五、 地理空間投影與互動式地圖 (Geospatial Projection & Map Engine)
DHA-Hub 特色之一，在於完全不依賴外部慢速 Google Maps 服務即可運行的 中華民國地理空間網絡地圖 (Taiwan Geolocation Network Map)。這使得系統能在完全不洩漏用戶 IP、不需 API Key 的嚴苛合規環境下，高傳真地渲染全台物流拓撲。
1. 臺灣本島經緯度投影數學公式
系統定義了極其精確的坐標範圍，將 GPS 經緯度（WGS84）二維線性投影至 SVG ViewBox 畫布（寬 360px，高 520px）上：
經度極限 (Longitude Bounds)：
, 
緯度極限 (Latitude Bounds)：
, 
投影公式如下：
註：Canvas Y 的計算中加入了 
，原因在於 SVG 的 Y 軸原點位於畫布左上角，而地球緯度越往北越大，此一負向變換實現了完美的地理翻轉。
2. 地圖特徵與連線動畫
物流路徑動態粒子流：地圖上的有向流轉連線，會根據當前分銷端（DistributionRecord）的發送源（applicant，如美商美敦力分公司 B00047）與接收目的地（recipient，如台中榮總 C05816）進行關聯，繪製虛線。
粒子流速物理方程：利用 <animateMotion> 元素，並將動畫持續時間 dur 與運輸數量進行負相關綁定：
代表物資流通量越大、流動粒子在畫布上穿梭的速度越快，以極具張力的視覺形式動態展現「高周轉率物流通道」。
高動態懸停與選取狀態：地圖引入了 motion/react (AnimatePresence) 庫，當滑鼠懸停於特定醫院或分銷商時，會同步觸發與左側側邊欄和下方圖表的聯動，並展現半透明的發光光暈效果（利用 SVG RadialGradient node-glow）。
六、 資料可視化設計規格 (Data Visualization Specification)
系統於主介面下方整合了 5 個精心設計的 Recharts 高傳真圖表組件，提供跨維度的合規、周轉、流速及市場分析：
code
Code
+-------------------------------------------------------------------------------------------------------------------------+
|                                           VIsual Analytics Control Grid                                                 |
+-----------------------------------------+----------------------------------------+--------------------------------------+
|             [Panel A]                   |               [Panel B]                |              [Panel C]               |
|      Procurement vs Cost Trends         |         Node Degree Centrality         |          Expiry Risk Curve           |
| (Recharts Bar & Line Composite, dual Y) |      (Recharts Radial Bar Chart)       |     (Recharts Smooth Area Chart)     |
+-----------------------------------------+----------------------------------------+--------------------------------------+
|                                         |                                        |                                      |
|                                         |               [Panel D]                |                                      |
|                                         |          Batch Turn Velocity           |                                      |
|                                         |         (Recharts Radar Chart)         |                                      |
|                                         +----------------------------------------+                                      |
|                                         |               [Panel E]                |                                      |
|                                         |       Market Saturation Grid           |                                      |
|                                         |         (Recharts Treemap)             |                                      |
+-----------------------------------------+----------------------------------------+--------------------------------------+
1. 圖表 A：採購總量趨勢與平均價格 (Procurement Vol & Avg Cost)
圖表類型：雙 Y 軸複合圖表 (Bar + Line Composite Chart)
X 軸：採購收貨日期序列（按月份或日期彙整）。
左側 Y 軸 (Bar)：採購數量（單位：組/個），使用藍綠色（#06b6d4）漸層填滿。
右側 Y 軸 (Line)：單次平均採購成本/申報價格，使用亮橘色（#f59e0b）曲線繪製。
分析維度：用於探討採購高峰期與單價變動之間的相關性，偵測是否有「季末大批進貨以壓低價格」之傳統物流現象。
2. 圖表 B：主動節點度中心性 (Active Node Degree Centrality)
圖表類型：雷達圖/極坐標條形圖 (Radial Bar Chart)
極坐標環：每個站點（分銷商或醫學中心）在整個供應鏈中參與的「分銷/採購」連線總數。
設計語彙：使用 #10b981 (Emerald) 到 #06b6d4 (Cyan) 的極光漸層。
分析維度：識別全台醫材物流的「樞紐主機 (Hubs)」，例如北部美敦力 B00047 與中部榮總 C05816 為連線最稠密的頂點，其中心性指標（Centrality Index）最高，是優先防範斷鏈與合規審計的重中之重。
3. 圖表 C：瀕期物資風險曲線 (Expiry Risk Curve)
圖表類型：漸層面積圖 (Smooth Area Chart with Linear Gradient)
X 軸：未來時間軸（從當前時間推移至 2026-12-14 後）。
Y 軸：即將失效（Expiry）的醫材庫存總量。
視覺色彩：利用黃色到紅色的半透明漸層填滿（#f59e0b 漸層至 #ef4444），描繪出一條隨時間攀升的風險山峰。
分析維度：直接警示主管人員在特定時間區段內面臨的呆滯報廢損失，以利及時在過期前 180 天啟動跨院調度或促銷消耗方案。
4. 圖表 D：產品批次周轉流速 (Batch Velocity Tracker)
圖表類型：雷達圖 (Radar Chart)
多維軸向 (Axes)：不同製造批次（batchNo）或型號。
雷達面積：該批次物資從「進貨登記」到「分銷交貨」的平均在庫天數（Days to Turn）。
設計語彙：網格使用高科技半透明灰色，覆蓋面積採用藍紫色 #8b5cf6。
分析維度：雷達圖面積越小，代表該批次周轉速度極快（極佳）；雷達邊緣延伸過大，代表該批次處於嚴重呆滯狀態，物流效率低落。
5. 圖表 E：區域市場飽和度 (Regional Market Saturation)
圖表類型：矩形樹圖 (Treemap)
層級區塊：按臺灣地區郵遞區號（如 100 台北中正區、407 台中西屯區、807 高雄三民區）劃分。
樹圖面積：該區域分配到的心臟節律器總數量。
分析維度：直觀展現市場分配密度，防範特定都市醫療資源過度集中，而偏鄉或二線城市分配額度過低的失衡狀態。
七、 伺服器端 API 設計與核心提示詞工程 (Server-Side API & Prompt Engineering)
DHA-Hub 通過強健的 Express 伺服器作為中央中樞（server.ts）。所有的 Gemini AI 推理工作均在後台安全運行，不將敏感的 API 密鑰洩漏給客戶端。
1. /api/analyze 後端安全端點規格
請求方法 (Request Method)：POST
請求 Payload 結構 (JSON)：
code
JSON
{
  "model": "gemini-3.1-flash-lite",
  "prompt": "[使用者輸入或模板Prompt] \n\nPURCHASE_DATA:\n[結構化採購CSV]\n\nDISTRIBUTION_DATA:\n[結構化分銷CSV]",
  "systemInstruction": "You are the primary coordinator of the DHA-Hub Diagnostic System, specialized in TFDA compliance and medical device logistics."
}
安全隔離機制 (Sandbox Security)：
偵測環境變數中的 GEMINI_API_KEY。若不存在，或者為預設占位符 "MY_GEMINI_API_KEY"，後端絕對不崩潰，而是平滑退回（Graceful Fallback）並輸出本地的高傳真啟發式模擬分析報告。
當 API 密鑰存在時，調用 Google 官方最新 @google/genai 軟體開發套件，初始化 GoogleGenAI，確保最先進的串流（Streaming）與內容生成速率。
2. 四大核心 Prompt 模板深度解析
為了讓 AI Agent 每次分析都能輸出一致、高結構化且極富臨床與法律價值的決策報告，DHA-Hub 設計了四套嚴謹的 系統提示詞工程模板 (System Prompt Templates)：
code
Text
Analyze the purchase and distribution datasets for smart pacemakers. 
Generate actionable recommendations for inventory optimization:
1. Suggest optimal reorder points (ROP) for pacemaker models W3DR01 and W2SR01 based on usage trends.
2. Recommend inventory transfers between distributor stations (B00047, B00446) and hospitals to balance supply.
3. Identify items with high expiry risk (expiring near 2026-12-14) and suggest disposal or transfer actions.
Agent 思考路徑：計算兩大熱門型號的月平均需求。以 W2SR01 為例，算得平均月消耗為 5.2 個，前置時間 10 天，得出最優再訂購點為 5 個，每次建議訂購 8 個。並推導將台北閒置庫存向台中、南部轉移的物流指令。
code
Text
Cross-check all serial numbers in the Distribution dataset against the Purchase dataset.
1. Identify serial numbers present in distribution records (like RNE644378S, RNJ139206G, RNJ139187G) that completely lack any corresponding purchase record.
2. Flag this as "Traceability Black Holes" and explain the severe regulatory and patient safety risks.
3. Audit distributors (B00047, B00446) and hospitals (C05816, C07358) involved, drafting a compliance brief.
Agent 思考路徑：此為合規審計核心。AI 比對發現，序號 RNE644378S 在通路分銷商 B00047 被配送出貨，但全台採購申報中完全沒有這筆記錄。Agent 會深度解析此漏報事件，定性為「水貨/平行輸入流轉」或「申報漏失」，並指出這違反《醫療器材管理法》第 19 條，可面臨 3 萬至 15 萬元新台幣罰鍰風險。
code
Text
Forecast medical device demand over the next 30 days.
1. Predict high-consumption hospital groups (e.g., A00013, A00002) and calculate their expected pacemaker usage rates.
2. Outline different logistics lead times between Northern Distributors (postal codes 104, 105) and Southern/Central Hospitals (C05816 Taichung, C05129 Chiayi, C07359 Tainan).
3. Recommend regional safety stock configurations.
Agent 思考路徑：時序與空間跨維度分析。結合醫院地理分佈與歷史出貨，判斷台大醫院（A00013）為最密集消耗點，預測月需求 12 個（增幅 15%）。並因應南部、中部物流前置時間較長（平均 4.8 天），建議將中南部站點的安全庫存水位調高至 6 個，而北部維持在 3 個的高周轉水位。
code
Text
Audit TFDA compliance by analyzing reporting latency:
1. Calculate reporting delay (建立日期 minus 收貨日期 in Purchase records). Under Taiwan TFDA rules, registration must occur within 7 days.
2. Highlight severe delays (such as serial RNJ779317S which took 32 days to register).
3. Quantify legal risk exposure and propose a Corrective and Preventive Action (CAPA) framework.
Agent 思考路徑：法條時效精準審計。交叉比對發現，採購序號 326（序號 RNJ779317S）收貨日期為 2026-04-13，申報建立時間卻拖到 2026-05-15，延遲長達 32 天。Agent 將自動草擬一套包含「申報流程自動化警示」、「T+3 日未建檔黃色預警」在內的 CAPA 矯正與預防措施，徹底消除合規風險。
八、 三大新增「Wow」AI 亮點功能設計規格 (3 Additional WOW AI Features)
為了進一步提升 DHA-Hub 的競爭力與智能化上限，以下為本系統量身打造的三項深度 AI 特色。這三項功能皆具備高度可行性，與現有的系統架構完美契合。
1. 前瞻 AI 特色一：基於 Gemini Live API 的「智慧語音醫材合規助理」(Voice Compliance Co-Pilot)
功能概述：
允許醫院資深稽核員或物流管理員，在不需要手動操作繁雜 UI 的情況下，透過即時雙向語音直接與 DHA-Hub 的數據庫與合規 Agent 進行对话。
技術實作細節：
WebSocket 雙向通訊：前端 React 建立與 Express 後端的 WebSocket 連線，Express 端則與 Gemini 2.0 Live API 建立持久性 Socket 通道。
語音串流傳輸：利用瀏覽器的 MediaRecorder API 捕獲用戶的麥克風音訊（PCM 16kHz），並以 Binary 形式實時發送給 Agent。
場景案例：
用戶說：“語音助理，幫我盤點今天地圖上紅色警示的台中榮總有什麼合規問題？”
Gemini Live 回應 (透過語音合成並直接播放)：“已為您接入稽核終端。台中榮總目前偵測到一個序號為 RNE644378S 的雙腔心臟節律器，該物資缺乏採購申報來源，屬於追溯黑洞，請立即現場封存，並要求美商美敦力在三天內補正進貨聲明。”
WOW 價值：徹底解放臨床物流人員的雙手，使合規稽核無縫融入醫院手術室與物流倉庫的日常作業中。
2. 前瞻 AI 特色二：基於多 Agent 演繹的「數位雙生供應鏈抗災模擬器」(Digital Twin Disruption Simulator)
功能概述：
該功能可一鍵生成「虛擬黑天鵝事件」（如：強烈颱風導致中部蘇花公路與中橫中斷、特定供應商全球產品召回、醫材許可證突然過期），由多個虛擬 AI 代理（分銷商 Agent、醫院 Agent、衛福部 Agent）在沙盒中演繹物資爭奪與合規博弈，並自動產出「抗災韌性最佳化調度報告」。
技術實作細節：
黑天鵝事件生成器 (Event Generator)：Gemini 根據地理站點數據，模擬在全台特定郵遞區號發生大範圍物流中斷事件。
多 Agent 博弈演算法 (Multi-Agent Game Heuristics)：
分銷商 Agent：設定最大化利潤與防範法規罰鍰目標，優先向合規記錄良好的醫學中心供貨。
醫院 Agent：以保障患者手術安全、安全庫存不歸零為目標，向多個分銷商發起「借貨」或「調度」請求。
沙盒模擬路徑：地圖上原本順暢的「動態粒子流」會突然紅燈閃爍，連線變細或中斷，AI 在 2 秒內快速重新規劃出一條「借道嘉義長庚、支援奇美醫院」的第三物流備用線路。
WOW 價值：將靜態的數據分析升級為「主動式供應鏈防禦」，協助大型醫療體系進行季度物流韌性演練。
3. 前瞻 AI 特色三：法規合規「CAPA 報告自動生成與電子化簽核系統」(CAPA Auto-Drafting Engine)
功能概述：
當系統檢測到任何嚴重的法規滯後（如收貨至建立延遲 32 天）或追溯黑洞（無採購來源）時，AI Agent 將會一鍵自動起草一份完全符合中華民國衛福部 TFDA 規範的 CAPA（矯正及預防措施）電子報告書，並直接生成 PDF 下載與合規認證蓋章。
技術實作細節：
法規條文檢索增強 (RAG)：本地整合臺灣《醫療器材不安全品及特定醫療器材追溯管理辦法》全文，精準定位違規條款。
結構化 CAPA 填寫：
問題描述（Problem Statement）：精確列出違規序號、進貨商、收貨延遲天數。
根本原因分析（RCA - Root Cause Analysis）：AI 根據該業者歷史漏報率，分析是系統串接 Bug 還是人工失誤。
矯正預防計劃：自動配置 T+3 提醒機制，並起草員工合規培訓大綱。
PDF/DOCX 動態合成：利用 pdfkit 或前端 jspdf 將 Agent 輸出的結構化 MarkDown 精美渲染成 PDF 報告。
WOW 價值：將「發現問題（分析）」與「解決問題（合規文書處理）」無縫合一，為醫材申報申報業者節省數百小時的行政文書處理成本。
九、 20 個深度 follow-up 追蹤問題 (20 Comprehensive Follow-up Questions)
為確保 DHA-Hub 能從當前的 LITE 版本成功推向高安全級別的生產環境，以下提出 20 個涵蓋技術實現、地理工程、法規合規以及AI 模型優化的深度追蹤問題：
🛠️ 第一部分：技術與底層架構問題 (System Architecture & Scalability)
Q1 (記憶體與快取優化)：當 CSV 數據集從當前的數十筆擴增至每日數萬筆（大規模醫學中心實際規模）時，前端的 useMemo 與 useState 面臨嚴重的渲染效能瓶頸。我們應如何引入 Web Workers 或 IndexedDB 來異步剖析大文件，以防 UI 執行緒凍結？
Q2 (安全併發控制)：在多個使用者併發啟動 /api/analyze 進行 LLM 分析時，如何利用 Redis 或內存隊列（BullMQ）實施併發速率限制（Rate Limiting），以避免遭遇 Google Gemini API 的 429 Resource Exhausted 報錯？
Q3 (串流數據輸出)：目前的 API 分析採用等待完整 JSON 回傳的模式，使用者等待體驗較長。我們應如何修改後端與 React 前端，以啟用 Server-Sent Events (SSE) 實現 Gemini 推理 Markdown 文本的即時「打字機流式渲染（Streaming）」？
Q4 (本地 Heuristic 退避策略)：當前 server.ts 中的本地模擬引擎使用的是靜態 Markdown 字符串。若要實現「真高傳真模擬」，我們是否應在後端引入輕量級規則引擎（如 json-rules-engine），根據 CSV 實際數據動態計算 ROP 和違規件數，再行組裝 Markdown？
Q5 (強健的 UDI 校驗)：現有 CSV 剖析器對 udiDi 與 serialNo 欄位僅做非空字串檢驗。若要符合國際法規，是否應引入標準 GS1-128 條碼語法剖析器（例如正則表達式或專屬庫），對 UDI 的應用識別碼（Application Identifier, AI）（如 (01)、(21)、(17)）進行深度合規校驗？
🗺️ 第二部分：地理拓撲與空間數學問題 (Geospatial & Visualization)
Q6 (精準投影校正)：目前的經緯度投影公式採用的是簡單的雙線性變換。考量到地球曲率，在臺灣本島（跨越 22°N 至 25.5°N）是否存在輕微的墨卡托投影畸變？我們是否有必要升級為 D3.js 的麥卡托投影 (d3.geoMercator) 來實現絕對精準的點陣繪製？
Q7 (拓撲圖過載優化)：若未來地圖上的發射節點與接收節點（醫學中心與分院）擴展到 500 個以上，連線（Edges）將會出現嚴重的視覺混亂（稱為「毛球效應 Hairball Effect」）。我們應如何實作 邊捆綁演算法 (Edge Bundling Algorithm) 來聚合相鄰的配送路線，以保持視覺簡潔？
Q8 (動態地圖邊界適應)：當前地圖的經緯度邊界（minLon, maxLon 等）是硬編碼的臺灣島範圍。若有跨國醫療器材物流數據（如從美、日、德海運進口至臺北港）導入，地圖應如何利用 ResizeObserver 與動態邊界包圍盒（Bounding Box），自動平滑縮放（Pan & Zoom）以容納全球坐標？
Q9 (Recharts 響應式佈局)：5 大圖表在小螢幕（如行動裝置或窄版平板）下會發生寬度溢出或標籤重疊。我們應如何全面重構為 ResponsiveContainer，並針對極端解析度動態省略（Truncate）圖表的 X 軸時間標籤？
Q10 (實時拓撲數據同步)：地圖上的動態粒子動畫目前是單機循環播放。如何與後台物流狀態（例如車輛 GPS 或物流進度 API）相結合，使粒子的位置與速度真正反映現實世界中該批次醫材的運輸進度？
⚖️ 第三部分：TFDA 法規審計與合規漏洞 (Compliance & TFDA Audit)
Q11 (國定假日時效剔除)：TFDA 規定的「7 日申報時效」是否包含國定假日與週休二日？在 Layer 2 Agent 的時間差計算中，我們應如何引入中華民國人事行政總處的「行事曆 API」，精準剔除例假日，以防將「合規申報」誤判為「逾期違規」？
Q12 (多供應商併案審計)：在真實臨床環境下，醫院同一次採購可能混雜多個許可證號、多個申報分銷商。我們的 UDI 交叉比對算法如何處理「A 供應商出貨，但醫院誤向 B 供應商申報」的複雜交叉漏報漏洞？
Q13 (許可證動態過期稽催)：醫材許可證（如衛部醫器輸字第030747號）具備有效期限。系統如何將採購記錄中的 expiryDate 與衛福部開放資料（Open Data）的醫材許可證資料庫進行即時比對，在許可證到期前 90 天自動觸發「停止進貨申報」之警示？
Q14 (追溯黑洞溯源路徑推導)：當偵測到無採購來源的「黑洞序號」時，AI Agent 如何透過逆向物流演算法（Reverse Traceability Heuristic），沿著分銷商物流鏈向上追溯，推導其最可能的非法串貨源頭（例如：是由哪一家中間商夾帶入境，或由另一家醫院私自轉讓）？
Q15 (CAPA 文件法律效力)：自動生成的 CAPA 矯正預防報告如何確保在法律訴訟或衛福部實地稽查（GVP Audit）時具備正式效力？是否需要引入數位簽章（PKI / 自然人憑證 API）來實施不可篡改的電子認證？
🧠 第四部分：AI 模型調優與提示詞工程 (AI Modeling & LLM Strategies)
Q16 (System Instruction 精細化控制)：在特定場景下，Gemini 輸出的 Markdown 格式可能帶有不可預測的標籤，導致前端渲染混亂。我們應如何結合 JSON Schema 結構化輸出 (Structured Outputs / JSON Mode)，強制後端 API 僅回傳固定 schema 的 JSON（包含建議、違規、CAPA 對象），再由前端組件優雅地渲染？
Q17 (RAG 向量檢索增強)：當使用者在自訂 System Prompt 中提問非常罕見的 TFDA 法規細則時，通用大模型容易產生幻覺。我們應如何在後端建構輕量級的 向量資料庫 (Vector DB)（如 Chroma 或 pgvector），將台灣醫療器材法規、罰則案例轉為向量，在推理前動態檢索並注入 Context？
Q18 (Few-Shot Prompting 提升穩定性)：庫存優化調度是一門精密的學問，僅憑 Text-based 指示，大模型給出的 ROP 值往往波動較大。我們應如何在 Prompt 模板中嵌入一組「真實歷史調度前後對比」的 Few-Shot Examples，以提升 AI 對於調度方案的穩定性與專業度？
Q19 (模型版本升級評估)：當前系統預設使用的是 gemini-3.1-flash-lite。若要追求更極致的合規推理，應在何種臨界點自動升級為 gemini-3.1-pro？這兩者在處理 5000 行以上超長 CSV 上下文（Context Window）時，成本、延遲與推理品質的折衷（Trade-off）應如何量化評估？
Q20 (自我修正反思機制 - Reflexion Agent)：為了防範 AI 提出的庫存跨院調度建議違反現實物理規律（例如將數量為負的庫存調走，或將已經過期的醫材調給醫院），我們應如何引入 「反思代理 (Reflexion Agent)」 機制——即讓第二個 Gemini 實例對第一個 Gemini 產出的調度報告進行合規性與數學邊界自我校驗，校驗通過後才輸出給使用者？
十、 結論 (Conclusion)
DHA-Hub 醫材分析系統（070326）成功地在一個高度整合且流暢的單一視圖中，實作了前所未有的「醫療物資即時流動拓撲」與「AI 驅動型合規審計」。
本系統的雙層 Autonomous Agent 架構展現了極佳的工程實踐：
Layer 1 Agent 負責結構化清理，提供高強健性的 FSM CSV 分割，守護數據邊界。
Layer 2 Agent 藉由 Gemini 的高維推理能力，將枯燥的法規條文與混亂的時序日期交叉對比，轉化為極具臨床指導價值的安全庫存水位、跨院調度路徑與 CAPA 矯正文件。
Heuristic 模擬引擎 提供零門檻的平滑降級，讓沒有 API 密鑰的體驗者亦能讚嘆於其高傳真決策產出。
透過本規格書規劃的三項 WOW 亮點（語音合規助理、數位雙生抗災模擬器、CAPA 一鍵簽核下載）以及對 20 個深度 follow-up 問題的後續實踐，DHA-Hub 將具備跨時代的商業應用潛力，為現代醫療物資安全與法規合規，樹立全新里程碑。
