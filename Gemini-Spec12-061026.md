SmartMed 2.0 Agentic AI FDA 510(k) 智慧審查平台 — 完整生產級技術規格書
(SmartMed 2.0 Agentic AI FDA 510(k) Smart Review Platform - Technical Specification)
壹、 系統總體定位與架構 (System Positioning & High-Level Architecture)
1.1 系統定位 (Product Positioning)
SmartMed 2.0 是一款專為醫療器材（Medical Device）法規註冊工程師（RA）與醫療器材審查專家（Reviewer）設計的生產級 Agentic AI 輔助 FDA 510(k) 暨 TFDA 智慧審查平台。平台的整合重心在於簡化、加速並標準化醫材申報資料的編修與預審流程。
本系統摒棄了傳統單向黑盒子（Black-box）的分析方式，轉而構建一個人機協同工作流引擎（Human-in-the-Loop Agentic Workflow），支援對法規申報材料、自定義審查指南（Skill.md）、AI 審查提示詞（Prompt）以及生成的最終報告進行即時編輯、版本調校與雙向綁定。
1.2 技術堆疊與平台約束 (Technology Stack & Architecture Matrix)
本平台架構完全相容於底層高速伺服器與前端容器。以下為系統所採用的核心技術與組件標準：
前端框架 (Client-Side SPA)：基於 React 18+ (TypeScript) 與 Vite 6+ 構建。
樣式與色彩系統 (Styling & Aesthetics)：採用 Tailwind CSS 4+ 原生 CSS 變數（CSS Variables）系統，支持 10 種 Pantone 極致主題調色盤與 Light / Dark 模式的瞬間切換。
動畫與流動感 (Motion design)：運用 motion（自 motion/react 導入）進行多 Agent 推理階段、動態 Live Log 和雷達圖的平滑微互動。
資料視覺化 (Visualization & Flowchart)：
Recharts 2+：用於高流暢、具懸停（Hover）提示的預審風險雷達圖。
Mermaid.js 11+：用於動態組裝並高亮顯示 FDA 510(k) 決策樹。
後端服務與安全代理 (Server-Side Proxy)：Express 4+ 搭配 Node.js。後端建立 /api/analyze-stream 安全代理端點，確保 GEMINI_API_KEY 與瀏覽器完全隔離。
AI 推理核芯 (AI Orchestrator)：使用 @google/genai (2.4.0+) SDK，首選 gemini-3.5-flash（高性價比、流式速度極佳）作為核心推理模型，同時支援切換至 gemini-3.1-pro-preview 進行大規模複雜法規數據的比對。
1.3 伺服器端 Gemini API 安全整合規約
為保障資訊安全並配合平台規格：
Lazy Initialization（延遲初始化） 模式：僅在第一個請求（API Route）到達時初始化 GoogleGenAI 化身物件，防止因未配置 Key 而在伺服器啟動時崩潰。
SSE (Server-Sent Events) 串流技術：後端採用 generateContentStream 流式獲取 AI 生成詞塊，並封裝成 SSE data: {} 協議格式傳輸至前端，為前端提供即時打字與解密特效（Typing & Glitch Effects）。
貳、 核心功能模組細部規格 (Core Functional Modules Specification)
code
Code
[多格式申報材料及PDF] ──> [文件解析引擎] ──> [結構化 JSON / Grid 對位]
                                                 │
                                                 ▼
[自訂法規 Skill.md] ─────> [LLM 推理控制器] ───> [多 Agent 並行推理與 SSE 串流]
                                                 │
                                                 ▼
