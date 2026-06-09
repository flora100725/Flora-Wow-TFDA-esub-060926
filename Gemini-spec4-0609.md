醫療器材查驗登記 AI 審查系統技術規格書
—— TFDA 衛生福利部食品藥物管理署初審專家代理系統 (TFDA Primary Expert Agent)
本技術規格書旨在深度剖析「醫療器材查驗登記 AI 審查系統」的系統架構、技術細節、AI 代理人工作流、核心 API 機制，並提出三項具備「WOW 效應」的下一代多模態人工智慧功能，最後以 20 個深度專業的後續追蹤問題作為系統演進之指引。
1. 系統設計藍圖與核心理念
本系統旨在解決 TFDA（Taiwan Food and Drug Administration，臺灣衛生福利部食品藥物管理署）在醫療器材第一等級查驗登記（Class I Medical Device Registration）初審過程中面臨的高重複性、法規判定標準繁雜、材料缺失率高之核心痛點。
1.1 核心痛點解決方案
在傳統審查流程中，承辦人員（審查員）需花費大量時間核對紙本或 PDF 檔案內文，而最常發生退件的缺失包括：
分類品項代碼與器材宣稱功能錯配（Classification & Intended Use Mismatch）：例如，本案廠商之中文名稱宣稱為「輪椅」，但其申請分類卻勾選「機械式助行器（O.3825）」，兩者在法規鑑別範圍（編碼 O.3825 與 O.3850）上存在本質之衝突。
作廢或無效文件的使用：廠商因版本控制不佳，上傳了蓋有「作廢」戳記的製造業藥商許可執照影本，依程序必須退審補件。
測試報告欄位錯置（Misplaced Technical Reports）：非帶電手動器械毋須檢附電性安全測試，但廠商常將機械力學測試（如 CNS 15037-2、ISO 11199-2）誤上傳至電性安全（IEC 60601-1）之欄位。
本系統透過雙重代理人架構（雙網/雙引擎代理機制），在前端提供即時的交互式響應、在後端建構嚴謹的 Gemini 2.x/3.x 推理流水線，直接在半秒內定位缺失，並提供可視化的高亮定位引導。
2. 系統架構與數據整合流水線
系統採用 Full-Stack（全棧）架構，前端基於 React 19 + Tailwind CSS 4 + Motion 進行高畫質的流暢交互渲染，後端則基於高併發的 Express 服務器，並深度整合了 @google/genai TypeScript SDK 以實現隱私與效能兼顧的伺服器端（Server-Side）LLM 推理。
code
Code
[前端：React 19 SPA]
                                         │
        ┌────────────────────────────────┴────────────────────────────────┐
        ▼                                                                 ▼
[多模態拖入/黏貼解析 UI]                                                 [交互式修復/可視化高亮面板]
  - 支援 PDF Base64、HTML 黏貼                                             - 點擊 Alert 自動追焦 Form 欄位
        │                                                                 ▲
        └────────────────────────────────┬────────────────────────────────┘
                                         │ JSON Payload (Base64 + Plaintext)
                                         ▼
                                [後端：Express 伺服器]
                             (Port 3000 / 0.0.0.0 容器部署)
                                         │
                 ┌───────────────────────┴───────────────────────┐
                 ▼                                               ▼
         [/api/transform]                                  [/api/review]
  (原始輸入結構化為 SubmissionMaterials型別)               (輸入 materials + guideline)
                 │                                               │
                 ├───────────────────────────────────────────────┤
                 ▼                                               ▼
      [Gemini 產品系列服務]                              [Gemini 推理決策引擎]
  - 溫度：0.1 (確保表格轉換極致準確)                     - 溫度：temperature 變數 (動態控制)
  - 系統 Role: 結構化語意解析大師                         - 系統 Role: TFDA 資深法規審查組長
                 └───────────────────────┬───────────────────────┘
                                         │
                                         ▼
                             [生成 Traditional Chinese 報告]
                             [更新 Dashboard 審查指標數據]
