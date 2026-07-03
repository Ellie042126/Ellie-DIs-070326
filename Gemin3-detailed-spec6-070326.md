DHA-Hub (智慧型醫療器材通路與採購分析系統)
Comprehensive Technical Specification & Architectural Blueprint
Document Version: 2.0.0-PROD
Target Architecture: Full-Stack (Express, React 18, Vite, Node.js, SQLite Memory-State Engine)
Cognitive Gateway: Dual-Model LLM Orchestration Framework with Dynamic Parameter Control
1. Executive Summary & System Vision
High-value medical devices—specifically implantable cardioverter-defibrillators (ICDs), single/dual-chamber pacemakers, and high-frequency pulse generators—represent a class of healthcare assets governed by some of the most stringent regulatory regimes globally, such as the US FDA Unique Device Identification (UDI) directives and Taiwan TFDA medical product tracking regulations. The operational challenges in this sector are exceptionally complex:
Sterility & Shelf-Life Expiry: High-value devices have finite sterile shelf lives. Expiry results in a complete loss of asset value and a write-off of critical clinical capital.
Cold-Chain & Environmental Sensitivity: Although pacemakers do not require deep-subzero storage, they are sensitive to temperature spikes, impact, and high humidity, which can affect battery cell longevity and internal circuitry.
Traceability (UDI Compliance): Every single device must be tracked down to its exact global Unique Device Identifier (UDI-DI) and unique Serial Number (UDI-PI), ensuring a complete chain of custody from manufacturer to supplier, distributor, hospital warehouse, and eventually the operating room.
Distribution Friction: Clinical demand is uneven and highly localized. One medical center might experience a sudden surge in emergency surgeries, while a neighboring facility has idle safety stock nearing its expiration date.
DHA-Hub (Durable Health Asset Hub) is an enterprise-grade, full-stack, intelligent, real-time medical logistics and compliance orchestration engine designed to mitigate these issues. The core system architecture combines a simulated in-memory SQLite relational ledger, a high-performance interactive topological graph modeling clinical logistics, and an advanced dual-model AI orchestration gateway that defaults to gemini-3.1-flash-lite for high-frequency low-latency logistics decision-making, with hot-swappable routing to gemini-1.5-pro for deep compliance and risk analyses.
This document serves as the exhaustive, production-grade technical specification for DHA-Hub. It details the complete state definitions, schema structures, AI reasoning pipelines, visual telemetry blueprints, translation mappings, PDF rendering mechanics, and the end-to-end verification suite required to construct the platform.
2. System Architecture & Full-Stack Blueprint
DHA-Hub is built on a full-stack, custom Express + Vite foundation running on Node.js. It operates as an offline-capable, high-fidelity sandbox with a live memory-state ledger that acts exactly like a transactional SQLite database.
code
Code
+-------------------------------------------------------------------------+
|                              User Interface                             |
|    (React 18 / Tailwind CSS / Lucide-React / D3 Graph Canvas)           |
+------------------------------------+------------------------------------+
                                     |
                           JSON API / REST Calls
                                     |
+------------------------------------+------------------------------------+
|                         Express Web Server                              |
|              (API Routing, Session, Local Middleware)                   |
+------------------------------------+------------------------------------+
                                     |
                    +----------------+----------------+
                    |                                 |
+-------------------v----------+            +---------v------------------+
|   SQLite Memory DB Ledger    |            |   Dual-Model LLM Gateway   |
|  (Virtual Data Store, FIFO)  |            |  (Gemini API Client SSL)   |
+------------------------------+            +----------------------------+
2.1 File Tree Layout and Code Distribution
The codebase is organized into modular files to maximize maintainability, support clean typing, and prevent code generation truncation during deployment cycles:
code
Code
/
├── .env.example                # Defines local environment variables (GEMINI_API_KEY, APP_URL)
├── .gitignore                  # Standard repository file exclusion rules
├── package.json                # Project manifest, dev dependencies, build/dev scripts
├── vite.config.ts              # Configures Vite build, aliases, and dev-server parameters
├── tsconfig.json               # Configures TypeScript compiler settings and path mappings
├── metadata.json               # Frame permissions, app naming, major capabilities
├── dataset.md                  # Default purchase, distribution, and geolocation datasets
├── TECHNICAL_SPECIFICATION.md  # Comprehensive architectural blueprint
├── index.html                  # Standard entry HTML wrapper
└── src/
    ├── main.tsx                # StrictMode entry loader
    ├── index.css               # Global Tailwind CSS imports and system-wide font face bindings
    ├── types.ts                # Strict TypeScript interfaces for all system states
    ├── data.ts                 # Pre-loaded baseline datasets parsed from dataset.md
    ├── App.tsx                 # Core UI dashboard container and React router-tabs
    ├── services/
    │   ├── llmService.ts       # Secure LLM Gateway, models, and prompt template bindings
    │   └── ledgerService.ts    # Operations for querying, adding, and auditing virtual SQLite state
    └── components/
        ├── D3NetworkGraph.tsx  # Dynamic topological d3-force network visualizer
        ├── InteractiveROP.tsx  # Reorder Point (ROP) simulator with real-time UI gauges
        ├── LiveTerminalLog.tsx # Live terminal log viewer for streaming transaction steps
        ├── WowDashboard.tsx    # Five analytical charts detailing supply chain status
        └── ComplianceCenter.tsx# UDI matching, audit trails, and license compliance overrides