[3K-4K 報告編輯與下載] <── [動態雷達與 Mermaid 決策圖] <── [彙整報告與 RFI 清單]
2.1 申報材料解析與即時協同編輯模組 (Submission Materials Parser & WYSIWYG Editor)
提供了一站式、無阻礙的多格式檔案匯入介面，支援 JSON、CSV、TXT、MD 與 PDF 的本地解析及模擬加載。
多格式文檔流式加載 (Dossier Parser)：
前端集成 pdf.js 技術將 PDF 以 ArrayBuffer 方式分頁非同步流式讀取。
透過 Regex 規則與模式匹配，自動擷取 510(k) 常見的核心大綱（如 Indications for Use, Device Description, Substantial Equivalence (SE) 等段落），並提取表格內容。
動態特徵網格與 Markdown 編輯器 (Data Grid Interaction)：
JSON 與 CSV 格式會被自動轉譯為動態網格表格，其中每一列欄位皆為獨立的可點選式輸入組件（Input Fields），支援即時的增加與刪除。
純文字與 Markdown 檔則投射於高對比雙欄 markdown 編輯器內，系統使用 React 狀態機（State Machine）即時保存在材料卡片中所做的全部異動，並提供一鍵「Reset to Original」的原型還原按鈕。
2.2 審查技術指引 (Skill.md) 的自定義與前置修調模組 (Guidance Customization Panel)
未來的審查在展開前，必須讓機器對準申報醫材對應的「黃金標準（Gold Standard）」。
三維指南來源選擇：
模式一：載入預設之 510(k) ＆ TFDA 中度風險 SaMD 審查指引（含 IEC 62304 軟體風險與 ISO 10993 生物相容性重點）。
模式二：支援法規人員直接在大文本框中貼上、修改自 FDA 官網下載之新版特定醫材特殊控制（Special Controls）指引。
即時語法高亮 (Markdown Highlighting Editor)：
指南編輯器支援雙欄分屏（Split Screen）預覽。左側提供行號、段落與警告符高亮；右側則是套用 Tailwind 完美排版的即時成品，讓審查前可以快速進行法規條款裁切（裁除不適用條款以大幅節省 Prompt Token 佔用深度）。
運行配置面板 (Orchestration Settings)：
包含微調滑桿控制參數：Temperature（溫度係數，預設 0.2 以確保高專業度與極低幻覺）、Max Tokens 以及 Model Select 下拉選單。
2.3 3k-4k 字繁體中文高規格審查報告生成模組 (High-Volume Report Composition)
產出一份正式、嚴謹的 510(k) 智慧預審與補件指引報告是本平台的核心產物。
並行 Worker 與鏈式思考 (CoT-Parallel Worker Architecture)：
為在短時間內產出字數扎實（3,000-4,000 字）、格式漂亮、結構清晰的報告，底層採用分段並行生成技術：
Worker 1 (SE Agent)：比對主申報器材與 Predicate (等效品) 之間物理及臨床規格的核心差異（約 1,200 字）。
Worker 2 (Safety Agent)：審計 biocompatibility（生物相容性）、EMC 電氣安全、網路安全與臨床偏差 CER（約 1,200 字）。
Worker 3 (Executive Agent)：草擬行政總覽（Executive Summary）及補件改善 RFI 表格（約 1,000 字）。
雙向綁定編輯與富文本下載 (Rich Document Exporter)：
報告生成後，會直接回傳並渲染於右側的工作窗格。支持直接點擊段落進行打字編修，提供「一鍵複製 HTML」、「下載為高階 Markdown」以及「匯出為高解析 PDF（由專用 @media print 列印 CSS 優化外觀，自動加入頁碼編排）」。
參、 三項殺手級 AI Wow Feature (Three Additional WOW AI Features)
為了將原本平淡的法規流程拉升至高互動性的 AI 引導工作流（Wow Demonstration Experience），特別設計了以下三項具突破性體驗的特色模組：
3.1 殺手功能 A：AI Predicate Finder (智慧等效品自動推薦與對比矩陣)
功能描述：在 510(k) 申報中，最核心的一關在於找尋已上市的「Predicate Device (等效對照品)」，並證明 Substantial Equivalence。本功能利用 AI 動態提取使用者解析出來的醫材功能、工作原理（Mechanism of Action）、以及預期用途（Intended Use），在背景建立起一個結構化特徵向量，由 Gemini 在其知識庫（或模擬呼叫外部 FDA 510(k) 資料庫）中自動檢索、匹配，並自動推薦出 3 款最適合的已上市 predicate devices（包含其 510(k) 號碼、製造商與 SE 比對摘要）。
前端呈現：推薦完成後，AI 會自動在網頁上建立一個極為精緻、跨多欄位的「三方同屏對比矩陣卡片（SE Matrix Table）」，將使用者申報器材與這三台 predicate devices 之「技術特徵（Technical Characteristics）、材質、功耗、能效、操作頻率、預期臨床效能」進行 20 項細緻的逐條對比，並高亮顯示「具有潛在法規審查風險、需補件證明其未引發新的安全問題（Different technological characteristics raising different safety questions）」之欄位。
3.2 殺手功能 B：AI Regulatory Risk Radar (法規風險多維度即時雷達分析與補件機率預測器)
功能描述：基於使用者上傳的申報材料與審查指南，AI 會透過語意提煉、臨床數據驗證與法規完整度偵測，建構出一個「法規合規度多維度分數卡」（包含：臨床有效性 Evidence, 安全性安全性驗證 Safety, 軟體/電器資訊完整度 Cybersecurity/EMC, 指南符合率 Guideline Alignment, 以及技術文檔嚴謹度 Technical Context Quality 等五個維度）。
前端呈現：此時，前端會以高度流暢、具有動畫淡入效果的 Recharts 互動式雷達圖（Radar Chart）呈現。滑鼠懸停於頂點時，會顯示該維度的詳細扣分與加分項目（例如：雷達圖顯示 EMC/Cybersecurity 分數為 40分，滑鼠懸停時 AI 會彈出提示："警告：缺乏 IEC 60601-1-2 EMC 測試報告的測試波形數據與校準配置說明"）。同時，雷達圖核心會以大字體顯示由 AI 多變量推理模型所預測之「FDA RTA (Refuse to Accept) 拒收機率」與「AI 預測一次補件 (AI Expected AI Request Information AI-AI) 成功通關概率」，讓 RA 人員一目了然其註冊策略。
3.3 殺手功能 C：Interactive Regulatory Flowchart & Smart Gap Navigator (互動式審查心智流程圖與盲點導航)
功能描述：本功能將複雜漫長的 FDA 審查決策樹（由 FDA 510(k) 決策流程與使用指引所組成的樹狀圖）與使用者上傳的醫材資料進行實時語意綁定，藉助 Mermaid.js 自動渲染出一個流程圖。
前端呈現：該圖不是靜態圖，而是一個「高亮高動態生命週期鏈路」。在 AI 解析申報材料後，會將使用者器材在決策樹中的每一條「Yes/No」路徑進行判定，將被擊中的那條鏈路以 Pantone Palette 的高對比亮色（比如：Pantone Illuminating 或 Tangerine Tango）進行高亮流光動畫展示。而針對缺失最嚴重的 Gap（缺漏點），心智圖上會閃爍紅色呼吸燈圓點（Smart Gap Navigator）。當使用者點擊該紅色圓點時，系統會從右側滑出一個「Gap 補件急救箱抽屜面板（Emergency Remediation Drawer）」，由 AI 自動根據 Guideline 條文生成該 Gap 的「標準回覆模板與數據補充 SOP」，甚至能幫 RA 草擬一封「問詢 FDA (Pre-Sub) 的 1對1 標準法規信件」。
肆、 炫酷視覺特效、互動指標、即時日誌與 Dashboard 設計 (Visual Fidelity & Logs Terminal)
本系統使用 Bento Grid（非對稱式板塊佈局）將極致美學與工程實用性融合：
code
Code
+-----------------------------------------------------------------------------------------+
|                                    SmartMed 2.0 SYSTEM HUD                              |
+-----------------------------------------------------------------------------------------+
| [Pantone Select] [Theme Light/Dark]                        [Agent Run Command Button]   |
+-----------------------------------------------------------------------------------------+
| +----------------------------+ +----------------------------+ +-------------------------+ |
| |       Import Portal        | |     Risk Radar Chart       | |     Live Agent Logs     | |
| |  [Drop Folder / PDF Array] | |                            | |   (Scroll Lock Support) | |
| |                            | |         Clinical           | |                         | |
| |  * K001-Z-100 Device specs | |        /        \          | | [INFO] API Connect OK   | |
| |  * Cytotoxicity Test PDF   | |      Bio         EMC       | | [OK] Extracted Spec 0.42| |
| |  * Cybersecurity doc       | |       \        /           | | [WARN] High TI > 1.5    | |
| |                            | |         Cyber              | | [STREAM] Composing SE   | |
| +----------------------------+ +----------------------------+ +-------------------------+ |
+-----------------------------------------------------------------------------------------+
4.1 AI 運行流體眩光特效 (Flowing Energy Particles)
當使用者點擊「執行智慧預審」時，主要運算卡片與按鈕外圍會產生一圈由 motion 驅動、呈波浪形擴散的「Pantone 智慧發光流動粒子場（Sparkle Trails）」。
當背景中的 Agent 完成階段大綱分析時，粒子會自「指南組件面板」沿著預設通道動態飛入「報告編輯器」，在終點爆發出微型光效，營造 AI 流體傳輸資料的強烈科技感。
4.2 互動數據指標健康度 (Compliance Thermometer)
於核心 Dashboard 搭載「法規健康水銀度溫度計（Compliance Health Gauge）」。
當使用者在左側編輯器修改不合規的器材常數（例如：將探頭最大頻率由違規的 1.5 MHz 修改回安全合規的 12.0 MHz）時，右側溫度計柱狀圖會以波浪彈跳動畫向上攀升，柱形顏色同時由違規警告的亮橙紅（Pantone Tangerine Tango）平滑漸變為安全合規的經典藍（Pantone Classic Blue）。
4.3 智慧審查實時日誌系統 (Live Agent Terminal)
底部或右下方設有高透明磨砂玻璃質感（Glassmorphism）的「Agent 執行日誌終端儀表板」，採用專屬等寬字型 'JetBrains Mono'。
雙向日誌語法與高亮 Badges：
以 [INFO]、[SUCCESS]、[WARNING ⚠]、[AGENT-COGNITIVE] 等微型彩色標籤展示系統執行進度。
日誌具備手動滾動監測機制：當用戶向上滾動查看歷史日誌時，控制台自動中斷鎖定（Scroll Lock），停止強制置底，並展示提示小微章，直到使用者再次置底或滑鼠移開。
伍、 光/暗主題與 10 種 Pantone 極致配色體系 (Color Psychology Specification)
SmartMed 2.0 內建 10 種代表不同專業氣場、科技感與情緒引導的 Pantone 配色方案，並在大框架上提供一鍵切換 Light（預設亮色保護眼睛，以米白搭配深卡其紙張質感為基底）/ Dark（適用於RA長夜加班，防藍光護眼高對比）的全體系主題。
不論在何種 Pantone 方案，我們皆嚴格遵循 W3C WCAG 2.1 AA 級對比度標準，文字段落對比度皆大於 4.5:1，標題大於 3:1。
#	Pantone 美學色彩	代表色彩隱喻	設計氣質與視覺核心
01	Classic Blue (經典藍)	嚴謹安穩與權威品質	作為官方 FDA 正式審查報告展示的預設美學色彩。
02	Ultimate Gray (終極灰)	精密岩石灰色調冷靜	象徵高合金精密醫療器具鋼、極簡現代化工業包浩斯風格。
03	Illuminating (亮麗黃)	法規曙光與科技突破	與終極灰聯袂，黃色高亮所有的合規漏洞，帶來明敏智慧。
04	Ultra Violet (紫外線)	AI 智慧算力與未來極限	螢光紫外發光字、深 indigo 邃紫背景，適合高科技感展演。
05	Greenery (草木綠)	生物相容性之生態健康	代表人體無毒、細胞活力，採用高對比森林草綠與象牙柔白。
06	Living Coral (珊瑚粉)	臨床熱度與溫暖人性生命	高雅珊瑚橙與玫瑰粉砂色，降低眼部壓力、重視溫和氣氛。
07	Marsala (瑪薩拉酒紅)	高階主管主管決策	沉穩、赤褐紅酒色系，為卡片增加厚重皮革框線，顯得尊貴。
08	Turquoise (綠松石綠)	無菌清爽之受信賴醫學度	蔚藍綠配磨砂消偏，視覺具高受信賴度與心血管醫療安定感。
09	Tangerine Tango (探戈橘)	高速運作與警示火眼金睛	激情橙紅，專門用來高亮提示缺漏的資料，適合主動分析時。
10	Black Onyx (墨黑 Onyx)	暗黑科技極簡主義 (Brutal)	純黑卡片與螢光亮白邊，極致對比，摒棄陰影，展現純粹技術面。
陸、 系統全域 CSS 變數定義 (CSS Variables Specification)
本套 10 種配色變數透過 HTML 屬性 data-palette 與 data-theme 在 React 中與底層主載體實現無縫耦合：
code
CSS
/* ==========================================================================
   SmartMed 2.0 Pantone Light Mode Styling Library
   ========================================================================== */
