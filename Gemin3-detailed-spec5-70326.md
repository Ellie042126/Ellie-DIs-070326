智慧型醫療器材通路與採購分析系統 (DHA-Hub)
系統架構、技術規格與 AI 擴充方案說明書 (Technical Specification Document)
本文件針對 智慧型醫療器材通路與採購分析系統 (DHA-Hub) 提供全面性、高保真度的系統技術規格與架構設計說明。本系統係為解決高價值、高風險醫療器材（如植入式心臟節律器）在進口通路商分銷、區域物流配送、以及最終醫療院所採購入庫與耗用申報三者之間，因資訊不對稱、申報時差、計量單位不一致及條碼誤植所導致的追溯性黑洞 (Traceability Black Holes) 而設計。
本規格書深度剖析現有 Full-Stack (React 19 + Vite 6 + Express 4) 程式碼結構與數據關聯，並提出 3 項極具震撼力之 WOW AI 創新功能，最後以 20 個深度架構與法規遵循問題作結。
1. 系統願景與核心價值定位 (System Vision & Value Proposition)
在現代醫療供應鏈中，高價值第三類醫療器材（Class III Medical Devices）如「植入式心臟節律器（Pacemakers）」與「磁振造影植入式心臟節律器-單腔」之追溯性直接關係到病患的生命安全。食品藥物管理署 (TFDA) 與國際組織均強制推行 唯一醫療器材識別系統 (Unique Device Identification, UDI)。然而，在實際產業運行中，仍面臨以下三大痛點：
申報時差與漏報（Temporal Discrepancy）：通路商交貨日期與醫療院所收貨入庫申報常有 24 至 72 小時之落差，致使即時合規稽核困難。
計量單位不一致（Unit Mismatch）：分銷商端以「個（Piece）」出貨，醫院端以「組（Set）」或「包（Package）」進貨，造成帳務與實體庫存虛高。
序號重疊與錯配（Serial Mismatch & Overlaps）：人工輸入 UDI 條碼或申報資料時，將產品序號（Production Identifier, PI）誤植或重複，導致追溯性鏈條斷裂，形成「追溯性黑洞」。
DHA-Hub (智慧型醫療器材通路與採購分析系統) 透過內建之雙層型自主 Agent 決報系統，實時解析分銷商與醫療院所的非結構化與半結構化大數據，精確捕捉供應鏈節點中的異常對帳，自動預測即期品呆滯損耗，並產出符合 TFDA 規範之法規報告，從動態宏觀的「通路網路」到微觀的「序號層級」進行全鏈路合規審計。
2. 系統架構與部署模型 (System Architecture & Deployment Model)
DHA-Hub 採用極簡且高效的全棧單一容器架構 (Full-Stack Monolithic Architecture)。前端基於 React 19、Vite 6 及 Tailwind CSS 4.0 進行高度響應式與視覺化呈現；後端使用 Express 4 (TypeScript) 提供 RESTful APIs，並結合 Gemini AI API (SDK @google/genai v2.4.0) 進行高維度推理。
2.1 系統實體與邏輯架構圖 (System Architecture Diagram)
code
Code
+-----------------------------------------------------------------------------------------+
|                                    CLIENT BROWSER                                       |
|                                                                                         |
|  +-----------------------------------------------------------------------------------+  |
|  |                                  React 19 SPA                                     |  |
|  |                                                                                   |  |
|  |  +-------------------+  +-------------------+  +-------------------+              |  |
|  |  |    NetworkHub     |  |    WowGraphs      |  |    AuditPanel     |              |  |
|  |  | (D3 Node Topology)|  | (Recharts Graphs) |  | (Anomaly Auditor) |              |  |
|  |  +-------------------+  +-------------------+  +-------------------+              |  |
|  |  +-------------------+  +-------------------+  +-------------------+              |  |
|  |  | InventoryOptim.   |  |    DataLedger     |  |     AICopilot     |              |  |
|  |  |  (Dynamic Alloc)  |  | (Raw Ledger view) |  | (Natural Lang UI) |              |  |
|  |  +-------------------+  +-------------------+  +-------------------+              |  |
|  +----------------------------------------+------------------------------------------+  |
|                                           | (JSON over HTTPS)                           |
+-------------------------------------------|---------------------------------------------+
                                            v
+-----------------------------------------------------------------------------------------+
|                                CLOUD RUN REVERSE PROXY                                  |
|                               (Nginx Gateway @ Port 3000)                               |
+-----------------------------------------------------------------------------------------+
                                            |
                                            v