2.1 雙 API 接口技術細節
A. 結構化轉換接口：POST /api/transform
本接口的核心職責是將非結構化（Unstructured）的混合文本、網頁 HTML 拷貝、或上傳之 PDF Base64 檔案，轉換為符合系統嚴格 TS 類型宣告的 SubmissionMaterials。為了確保絕對的提取精度，此處將 Gemini 的調用參數 temperature 設為接近 0.1，並注入強制的 JSON 格式限制（JSON Schema constraint）。
B. 智慧合規稽核接口：POST /api/review
本接口是整個 Agentic System 的「法規大腦」。它同時引入三個維度的輸入：
materials（申報材料實體）：由 /api/transform 生成或由審查員在前端動態核對修正後的結構化數據。
guideline（TFDA 評估法規指引）：可自定義並進行多輪覆寫的法規審查基準。
customPrompt（審查增幅提示詞）：支持初審審查員針對特定樣態（如：高齡者防跌特殊宣告、複合式多功能等）注入額外的專屬判定規則。
3. 核心資料結構與領域枚舉 (Domain Enums & TS Types)
為了確保代碼庫的健壯性與型別安全，我們在 /src/types.ts 中精心定義了醫療器材特殊語境下的資料模型。
code
TypeScript
export interface AttachmentItem {
  id: string;
  sectionName: string; // 申報項目（如：檢送簡表、申請書、製造業許可等）
  fileName: string;
  status: "適用" | "不適用" | "需補件" | "免附";
  isVoided: boolean;   // 是否包含作廢/效期異常標籤
  note?: string;       // 備註說明
}

export interface SubmissionMaterials {
  officialDocumentNo?: string;  // 公文發文字號
  applicationType: "新案" | "變更" | "延展" | "補發";
  origin: "國產" | "輸入";
  riskLevel: "第一等級" | "第二等級" | "第三等級";
  chineseName: string;         // 中文名稱
  englishName?: string;        // 英文名稱
  mainCategoryNum: string;     // 醫材大類代碼 (如 O, A, B 等)
  mainCategoryName: string;    // 醫材大類名稱
  subCategoryCode: string;     // 子類別編號 (如 O.3825, O.3850)
  subCategoryName: string;     // 子類別名稱 (如 機械式助行器, 手動輪椅)
  intendedUse: string;         // 效能或預期用途
  specifications?: string;     // 規格說明
  applicantTaxId: string;      // 藥商/申請商統一編號
  applicantName: string;       // 申請商名稱
  applicantAddress: string;    // 申請商地址
  responsiblePerson: string;   // 負責人
  contactName: string;         // 聯絡人
  contactPhone: string;        // 聯絡電話
  contactEmail: string;        // 聯絡電子郵件
  manufacturerType: "單一製造廠" | "委託製造" | "分裝包裝";
  manufacturerName: string;    // 製造廠名稱
  manufacturerAddress: string; // 製造廠地址
  paymentMethod: string;       // 繳法方式
  paymentAmount: number;       // 繳法金額
  attachments: AttachmentItem[]; // 附件清單二進位與作廢驗證狀態
}

export interface LiveLog {
  id: string;
  timestamp: string;
  stage: "PARSING" | "CLASSIFYING" | "VALIDATING" | "CROSS_CHECKING" | "FINALIZING";
  message: string;
  status: "info" | "warning" | "success" | "error";
}

export interface CriticalAlert {
  id: string;
  type: "danger" | "warning" | "info";
  title: string;
  description: string;
  field?: keyof SubmissionMaterials | "attachments"; // 對應的前端 Form 欄位
}

export interface RegulatoryMatch {
  rule: string;
  status: "pass" | "fail" | "warning";
  details: string;
}

export interface DashboardMetrics {
  documentCompleteness: number;    // 文件完整度百分比 (0-100)
  regulatoryRiskScore: number;     // 法規合規風險評分 (0-100)
  tfdaApprovalProbability: number; // 建議核准機率預估 (0-100)
  criticalAlerts: CriticalAlert[]; // 致命缺失
  regulatoryMatches: RegulatoryMatch[]; // 與現行法規核對之清單
}
4. 稽核演算法與核心邏輯深度解析
本案之核心程式邏輯中，針對 TFDA 法規的自動化合規性稽核（Regulatory Compliance Auditing）有三層關鍵交叉檢索機制：
4.1 語意相關性分析演算法 (Semantic Consistency Matching)
審查引擎將比對下列核心欄位：