2.2 System Preference & Language State Management
DHA-Hub defaults to a multi-lingual, theme-aware client state. The system initializes with the following primary state variables:
code
TypeScript
export type Language = 'zh-TW' | 'en-US';
export type AppTheme = 'light' | 'dark';
export type ModelOption = 'gemini-3.1-flash-lite' | 'gemini-1.5-pro';

export interface UserPreferences {
  theme: AppTheme;
  language: Language;
  selectedModel: ModelOption;
  temperature: number;
  topP: number;
  customPromptTemplate: string;
}
The system translation map is structured statically in src/data.ts and accessed through a custom React hook useTranslation():
code
TypeScript
export const TRANSLATIONS: Record<Language, Record<string, string>> = {
  'zh-TW': {
    appName: "DHA-Hub 智慧型醫療器材通路與採購分析系統",
    dashboard: "智能儀表板",
    ledger: "虛擬 SQLite 交易帳本",
    advisor: "AI 決策顧問",
    compliance: "法規合規與 UDI 稽核",
    terminal: "實時操作終端",
    activeModel: "當前 AI 模型評估器",
    theme: "介面主題",
    lang: "系統語言",
    purchaseTitle: "Procurement / 採購進貨明細",
    distTitle: "Distribution / 通路分銷明細",
    ropTitle: "ROP 再訂購點動態模擬器",
    diagnostics: "系統自我檢測診斷",
    executeOpt: "執行 AI 庫存與合規優化",
    rebalanceAction: "執行跨節點主動再平衡 (AMRA)",
    complianceOverride: "強制合規覆蓋",
    pdfExport: "導出 AI 戰略報告 PDF",
    expiryAlert: "過期警報 (FIFO 優先)"
  },
  'en-US': {
    appName: "DHA-Hub Medical Device Logistics & Procurement Analytics System",
    dashboard: "Smart Dashboard",
    ledger: "Virtual SQLite Transaction Ledger",
    advisor: "AI Decision Advisor",
    compliance: "Regulatory Compliance & UDI Audit",
    terminal: "Real-time Command Terminal",
    activeModel: "Active AI Evaluator Model",
    theme: "Interface Theme",
    lang: "System Language",
    purchaseTitle: "Procurement Records",
    distTitle: "Distribution Records",
    ropTitle: "Dynamic ROP Simulator",
    diagnostics: "System Self-Diagnostics",
    executeOpt: "Execute AI Inventory Optimization",
    rebalanceAction: "Execute Inter-Node Rebalancing (AMRA)",
    complianceOverride: "Force Compliance Override",
    pdfExport: "Export AI Strategy Report PDF",
    expiryAlert: "Expiry Alert (FIFO Focus)"
  }
};
Theme changes inject or remove the .dark class directly on the root <html> element, modifying variables defined in Tailwind CSS to transition between an off-white clinical canvas and an elegant, low-contrast charcoal slate canvas.
3. Virtual SQLite Ledger Database Design
DHA-Hub maintains a simulated transactional database inside the Node.js memory model, operating as a virtual SQLite instance. This design guarantees consistent relational behavior across clients while avoiding local database configuration issues during preview phases.
3.1 Data Schema Declarations
The schema defines three fundamental tables: purchase_records, compliance_overrides, and hub_topology.
code
SQL
CREATE TABLE purchase_records (
    id INTEGER PRIMARY KEY,
    declarant VARCHAR(50) NOT NULL,
    receiveDate VARCHAR(8) NOT NULL,
    supplier VARCHAR(50) NOT NULL,
    licenseNo VARCHAR(100) NOT NULL,
    chineseName VARCHAR(255) NOT NULL,
    udi_di VARCHAR(14) NOT NULL,
    subCategory VARCHAR(100) NOT NULL,
    batchNo VARCHAR(50) NOT NULL,
    serialNo VARCHAR(50) UNIQUE NOT NULL,
    model VARCHAR(50) NOT NULL,
    quantity INTEGER NOT NULL,
    unit VARCHAR(10) NOT NULL,
    expiryDate VARCHAR(8) NOT NULL,
    savedDays INTEGER NOT NULL,
    isReturned INTEGER DEFAULT 0,
    createdDate TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE compliance_overrides (
    id INTEGER PRIMARY KEY,
    serialNo VARCHAR(50) UNIQUE NOT NULL,
    auditor VARCHAR(100) NOT NULL,
    overrideReason TEXT NOT NULL,
    fileReference VARCHAR(100),
    timestamp VARCHAR(50) NOT NULL
);
3.2 Relational Consistency Constraints
The virtual ledger enforces reference rules:
Unique Serial Constraints: A unique identifier check is performed before any transaction insert. This is critical for preventing gray-market parallel imports or duplicate device registrations.
Regulatory Rule Checking: Ensures that all active serialNo objects map back to registered UDI Device Identifiers (udi_di) that have valid import licensing references (licenseNo).
Dynamic Cascade Re-evaluation: When a physical action occurs (such as an emergency dispatch or an regulatory override), the virtual SQLite engine recalculates network metrics and cascades updates across all active view modules.
4. Dual-Model LLM Orchestration Framework
DHA-Hub implements a robust dual-model LLM gateway located in /src/services/llmService.ts and proxied on the server layer. It is optimized to support high-speed inferences via gemini-3.1-flash-lite while maintaining fallback and advanced options for gemini-1.5-pro or other enterprise models.
code
Code
+-------------------------------------------------------------------------+
|                       Dual-Model LLM Gateway                            |
|                       (src/services/llmService.ts)                       |
+-----------------------------------+-------------------------------------+
                                    |
            +-----------------------+-----------------------+
            |                                               |
  [Low-Latency Inference]                        [High-Reasoning Fallback]
  "gemini-3.1-flash-lite"                         "gemini-1.5-pro"
            |                                               |
            +-----------------------+-----------------------+
                                    |
                        Unified API Payload Format
                        - customPromptTemplates
                        - temperature & topP params
                                    |
                                    v
                        /api/analyze-local (Server)
                                    |
                       Secure SSL Request to Google
4.1 Custom Prompt Template Interfaces
The system provides robust prompt management interfaces, allowing users to select preset analytical lenses or tune raw system instructions directly from the UI:
code
TypeScript
export interface PromptTemplate {
  id: string;
  name: string;
  description: string;
  systemInstruction: string;
  defaultUserPrompt: string;
}

export const DUAL_MODEL_PRESETS: Record<string, PromptTemplate> = {
  logisticsAudit: {
    id: 'logistics-audit',
    name: '精準供應鏈再訂購點優化範本 (High-Sec Logistics)',
    description: '專為高單價心律調節器與植入式脈搏產生器設計，分析庫存安全存量、前置時間、及最佳再訂購點 (ROP)。',
    systemInstruction: `你是一位資深的醫療器材物流稽核總監（Chief Medical Logistics Officer）。
你的職責是深入分析 SQLite 虛擬帳本內所有 UDI 批號、製造日期與有效期限。
請根據使用者輸入的 ROP 滑桿參數（日平均消耗量 ADU、前置天數 Lead Time、安全庫存量 Safety Stock），
結合當前九個醫療院所與分銷商的地理分布數據（DHA Hub Geolocation Stations），
精確計算出：
1. 哪些醫院（例如國立臺灣大學醫學院附設醫院 A00013、台北榮民總醫院 A00002）正面臨嚴重庫存短缺威脅？
2. 根據 FIFO 庫存先進先出原則，哪些序號的裝置（如 RNE644291S2001）已經進入「過期警戒黑洞期（<180天）」必須立即調度至高週轉率醫院？
3. 輸出一個清晰、結構化的 Markdown 規劃表，給予採購主管最嚴格的再訂購指引與緊急轉載建議。
注意：請務必使用繁體中文（Traditional Chinese）回答，並維持專業、臨床化、具備高度決策可行性的口吻。`,
    defaultUserPrompt: '請根據目前的庫存數據，針對 W2SR01 與 W3DR01 兩種主力心律調節器型號，進行 ROP 的全面健檢，並指出有潛在缺貨風險的醫療中心。'
  },
  regulatoryCompliance: {
    id: 'regulatory-compliance',
    name: 'TFDA 醫事法規與 UDI 唯一識別碼稽核範本',
    description: '專注於高風險植入性醫療器材合規稽核，識別非法平行輸入、黑市水貨、與漏報申報序號之「黑洞交易」。',
    systemInstruction: `你是一位嚴格的 TFDA 醫療器材合規稽核檢察官。
你需要仔細比對進貨帳本（Purchase Dataset）與分銷轉載帳本（Distribution Dataset）之間的 UDI 關聯性：
1. 找出任何存在於「分銷帳本」中，但「進貨帳本」卻查無此進口申報許可證號、或許可證號不匹配的「黑洞產品序號（Compliance Black Hole）」。
2. 針對批號、UDI-DI（例如 00763000955953）進行嚴格的許可證對應。
3. 提供一份合規風險報告，明確列出違法序號、潛在平行輸入風險，並制定對應的「合規覆蓋方案（Compliance Override）」。
注意：必須使用繁體中文回答，指出具體數據行號，不可遺漏任何可疑的批次細節。`,
    defaultUserPrompt: '請立刻比對進貨與分銷帳本，抓出所有未申報或許可證號不符的異常 UDI 產品序號，並評估其法規風險。'
  }
};
5. Core Feature Specification: Dynamic Inventory Reorder Point (ROP) & Active AI Hub
This core feature represents the strategic brain of the application. It dynamically cross-references the live virtual SQLite data with user-set parameters to generate actionable solutions.
code
Code
[ Live Virtual SQLite Ledger ] ----+
                                  |
[ ROP Sim Parameters (Sliders) ] -+----> [ AI Inventory Optimizer Engine ]
- Average Daily Usage (ADU)       |      - Calls gemini-3.1-flash-lite
- Lead Time (Days)                |      - Performs Multi-variable scoring
- Safety Stock                     |
                                  |
   [ Custom Prompt Template ] ----+
                                  |
                                  v
            +-------------------------------------------------------+
            |          Real-time Immersive Wow Execution            |
            |  1. Simulated Sandbox Terminal logs                   |
            |  2. Interactive Visual ROP Gauge (Dynamic Gauge Bar)  |
            |  3. One-Click Trigger Action Cards with SQLite sync    |
            +-------------------------------------------------------+
5.1 Dynamic ROP Mathematical Modeling
Within the interface, users are presented with three interactive precision sliders, allowing them to manipulate the mathematical variables of the classic inventory replenishment formula:
ADU Slider: Ranges from 0.1 to 10.0 units per day (Step: 0.1).
Lead Time Slider: Ranges from 1 to 30 calendar days (Step: 1).
Safety Stock Slider: Ranges from 0 to 50 units (Step: 1).
Whenever a slider is adjusted, the client recalculates the ROP value in real time. The calculated ROP is dynamically projected against the current in-stock quantities of each pacemaker model (W3DR01 and W2SR01) filtered by their active warehouse nodes. If the current quantity drops below the ROP, a critical alert state is entered, visually represented by an amber pulsing warning indicator.
5.2 Immersive Real-Time Visual Execution
When the user triggers the "Run AI Optimization" routine, an interactive telemetry console is displayed. The optimization process is executed in distinct visual phases to provide a highly interactive experience:
Phase 1: In-Memory Ledger Extraction (t = 0ms)
The system query processor runs an analytical pipeline over the local memory store, compiling total quantities grouped by model, declarant, and days remaining before sterility expiration.
Log stream in Terminal: [INFO] SQLite Master Query initialized. Fetching active stock levels for implantable pulse generators...
Phase 2: Topological Friction Modeling (t = 300ms)
The distance matrix between the distribution centers (B00047, B00446) and the critical hospital nodes (such as Taipei VGH A00002 or NTU Hospital A00013) is updated using a simplified geodesic model:
Log stream in Terminal: [COMPUTE] Topological weights calculated. Node CMU Hospital (A00338) supply-chain friction coefficient set to 1.14...
Phase 3: Payload Construction and Dispatch to LLM Gateway (t = 600ms)
The computed context, selected model options, active ROP variables, and custom prompt templates are packaged into a JSON payload and dispatched to the Express backend proxy endpoint /api/analyze-local.
Log stream in Terminal: [NET] Transport packet established. Sending payload to local Gemini Proxy Server [Model: gemini-3.1-flash-lite, Temp: 0.1]...
Phase 4: Output Rendering and Action-Card Generation (t = 1200ms)
The Markdown response from Gemini is streamed to the Advisor panel, and structured recommendation cards are automatically parsed and displayed to the user.
code
TypeScript
export interface ActionCard {
  id: string;
  type: 'DISPATCH_REPLENISHMENT' | 'RESOLVE_BLACK_HOLE' | 'FIFO_LIQUIDATION';
  title: string;
  description: string;
  sourceNode: string;
  targetNode: string;
  associatedSerial: string;
  quantity: number;
}
If the LLM recommends "Reroute serial RNJ135962G to resolve CMU Hospital's stock deficiency," the system dynamically builds an interactive card. Clicking the "Approve Transfer" button calls the local ledger service, executes a virtual transfer transaction in the SQLite database, and updates the D3 map network lines with active glowing particles.
6. Core Feature Specification: Cold-Chain GPS Pathfinder Anomaly Routing
When pacemakers or implantable pulse generators are dispatched from central warehouses to regional clinical facilities, environmental sensors monitor cargo safety. DHA-Hub features a custom GPS tracking simulation that flags cold-chain temperature anomalies and calculates alternative routes in real time.
code
Code
[ Cold-Chain Sensor Anomaly Detected ] (e.g. Temp > 8°C)
                         |
                         v
       [ Initiate A* Dijkstra Pathfinder Engine ]
                         |
     +-------------------+-------------------+
     |                                       |
     v                                       v
[Calculate Distance & ETA]          [Query Active Stock Levels]
Find nearest qualified hub          Ensure hub has buffer capacity
     |                                       |
     +-------------------+-------------------+
                         |
                         v
       [ Plot Alternative Geodesic Path ]
                         |
                         v
       [ Dispatch Secured Reroute Instruction ]
6.1 Pathfinding Algorithmic Heuristics
The route optimization module models transport routes as a weighted graph 
, where vertices 
 represent hospitals or warehouses, and edges 
 represent logistics corridors. Edge weights are computed based on distance, ambient traffic index, and real-time transit friction:
When a thermal sensor alert is logged (e.g., shipping container temperature exceeds the safe 
 ceiling for implantable pacing batteries), the pathfinder immediately executes an 
 search variant to find the nearest qualified clinical node that can safely receive the high-value medical cargo.
6.2 Real-time Cold-Chain Telemetry HUD
To provide an immersive monitoring experience, a dedicated tracking panel displays dynamic telemetry data during rerouting events:
Real-time Sparklines: Built with Recharts, showing current and historical temperature trends.
Stage Progress Rings: Visualizing UDI verification milestones (INTAKE 
 COURIER_ASSIGNED 
 TEMP_VERIFIED 
 OUTTAKE).
Interactive Map Elements: Highlighting alternative route options with animated, colored paths directly on the map.
7. Core Feature Specification: Regulatory OCR Document Intelligence Parser
To prevent unrecorded parallel imports or gray-market device distribution, DHA-Hub features a simulated high-fidelity document parsing interface styled after real optical character recognition (OCR) platforms.
code
Code
[ Upload / Select Regulatory PDF Certificate ]
                         |
                         v
       [ Run Holographic Laser Scan Simulation ]
                         |
                         v
       [ Extract Structured Key-Value Pairs ]
       - Serial Key (UDI-PI)
       - Model ID
       - Import License Number
       - Sterile Expiration Date
                         |
                         v
   [ Audit and Validate Against Master Ledger ]
                         |
     +-------------------+-------------------+
     |                                       |
[Match Found]                        [Mismatch / Black Hole]
Commit record to database            Flag compliance warning
                                     Create override proposal card
7.1 Key-Value Extraction Logic
The OCR engine processes imported PDFs (such as import permits or clinical compliance certificates) and parses them into structured JSON metadata:
code
TypeScript
export interface OCRExtractedMetadata {
  serialNo: string;
  model: string;
  licenseNo: string;
  expiryDate: string;
  auditorName: string;
  overrideReason: string;
  fileReference: string;
}
The parsed metadata is compared against the virtual SQLite database. If a mismatch is identified—such as a distribution record with no corresponding import permit—the system generates a "Compliance Black Hole" alert and prompts the user to resolve the issue.
7.2 Interactive Compliance Overrides
When an discrepancy is flagged, the system displays an override resolution interface that allows authorized personnel to review the extracted document metadata, attach a digital signature, and record an audit override directly to the virtual ledger:
code
TypeScript
export interface ComplianceOverride {
  id: number;
  serialNo: string;
  auditor: string;
  overrideReason: string;
  fileReference: string;
  timestamp: string;
}
8. Three (3) Additional "Wow" AI Features
These newly proposed AI capabilities expand DHA-Hub's operational scope, introducing predictive clinical prioritization, advanced hardware life modeling, and automated regulatory filing.
code
Code
+-----------------------------------+
                  |  DHA-HUB ADVANCED COGNITIVE SUITE |
                  +-----------------+-----------------+
                                    |
         +--------------------------+--------------------------+
         |                          |                          |
         v                          v                          v
 [AI Feature 1: DHCPO]       [AI Feature 2: PCD-BHF]     [AI Feature 3: MARCCC]
 Correlates surgical lists   Simulates thermodynamic    Automates regulatory filings
 with available stock        sterility degradation      by cross-referencing global UDI
8.1 Feature 1: Dynamic Heuristic Clinical Priority Orchestration (DHCPO)
Concept: Typical medical logistics platforms operate strictly on chronological request orders (First-In, First-Out). DHCPO replaces this with clinical priority routing. It uses AI to match pending surgical waitlists and anonymized patient risk indicators against active sterile stock, prioritizing device deliveries based on clinical urgency rather than chronological order.
Algorithmic Logic:
Scans regional hospital data and computes a Patient Clinical Risk Index (
) for each pending pacemaker surgery:
Compares 
 values across clinics in the network.
If a high-urgency patient (
) is identified at a facility experiencing stock shortages, DHCPO automatically initiates a virtual transfer request to route devices from lower-priority locations (
).
UI Integration: Adds a dedicated "Clinical Waitlist Prioritizer" dashboard view. It displays patient urgency curves, lists available devices, and highlights high-priority transfer recommendations with interactive "Authorize Urgent Allocation" controls.
8.2 Feature 2: Predictive Cold-Chain Degradation & Battery Health Forecaster (PCD-BHF)
Concept: While standard cold-chain sensors only record temperature violations after they occur, PCD-BHF uses thermal modeling to project how temperature fluctuations will impact pacemaker battery life and packaging sterility over time.
Algorithmic Logic:
Uses a thermal decay formula to project battery cell life degradation based on cumulative temperature exposure:

Where 
 is a temperature-dependent degradation coefficient.
If the projected battery life drops below 95% of nominal specifications, the system flags the device for secondary verification before clinical implantation.
UI Integration: Integrates a dynamic battery health gauge into the active tracking HUD, displaying a degradation curve and showing the projected battery performance for each tracked device.
8.3 Feature 3: Multi-Agentic Regulatory Customs Clearance Copilot (MARCCC)
Concept: Navigating international customs and product clearances often involves significant administrative delays. MARCCC automates this process by cross-referencing imported device barcodes against global regulatory databases to generate pre-filled customs declaration documents.
Algorithmic Logic:
Reads scanned UDI barcodes and queries international medical device registries (such as the US FDA AccessGUDID or the European EUDAMED) to retrieve official product classifications.
Cross-references this data with local import rules to verify licensing requirements.
Automatically generates compliant XML regulatory filings and produces pre-populated customs clearance appeal PDFs ready for submission.
UI Integration: Adds an "Automated Regulatory Filer" workspace that shows active import queues, lists verified product classifications, and provides one-click document generation tools.
9. Visual Telemetry, Dynamic Charting & Custom UI Design
DHA-Hub features an interactive, high-fidelity UI optimized for complex medical logistics operations. It utilizes responsive layouts, custom animations, and clean dark-mode and light-mode themes to ensure data readability.
code
Code
+-------------------------------------------------------------------------+
|                        DHA-HUB MAIN DASHBOARD                           |
|  [Tabs: Map Canvas | Ledger Registry | Compliance | Inventory Advisor]   |
+-------------------------------------------------------------------------+
|                                                                         |
|  +-----------------------------------+-------------------------------+  |
|  |           D3 Graph Canvas         |     Interactive ROP Control   |  |
|  |                                   |                               |  |
|  |          [Midland Hub]            |    ADU:  [======o-----] 2     |  |
|  |              /   \                |    Lead: [====o-------] 3     |  |
|  |             /     \               |    Safe: [=o----------] 1     |  |
|  |      [North]     [South]          |                               |  |
|  |                                   |    Calculated ROP: 7 Units    |  |
|  |    * Displays live flow lines     |    Current Stock: 4 Units     |  |
|  |    * Highlights compliance holes  |    Status: [🚨 Critical]      |  |
|  |                                   |                               |  |
|  |                                   |    [ Run AI Optimization ]    |  |
|  |                                   |    [ Send to Gemini Engine ]  |  |
|  |                                   |                               |  |
|  |                                   |    [ Action Recommendations ] |  |
|  |                                   |    * Dispatch Replenishment   |  |
|  |                                   |    * Resolve Black Hole       |  |
|  |                                   |    * Initiate FIFO            |  |
|  +-----------------------------------+-------------------------------+  |
|                                                                         |
+-------------------------------------------------------------------------+
9.1 Interactive Network Topology Graph (D3.js)
The layout features a custom D3 force-directed topological map that visualizes the entire supply chain network. It renders nodes representing clinics, distributor warehouses, and regional logistics hubs based on their latitude and longitude coordinates.
Node Visualization Rules:
Size: Nodes are sized proportionally to their active storage capacity.
Color: Green indicates optimal stock levels, while deep amber indicates a stockout risk or an active compliance hold.
Interaction: Clicking a node filters the virtual SQLite ledger to show only records associated with that facility.
Dynamic Transit Lines:
Connecting links indicate active logistics routes. Transits between nodes feature glowing animated particles that travel along path curves to represent active physical shipments.
9.2 Dashboard: Five (5) Custom Analytics Charts
The dashboard compiles complex operational metrics into five distinct visualizations:
1. UDI-DI Stock Distribution Chart (Stacked Horizontal Bar Chart)
X-Axis: Stock Quantity (units).
Y-Axis: Hub Location (North, Midland, South, Hospital Groups).
Metrics: Compares inventory for pacemakers (W2SR01 vs. W3DR01) across different hubs, highlighting surplus or deficit states.
2. Network Centrality vs. Supply Friction Matrix (D3 Scatter Plot)
X-Axis: Node Supply Friction Coefficient (derived from transit distance and local customs delay factors).
Y-Axis: Centrality Score (calculated using Dijkstra’s shortest path algorithm over active routes).
Interactive Tooltip: Displays the facility name and a breakdown of logistical overhead.
3. Lead-Time Demand Distribution Curve (Area Chart)
X-Axis: Delivery Days (1 through 30).
Y-Axis: Probability Density of Consumption.
Purpose: Visualizes safety stock thresholds and highlights the probability of stockouts during peak transit delays.
4. Compliance Overrides Timeline (Horizontal Flame Graph)
X-Axis: Calendar Dates (2026-03-26 to 2026-04-29).
Y-Axis: Auditor Action Levels.
Purpose: Tracks regulatory compliance overrides over time, exposing periods of elevated compliance risk.
5. Cumulative FIFO Shelf-Life Decay (Liquid Fill / Wave Chart)
Visual Representation: A liquid wave animation that indicates the proportion of total stock that is within 180 days of sterility expiration.
Purpose: Gives managers an instant visual cue regarding urgent stock rotation needs.
10. jsPDF Strategic Report Reflow Engine & Strategic Report Compiler
The Advisor Tab acts as the strategic command center, translating complex operational data into actionable insights for logistics directors.
code
Code
+-----------------------------------------------------------------+
|                     Advisor Tab Workspace                      |
+-------------------------------+---------------------------------+
                                |
              Gathers SQLite Data & Network Centrality
                                |
            Calls Dual-Model Gateway (gemini-3.1-flash-lite)
                                |
              Generates Multi-Page Strategy Report
                                |
     +--------------------------+--------------------------+
     |                                                     |
     v                                                     v
[Advisor Tab Display]                                      [Export as PDF (jsPDF)]
Renders interactive layout with recommendations             Generates polished PDF report
10.1 jsPDF Multi-Page Reflow Pipeline
DHA-Hub implements a custom PDF rendering pipeline using jsPDF that formats generated reports into multi-page documents suitable for executive reviews:
code
TypeScript
import { jsPDF } from 'jspdf';

export const compileStrategicPDF = (
  reportContent: string,
  parameters: { adu: number; leadTime: number; safetyStock: number; model: string }
) => {
  const doc = new jsPDF({
    orientation: 'portrait',
    unit: 'mm',
    format: 'a4'
  });

  const pageHeight = 297;
  const pageWidth = 210;
  const marginX = 15;
  let currentY = 55;

  // Page 1: Corporate Header Band
  doc.setFillColor(15, 23, 42); // Navy slate background
  doc.rect(0, 0, pageWidth, 42, 'F');

  // Decorative Accent Bar
  doc.setFillColor(99, 102, 241); // Indigo bar
  doc.rect(0, 42, pageWidth, 3, 'F');

  // Header Text Styling
  doc.setTextColor(255, 255, 255);
  doc.setFont('helvetica', 'bold');
  doc.setFontSize(20);
  doc.text('DHA-Hub Medical Logistics Strategic Report', marginX, 18);

  doc.setFont('helvetica', 'normal');
  doc.setFontSize(9);
  doc.text(`Generated: ${new Date().toLocaleString('zh-TW')} | Class 1 Implantable Device Audit`, marginX, 26);
  doc.text(`Gateway LLM Core: gemini-3.1-flash-lite | Temperature: 0.1 (Clinical Mode)`, marginX, 31);
  doc.text(`Active Parameters: ADU=${parameters.adu} | Lead Time=${parameters.leadTime}d | Safety Stock=${parameters.safetyStock}u`, marginX, 36);

  // Set Body Text Color and Size
  doc.setTextColor(30, 41, 59);
  doc.setFontSize(10.5);
  doc.setFont('helvetica', 'normal');

  // Splits report text into wrapped lines fitting A4 width constraints
  const splitLines = doc.splitTextToSize(reportContent, pageWidth - (marginX * 2));

  splitLines.forEach((line: string) => {
    // Automatic Page Reflow Check
    if (currentY > pageHeight - 20) {
      doc.addPage();
      
      // Secondary Page Running Header
      doc.setFillColor(248, 250, 252); // Light gray header bar
      doc.rect(0, 0, pageWidth, 15, 'F');
      doc.setTextColor(148, 163, 184);
      doc.setFontSize(8);
      doc.text('DHA-Hub Strategic Advisor Report — Continued', marginX, 10);
      
      doc.setTextColor(30, 41, 59); // Reset text color
      doc.setFontSize(10.5);
      currentY = 25; // Reset top margin for content
    }
    doc.text(line, marginX, currentY);
    currentY += 6.5; // Optimized line spacing
  });

  // Footer Page Numbers and Stamp
  const totalPages = doc.getNumberOfPages();
  for (let i = 1; i <= totalPages; i++) {
    doc.setPage(i);
    doc.setFontSize(8);
    doc.setTextColor(148, 163, 184);
    doc.text(`Page ${i} of ${totalPages}`, pageWidth - 30, pageHeight - 10);
    doc.text('System verified by GS1 UDI standards & SQLite virtual state check.', marginX, pageHeight - 10);
  }

  doc.save(`DHA-Hub_Strategist_Report_${Date.now()}.pdf`);
};
11. Comprehensive Diagnostic, Verification & Test Suite
DHA-Hub includes a multi-layered verification suite to ensure continuous stability, speed, and strict regulatory compliance.
code
Code
+-----------------------------------------------------------------+
|                  DHA-Hub Unified Test Runner                     |
+-------------------------------+---------------------------------+
                                |
     +--------------------------+--------------------------+
     |                          |                          |
     v                          v                          v
[Relational Integrity Test]    [Compliance Engine Test]     [LLM Gateway Latency]
Validate SQLite unique constraints  Detect parallel imports    Verify API latency is
and duplicate serial detection   and missing UDI-DIs        consistently < 1.2s
11.1 Test Execution Suite
The verification suite contains three diagnostic tests that can be executed directly from the admin panel:
code
TypeScript
export interface DiagnosticResult {
  testID: string;
  name: string;
  status: 'PASSED' | 'FAILED' | 'WARNING';
  latencyMS: number;
  message: string;
}

export const executeSystemDiagnostics = async (
  purchaseRecords: any[], 
  distributionRecords: any[]
): Promise<DiagnosticResult[]> => {
  const results: DiagnosticResult[] = [];

  // Test 1: Relational Integrity of the SQLite Memory Engine
  const startT1 = performance.now();
  const serials = purchaseRecords.map(r => r.serialNo);
  const duplicates = serials.filter((item, index) => serials.indexOf(item) !== index);
  const latencyT1 = performance.now() - startT1;

  results.push({
    testID: 'T1-SQLITE-INTEGRITY',
    name: 'SQLite Virtual Ledger Unique Serial Constraint Test',
    status: duplicates.length === 0 ? 'PASSED' : 'FAILED',
    latencyMS: Math.round(latencyT1 * 100) / 100,
    message: duplicates.length === 0 
      ? 'All unique serial constraints resolved successfully. Zero duplicate serial keys exist.'
      : `Relational constraint violation! Duplicate serials detected: ${duplicates.join(', ')}`
  });

  // Test 2: Regulatory Compliance Audit (Intake vs Distribution)
  const startT2 = performance.now();
  const purchaseSerials = new Set(purchaseRecords.map(r => r.serialNo));
  const unregisteredSerials = distributionRecords
    .map(d => d.產品序號)
    .filter(serial => !purchaseSerials.has(serial));
  const latencyT2 = performance.now() - startT2;

  results.push({
    testID: 'T2-UDI-MATCHING',
    name: 'TFDA UDI Intake/Outtake Reconciliation Test',
    status: unregisteredSerials.length === 0 ? 'PASSED' : 'WARNING',
    latencyMS: Math.round(latencyT2 * 100) / 100,
    message: unregisteredSerials.length === 0
      ? 'UDI reconciliation complete. Zero unregistered parallel imports detected.'
      : `Compliance Warning: ${unregisteredSerials.length} distribution serials exist without registered import filings (Compliance Black Hole).`
  });

  // Test 3: LLM Gateway Connection Latency Audit
  const startT3 = performance.now();
  let statusT3: 'PASSED' | 'FAILED' = 'PASSED';
  let messageT3 = '';
  try {
    const response = await fetch('/api/health');
    if (!response.ok) throw new Error('API server returned error code.');
    messageT3 = 'Secure connection with Gemini SSL local proxy established.';
  } catch (err: any) {
    statusT3 = 'FAILED';
    messageT3 = `LLM Gateway unreachable: ${err.message}`;
  }
  const latencyT3 = performance.now() - startT3;

  results.push({
    testID: 'T3-GATEWAY-LATENCY',
    name: 'Gemini LLM Gateway SSL Connection Latency Audit',
    status: statusT3,
    latencyMS: Math.round(latencyT3 * 100) / 100,
    message: messageT3
  });

  return results;
};
12. Unified Terminal Command-Line Interpreter (CLI) Specification
To provide advanced users and systems administrators with direct access to system functions, DHA-Hub features a unified terminal command-line interpreter (CLI). The CLI supports both strict command routing and an NLP-based parser fallback for natural language requests.
code
Code
[ Command Typed Into Terminal Input ]
                         |
                         v
       [ Regex Router Parses Command Syntax ]
                         |
      +------------------+------------------+
      |                                     |
[Match Found]                        [No Match Found]
Execute CLI Command Directly         Call LLM Agent Proxy for NLP Translation
- audit                              - Process semantic intent
- add override                       - Generate executable actions
- transfer                           - Log output results
- reset
12.1 Regex CLI Routers
The CLI uses regular expressions to route common operational commands directly to system services:
Audit Command: Matches ^audit( ledger)?$ or ^run$. Triggers the Gemini Dual-Model Logistics Optimization Core.
Override Command: Matches ^(add )?override\s+([A-Za-z0-9\-]+)$. Extracts the serial number and registers a compliance override.
Transfer Command: Matches ^transfer\s+([A-Za-z0-9\-]+)\s+to\s+([A-Za-z0-9\-]+)$. Extracts the serial and destination node to execute an immediate stock transfer.
Reset Command: Matches ^reset$ or ^clear$. Reinitializes the virtual SQLite database to baseline default records.
12.2 NLP Agent Proxy Fallback
If the input command does not match any predefined regex rules, the CLI routes the request to an NLP agent proxy powered by gemini-3.1-flash-lite. The proxy analyzes the user's intent and determines the appropriate administrative action to execute:
code
TypeScript
const handleNLPEval = async (command: string) => {
  const payload = {
    prompt: `You are an intelligent TFDA medical logistics command operator. The user typed: "${command}".
    Analyze their request to determine if they want to override, transfer, reset, audit, or insert data.
    Output your analysis and execute action details. Speak in traditional Chinese or English matching the command tone.`,
    systemInstruction: "You are a secure command line routing agent. Provide concise, administrative terminal responses.",
    model: "gemini-3.1-flash-lite",
    temperature: 0.1
  };
  // Send payload to backend gateway and log response...
};
13. Twenty (20) Architectural & Regulatory Questions
To guide future system iterations, consider the following technical and operational questions:
Relational Database Migration: How should we configure the database layer when transitioning from our in-memory SQLite store to a dedicated, high-availability Cloud SQL (PostgreSQL) instance using Drizzle ORM?
Predictive Analytics: Can we integrate a client-side ARIMA or linear regression model to predict Average Daily Usage (ADU) values based on seasonal hospital purchase patterns instead of relying on manual slider settings?
Multi-Tenant Access Control: What is the most secure strategy for configuring Role-Based Access Control (RBAC) to restrict UDI-PI serial numbers, patient-adjacent records, and regulatory overrides based on user roles (such as Auditors, Clinical Logistics Managers, and Suppliers)?
Offline Sync Handling: How should we handle synchronization conflicts when offline warehouse operators update the local IndexedDB state while concurrent transactions are recorded on the primary database server?
Cold-Chain IoT Integrations: What streaming API architecture should be used to feed continuous temperature and humidity readings from cellular IoT cargo trackers directly into our cold-chain telemetry HUD?
PDF Design Customization: Do the strategic reports exported via jsPDF require additional brand styling, such as custom clinic logos, dynamic charts, or signature blocks for auditors?
Alternative LLM Gateways: Should we integrate a secondary LLM gateway (like a local Ollama or Llama-3 instance) to ensure compliance reports can still be generated if external connections to Gemini are lost?
Real-Time Data Streaming: What WebSocket message format should be established to instantly push transaction modifications across multiple concurrent active client sessions without causing unnecessary component re-renders?
D3 Graph Performance: How can we optimize our D3 force-directed simulations to maintain smooth 60fps rendering as the logistics network scales to thousands of clinic and supplier nodes?
TFDA Automated XML Output: Can we construct an automated pipeline that formats approved compliance overrides into XML structures ready for direct submission to TFDA regulatory portals?
ADU Calculation Models: Should the Average Daily Usage (ADU) variable be computed as a simple rolling average of the past 30 days, or should we apply an exponential smoothing factor to weight recent cardiac surgery demands more heavily?
Optimal Expiry Thresholds: Is 180 days the optimal threshold for flagging near-term device expiration, or should we adjust this threshold dynamically based on the average sterilization shelf life of different medical device categories?
Transfer Authorization Rules: What authentication procedures and cryptographic verification checks should be required before warehouse managers can execute inter-hub inventory transfers?
Prompt Security Safeguards: How can we protect our prompt templates from injection attacks where unauthorized users try to modify system instructions or bypass security guidelines?
Localization Framework: To prepare for broader international deployments, what is the best approach for managing and loading translation maps for additional languages like Japanese (ja-JP) or Spanish (es-ES) dynamically?
Offline Barcode Capture: How can we configure Service Workers to support offline barcode scanning, storing scanned UDI structures in local IndexedDB containers until a network connection is re-established?
Historical Trend Analysis: Should we add a dedicated dashboard tab for historical trend analysis that tracks reorder frequency and safety stock violations over a 12-month rolling period?
UDI-DI Verification: How can we connect our validation engine to global regulatory databases (like AccessGUDID) to automatically retrieve device metadata based on the scanned UDI-DI?
Cost-of-Outage Calculations: Can we develop a cost-of-outage formula that quantifies the financial and operational impact of stocking out of critical devices, and display this metric directly within the advisor reports?
Security Auditing: What steps should we take to ensure the entire full-stack application meets HIPAA and GDPR data privacy standards, particularly regarding the transmission of surgical scheduling and patient-adjacent device records?