+-----------------------------------------------------------------------------------------+
|                              EXPRESS BACKEND SERVER (Node.js)                           |
|                                                                                         |
|   +---------------------------------------------------------------------------------+   |
|   | /api/dataset (Active CSV Parsing of dataset.md & Memory Cache)                  |   |
|   +---------------------------------------------------------------------------------+   |
|   | /api/optimize (Stateful Multi-Echelon Stock Allocation Logic)                    |   |
|   +---------------------------------------------------------------------------------+   |
|   | /api/chat (Interactive Auditor Assistant Context Grounding)                     |   |
|   +---------------------------------------------------------------------------------+   |
|   | /api/predict-disruptions (Vulnerability Evaluator using High-Order Reasoning)   |   |
|   +---------------------------------------------------------------------------------+   |
|   | /api/generate-tfda-report (Official Compliance Template Generation Engine)      |   |
|   +---------------------------------------------------------------------------------+   |
|                                           |                                             |
|                                           | (Google GenAI SDK v2.4.0)                   |
|                                           v                                             |
|                             +----------------------------+                              |
|                             |    Gemini Pro / Flash      |                              |
|                             |   (Server-Side API Keys)   |                              |
|                             +----------------------------+                              |
+-----------------------------------------------------------------------------------------+
2.2 部署策略與網路配置
Containerization: 本系統完全相容於 Google Cloud Run。所有靜態資源之編譯、Node.js 伺服器之載入均整合於一個容器中。
Port Ingress Constraints: 系統在生產環境與開發環境皆硬性綁定 PORT 3000 且監聽 0.0.0.0。Nginx 作為反向代理層（Reverse Proxy），統一轉發外部 HTTPS 流量。
Vite Middleware Integration: 在開發模式（Development Mode）下，Express 伺服器會動態啟動內嵌之 Vite Server，並以 middlewareMode: true 方式掛載於 Express 中，實現零延遲的前後端協同開發。在生產模式（Production Mode）下，系統直接載入 /dist 下預編譯之靜態資產，並提供完美的 Single-Page Application (SPA) 路由回退（Fallback）機制。
3. 資料架構與強型別定義 (Data Architecture & Strong Typing)
本系統的最高指導原則之一是資料的高保真度與型別安全。在 /src/types.ts 中，定義了多維度的資料結構，以嚴格映射採購、通路分銷以及異常追溯的事實欄位。
3.1 採購資料集強型別 (PurchaseItem)
對應醫院端的採購入庫申報資料，精確對齊 UDI-DI 以及產品序列號：
code
TypeScript
export interface PurchaseItem {
  id: number;
  申报業者: string;       // 申報醫院 ID，例如 "A00013" (台大醫院)
  收货日期: string;       // 格式：YYYYMMDD
  供应商: string;         // 供貨廠商 ID，例如 "C00306" (美敦力台灣)
  许可证号: string;       // TFDA 醫療器材許可證字號
  中文品名: string;       // 醫材官方中文名稱
  UDI_DI: string;         // 14 碼 UDI 識別碼 (例如: 00763000956004)
  医疗器材次类别: string;  // 例如 "E.3610植入式心律器之脈搏產生器"
  产品批号: string;       // 批次代碼 (Lot Number)
  产品序列: string;       // 唯一產品序列號 (Serial Number, PI)
  产品型号: string;       // 產品型號 (Model Number, e.g., "W3DR01")
  数量: number;           // 採購數量
  单位: string;           // 申報單位，例如 "個"、"組"、"套"
  制造日期: string;       // 格式：YYYYMMDD
  有效期间: string;       // 效期長度
  保存期限: string;       // 格式：YYYYMMDD (到期日)
  退货资讯: string;       // 退貨狀態與說明
  剩余数量: number;       // 庫存餘額
  建立日期: string;       // 資料建檔時間
}
3.2 通路配送資料集強型別 (DistributionItem)
對應進口商/分銷商出貨配送至醫院端的資料流：
code
TypeScript
export interface DistributionItem {
  id: number;
  申报業者: string;       // 分銷商 ID，例如 "B00047" (美敦力分公司)
  交货日期: string;       // 格式：YYYYMMDD
  供应对象: string;       // 目標醫院或二級分銷商，例如 "C05816" (台中榮總)
  许可证号: string;
  医疗器材次类别: string;
  UDID: string;           // 對應 UDI 條碼資訊
  中文品名: string;
  产品批号: string;
  产品序列: string;       // 唯一產品序列號 (Serial Number)
  产品型号: string;
  数量: number;
  单位: string;           // 配送申報單位，如 "組" 或 "個"
  制造日期: string;
  有效期间: string;
  保存期限: string;
}
3.3 地理空間站點強型別 (GeolocationStation)
用於在 NetworkHub 中進行地理拓撲渲染與多級物流路由分析：
code
TypeScript
export interface GeolocationStation {
  entity_id: string;      // 機構代碼 (e.g., "B00047", "A00013")
  official_name: string;  // 機構官方中文名稱
  entity_type: "Distributor" | "Hospital_Group"; // 分銷商或醫院體系
  postal_code: string;    // 郵遞區號 (對應台灣三碼郵區)
  street_address: string; // 實體通訊地址
  latitude: number;       // 緯度
  longitude: number;      // 經度
}
3.4 追溯性黑洞審計模型 (TraceabilityBlackHole)
描述進銷對帳過程中捕捉到的嚴重異常實體：
code
TypeScript
export interface TraceabilityBlackHole {
  id: string;
  serialNumber: string;   // 衝突或異常的產品序號
  productModel: string;   // 產品型號
  productName: string;    // 產品名稱
  issueType: "Serial Mismatch" | "Unmapped Transit" | "Discrepancy In Quantities";
  distributorId: string;  // 出貨分銷商 ID
  hospitalId: string;     // 收貨醫院 ID
  date: string;           // 發現日期
  quantity: number;       // 涉案數量
  status: "pending" | "resolved" | "false-positive"; // 審計核決狀態
  notes?: string;         // 技術性排查備註
  auditor?: string;       // 核核審計員名稱
}
4. 核心技術引擎與演算法設計 (Core Technical Engines)
DHA-Hub 的核心價值在於四大智慧型模組，以下詳細說明其演算法邏輯與資料流轉。
4.1 雙層型自主 AI Agent 決報系統 (Double-Layer Autonomous AI Agent Architecture)
系統採用業界先進的雙層式 Agent 設計。
code
Code
+-------------------------------+
                       |      Auditor Interface        |
                       +---------------+---------------+
                                       |
                                       v
                     +-----------------+-----------------+
                     |    Layer 1: Orchestrator Agent    |
                     +-----------------+-----------------+
                                       |
                 +---------------------+---------------------+
                 |                                           |
                 v                                           v
  +--------------+--------------+             +--------------+--------------+
  |  Layer 2: Diagnostic Agent  |             | Layer 2: Inventory Optimizer |
  | (Vulnerability & Anomaly)   |             |    (Transfer & Reordering)   |
  +--------------+--------------+             +--------------+--------------+
                 |                                           |
                 +---------------------+---------------------+
                                       |
                                       v
                       +---------------+---------------+
                       |   Unified Compliance Synthesis |
                       +-------------------------------+