若中文名稱判定出含有「輪椅」等強力特特徵詞（如：輪椅、Wheelchair、Comfort Chair），而子類別編碼設定為 O.3825 (機械式助行器)，系統會主動觸發語意錯配警告，同時調降 tfdaApprovalProbability 至 20% 以下。
4.2 文件狀態與效力追溯規則
遍歷附件陣列 attachments，當 isVoided === true，或偵測到 藥商許可執照 內文含蓋特定不當關鍵字（"DEPRECATED", "作廢", "INVALID"），系統會觸發：
一個致命缺失警報（CriticalAlert，型別為 danger，綁定至 attachments 欄位）。
文件完整度指標 documentCompleteness 自行調降 15%。
4.3 規格匹配與分級穿透預警
當 riskLevel 標記為「第一等級」（Class I），但在 intendedUse（預期用途）或 specifications（規格）中出現「除褥瘡」、「交替壓力」、「防褥瘡氣墊」、「動力式」等帶電或壓力交互作用之描述時，系統會立即警告「有超越第一等級範疇、應屬第二等級（Class II）醫療器材之嫌疑」，以阻斷不合規之查驗。
5. 三項 WOW 級全新 AI 功能技術提案
為了大幅超越現有同類工具、為 TFDA 審查生態圈帶來無與倫比的智慧變革，我們在維持目前優秀代碼儲備的前提下，技術性提案以下三個前沿的多模態、代理人化（Agentic）進階技術功能：
🔥 Wow AI 功能 1：多模態臨床文獻與生體相容性關聯稽核引擎 (Multimodal Bio-compatibility & Clinical Literature Verification Engine)
技術描述：
第一等級或第二等級醫材申報中，常附帶材料的生體相容性（Bio-compatibility）測試報告。本功能結合 Google Gemini 2.x/3.x 的多模態圖像理解能力（Multimodal Vision Capacity）。
技術運作流程：
當廠商拖入皮膚刺激、致敏、細胞毒性測試之 PDF 掃描圖檔（如 ISO 10993 系列標準報告）時，多模態代理人直接將 PDF 圖表像素化傳送給後端。
引擎不僅檢核文字，更自動分析掃描件中的 Excel 試驗數據截圖、化學色譜分析圖譜、動物皮膚紅腫實驗原相片。
AI 主動提取相片中實驗組之「紅斑與水腫評分（Erythema and Edema Scores）」，並比對 ISO 10993-10 中定義的臨界分值。
一旦圖片與報告宣稱在文字不一致（例如測試圖片已有中度红斑現象，文字報告卻聲稱無刺激性），系統會直接在 PDF 上標註高亮，向初審人員報警。
商業與實用價值：
從根本上杜絕「文字報告造假」與「張冠李戴、濫用他人檢驗圖表」之現象，實現影像級法規實體稽核。
🔥 Wow AI 功能 2：動態 TFDA 法規符合性自適應補件模擬器 (TFDA Adaptive Request-for-Information (RFI) Simulator & Automated Reply Generator)
技術描述：
在傳統流程中，若發現缺失，承辦人需要花費一至兩小時撰寫「補件通知公文（RFI Official Letter）」。本功能透過 Agentic Reasoning Loop 實現一鍵式補件公文擬定、並為廠商提供高度仿真的「補件自我修正交互式模擬器」。
技術運作流程：
審查結束後，AI 自動提取所有的 CriticalAlert（如：力學報告放錯欄位、製造業執照蓋作廢章、名不符實）。
系統調用精確微調的 TFDA 政府機關公文格式提示詞（Official Regulatory Document Prompt），自動渲染出高度格式化的 Traditional Chinese 「補件說明通知書」草案，內容精確對應現行法規（如《醫療器材管理法》第25條）。
初審員一鍵匯出（PDF 或 Word），廠商在補件專區可以看到 AI 根據補件缺失生成的**「引導式補件練習箱」**。廠商將正確的 CNS 符合性報告或未作廢的執照重新拖入此箱。
模擬器秒級預評分，告知廠商「重新校正後，您本案的核准機率已從 15% 提升至 95%，可以正式提交補件公文了」。
商業與實用價值：
大幅降低衛生機關與申報藥商之間的溝通屏障，使補件往返次數平均降低 70% 以上，顯著釋放基層審查體系之行政負載。
🔥 Wow AI 功能 3：智慧供應鏈 QSD/GMP 製造業者交叉查驗與多源追溯圖譜 (Smart Manufacturing Quality System (QSD/GMP) Cross-Verification & Global Traceability Graph Engine)
技術描述：
第一等級與第二等級醫材查驗登記時，極其看重其生產品項是否確實落在「醫療器材製造業許可產線」或國外製造廠 QSD (Quality System Documentation，醫療器材品質管制系統) 的核准範疇內。本功能引入 動態圖數據庫推理（Graph-based RAG）。
技術運作流程：
AI 初審專家讀取申請書中的 applicantTaxId 藥商統編，以及 manufacturerAddress（製造廠地址）與 manufacturerName（伍和儀器廠股份有限公司）。
Agent 在後端啟動隱私法規受保護的即時圖譜檢索，穿透並對接 TFDA 歷年核准之公佈數據庫。
系統將本次申請之醫材名稱（舒適機械式輪椅）與圖譜中該製造廠已核准之 「登錄品項代碼、GMP 生產線類別、違規紀錄、不合格產品回收通報紀錄」 進行圖聯網（Knowledge Graph）多維交叉驗證。
若系統發現該地址登錄之 GMP 製造業許可證「僅核准生產機械式助行器，且去年曾有因鋼管焊接裂紋導致海外回收之重大通報（Recall Notification）」，AI 便會在儀表板中建立高風險紅旗。
商業與實用價值：
將孤立的「文件審查」升級為三維的「供應鏈實時風險動態監控」，防範無證代工廠越權代工高風險產品、或由已被作廢 GMP 執照之產線出貨。
6. 指標演算法與技術公式設計
為了讓審查員清晰掌握申報案的整體品質狀況，本系統在前端提供了三個科學化的動態儀表板指標（Dashboard Metrics）。以下為後端的核心指標精確計算邏輯：
6.1 文件完整度百分比 (Document Completeness, 
)
文件完整度反映了必檢文件是否基本齊全且無致命瑕疵。其計算公式定義如下：
其中，
 為法規要求的必檢附件總量。
 為第 
 項附件的權重分值（例如，申請書權重為 