:root[data-theme="light"] {
  /* 01. Classic Blue (Default) */
  --bg-cb-light: #F4F6F9;
  --card-cb-light: #FFFFFF;
  --border-cb-light: #D2D7E0;
  --text-cb-light: #1A2E40;
  --color-primary-cb-light: #003366; 
  --color-secondary-cb-light: #336699;
  --color-accent-cb-light: #FF6600;

  /* 02. Ultimate Gray */
  --bg-ug-light: #F2F2F2;
  --card-ug-light: #FFFFFF;
  --border-ug-light: #D9D9D9;
  --text-ug-light: #333333;
  --color-primary-ug-light: #939597; 
  --color-secondary-ug-light: #5A5C5E;
  --color-accent-ug-light: #E0A926;

  /* 03. Illuminating */
  --bg-il-light: #F9FAF2;
  --card-il-light: #FFFFFF;
  --border-il-light: #EBECC5;
  --text-il-light: #2C2D13;
  --color-primary-il-light: #F5DF4D; 
  --color-secondary-il-light: #BFAB21;
  --color-accent-il-light: #2D2D2D;

  /* 04. Ultra Violet */
  --bg-uv-light: #F5F3FA;
  --card-uv-light: #FFFFFF;
  --border-uv-light: #DFD7ED;
  --text-uv-light: #2D1A3B;
  --color-primary-uv-light: #5F4B8B; 
  --color-secondary-uv-light: #8E77B8;
  --color-accent-uv-light: #FF2E93;

  /* 05. Greenery */
  --bg-gr-light: #F2F6F1;
  --card-gr-light: #FFFFFF;
  --border-gr-light: #D3E2D0;
  --text-gr-light: #1E2D1A;
  --color-primary-gr-light: #88B04B; 
  --color-secondary-gr-light: #5E8229;
  --color-accent-gr-light: #D65A31;

  /* 06. Living Coral */
  --bg-lc-light: #FAF4F2;
  --card-lc-light: #FFFFFF;
  --border-lc-light: #EDE0DB;
  --text-lc-light: #4A251B;
  --color-primary-lc-light: #FF6F61; 
  --color-secondary-lc-light: #C94C3F;
  --color-accent-lc-light: #3D8B84;

  /* 07. Marsala */
  --bg-ma-light: #F7F1F2;
  --card-ma-light: #FFFFFF;
  --border-ma-light: #E3D2D4;
  --text-ma-light: #3B1B1C;
  --color-primary-ma-light: #964F4C; 
  --color-secondary-ma-light: #6E302E;
  --color-accent-ma-light: #486E78;

  /* 08. Turquoise */
  --bg-tq-light: #EEF7F7;
  --card-tq-light: #FFFFFF;
  --border-tq-light: #CCE2E2;
  --text-tq-light: #0B3333;
  --color-primary-tq-light: #45B8AC; 
  --color-secondary-tq-light: #1F8c84;
  --color-accent-tq-light: #D9534F;

  /* 09. Tangerine Tango */
  --bg-tt-light: #FAF2EF;
  --card-tt-light: #FFFFFF;
  --border-tt-light: #EDE0DB;
  --text-tt-light: #45170B;
  --color-primary-tt-light: #DD4124; 
  --color-secondary-tt-light: #A82410;
  --color-accent-tt-light: #2A4B7C;

  /* 10. Black Onyx */
  --bg-bo-light: #F5F5F5;
  --card-bo-light: #FFFFFF;
  --border-bo-light: #E3E3E3;
  --text-bo-light: #121212;
  --color-primary-bo-light: #2B2C2C; 
  --color-secondary-bo-light: #525353;
  --color-accent-bo-light: #FF3B30;
}