Layer 1: Orchestrator Agent (調度編排主代理)
職責：負責解析審計人員（Auditor）輸入的自然語言指令。識別該查詢屬於「合規追溯對帳」、「安全庫存優化」、「鏈條脆弱性評估」還是「TFDA 法規報告編撰」範疇。
工作流：從前端收集即時的 purchases、distributions 與 blackholes 狀態，過濾並重構其上下文語義（Context Grounding），動態路由至特定的 Layer 2 代理，同時整合系統內置的 fallback 機制。
Layer 2: Specialized Diagnostic Agents (專業診斷子代理)
Agent 2A (合規異常診斷代理)：專精於解析序列號衝突。當偵測到「Serial Mismatch」時，它會主動追蹤該序列號在分銷商與多個醫院間的流轉鏈結，調用知識庫分析其是否存在重複張貼、退貨二次上架或輸入錯誤，並指出可能的涉案責任方。
Agent 2B (多級庫存調撥與優化代理)：專精於分析供需預測與耗用率。依據「安全庫存演算法」與「即期呆滯損耗比率」，給出跨機構調撥（Transfer）或向特定進口商增購（Reorder）的決策。
4.2 UDI 追溯性黑洞核對演算法 (Traceability Black Hole Reconciling Algorithm)
此演算法用以實時對抗「資訊斷鏈」問題。系統以後端 CSV/JSON 緩存為基準，進行雙向滾動關聯（Two-Way Rolling Reconciliation）：
其中 
 為通路配送集合（Distributions），
 為醫院採購集合（Purchases）。
對帳狀態機核心邏輯流程：
code
Code
+---------------------------------------+
           |   New Distribution/Purchase Record    |
           +-------------------+-------------------+
                               |
                               v
               +---------------+---------------+
               |    Match by Serial Number?    |
               +-------+---------------+-------+
                       |               |
                    YES|               |NO
                       v               v
       +---------------+-------+   +---+-------------------+
       | Compare UDI-DI & Model|   | Is there unmapped     |
       +-------+---------------+   | logistics transit?    |
               |                   +---+---------------+---+
       +-------+-------+               |               |
       |               |            YES|               |NO
    YES|               |NO             v               v
       v               v       +-------+-------+   +---+---+
   +---+---+   +-------+---+   | Anomaly:      |   | SAFE  |
   | SAFE  |   | Anomaly:  |   | Unmapped      |   +-------+
   +-------+   | Serial    |   | Transit       |
               | Mismatch  |   +---------------+
               +-----------+
Serial Mismatch（序號不匹配）：
當 
 但 
 或 
 時。
成因：條碼輸入錯誤或包裝替換。
Discrepancy In Quantities（計量單位與申報量衝突）：
當 
 或 
 時（如百特出貨 1 個，但中榮登記進貨 1 組）。
成因：包裝階層（Packaging Hierarchy）申報基準未統一。
Unmapped Transit（未註冊在途交易）：
醫院申報了特定序號的入庫，但通路分銷商卻無對應的交貨申報。
成因：涉嫌非授權平行輸入（水貨）或院外調撥漏報。
4.3 供應鏈脆弱性評估演算法 (Supply Chain Vulnerability Predictor)
脆弱性演算法基於節點間的時空偏離度（Spatiotemporal Deviation） 與 歷史退貨呆滯度（Historical Retardation Rate） 進行加權計算：
DelayHrs (時差偏離)：配送申報交貨日與醫院登記收貨日之差距（超過 48 小時即觸發 Medium 警報）。
ReturnRate (退貨率)：特定路徑歷史退貨頻率。
ExpiryRisk (效期風險)：剩餘保存期限小於 180 天之產品佔比。
系統將此模型封裝於 /api/predict-disruptions 接口，回傳高解析度的脆弱性路徑與控制（Mitigation）方案。
4.4 庫存主動優化演算法 (Proactive Multi-Echelon Inventory Optimizer)
此模組透過對比近期採購耗用速度（Velocity of Consumption, VoC）與各點的安全庫存水平（Reorder Point, ROP）：
當預測到某醫院的現存量（OnHand Stock）在 
 內將低於安全庫存，且同區域有其他分院存在呆滯庫存（即有效期限不足 180 天，且 VoC 近乎為零）時，系統不採用向供應商增購的昂貴途徑，而是主動生成跨院調撥建議 (Inter-hospital Transfer Recommendation)，就近完成調撥，最大化保存期限效能。