，一般型錄權重為 
）。
 為檢核分數：
適用 (Applied) 
免附 (Exempt) 
不適用/需補件 (RFI) 
 是作廢扣分係數。若任何必檢文件中被發現 isVoided === true，則 
，否則為 
。
6.2 法規風險評估值 (Regulatory Risk Score, 
)
法規風險評估值係指本案違反 TFDA 現行管理辦法的內在風險，數值越高表示風險越大：
其中，
 為檢出缺失的安全嚴重度扣分：
Danger 等級缺失：每次分值 +30
Warning 等級缺失：每次分值 +15
Info 等級缺失：每次分值 +5
 為「類別與名稱宣稱錯配」之穿透懲罰值。當偵測到「輪椅名稱」對應「助行器代碼」之類嚴重情況，自動令 
。
6.3 建議核准機率 (TFDA Approval Probability, 
)
融合前兩者指標，估算出直接過審（無須補件而核准）的綜合概率：
 為致命缺陷乘法因子。
當 criticalAlerts 陣列中存在任意一項 type === "danger" 的核心法規缺失（例如，中文名與編代碼嚴重錯置、關鍵原廠臨床性能缺失、或主要證明文件作廢），則：
若無任何致命重大缺陷，則 
。
7. 系統擴充性與資訊安全合規指南
為配合醫療器材管理法規中對於「資訊安全與個資保護（Information Security & Privacy Protection）」之嚴格要求，本技術規格書在架構設計上特別設定以下高階安全防護策略：
7.1 去識別化 (De-identification) 數據預處理管道
為了防範廠商上傳之申請書草案包含敏感的受試者隱私（PHI）或商業機密：
地端模糊化解析（Edge Obfuscation）：在將 Base64 發送至 API 之前，前端會利用輕量正則表達式，自動檢索「負責人身分證號、特定病患個人健康資訊 (PHI)」，並在發送端實施字元屏蔽（Masking，如 F127***890）。
安全專屬金鑰儲存：所有的 GEMINI_API_KEY 均僅在 Server-Side 環境變量中流通，客戶端程式（Client UI）完全無法直接讀取或請求金鑰，完美規避了逆向工程導致的密鑰外洩。
7.2 零信任安全架構與 CORS 防護
在 server.ts 中，嚴格限制 HTTP 請求來源。
所有的數據在處理完畢後，後端 Express 僅緩存於內存（Memory-Based Temp），不落地存儲於容器的硬碟空間內。若要啟用持久化紀錄，將遵循系統規定的 Firebase Firestore 整合技能（firebase-integration），使用強大的安全規則（firestore.rules）進行藥商主動權限認證（Auth Claim Validation）。
8. 專案未來展望與 20 個深度專業追蹤問題
為使系統可以依循健康方向持續演進，並在未來能順利通過醫療軟體安全與 TFDA 正式認證（SaMD，Software as a Medical Device），我們為專案核心人員設計了以下 20 個深度探討問題，作為下一步技術推進與架構轉型時的探路基石：
🗂️ 類別一：AI 模型能力與多模態技術 (5 題)
在多模態 PDF 掃描件理解中，面對低解析度（< 150 DPI）、歪斜且印有大面積半透明浮水印的力學測試報告圖表，我們該如何優化 Gemini 2.x/3.x 的 Vision 提示詞（Vision Prompting）以提高文字提取率？
是否有必要在後端引入專屬的 LayoutLM 或其他輕量級本地 OCR 預處理架構，先進行版面分析（Layout Analysis），再由 Gemini 進行法規審查，以此減少 API 的 Token 消耗？
當用戶上傳容量多達 200 頁、包含大量圖片與臨床試驗表單的醫材綜合申報 PDF 時，如何有效劃分文件區塊以防止單次 API 請求超越 Gemini 的 Context Window 限制並降低模型幻覺（Hallucination）？
本系統如何保障對台灣 TFDA 歷年法規更新（例如《醫療器材管理法》最新修正案、歷屆行政命令公告）進行即時更新？是否考慮構建動態的合規知識庫，並引入法規級的 RAG（檢索增強生成）架構？
面對多樣性的臨床相片，我們應如何設計「模型微調（Fine-tuning）」或是「高階上下文學習（In-context Learning）」機制，確保 AI 判讀「生體相容性」相片時，不會產生誤判（False Positive）進而影響合規準確率？
🔒 類別二：數據隱私、個資合規與網路安全 (5 題)
由於醫療器材查驗登記涉及極其敏感的產品機密、專利技術細節（如核心力學配方、專利材料結構），我們該如何在 server.ts 層級設計一套高強度的數據不落地機制，使上傳的文件與 AI 推理內容完全不留存於第三方主機？
如果要將系統部署在衛生福利部內聯網（Government Intranet）的高安全專網環境，我們如何支持企業級 Gemini 模型的「私有化雲端隔離部署（VPC Service Controls）」與 API 白名單對接？
當前系統使用的 PDF Base64 傳輸方案，在面對超大檔案（> 100MB）時會對 Express 伺服器內存造成顯著壓迫，如何將其重構為「分片分塊上傳、直接利用 Cloud Storage 預簽名 URL（Presigned URL）讓 Gemini 服務直接讀取過濾」的安全傳輸架構？
為了防範有惡意攻擊者透過特製的 PDF（例如包含惡意腳本的 PDF、或是專門對大語言模型進行指令注入 Prompt Injection 的文檔）來癱瘓 AI 審查引擎，我們在後端應建立哪些前置過濾（Sanitization）與文本防禦層？
當有不同權限等級的審查員（如：一般科員、初審組長、外部專科審查委員）使用此 App，系統應如何在不修改前端交互流的前提下，落實符合 HIPAA 及 CNS 27001 之細粒度基於角色的存取控制 (RBAC)？
⚙️ 類別三：系統工程、可擴展性與 API 效能優化 (5 題)
目前 API /api/transform 與 /api/review 分開執行。為了應對高併發的申報季，我們如何使用消息隊列（如 BullMQ 或 Redis-based Queue）來異步處理耗時的 Gemini 分析，並透過 WebSockets (基於 realtime_guideline) 向前端推送即時審查虛擬日誌（liveLogs）？
系統如何有效降低每次審查的延遲（Latency）？是否可對相對固定的「法規評估指引（DEFAULT_GUIDELINE）」進行嵌入向量（Embedding）緩存，從而免除每次請求都發送大段 Guideline System Instruction 的高昂開銷？
若將本系統升級為「多人協同審查雲端平台」，其 React 19 Frontend 多個 Component 之間如何透過穩定精確的 Context 或 Zustand 進行狀態傳遞，從而規避 useEffect 短路導致的頁面無限循環重新渲染？
面對多品項審查，如何重構 /src/types.ts 中，使 SubmissionMaterials 除了適用於「O. 物理醫學科學」器材之外，還能通過靈活的 schema 註冊，相容於「A. 臨床化學及臨床毒理學」等更複雜的帶電化學檢測試劑（IVD）？
針對前端 UI 的「Alert 點擊自動聚焦 Form 欄位及亮點高亮」之卓越 WOW 體驗（Wow Ref Focus），如何在動態新增或刪除附件（Attachment Items）時，保障各 Refs 引用不為 null 或不發生記憶體洩漏（Memory Leaks）？
⚖️ 類別四：法規演進、補件自動生成與 SaMD 認證 (5 題)
TFDA 針對第二等級、第三等級醫材之變更與延展案（Renewal & Amendment）有着更為複雜的「新舊規格對照表」檢核。本系統在架構上應如何設計，方能實現雙份（甚至是多個歷史版本）申報材料的「AI 語意 Diff 差分自動化生成報告」？
目前系統預估的「TFDA 核准機率（TFDA Approval Probability）」算力模型屬於靜態公式。我們應如何引入對實際歷史已過審/退件之去識別化真實公文數據集進行「決策森林（Decision Forest）或小模型迴歸」訓練，以使 Approval Probability 預測精度逼近 ±3% 誤差值？
RFI 補件公文生成器在撰寫法規依據時，若遭遇複合型醫材（例如同時具備助行、自動感測防拐避障等跨學科跨編碼型態），系統如何調和不同大類法規（如 O.物理醫學科學 與 K.一般醫院及個人使用科學）之核發標準衝突？
為了滿足醫療器材軟體（SaMD）生命週期標準（IEC 62304），我們是否能自動生成系統的審查邏輯跟蹤（Audit Trail）及自動化缺陷回溯紀錄，以作為未來的法規符合性依據？
本系統如何與衛生福利部的現有線上申報網（Medical Device Registration Online System）無縫串接？如何通過編寫自動化 API 網關（API Gateway）來取代前端的人工文字複製與黏貼，實現全直連、端到端（End-to-End）無縫對接初審流水線？
9. 總結與展望
本《技術規格書》確立了「醫療器材查驗登記 AI 審查系統」的底層軟體架構，精緻的資料定義全面與 TFDA 第一等級醫療器材查驗登記審查實務接軌。透過對「多模態生體相容性關聯稽核」、「自適應 RFI 補件模擬器」與「智慧 QSD/GMP 多源追溯圖譜」三項革命性技術功能的前瞻設計，本平台已經具備朝向企業級 SaMD 產品邁進的堅實技術主幹。
緊扣「設計至上、嚴控代碼、深度法規相容」的理念，此案將持續為海內外醫材廠商及一線法規審查員，提供兼具極致美學、驚艷效能、與全生命週期安全合規的下一代智慧法規代理人體驗。