/* ==========================================================================
   SmartMed 2.0 Pantone Dark Mode Styling Library
   ========================================================================== */
:root[data-theme="dark"] {
  /* 01. Classic Blue (Default) */
  --bg-cb-dark: #091223;
  --card-cb-dark: #101F3C;
  --border-cb-dark: #1E345E;
  --text-cb-dark: #ECEEF2;
  --color-primary-cb-dark: #1D549E; 
  --color-secondary-cb-dark: #4189DF;
  --color-accent-cb-dark: #FFA340;

  /* 02. Ultimate Gray */
  --bg-ug-dark: #181919;
  --card-ug-dark: #262728;
  --border-ug-dark: #3B3C3E;
  --text-ug-dark: #EDEDED;
  --color-primary-ug-dark: #ADB3B5; 
  --color-secondary-ug-dark: #D4D8D9;
  --color-accent-ug-dark: #F0CE4A;

  /* 03. Illuminating */
  --bg-il-dark: #1B1E05;
  --card-il-dark: #2A2E0F;
  --border-il-dark: #484E1F;
  --text-il-dark: #EBF2A6;
  --color-primary-il-dark: #FFE866; 
  --color-secondary-il-dark: #FFCA45;
  --color-accent-il-dark: #FFFFFF;

  /* 04. Ultra Violet */
  --bg-uv-dark: #160B29;
  --card-uv-dark: #231245;
  --border-uv-dark: #3F237A;
  --text-uv-dark: #EDE5F7;
  --color-primary-uv-dark: #9175C4; 
  --color-secondary-uv-dark: #B99AF2;
  --color-accent-uv-dark: #FFA6FF;

  /* 05. Greenery */
  --bg-gr-dark: #0D1609;
  --card-gr-dark: #1A2B14;
  --border-gr-dark: #304E27;
  --text-gr-dark: #EDFBEC;
  --color-primary-gr-dark: #AED565; 
  --color-secondary-gr-dark: #D4FC9C;
  --color-accent-gr-dark: #FF8552;

  /* 06. Living Coral */
  --bg-lc-dark: #240F0A;
  --card-lc-dark: #3E1B15;
  --border-lc-dark: #662E25;
  --text-lc-dark: #FBECEB;
  --color-primary-lc-dark: #FFA396; 
  --color-secondary-lc-dark: #FFC0B5;
  --color-accent-lc-dark: #7FE8DF;

  /* 07. Marsala */
  --bg-ma-dark: #1D0C0D;
  --card-ma-dark: #351618;
  --border-ma-dark: #582B2E;
  --text-ma-dark: #FAEDED;
  --color-primary-ma-dark: #C27471; 
  --color-secondary-ma-dark: #E6A3A1;
  --color-accent-ma-dark: #71AFBF;

  /* 08. Turquoise */
  --bg-tq-dark: #051A18;
  --card-tq-dark: #0B3330;
  --border-tq-dark: #145952;
  --text-tq-dark: #EBFCF9;
  --color-primary-tq-dark: #6ADCBD; 
  --color-secondary-tq-dark: #99FFDF;
  --color-accent-tq-dark: #FF8080;

  /* 09. Tangerine Tango */
  --bg-tt-dark: #240801;
  --card-tt-dark: #400F05;
  --border-tt-dark: #6C1E11;
  --text-tt-dark: #FBECEB;
  --color-primary-tt-dark: #FF7056; 
  --color-secondary-tt-dark: #FFA896;
  --color-accent-tt-dark: #568AFF;

  /* 10. Black Onyx */
  --bg-bo-dark: #000000;
  --card-bo-dark: #121212;
  --border-bo-dark: #333333;
  --text-bo-dark: #FFFFFF;
  --color-primary-bo-dark: #E2E2E2; 
  --color-secondary-bo-dark: #A6A6A6;
  --color-accent-bo-dark: #FF453A;
}
柒、 系統關鍵資料模型與 Type 宣告 (System Data Architecture)
前端模組化與與後端進行資料調用時，依據本技術規格書標準實作之 src/types.ts 定義如下：
code
TypeScript
export enum PantonePalette {
  CLASSIC_BLUE = "classic-blue",
  ULTIMATE_GRAY = "ultimate-gray",
  ILLUMINATING = "illuminating",
  ULTRA_VIOLET = "ultra-violet",
  GREENERY = "greenery",
  LIVING_CORAL = "living-coral",
  MARSALA = "marsala",
  TURQUOISE = "turquoise",
  TANGERINE_TANGO = "tangerine-tango",
  BLACK_ONYX = "black-onyx"
}