當全球供應鏈因不可抗力中斷時，調用 Gemini 模型的推理能力，生成加急採購（Reorder）或即期促銷（Expedite Sale）方案。
5. 後端接口架構與實現細節 (API Endpoints & Server-Side Implementation)
後端採用 TypeScript 精確實現。以下為核心控制器（Controllers）的關鍵代碼邏輯架構：
5.1 唯一資料源加載與快取解析 (dataset.md Parsing Mechanism)
為確保數據一致性，後端不採用靜態死代碼，而是在系統啟動時，動態讀取專案根目錄下的 /dataset.md 文件，利用正規表達式與 CSV 分流解析引擎加載原始數據。
code
TypeScript
// /server.ts 核心啟動加載片段
function loadAndParseDatasetMD(): {
  purchase: any[];
  distribution: any[];
  geolocation: any[];
} {
  try {
    const filePath = path.join(process.cwd(), "dataset.md");
    if (!fs.existsSync(filePath)) {
      console.error("dataset.md not found at", filePath);
      return { purchase: [], distribution: [], geolocation: [] };
    }
    const mdContent = fs.readFileSync(filePath, "utf-8");
    
    // 正則定位 CSV / JSON 代碼塊
    const purchaseMatch = mdContent.match(/## 1\. 採購資料集[\s\S]*?```csv([\s\S]*?)```/);
    const purchaseCSV = purchaseMatch ? purchaseMatch[1].trim() : "";
    
    const distributionMatch = mdContent.match(/## 2\. 通路資料集[\s\S]*?```csv([\s\S]*?)```/);
    const distributionCSV = distributionMatch ? distributionMatch[1].trim() : "";
    
    const geoMatch = mdContent.match(/## 3\. 地理空間站點資料集[\s\S]*?```json([\s\S]*?)```/);
    const geoJSON = geoMatch ? geoMatch[1].trim() : "[]";
    
    const purchase = parseCSV(purchaseCSV);
    const distribution = parseCSV(distributionCSV);
    let geolocation: any[] = JSON.parse(geoJSON);
    
    return { purchase, distribution, geolocation };
  } catch (err) {
    console.error("Error reading/parsing dataset.md:", err);
    return { purchase: [], distribution: [], geolocation: [] };
  }
}
5.2 延遲初始化與健全安全降級機制 (Lazy Initialization & Safe Fallback Pattern)
在容器雲環境中，API Key 的暫時性缺失是極為常見的。為防止系統在啟動時崩潰，DHA-Hub 後端完美實施了 延遲初始化機制 (Lazy Initialization) 配合 本地高保真模擬器 (High-Fidelity Offline Mock Generators)：
code
TypeScript
let aiClient: any = null;
function getAI() {
  if (!aiClient) {
    const apiKey = process.env.GEMINI_API_KEY;
    if (apiKey) {
      aiClient = new GoogleGenAI({ apiKey });
    }
  }
  return aiClient;
}
當 getAI() 返回空，或者調用官方 API 逾時、超出 Quota 限制時，控制器會優雅地補獲 Exception，調用本地優化模擬器，保障前端運作不中斷。
6. 前端組件生態與視覺化設計 (Front-End Components System)
DHA-Hub 前端介面嚴格遵循高質感的瑞士/現代極簡主義 (Swiss Modern Minimalism)，拒絕對 UI 進行不必要的裝飾。全片使用 Lucide-React 圖標，並採用 Tailwind CSS 4.0 的深度配色方案與動態布局動畫。
code
Code
+-----------------------------------------------------------------------------------------+
|                                DHA-HUB COMPLIANCE PORTAL                                |
+-----------------------------------------------------------------------------------------+
| [A00013 NTU Hosp] [System Status: SECURE] [User: Auditor-Chief] | Theme: [Dark] [ZH]   |
+-------------------+---------------------------------------------------------------------+
| SIDEBAR NAV       | MAIN WORKSPACE                                                      |
|                   |                                                                     |
| (•) Topology Hub  | +-----------------------------------------------------------------+ |
| ( ) Optimizer     | |                       NETWORK TOPOLOGY HUB                      | |
| ( ) Audit Panel   | |  (Active Geolocation Stations & Multi-Echelon Flow Paths)       | |
| ( ) Data Ledger   | |                                                                 | |
| ( ) AI Copilot    | |                 [ NTU Hosp ] <=========> [ Medtronic ]          | |
|                   | |                      \\                   //                    | |
| [ Terminal Logs ] | |                       \\                 //                     | |
| [ > API Res OK  ] | |                        v                 v                      | |
| [ > Cache Synced] | |                             [ CMU Hosp ]                        | |
|                   | +-----------------------------------------------------------------+ |
|                   | +-------------------------------+ +-------------------------------+ |
|                   | |         AUDIT LEDGER          | |          AI COPILOT           | |
|                   | | ID: BH-001  Status: Pending   | | User: Find Serial discrepancies | |
|                   | | ID: BH-002  Status: Pending   | | AI: Analyzing 36 nodes...     | |
|                   | +-------------------------------+ +-------------------------------+ |
+-------------------+---------------------------------------------------------------------+
6.1 NetworkHub.tsx - 地理拓撲與物流網絡可視化
技術實現：本組件為系統核心之視覺高地。利用 D3.js (v7) 的強力力導向圖（Force-Directed Graph）或精確緯度雙向投影技術，將 GeolocationStation 的物理座標投影至 SVG 畫布。
互動設計：分銷商與醫院分別以特定的幾何幾何和高反差色彩（如亮藍色與翠綠色）進行呈現。物流路徑以 animate-dash 的 SVG Dash Array 移動特效標示其流速與流量。用戶懸停於特定路徑上，可實時高亮該通路正在配送的節律器型號、數量與潛在風險係數。
6.2 WowGraphs.tsx - 高維數據關聯圖表
技術實現：基於 Recharts 庫，精確繪製「採購申報趨勢圖」、「分銷交貨時差直方圖」以及「即期保存期限漏斗圖」。
圖表設計：避免常規過飽和彩虹色，統一採用 Slate 灰色調搭配琥珀色（Amber）作為警告色，為審計官提供毫無視覺干擾的高密度數據洞察。
6.3 AuditPanel.tsx - 異常黑洞治理與通報中心
技術實現：此組件提供交互式網格（Grid），將檢測到的 TraceabilityBlackHole 完美條列。審計員可在此處切換黑洞的治理狀態（pending ➔ resolved / false-positive）。
一鍵申報：點擊「生成 TFDA 通報書」，系統發送 POST 請求至 /api/generate-tfda-report，拉起 Layer 2 Diagnostic Agent，利用 Gemini AI 將原本枯燥、繁雜的技術對帳資料，一鍵渲染成符合國家法規格式、具備標準公文段落結構的 Traditional Chinese PDF/Markdown 格式正式通報書。
6.4 InventoryOptimizer.tsx - 主動式多級調撥面板
技術實現：展示當前各節點之預警庫存、呆滯庫存以及即期庫存，並實時列出 AI 生成之 Transfer/Reorder Action 列表。
沙盒模擬 (Sandbox)：審計人員可在不改動實際生產庫存的情況下，拖動安全儲備滑桿（Thresholds），實時查看庫存優化拓撲的動態再分配演算法響應。
6.5 DataLedger.tsx - 進銷存底層原始數據分類帳
技術實現：提供超高載入性能的虛擬滾動表格（Virtual Scroll Table），支援分開對「採購（Purchase）」與「通路配送（Distribution）」兩大資料集進行高解析度關鍵字模糊查詢（Fuzzy Search）、UDI 精確篩選、製造日期及到期日期排序，確保審計員隨時核對底層細節。
6.6 AICopilot.tsx - 自然語言交互審計協作艙
技術實現：採用聊天室交互界面（Chat Workspace）。深度整合 Layer 1 Orchestrator，並在 UI 上提供即時的 Token 消耗、推理延遲統計，同時在 Copilot 邊側顯示系統運行之實時 Terminal Logs，彰顯系統運行的全透明度（Observability）。
7. 三大全新 WoW AI 特色功能設計方案 (Proposed 3 Additional WOW AI Features)
為了進一步將 DHA-Hub 的功能推向業界頂尖層次，在此特別規劃並設計 3 項「WOW」級別的 AI 深度整合功能。以下提供其詳細的技術實現、架構擴充與交互方案：
7.1 WOW AI 1: 基於時空關聯的「實時全息空間風險熱圖與動態路徑重導向引擎」 (Real-Time Predictive Spatiotemporal Routing Heatmap)
7.1.1 業務痛點與技術原理
常規的物流網路僅能展示「過去已發生的配送」。當颱風襲台、突發性公衛事件導致特定地區醫院（如南台灣或東部山區）耗用量暴增，或是特定港口關稅清關延遲時，靜態拓撲圖完全無法應變。本功能引入時空軌跡多變量分析（Spatiotemporal Vector Trajectory Analysis）。
7.1.2 架構與 API 實現
在後端新增 /api/spatial-risk 接口。該接口整合即時台灣氣象 API、交通部路況 API 與當前 DHA-Hub 各物流站點的 UDI 即時數據，將其轉換為三維矩陣（維度為：[經度, 緯度, 時間維度]）。使用 Gemini API 的多模態與高維推理能力，輸入當前所有節點的經緯度與庫存，輸出預測性的脆弱路徑，並重構路徑。
7.1.3 API 接口定義 (POST /api/predict-routing)
code
TypeScript
interface SpatialRiskRequest {
  model: string;
  weatherAlerts: boolean; // 是否納入即時天候因子
  currentInventoryState: any;
}

interface SpatialRiskResponse {
  riskHeatmapPoints: Array<{
    latitude: number;
    longitude: number;
    intensity: number;  // 風險強度 0 到 1.0
    description: string;
  }>;
  recommendedRerouting: Array<{
    routeId: string;
    originalPath: string[];
    alternativePath: string[];
    delayMitigationMinutes: number;
    reason: string;
  }>;
}
7.1.4 前端視覺呈現
在 NetworkHub 中引入 WebGL 著色器（Shader），於 D3.js 拓撲網格底層疊加一層動態流動、色彩斑斕的「風險熱圖（Risk Heatmap）」。當某配送路線風險過高時，該路徑之 SVG 動態虛線會由「常規藍」漸變為「高警示脈動紅（Pulsing Red）」，並實時在路徑旁浮現一條虛擬的「AI 推薦替代分銷路徑」，極具震撼性視覺張力。
7.2 WOW AI 2: 雙層 Agent 自主法規合規模擬沙盒與 TFDA 交互式通報合規演練系統 (TFDA Compliance Simulator Sandboxing)
7.2.1 業務痛點與技術原理
當審計官發現重大合規黑洞時（例如：某批心臟節律器序號在多個地區醫院重複出現，涉嫌水貨或重貼條碼之重大刑案），直接通報食藥署可能涉及繁雜的法律核實程序，若發生誤判則對醫院名譽損害極大。本功能提供一個自主式合規演練沙盒，讓審計官在正式提交報告前，能以模擬形式與「虛擬 TFDA 稽查官」進行合規答辯演練。
7.2.2 架構與 API 實現
在後端新增 /api/compliance-sandbox 路由。此路由利用 Gemini 的角色扮演與長上下文檢索 (Role-Playing & Long-Context Retrieval)。載入中華民國《醫療器材管理法》及《醫療器材優良製造規範 (GMP)》全本條文作為 Grounding。創建一個獨立的 Layer 2 Agent 扮演「TFDA 資深法規合規處長」，前端審計員可上傳其調查佐證，AI 將嚴苛、專業、模擬法律程序地質詢審計員，指出其證據鏈之漏洞。
7.2.3 API 接口定義 (POST /api/compliance-sandbox/chat)
code
TypeScript
interface SandboxRequest {
  model: string;
  caseId: string;
  userMessage: string;
  evidenceFiles?: string[]; // 支持上傳 PDF 診斷證據
  auditStateContext: any;
}

interface SandboxResponse {
  agentRole: "TFDA_Chief_Inspector";
  replyMessage: string; // 嚴肅、專業、引經據典的質詢
  complianceVulnerabilityScore: number; // 當前通報案例在法律層面的漏洞分數 (0-100)
  suggestedActionCorrection: string;   // AI 指出應如何補充證據
}
7.2.4 前端視覺呈現
前端以「法庭對決/法規聽證會」的雙視窗形式呈現。左側是審計官持有的 DHA-Hub 進銷分析圖表、序號匹配軌跡；右側是神祕高反差暗黑風格的「虛擬食藥署聽證席」。AI 會實時對審計官提報的案件進行打分。當審計官答辯合規度提升時，聽證席的安全指示燈會漸漸變為綠色，將硬核的合規治理工作轉化為極具黏性與教育意義的「合規防禦沙盒遊戲」。
7.3 WOW AI 3: 語音喚醒、免手動 UDI 語音對帳與「AI 視覺條碼多模態校驗助手」 (Multimodal Hands-Free Voice & Visual UDI Assistant)
7.3.1 業務痛點與技術原理
在醫院地下庫房或開刀房進行實體庫存盤點時，醫護人員或審計官雙手需要拿取、核對心臟節律器實體盒裝，極其不便騰出手來操作鍵盤。且心臟節律器包裝上往往印有極其複雜、字體極小的 UDI 條碼與多個輔助識別標籤。本功能結合 多模態電腦視覺（Multimodal Computer Vision） 與 即時語音流式辨識（Streaming STT），實現完全免手動的雙重審計。
7.3.2 架構與 API 實現
利用 WebRTC 或是前端 MediaDevices API 捕獲相機影像，並利用 Web Speech API 進行本機端語音捕捉。後端接口 /api/multimodal-verify 接收前端傳送的「盒裝實體照」與語音辨識文本。調用支援多模態的 Gemini (如 Gemini 2.5 Flash 或 Pro)，實時進行「視覺 UDI 提取 ➔ 底層資料庫比對 ➔ 語音語意比對」的三向核對。
7.3.3 API 接口定義 (POST /api/multimodal-verify)
code
TypeScript
interface MultimodalVerifyRequest {
  imageBytes: string; // Base64 醫療器材包裝實體相片
  voiceTranscript: string; // 醫護人員用語音唸出的序號/型號，如 "型號W3DR01，序號RNJ一三五"
  currentStationId: string;
}

interface MultimodalVerifyResponse {
  matchStatus: "MATCHED" | "MISMATCH_DETECTED" | "UNKNOWN_DEVICE";
  visualExtractedUDI: {
    di: string;
    pi_lot: string;
    pi_serial: string;
  };
  databaseMatchedRecord: any;
  discrepancyReport?: {
    field: string;
    visualValue: string;
    databaseValue: string;
    gravity: "CRITICAL" | "WARNING";
  };
  audioInstruction: string; // 系統將此文字轉為 TTS，回播給醫護人員，如 "警告！序列號與資料庫登記之百特醫療不符，請核對包裝。"
}
7.3.4 前端視覺呈現
在 AICopilot 中提供一個「盤點校對相機鏡頭（AR Overlay）」。當相機對準醫材包裝時，AI 會利用網格動態在畫面上圈出 UDI 14碼、批號、效期等關鍵字，並以綠色或紅色的虛擬外框進行高亮（AR Annotation）。系統同時播放溫和、具磁性的「AI 語音播報」，直接用語音提示「此節律器批號無誤，但與本院登記之 A00338 進貨批號不匹配，已自動將其歸類為黑洞 BH-003，並記錄於 Data Ledger 中」。這將帶來無與倫比的前沿科技感與臨床實用度。
8. 安全性、隱私保護與法規遵循 (Security & Regulatory Alignment)
作為一款處理醫療通路、機敏醫院代碼與病患潛在涉案數據的系統，DHA-Hub 設計了嚴格的合規框架：
8.1 數據去識別化與隱私邊界 (Data De-identification)
系統不直接儲存、不傳輸任何病患個人姓名、身分證字號或詳細病歷。所有涉及「病患」層面的追溯性，一律終止於醫療器材唯一的「產品序列號 (SerialNumber)」與「受贈機構 (A00013)」。
將醫院真實名稱與 ID 進行雜湊（Hashing）或使用標準代碼對齊，防範惡意第三方推導出特定名醫或特定科室之採購商業機密。
8.2 傳輸與儲存安全
TLS 1.3 傳輸加密：前後端的所有數據傳輸均強制透過 Nginx TLS 1.3 加密層進行。
Gemini 安全合規配置 (API Safety Settings)：在呼叫 Gemini API 時，設置合規的安全邊界（Safety Filters），防止 AI 在合規報告中生成虛假病患隱私，杜絕幻覺。
API 密鑰隱匿：嚴禁任何 VITE_ 前綴的敏感 API key 曝露於瀏覽器 DevTools 中。所有與 Gemini 相關的密鑰統一保存在生產環境之環境變量（Cloud Run Secrets Manager）中，由 Node.js 伺服器在後端安全呼叫，防範密鑰洩漏。
8.3 國際與國家標準對齊
系統架構完全相容於中華民國食品藥物管理署 TFDA「醫療器材唯一識別（UDI）系統申報規範」。
資料欄位精確映射 FDA UDI 格式、GS1 條碼標準及 ISO 13485 醫療器材品質管理系統標準。
9. 系統運作視覺日誌分析 (Observability & Terminal Console Logs)
為維持高透明度（Observability），DHA-Hub 配備了獨特的底層「執行日誌視覺化系統」（內嵌於 App 左側底部）。以下詳解兩類高典型之運行日誌，說明其在底層的數據通訊：
9.1 成功合規同步與對帳流程
code
Code
[2026-07-02 20:47:49] [INFO]    DHA-Hub Compliance Gateway loaded. Initializing diagnostic pipelines...
[2026-07-02 20:47:50] [INFO]    Fetch request dispatched to: GET /api/dataset
[2026-07-02 20:47:51] [SUCCESS] Data synchronization complete. Loaded 36 purchases, 26 distributions, 9 geo nodes.
[2026-07-02 20:47:51] [INFO]    Reconciliation Engine triggered. Scanning for overlapping Serial Numbers...
[2026-07-02 20:47:52] [SUCCESS] Scanning complete. Detected 2 active Traceability Black Holes: BH-001, BH-002.
[2026-07-02 20:47:52] [INFO]    Network Topology rendering dispatched to SVG Layer. Applied Fruchterman-Reingold Force Model.
9.2 API 逾時、Quota 限制與自動降級本地引擎流程
當雲端網路不穩定，或 Gemini 帳戶額度用盡時，後端實施安全降級（Reversion），確保系統不發生死當：
code
Code
[2026-07-02 20:48:01] [INFO]    Dispatching request to model: models/gemini-flash-latest
[2026-07-02 20:48:01] [INFO]    [Dataset] Compiling context: 15 purchase entries, 15 distribution entries.
[2026-07-02 20:48:02] [INFO]    [LLM] Transmitting request to Gemini API...
[2026-07-02 20:48:05] [WARNING] [LLM] Connection timeout or resource exhausted. HTTP Code 429.
[2026-07-02 20:48:05] [WARNING] Backend connection refused. Initializing high-fidelity mock fallback: API_KEY_EXHAUSTED
[2026-07-02 20:48:05] [INFO]    [Local Engine] Reverting to secure local optimization helper.
[2026-07-02 20:48:06] [SUCCESS] Offline Smart AI Optimizer generated 3 local recommendations. Synthesis complete.
10. 深度技術思考與架構驗證問題 (20 Comprehensive Follow-up Questions)
為了確保 DHA-Hub 在從概念原型（MVP）邁向高併發、極致安全、跨醫院大規模部署的實戰過程中，系統架構能保持堅不可摧，研發團隊、法規專家與安全架構師必須深入探討並解答以下 20 個關鍵問題：
10.1 系統架構、效能與擴充度問題 (Architecture, Performance & Scalability)
問：當前系統直接讀取專案根目錄之 dataset.md 文件作為唯一真實資料源並緩存在內存中。若醫院日均採購與通路配送數據量暴增至 100 萬筆以上，如何將此機制遷移至以 Drizzle ORM 為基礎之 PostgreSQL (Cloud SQL) Relational Database？
思考方向：需要設計增量同步機制、分頁加載（Pagination）以及複合索引（Composite Indexes），特別是針對 SerialNumber、UDI_DI 與 建立日期 欄位建立 B-Tree 索引，以優化檢索速度。
問：在多級庫存調撥演算法中，當兩家大型醫學中心（如台大醫院與台北榮總）同時爭奪某個高價值即期心臟節律器之調撥配額時，系統該如何實施「分散式併發控制」與「樂觀鎖（Optimistic Locking）」機制，以防止產生「重複調撥」或「超賣」現象？
思考方向：可以探討在 PostgreSQL 交易中使用 SELECT FOR UPDATE 或引入 Redis 分散式鎖，確保調撥狀態在變更時具備強一致性（ACID）。
問：前端的 NetworkHub.tsx 使用 D3.js 渲染 2D 地理拓撲圖。若系統需要同時渲染全台 500 家分銷商與 2000 家醫療診所之動態物流拓撲網格，D3.js 繪製 SVG 的 DOM 操作會成為嚴重效能瓶頸。應如何利用 HTML5 Canvas 或是 PixiJS (WebGL) 進行 Canvas-Based 渲染優化？
思考方向：將數千個節點和連線的繪製，從 SVG DOM 樹轉移至單一 Canvas 畫布，利用雙緩衝區（Double Buffering）技術，或 Web Worker 異步計算節點力學矩陣，維持 60 FPS 的流暢動畫體驗。
問：如何利用 React 18+ 的 useTransition 或 useDeferredValue 勾子（Hooks）來防止當用戶在 DataLedger.tsx 的原始分類帳中進行超高速全文字串模糊篩選（Fuzzy Search）時，導致整個前端渲染線程（Main Thread）發生卡頓與 UI 失去響應？
思考方向：降低非關鍵渲染（如底層幾千行表格的過濾）之優先級，優先響應輸入框的輸入與標籤切換，提升系統之感知效能（Perceived Performance）。
10.2 AI、大語言模型與 Agent 設計問題 (AI, LLM & Agent Design)
問：在 /api/optimize 中，後端硬性截取前 15 筆採購與分銷記錄傳送給 Gemini。這種「上下文截斷」會導致 AI 無法看到全局歷史。在不超出 LLM Token Quota 且維持低推理延遲的前提下，如何設計一套「高維數據特徵提取器（Feature Extractor）」或基於 RAG 的「向量特徵檢索系統」，僅傳送最相關的即期、呆滯數據給 Gemini 進行決策？
思考方向：預先在後端進行聚合（Aggregation），如僅輸出「庫存周轉率異常的 Top 50 產品摘要」或「即期保存期限小於 10% 的產品清單」，降低上下文體積，並提升模型推理精準度。
問：若 Gemini 生成的優化 JSON 結構發生「格式畸變」（例如：返回的 JSON 中，部分欄位缺失或陣列結構受損），當前 Express 伺服器會直接崩潰並降級。如何結合 zod 庫實施強大的運行時 Schema 校驗（Runtime Validation），並在後端實現「自我修正機制」（Self-Repairing Loop），即當校驗失敗時，自動將錯誤報告發回給 Gemini 要求其重構？
思考方向：使用 zod.safeParse() 捕獲結構異常，將錯誤定位（Validation Error Path）作為 Context 再次發送，要求 LLM 回傳 JSON Patch，或直接利用 SDK 的 responseSchema 強制模型輸出指定格式。
問：雙層型 Agent 系統中，Layer 1 Orchestrator 進行分類路由時，如何防止將合規審計問題「錯誤分類」為庫存調撥優化，導致向用戶輸出完全無關之回答？需要設定哪些精密的「Few-Shot Prompting」範例或評估基準（Evaluation Benchmarks）？
思考方向：在 System Instruction 中明確定義分類決策樹，並提供 3 至 5 個極端邊界案例（Edge Cases），同時在後端利用輕量化模型（如 Gemini Flash）先進行極速意圖分類，再由旗艦模型（如 Gemini Pro）執行重推理。
問：如何設計一套自動化機制來評估 Gemini 模型在生成跨院調撥優化時的「幻覺率（Hallucination Rate）」？如何確保 AI 推薦的調撥型號（如 W3DR01）確實存在於來源醫院的剩餘申報庫存中，而非憑空捏造？
思考方向：在後端建立「確定性真理校驗層（Deterministic Validation Layer）」。在 AI 產出 JSON 優化方案後，後端必須自動以傳統演算法掃描一次資料庫，驗證 sourceId 的 remainingQuantity 是否真的 
 quantity，若否，直接在 API 內部將該 Recommendation 標示為不合規或予以剔除。
10.3 安全性、隱私、加密與法規遵循問題 (Security, Privacy & Compliance)
問：雖然 DHA-Hub 目前僅處理機構與序列號資料，但若惡意攻擊者透過「庫存重建攻擊（Inventory Reconstruction Attack）」，將分銷商出貨量與醫院公開之特定手術量進行比對，推導出特定機敏患者（例如：知名政治人物）使用植入式心臟節律器的具體日期與醫院，系統該如何防範？是否需要引入「差分隱私（Differential Privacy）」或「資料擾動技術（Data Perturbation）」？
思考方向：在發送給外部 APIs 或進行宏觀視覺化前，對交貨數量與日期進行合理的微幅時空微調或聚合，防止精確時間戳與數量被惡意關聯。
問：本系統使用 Node.js 後端安全呼叫 process.env.GEMINI_API_KEY。在實際的多醫院聯合運營模式下（Multi-Tenant Deployment），如果每家醫院都想使用自己專屬的 Gemini 帳戶以控制 API 開銷，系統應如何設計「多租戶密鑰安全隔離（Multi-Tenant Secrets Segregation）」與託管機制？
思考方向：可探討將 API key 加密儲存於資料庫中，或整合 GCP Secret Manager，為每個 Tenant ID 建立對應的密鑰路徑，在請求上下文載入時動態解密獲取。
問：若本系統在歐洲分院部署，需遵循嚴格之 GDPR。而 UDI 序列號（SerialNumber）在某些法庭釋憲中被認定為「間接識別個人之數據（Indirect PII）」。系統應如何設計完善的「被遺忘權（Right to be Forgotten）」與「資料匿名化流水線」？
思考方向：實施一次性雜湊鹽值加密（Salted Hashing）處理序列號，或者對過期且已耗用的序列號進行定期物理擦除，僅保留宏觀的審計統計數據。
問：若 TFDA 的監管法規發生動態更新（例如：要求植入式心臟節律器申報必須新增「無線射頻安全認證代碼」），系統如何保證資料模型（PurchaseItem / DistributionItem）具備足夠的「可擴充半結構化特徵」（如使用 PostgreSQL 的 JSONB 欄位），以實現零停機（Zero-Downtime）無縫升級？
思考方向：在 ORM 數據庫設計中預留一個靈活的 metadata 或 attributes JSONB 欄位，允許在不更改實體表結構（Table Schema Migration）的情況下，動態追加法規要求的新欄位。
10.4 業務、營運與供應鏈整合問題 (Business, Operations & Supply Chain)
問：在 InventoryOptimizer.tsx 中，系統提出「將高雄醫學大學附設醫院之即期心臟節律器（W2SR01）調撥至台中榮總」之優化方案。然而在實務上，此舉會受到兩家醫院間「不同預算體系」、「不同採購合約」與「醫療器材產權歸屬」之法律限制。系統應如何設計一套「調撥合約工作流（Transfer Contract Workflow）」，在實施物理調撥前自動協調兩端法務之電子簽章？
思考方向：結合工作流引擎（如 Temporal 或 Camunda），在 AI 生成調撥建議後，先向兩端醫院管理層發送「調撥意向同意書」，經由 API 完成產權轉讓電子簽章登記後，再行觸發物流配送單號生成。
問：高價值植入式心臟節律器對溫濕度極為敏感（需維持在常溫 15°C 至 25°C 之間）。若在區域調撥過程中（如高雄調撥至台中），因冷鏈失效導致器材受損。系統如何透過擴充「物聯網低功耗藍牙 (BLE) / NFC 實時感測數據對接機制」，將冷鏈數據納入追溯性黑洞審計的判斷指標？
思考方向：在 TraceabilityBlackHole 的 issueType 中，擴充一個 "Cold-Chain Breach" 異常型別。當物流感測器檢測到異常升溫，自動在 DataLedger 中將此 UDI 序列號鎖定為「已汙染」，防止臨床誤用。
問：如何設計本系統與醫院現有之 HIS (Hospital Information System, 醫療資訊系統) 與 ERP (Enterprise Resource Planning, 企業資源計劃系統)（如 SAP 或 Oracle）的標準 HL7 FHIR (Fast Healthcare Interoperability Resources) 接口串接，實現盤點與對帳之自動化，免除人工導入 CSV 的繁重步驟？
思考方向：開發符合 HL7 FHIR 標準之微服務（Microservices），動態訂閱醫院 HIS 的「開刀房耗用事件」與 ERP 的「驗收入庫事件」，以實時串流（Event-Driven Streaming）取代批次（Batch）對帳。
問：對於「美商美敦力」與「台灣百特醫療」等進口代理商，其出貨常採用「寄售模式（Consignment Stock）」，即器材物理上已存放在醫院庫房，但產權仍屬分銷商，直到手術消耗時才開立發票。DHA-Hub 應如何調整帳務對帳演算法，以清晰區分「實體庫存持有權」與「財務產權所有人」？
思考方向：在 PurchaseItem 中追加 consignmentStatus 欄位（寄售、已買斷、已消耗），並對寄售狀態之 UDI 進行特別追蹤，防止通路端重複申報銷售收入。
10.5 系統可靠性、可觀測性與容錯機制 (Reliability, Observability & Resilience)
問：在生產環境中，AI Studio 開發伺服器與 Preview 框架在檢測到 HMR 時可能產生 Benign Websocket 報錯。在部署至 Cloud Run 生產環境時，應如何精確配置容器之 Health Checks 與 Liveness/Readiness Probes，確保在高流量壓力下若 Node.js 伺服器發生 Uncaught Exception 能在 1 秒內由雲端主動拉起新容器？
思考方向：在 server.ts 中設計獨立的 /api/health 輕量化探針，僅做內存與資料庫連接檢驗。設定 Express 之錯誤捕捉中間件（Error Handling Middleware），捕獲所有未處理的例外並優雅降級，防止容器崩潰。
問：在面對外部 API 發生極端雪崩（Cascade Failure）或斷網（如海底電纜斷裂導致無法與 Google Core API 連線）時，DHA-Hub 的「高保真度本地優化引擎 (Offline Emulation Engine)」如何根據當前緩存的歷史數據進行「本地線性預測（Linear Extrapolation）」，以維持至少 30 天的基本預警優化運作？
思考方向：本地端內置輕量化預測演算法（如移動平均線、簡單指數平滑），在無 AI 連線時，依託本機邏輯（Rule-Based Rules）持續過濾即期品、計算安全庫存，保障醫療體系在災難發生時不停擺。
問：當前系統之 Console 日誌（consoleLogs）皆存儲於 React Client-Side State 中，一旦用戶刷新頁面，歷史日誌將全部丟失。如何設計一個「持續性審計日誌通道（Persistent Audit Trail Logger）」，將所有的 AI 執行軌跡、審計官治理黑洞之動作，異步傳輸並保存至安全之雲端日誌分析平台（如 Google Cloud Logging / Winston Logger）以供日後司法合規備查？
思考方向：後端建立專門的 /api/logs 接收器，前端將重要審計日誌以異步佇列（Queue）形式推播至後端，後端寫入唯讀（Append-Only）之合規日誌庫，保障日誌之防篡改性。
問：若某家醫院之惡意管理員試圖通過「竄改本地資料庫」來掩蓋非授權醫療器材之流入，系統如何整合「區塊鏈/分散式共識帳本機制（Consortium Blockchain Ledger）」或採用具備加密學雜湊鏈結之「防篡改分帳（Cryptographically Verifiable Ledger）」，確保每一筆 UDI 進銷存的變更均具備不可篡改的司法級證據效力？
思考方向：探討使用 AWS QLDB 技術，或在每次完成 UDI 雙向對帳時，將其 SHA-256 雜湊鏈結（Hash Chain）寫入聯盟鏈節點，確保任何事後修改底層資料之行徑皆會立即導致雜湊鏈斷裂，觸發 Critical 安全警報。
11. 總結 (Summary)
DHA-Hub（智慧型醫療器材通路與採購分析系統）以其創新的雙層型自主 Agent 決報技術、高質感的瑞士現代極簡視覺風格，以及對 TFDA / UDI 合規痛點的精準打擊，在架構上展現出卓越的設計美學與工程嚴謹性。
本技術規格書詳盡闡述了系統現有設計理念，規劃了三大 WOW 級 AI 延伸特色，並拋出 20 個深邃的工程與法規實踐問題，為 DHA-Hub 從當前的先進原型成功躍升為「國家級、高保真、多醫院安全互聯之醫療物資合規監管與審計平台」奠定了無可撼動的堅實理論與技術基石。