export enum ThemeMode {
  LIGHT = "light",
  DARK = "dark"
}

export interface MaterialFile {
  id: string;
  name: string;
  size: number;
  mimeType: string;
  parsedContent: string; // PDF 解析出的純文字
  uploadedAt: Date;
}

export interface StructuredMaterial {
  caseId: string;
  deviceName: string;
  manufacturer: string;
  classificationCode: string;
  resolution: string;
  frequency: string;
  safetyStandard: string;
  biocompatibility: string;
  cyberSecurity: string;
}

export interface LiveLog {
  id: string;
  timestamp: string;
  level: "info" | "success" | "warning" | "error";
  message: string;
}

export interface DeficiencyItem {
  id: string;
  severity: "critical" | "warning" | "recommendation";
  section: string;
  finding: string;
  remediationSop: string; // 補件行動指南
}

export interface RadarData {
  axis: string;      // 例如 "臨床試驗(Clinical)", "資安防護(Cybersecurity)"
  score: number;     // 0 - 100
  detail?: string;
}

export interface PredicateDevice {
  kNumber: string;
  manufacturer: string;
  deviceName: string;
  isEquivalent: boolean;
  differentialSummary: string;
}
捌、 多 Agent 推理協同與報告生成工作流詳解與防幻覺機制
當點擊「執行智慧預審 (Execute Review)」時，後端 AI 系統將以下列 多階層 Agentិក 工作流 (Multi-Agentic Lifecycle) 循序並行非同步展開。此設計能自主分配推理強度、降低單點 LLM 錯誤幻覺（Hallucination），並最大化生成 3000-4000 字繁體中文高階報告：
code
Code
[開始預審]
                                  │
                                  ▼
                     [臨床/技術數據提煉 Agent]
                                  │
                                  ▼
                     [法規對比與 Deficiencies 偵測 Agent]
                                  │
                                  ▼
      +───────────────────────────┴───────────────────────────+
      │                                                       │
      ▼                                                       ▼
[SE 分析並行 Agent]                                   [安全測試審計並行 Agent]
(Substantial Equivalence)                               (Biocompatibility, EMC)
      │                                                       │
      +───────────────────────────┬───────────────────────────+
                                  │
                                  ▼
                     [報告格式總校與審核 Agent]
                     - 3000-4000字 繁體中文 Markdown 報告產出
                     - 決策 Mermaid 流水圖
                     - Recharts 五維雷達圖
8.1 第一階段：臨床技術參數提煉 (Extraction Agent)
職責：提取輸入之多维格式材料，過濾雜音。
安全保障（防幻覺）：對於提取出的每個關鍵參數（例如：resolution、frequency），Agent 必須在回傳 JSON 中強行檢附 Source Citation（定位字句或文件頁碼）。
若原文中無該項目，則標註為 "NOT_FOUND"，杜絕 LLM 加載預訓練權重自主猜測、捏造未申報數據的漏洞。
8.2 第二階段：法規標準對比與缺失比審計 (Analysis Agent)
職責：將第一階段參數，對齊用戶修改好的 Skill.md 法規條文進行逻辑推導。
比對實例：若 Skill.md 明定「探頭最大熱指數（TI）應控制在 1.5 以下，超出必須加入鎖定機制」，而材料顯示「設備極限情況下 TI 達 1.8 且缺乏鎖定記錄」，Analysis Agent 將主動捕獲此 Gap，生成一筆等級為 critical 的不合規清單（DeficiencyItem），包含一項詳細的補件 SOP 指南。
3.3 第三階段：分模組並行高容量合成 (Generator Worker Chain)
為繞過 LLM 單次最大 Output Tokens 上限（避免報告段落崩坍、過早截斷、格式缺漏），後端 Express 代理端點會建立並發多 Worker 機制：
Worker A 專寫行政大綱與前言（約 800 字）。
Worker B 專攻 Substantial Equivalence（等效品對比，約 1,200 字）。
Worker C 審計非臨床安全與臨床對照組偏差檢索（約 1,000 字）。
Worker D 構建全面缺陷清單（RFI）及仿單警告事項（約 1,000 字）。
3.4 第四階段：繁體中文（Taiwan Standard）總校正與拼裝 (Consolidation Agent)
加載最後的「總編輯 Prompt」，將 A, B, C, D 的片段串聯，擦除重複段落，消除拼接處的語法生硬感。
強制套用標準 TFDA 法律與臨床文獻術語規範（例如：將 “Predicate Device” 統譯為 “對照組” 或 “實質等同對照器材”，而 “Indicators” 譯為 “指標”），輸出最終 3k-4k 字 Markdown。
玖、 平台效能與非功能性規格 (Performance & Security Policies)
首字響應延遲 (Time to First Token - TTFT)：
使用流式 Server-Sent Events（SSE）通道，在啟動預審後 500 毫秒 (0.5s) 內，前端 Terminal 日誌控制台必須滾動出第一行真實 Agent 運算進度，消除使用者的等待焦慮。
時效管理 (Generation Constraint Time)：
整份 3000-4000 字的高精細繁體中文預審審報，整體並行合成與校正輸出應控制在 12 秒 入庫，過程中不得有 broken markdown 標籤或結構傾斜。
無落盤隱私保護 (Stateless Zero-Retention Trust)：
系統不儲存用戶上傳的任何智慧財產 PDF 或 CSV 原檔，全部參數皆在內存（RAM）快取中流動。用戶關閉 session 30 分鐘後，數據會被主動進行垃圾回收（Garbage Collected），確保對第三方零外洩风险。
回應式佈局與觸控範圍 (Responsive Density)：
桌面端採用 100vh 覆蓋之三欄 Bento Grid 排版，提供獨立溢出捲軸（Overflow Scroll）。而當視窗縮小至行動端時，系統自動啟用置頂的 Tabs 控制條（包含材料、日誌、雷達、報告），並保證全部操作按鍵大於 44px 觸碰目標。
拾、 二十個醫療器材審查平台關鍵痛點追蹤問題 (20 Tracking Questions for FDA Review Platform)
在進入後續產品化以及多輪演入部署階段時，設計、開發以及法規專家團隊必須針對以下 20 個涉及系統瓶頸、安全性與體驗層面的痛點問題 進行深度的技術考量：
10.1 架構與系統性能層面 (Technical & Performance Constraints)
問題 1：當用戶上傳的單個 PDF 申報文件高達 150 頁、檔案大小超過 80MB 時，純前端的 pdf.js 解析是否會造成瀏覽器 UI 主執行緒（Main Thread）當機？我們應如何引入並配置 Web Worker 以執行背景非同步 PDF 的文本提取與分流？
問題 2：在 Express 串流代理模式下，若遇到 API 網絡回響緩慢或 LLM 推理深度極大，整體 Stream 超過 3 分鐘，如何配置防禦超時（Keep-alive Ping），避免部署於 Cloud Run 容器或 Nginx 層的 60 秒預設超時機制強制切斷 HTTPS 連線？
問題 3：如何在完全無資料庫（Client-side local state / LocalStorage）的限制架構下，實現完美且安全的「案件版本追蹤與管理」？當用戶清空瀏覽器快取時，如何提供一鍵導出完整加密規格（.json dossier）的下載與還原機制？
問題 4：面對結構各不相同的 CSV 物理實驗數據，前端 Dynamic Data Grid 如何自主偵測欄位對齊、對特定編碼溢出（如 UTF-8 BOM）進行智慧容錯，並在網格列數大於 1,000 時依然達成 60fps 的超流暢互動？
問題 5：當用戶在 10 種 Pantone 調色盤之間高頻率切換時，前端底層 CSS 變數在 canvas 與 SVG 多次重繪時，如何優化渲染效能、降低 GPU 開銷，以消除任何可能發生的轉場微幅卡頓？
10.2 AI 法規推理、符合性與防幻覺層面 (Clinical Reasoning & Anti-Hallucination)
問題 6：合規健康雷達圖（Risk Radar）的五個百分比維度分數（0-100%），是由 LLM 隨機生成的感性值，還是有具體的「法規扣分扣牌公式」在 Prompt 及後端 Agent 架構中獲得嚴格、可控的公式計算支持？
問題 7：當系統為用戶推薦出 3 款 510(k) 實質等同對照品（Predicate Devices）時，若推薦結果不盡人意，AI 代理能否主動顯示其推薦的理據（例如關鍵字權重、使用目的、相近物理結構），並允許用戶手動編輯並儲存這項 SE 矩陣特徵？
問題 8：當 FDA 或 TFDA 在今年公布了新的 SaMD 指引（例如新增 AI 資料分組盲性標註防偏移要求），我們的系統該如何實現一鍵式「法規庫在線熱更新」？是否能開發一個訂閱 FDA RSS 新聞源並動態刷新 Skill.md 源起部分的微服務？
問題 9：面對縮寫相同、但醫學邏輯完全相異的醫學名詞（如 RF 在不同報告中可能指射 "Radio Frequency 射頻" 或 "Rheumatoid Factor 類風濕因子"），我們該如何通過 SystemInstruction 與 Context Guard，防止 AI 在流式生成長篇報告時發生致命名詞混淆与邏輯翻車？
問題 10：如何完全防範 AI 產生「幽靈不合規點（Hallucinated Deficiency）」— 即 AI 為了勉強湊足 3500 字，在報告中憑空虛擬、偽編了申報材料內「從未出現過」的測試數據與缺失點？我們該設計何種 Verification Step（二次自我審核 Worker）來核實 RFI 清單的真偽？
10.3 前端視覺美學、細節與 Wow 特效層面 (Visual Design & Human factors)
問題 11：在 “Black Onyx (黑瑪瑙)” 或是 “Ultra Violet (紫外光)” 這種高飽和、深邃暗黑的主題下，如何精確調整 'JetBrains Mono' 代碼控制台中的警告（橘色）與成功（綠色）字體亮度，以完美落實 WCAG 2.1 AAA 級最高無障礙對比標準？
問題 12：Mermaid.js 預設不支援主題色彩與外圍 CSS 變數的主動響應。我們應如何在每次切換 Pantone 主題時，動態讀取當前 CSS 變數顏色，並將其動態覆寫進 Mermaid 圖示的「節點渲染 API (render class definition)」中，使高亮路徑顏色與調色盤色系渾然一體？
問題 13：在 LLM 長文串流（Stream UTF-8 Word-chunks）生成時，Live Terminal 控制台如何精確辨識用戶手動向上滑起的動作，以「優雅暫停強制置底滾動、顯示置底鎖定提示」，並在用戶滑鼠移開或回滾時無縫恢復「自滾動」？
問題 14：當生成的 Mermaid 審查心智流程圖、決策樹分支極其繁茂時，前端 Bento Grid 容器該如何提供優雅的「SVG 動態滑鼠縮放、平移（Pan & Zoom）」機能，以完全防止圖示超出邊框、被卡片切邊的跑版悲劇？
問題 15：在合規防護罩溫度計與通關預測率等 Widget 中，百分比數字從 30% 動態爬升至 85% 時，如何優化 React 渲染迴圈、避免多餘的 React Component Tree Re-renders，以極致流暢地完成流式進度增長動畫？
10.4 臨床安全、資安風險與上市合規層面 (Safety, Security & Validation)
問題 16：如果用戶直接採信了本 AI 預審平台生成的「標準補件 SOP 信件模板（RFI Remediation Draft）」並遞交給 FDA 審查官，卻因 AI 推理中的隱蔽邏輯缺失遭致拒絕申報，我們在 UI / 報告底端應如何設計卓越的「RA 法規法律免責告知（Legal Advisory Disclaimer）」？
問題 17：是否為使用者任意貼入文字的 Ingesting Panel 整合防注入檢驗（Prompt Injection Guardrail）？如何防範有毒的 CSS、HTML、或惡意 JavaScript 程式碼被偽裝成 XML/JSON 形式輸入，從而對平台造成 XSS 跨站指令碼與底層 LLM 的資安劫持？
問題 18：當這套智慧審查平台交付給視力不佳、或色弱的資深醫材審報專家使用時，10 種 Pantone 配色搭配 UI 的 Bento Grid 佈局是否能在系統字體比例放大 150-200% 時，提供精美流暢地自動網格折行折角，而完全不起任何重疊？
問題 19：在生成 3k-4k 字 Markdown 報告並列印為實體紙張時，我們如何在 @media print 列印媒體集中，將鮮艷的 Pantone 卡片背景、發光邊框在 100 毫秒內平滑抹除、對齊轉印為白底、極致高灰度（High-contrast Gray）、清晰實線黑字的「環境友好（Eco-Print Grayscale）」格式？
問題 20：當面對極為重大的多中心 RCT 臨床試驗數據表（包含 thousands of patient details），AI 在進行第五章“臨床偏差與偏見分析（Clinical Trial Bias Check）”時，如何設計 Prompt 的數據滑動窗口（Sliding Context Window），在兼顧模型 Input 限流的同時，依舊精準嗅探檢索到跨年齡、性別分組信賴區間的偏差不一致點？
結語：本規格書旨在成為 SmartMed 2.0 系統步入最完整生產線研製的技術燈塔。規格不僅在美學（Pantone 體系）、AI 智導控制以及工程防護上設定了極高極細緻的標準，也為未來的研發演化與邊界防禦定下了堅實的基底。
