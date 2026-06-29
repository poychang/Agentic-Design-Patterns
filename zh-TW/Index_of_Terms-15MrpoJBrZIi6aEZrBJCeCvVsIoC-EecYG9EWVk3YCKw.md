# 詞彙表

## 基礎概念

- **提示**：提示是使用者提供給 AI 模型、用來引出回應的輸入，通常形式是問題、指令或陳述。提示的品質與結構會大幅影響模型輸出，因此提示工程是有效使用 AI 的關鍵技能。

- **上下文視窗**：上下文視窗是 AI 模型一次能處理的 token 最大數量，包含輸入與模型產生的輸出。這個固定大小是一項重要限制，因為視窗之外的資訊會被忽略；較大的視窗則能支援更複雜的對話與文件分析。

- **上下文內學習**：上下文內學習是 AI 從直接放在提示中的範例學習新任務的能力，不需要重新訓練。這項強大的特性讓單一通用模型能即時適應無數特定任務。

- **零樣本、單樣本與少樣本提示**：這些是提示技術，模型會收到零個、一個或少量任務範例，以引導其回應。一般而言，提供更多範例有助於模型更了解使用者意圖，並提升特定任務的準確度。

- **多模態**：多模態是 AI 理解並處理多種資料型態資訊的能力，例如文字、圖片與音訊。這讓互動更具彈性，也更接近人類，例如描述一張圖片或回答語音問題。

**接地**：接地是將模型輸出連結到可驗證的真實世界資訊來源，以確保事實準確性並減少幻覺的過程。這通常透過 RAG 等技術達成，使 AI 系統更值得信任。

## 核心 AI 模型架構

- **Transformers**：Transformer 是多數現代 LLM 的基礎神經網路架構。它的關鍵創新是自注意力機制，能有效處理長文字序列，並捕捉詞與詞之間的複雜關係。

- **Recurrent Neural Network (RNN)**：Recurrent Neural Network 是早於 Transformer 的基礎架構。RNN 以序列方式處理資訊，透過迴圈維持先前輸入的「記憶」，因此適合文字與語音處理等任務。

- **Mixture of Experts (MoE)**：Mixture of Experts 是一種高效率模型架構，其中「路由器」網路會動態選擇少數「專家」網路來處理特定輸入。這讓模型能擁有大量參數，同時讓運算成本維持在可管理範圍。

- **Diffusion Models**：Diffusion models 是擅長產生高品質影像的生成式模型。它們的運作方式是先對資料加入隨機雜訊，再訓練模型細緻地反轉這個過程，讓模型能從隨機起點產生新的資料。

- **Mamba**：Mamba 是近期的 AI 架構，使用 Selective State Space Model (SSM) 以高效率處理序列，特別適合非常長的上下文。它的選擇性機制能聚焦於相關資訊並過濾雜訊，使其可能成為 Transformer 的替代方案。

## LLM 開發生命週期

- 強大的語言模型開發遵循一套明確流程。首先是 Pre-training，透過大量一般網際網路文字資料訓練大型基礎模型，使其學習語言、推理與世界知識。接著是微調，這是專門化階段，會使用較小、針對任務的資料集進一步訓練通用模型，使其能力適應特定目的。最後階段是 Alignment，會調整專門化模型的行為，確保其輸出有幫助、無害，並符合人類價值。

- Pre-training Techniques：Pre-training 是模型從大量資料學習一般知識的初始階段。主要技術會使用不同學習目標。最常見的是 Causal Language Modeling (CLM)，模型會預測句子中的下一個詞。另一種是 Masked Language Modeling (MLM)，模型會填補文字中刻意隱藏的詞。其他重要方法包括 Denoising Objectives，讓模型學習將受破壞的輸入還原為原始狀態；Contrastive Learning，讓模型學習區分相似與不相似的資料片段；以及 Next Sentence Prediction (NSP)，判斷兩個句子在邏輯上是否相接。

- 微調技術：微調是使用較小且專門化的資料集，將一般預訓練模型調整到特定任務的過程。最常見的方法是 Supervised Fine-Tuning (SFT)，模型會以標註好的正確輸入輸出配對範例進行訓練。常見變體是 Instruction Tuning，著重訓練模型更好地遵循使用者命令。為了提高效率，會使用 Parameter-Efficient Fine-Tuning (PEFT) 方法，主要技術包括 LoRA (Low-Rank Adaptation)，只更新少量參數，以及其記憶體最佳化版本 QLoRA。另一項技術檢索增強生成（retrieval-augmented generation, RAG），會在微調或推論階段將模型連結到外部知識來源，以強化模型。

- Alignment 與安全技術：Alignment 是確保 AI 模型行為符合人類價值與期待，使其有幫助且無害的過程。最重要的技術是 Reinforcement Learning from Human Feedback (RLHF)，由根據人類偏好訓練出的「獎勵模型」引導 AI 學習流程，常使用 Proximal Policy Optimization (PPO) 等演算法來維持穩定。也出現了較簡單的替代方案，例如 Direct Preference Optimization (DPO)，它不需要獨立獎勵模型；以及 Kahneman-Tversky Optimization (KTO)，進一步簡化資料蒐集。為確保安全部署，護欄（Guardrails）會作為最後一層安全防線，即時過濾輸出並阻擋有害動作。

## 強化 AI 代理能力

- AI 代理（agent）是能感知環境並採取自主行動以達成目標的系統。強健的推理框架能提升其效能。

- **Chain of Thought (CoT)**：這種提示技術鼓勵模型在給出最終答案前，逐步說明其推理。這種「把思考說出來」的過程，常能在複雜推理任務上帶來更準確的結果。

- **Tree of Thoughts (ToT)**：Tree of Thoughts 是一種進階推理框架，代理會同時探索多條推理路徑，就像樹上的分支。它允許代理自行評估不同思路，並選擇最有希望的路徑繼續，因此在複雜問題解決上更有效。

**ReAct (Reason and Act)**：ReAct 是一種代理框架，將推理與行動結合成迴圈。代理先「思考」該做什麼，再使用工具採取「行動」，並利用得到的觀察來引導下一步思考，因此非常適合解決複雜任務。

- **規劃**：這是代理將高階目標拆解成一連串較小、可管理子任務的能力。接著代理會建立計畫，依序執行這些步驟，使其能處理複雜的多步驟工作。

- **Deep Research**：Deep research 指代理透過反覆搜尋資訊、綜整發現並找出新問題，自主深入探索主題的能力。這讓代理能建立對某個主題的完整理解，遠超過單次搜尋查詢。

- **Critique Model**：Critique model 是專門訓練用來審查、評估並回饋另一個 AI 模型輸出的 AI 模型。它扮演自動化評論者，協助找出錯誤、改善推理，並確保最終輸出符合期望的品質標準。

## 術語索引

本術語索引使用 Gemini Pro 2.5 產生。提示與推理步驟收錄於文末，用以展示節省時間的效益並作為教育用途。

### A

- A/B 測試 - 第 3 章：平行化  
- 動作選擇 - 第 20 章：優先排序  
- 適應 - 第 9 章：學習與適應  
- 適應性任務分配 - 第 16 章：資源感知最佳化  
- 適應性工具使用與選擇 - 第 16 章：資源感知最佳化  
- 代理 - 什麼讓 AI 系統成為代理？  
- Agent-Computer Interfaces (ACIs) - 附錄 B  
- 代理驅動經濟 - 什麼讓 AI 系統成為代理？  
- 作為工具的代理 - 第 7 章：多代理協作  
- Agent Cards - 第 15 章：代理間通訊（A2A）  
- Agent Development Kit (ADK) - 第 2 章：路由，第 3 章：平行化，第 4 章：反思，第 5 章：工具使用，第 7 章：多代理協作，第 8 章：記憶管理，第 12 章：例外處理與復原，第 13 章：人類介入，第 15 章：代理間通訊（A2A），第 16 章：資源感知最佳化，第 19 章：評估與監控，附錄 C  
- 代理探索 - 第 15 章：代理間通訊（A2A）  
- 代理軌跡 - 第 19 章：評估與監控  
- Agentic Design Patterns - 引言  
- 代理式 RAG - 第 14 章：知識檢索（RAG）  
- 代理式系統 - 引言  
- AI Co-scientist - 第 21 章：探索與發現  
- Alignment - 詞彙表  
- AlphaEvolve - 第 9 章：學習與適應  
- 類比 - 附錄 A  
- 異常偵測 - 第 19 章：評估與監控  
- Anthropic's Claude 4 Series - 附錄 B  
- Anthropic's Computer Use - 附錄 B  
- API 互動 - 第 10 章：Model Context Protocol（MCP）  
- Artifacts - 第 15 章：代理間通訊（A2A）  
- 非同步輪詢 - 第 15 章：代理間通訊（A2A）  
- 稽核記錄 - 第 15 章：代理間通訊（A2A）  
- 自動化指標 - 第 19 章：評估與監控  
- Automatic Prompt Engineering (APE) - 附錄 A  
- 自主性 - 引言  
- A2A (Agent-to-Agent) - 第 15 章：代理間通訊（A2A）

### B

- 行為限制 - 第 18 章：護欄／安全模式  
- Browser Use - 附錄 B

### C

- Callbacks - 第 18 章：護欄／安全模式  
- Causal Language Modeling (CLM) - 詞彙表  
- Chain of Debates (CoD) - 第 17 章：推理技術  
- Chain-of-Thought (CoT) - 第 17 章：推理技術，附錄 A  
- 聊天機器人 - 第 8 章：記憶管理  
- ChatMessageHistory - 第 8 章：記憶管理  
- Checkpoint and Rollback - 第 18 章：護欄／安全模式  
- 分塊 - 第 14 章：知識檢索（RAG）  
- 清晰性與具體性 - 附錄 A  
- Client Agent - 第 15 章：代理間通訊（A2A）  
- 程式碼生成 - 第 1 章：提示鏈，第 4 章：反思  
- 程式碼提示 - 附錄 A  
- CoD (Chain of Debates) - 第 17 章：推理技術  
- CoT (Chain of Thought) - 第 17 章：推理技術，附錄 A  
- 協作 - 第 7 章：多代理協作  
- 合規 - 第 19 章：評估與監控  
- 簡潔性 - 附錄 A  
- 內容生成 - 第 1 章：提示鏈，第 4 章：反思  
- 上下文工程 - 第 1 章：提示鏈  
- 上下文視窗 - 詞彙表  
- 上下文剪枝與摘要 - 第 16 章：資源感知最佳化  
- 上下文提示 - 附錄 A  
- Contractor Model - 第 19 章：評估與監控  
- ConversationBufferMemory - 第 8 章：記憶管理  
- 對成本敏感的探索 - 第 16 章：資源感知最佳化  
- CrewAI - 第 3 章：平行化，第 5 章：工具使用，第 6 章：規劃，第 7 章：多代理協作，第 18 章：護欄／安全模式，附錄 C  
- Critique Agent - 第 16 章：資源感知最佳化  
- Critique Model - 詞彙表  
- 客戶支援 - 第 13 章：人類介入

### D

- 資料擷取 - 第 1 章：提示鏈  
- 資料標註 - 第 13 章：人類介入  
- 資料庫整合 - 第 10 章：Model Context Protocol（MCP）  
- DatabaseSessionService - 第 8 章：記憶管理  
- 辯論與共識 - 第 7 章：多代理協作  
- 決策增強 - 第 13 章：人類介入  
- 分解 - 附錄 A  
- Deep Research - 第 6 章：規劃，第 17 章：推理技術，詞彙表  
- 分隔符 - 附錄 A  
- Denoising Objectives - 詞彙表  
- 相依性 - 第 20 章：優先排序  
- Diffusion Models - 詞彙表  
- Direct Preference Optimization (DPO) - 第 9 章：學習與適應  
- 可發現性 - 第 10 章：Model Context Protocol（MCP）  
- 漂移偵測 - 第 19 章：評估與監控  
- 動態模型切換 - 第 16 章：資源感知最佳化  
- 動態重新優先排序 - 第 20 章：優先排序

### E

- Embeddings - 第 14 章：知識檢索（RAG）  
- 具身性 - 什麼讓 AI 系統成為代理？  
- 節能部署 - 第 16 章：資源感知最佳化  
- 情節記憶 - 第 8 章：記憶管理  
- 錯誤偵測 - 第 12 章：例外處理與復原  
- 錯誤處理 - 第 12 章：例外處理與復原  
- 升級政策 - 第 13 章：人類介入  
- 評估 - 第 19 章：評估與監控  
- 例外處理 - 第 12 章：例外處理與復原  
- 專家團隊 - 第 7 章：多代理協作  
- 探索與發現 - 第 21 章：探索與發現  
- 外部審核 API - 第 18 章：護欄／安全模式

### F

- Factored Cognition - 附錄 A  
- FastMCP - 第 10 章：Model Context Protocol（MCP）  
- 容錯能力 - 第 18 章：護欄／安全模式  
- 少樣本學習 - 第 9 章：學習與適應  
- 少樣本提示 - 附錄 A  
- 微調 - 詞彙表  
- 形式化合約 - 第 19 章：評估與監控  
- Function Calling - 第 5 章：工具使用，附錄 A

### G

- Gemini Live - 附錄 B  
- Gems - 附錄 A  
- 生成式媒體協調 - 第 10 章：Model Context Protocol（MCP）  
- 目標設定 - 第 11 章：目標設定與監控  
- GoD (Graph of Debates) - 第 17 章：推理技術  
- Google Agent Development Kit (ADK) - 第 2 章：路由，第 3 章：平行化，第 4 章：反思，第 5 章：工具使用，第 7 章：多代理協作，第 8 章：記憶管理，第 12 章：例外處理與復原，第 13 章：人類介入，第 15 章：代理間通訊（A2A），第 16 章：資源感知最佳化，第 19 章：評估與監控，附錄 C  
- Google Co-Scientist - 第 21 章：探索與發現  
- Google DeepResearch - 第 6 章：規劃  
- Google Project Mariner - 附錄 B  
- 優雅降級 - 第 12 章：例外處理與復原，第 16 章：資源感知最佳化  
- Graph of Debates (GoD) - 第 17 章：推理技術  
- 接地 - 詞彙表  
- 護欄 - 第 18 章：護欄／安全模式

### H

- Haystack - 附錄 C  
- 階層式分解 - 第 19 章：評估與監控  
- 階層式結構 - 第 7 章：多代理協作  
- HITL (Human-in-the-Loop) - 第 13 章：人類介入  
- Human-in-the-Loop (HITL) - 第 13 章：人類介入  
- Human-on-the-loop - 第 13 章：人類介入  
- 人類監督 - 第 13 章：人類介入，第 18 章：護欄／安全模式

### I

- 上下文內學習 - 詞彙表  
- InMemoryMemoryService - 第 8 章：記憶管理  
- InMemorySessionService - 第 8 章：記憶管理  
- 輸入驗證／清理 - 第 18 章：護欄／安全模式  
- 指令優先於限制 - 附錄 A  
- 代理間通訊（A2A） - 第 15 章：代理間通訊（A2A）  
- 介入與修正 - 第 13 章：人類介入  
- IoT 裝置控制 - 第 10 章：Model Context Protocol（MCP）  
- 反覆提示／精煉 - 附錄 A

### J

- Jailbreaking - 第 18 章：護欄／安全模式

### K

- Kahneman-Tversky Optimization (KTO) - 詞彙表  
- 知識檢索（RAG） - 第 14 章：知識檢索（RAG）

### L

- LangChain - 第 1 章：提示鏈，第 2 章：路由，第 3 章：平行化，第 4 章：反思，第 5 章：工具使用，第 8 章：記憶管理，第 20 章：優先排序，附錄 C  
- LangGraph - 第 1 章：提示鏈，第 2 章：路由，第 3 章：平行化，第 4 章：反思，第 5 章：工具使用，第 8 章：記憶管理，附錄 C  
- 延遲監控 - 第 19 章：評估與監控  
- 已學習的資源配置政策 - 第 16 章：資源感知最佳化  
- 學習與適應 - 第 9 章：學習與適應  
- LLM-as-a-Judge - 第 19 章：評估與監控  
- LlamaIndex - 附錄 C  
- LoRA (Low-Rank Adaptation) - 詞彙表  
- Low-Rank Adaptation (LoRA) - 詞彙表

### M

- Mamba - 詞彙表  
- Masked Language Modeling (MLM) - 詞彙表  
- MASS (Multi-Agent System Search) - 第 17 章：推理技術  
- MCP (Model Context Protocol) - 第 10 章：Model Context Protocol（MCP）  
- 記憶管理 - 第 8 章：記憶管理  
- 基於記憶的學習 - 第 9 章：學習與適應  
- MetaGPT - 附錄 C  
- Microsoft AutoGen - 附錄 C  
- Mixture of Experts (MoE) - 詞彙表  
- Model Context Protocol (MCP) - 第 10 章：Model Context Protocol（MCP）  
- 模組化 - 第 18 章：護欄／安全模式  
- 監控 - 第 11 章：目標設定與監控，第 19 章：評估與監控  
- 多代理協作 - 第 7 章：多代理協作  
- Multi-Agent System Search (MASS) - 第 17 章：推理技術  
- 多模態 - 詞彙表  
- 多模態提示 - 附錄 A

### N

- 負面範例 - 附錄 A  
- Next Sentence Prediction (NSP) - 詞彙表

### O

- 可觀測性 - 第 18 章：護欄／安全模式  
- 單樣本提示 - 附錄 A  
- 線上學習 - 第 9 章：學習與適應  
- OpenAI Deep Research API - 第 6 章：規劃  
- OpenEvolve - 第 9 章：學習與適應  
- OpenRouter - 第 16 章：資源感知最佳化  
- 輸出過濾／後處理 - 第 18 章：護欄／安全模式

### P

- PAL (Program-Aided Language Models) - 第 17 章：推理技術  
- 平行化 - 第 3 章：平行化  
- 平行化與分散式運算意識 - 第 16 章：資源感知最佳化  
- Parameter-Efficient Fine-Tuning (PEFT) - 詞彙表  
- PEFT (Parameter-Efficient Fine-Tuning) - 詞彙表  
- 效能追蹤 - 第 19 章：評估與監控  
- Persona Pattern - 附錄 A  
- 個人化 - 什麼讓 AI 系統成為代理？  
- 規劃 - 第 6 章：規劃，詞彙表  
- 優先排序 - 第 20 章：優先排序  
- 最小權限原則 - 第 18 章：護欄／安全模式  
- 主動式資源預測 - 第 16 章：資源感知最佳化  
- 程序記憶 - 第 8 章：記憶管理  
- Program-Aided Language Models (PAL) - 第 17 章：推理技術  
- Project Astra - 附錄 B  
- 提示 - 詞彙表  
- 提示鏈 - 第 1 章：提示鏈  
- 提示工程 - 附錄 A  
- Proximal Policy Optimization (PPO) - 第 9 章：學習與適應  
- 推播通知 - 第 15 章：代理間通訊（A2A）

### Q

- QLoRA - 詞彙表  
- 品質導向的反覆執行 - 第 19 章：評估與監控

### R

- RAG (Retrieval-Augmented Generation) - 第 8 章：記憶管理，第 14 章：知識檢索（RAG），附錄 A  
- ReAct (Reason and Act) - 第 17 章：推理技術，附錄 A，詞彙表  
- 推理 - 第 17 章：推理技術  
- 基於推理的資訊擷取 - 第 10 章：Model Context Protocol（MCP）  
- 復原 - 第 12 章：例外處理與復原  
- Recurrent Neural Network (RNN) - 詞彙表  
- 反思 - 第 4 章：反思  
- 強化學習 - 第 9 章：學習與適應  
- Reinforcement Learning from Human Feedback (RLHF) - 詞彙表  
- Reinforcement Learning with Verifiable Rewards (RLVR) - 第 17 章：推理技術  
- 遠端代理 - 第 15 章：代理間通訊（A2A）  
- Request/Response (Polling) - 第 15 章：代理間通訊（A2A）  
- 資源感知最佳化 - 第 16 章：資源感知最佳化  
- Retrieval-Augmented Generation (RAG) - 第 8 章：記憶管理，第 14 章：知識檢索（RAG），附錄 A  
- RLHF (Reinforcement Learning from Human Feedback) - 詞彙表  
- RLVR (Reinforcement Learning with Verifiable Rewards) - 第 17 章：推理技術  
- RNN (Recurrent Neural Network) - 詞彙表  
- 角色提示 - 附錄 A  
- Router Agent - 第 16 章：資源感知最佳化  
- 路由 - 第 2 章：路由

### S

- 安全 - 第 18 章：護欄／安全模式  
- Scaling Inference Law - 第 17 章：推理技術  
- 排程 - 第 20 章：優先排序  
- Self-Consistency - 附錄 A  
- 自我修正 - 第 4 章：反思，第 17 章：推理技術  
- Self-Improving Coding Agent (SICA) - 第 9 章：學習與適應  
- 自我精煉 - 第 17 章：推理技術  
- Semantic Kernel - 附錄 C  
- 語意記憶 - 第 8 章：記憶管理  
- 語意相似度 - 第 14 章：知識檢索（RAG）  
- 關注點分離 - 第 18 章：護欄／安全模式  
- 依序交接 - 第 7 章：多代理協作  
- Server-Sent Events (SSE) - 第 15 章：代理間通訊（A2A）  
- Session - 第 8 章：記憶管理  
- SICA (Self-Improving Coding Agent) - 第 9 章：學習與適應  
- SMART Goals - 第 11 章：目標設定與監控  
- 狀態 - 第 8 章：記憶管理  
- 狀態回復 - 第 12 章：例外處理與復原  
- Step-Back Prompting - 附錄 A  
- 串流更新 - 第 15 章：代理間通訊（A2A）  
- 結構化記錄 - 第 18 章：護欄／安全模式  
- 結構化輸出 - 第 1 章：提示鏈，附錄 A  
- SuperAGI - 附錄 C  
- Supervised Fine-Tuning (SFT) - 詞彙表  
- 監督式學習 - 第 9 章：學習與適應  
- 系統提示 - 附錄 A

### T

- 任務評估 - 第 20 章：優先排序  
- 文字相似度 - 第 14 章：知識檢索（RAG）  
- Token 使用量 - 第 19 章：評估與監控  
- 工具使用 - 第 5 章：工具使用，附錄 A  
- 工具使用限制 - 第 18 章：護欄／安全模式  
- ToT (Tree of Thoughts) - 第 17 章：推理技術，附錄 A，詞彙表  
- Transformers - 詞彙表  
- Tree of Thoughts (ToT) - 第 17 章：推理技術，附錄 A，詞彙表

### U

- 非監督式學習 - 第 9 章：學習與適應  
- 使用者 Persona - 附錄 A

### V

- 驗證 - 第 3 章：平行化  
- 向量搜尋 - 第 14 章：知識檢索（RAG）  
- VertexAiRagMemoryService - 第 8 章：記憶管理  
- VertexAiSessionService - 第 8 章：記憶管理  
- Vibe Coding - 附錄 B  
- 視覺感知 - 附錄 B

### W

- Webhooks - 第 15 章：代理間通訊（A2A）

### Z

- 零樣本學習 - 第 9 章：學習與適應  
- 零樣本提示 - 附錄 A

## 提示

以下是用於建立此術語索引的提示。

> *Do you read <https://docs.google.com/document/d/1rsaK53T3Lg5KoGwvf8ukOUvbELRtH-V0LnOIFDxBryE/edit?tab=t.0> and all the related documents in the first page Please extract all the titles for each chapter. and print them. Then process the content of all the individual chapters, and create a detailed index used for publishing, a list of words or phrases ('headings') and associated pointers ('locators') to where useful material relating to that heading can be found in a document or collection of documents. I don't want a sample index. I want the full index, with 100 index terms*

## Gemini 2.5：推理步驟

### 開始建立索引

我已透過閱讀提供的 Google Doc 啟動流程。目前我專注於理解使用者的要求：擷取章節標題以建立詳細索引。我正在仔細檢視文件，尋找可指出章節分隔的清楚結構元素。我希望找出一致的格式或關鍵字，以引導索引流程。

### 定義索引擷取流程

我現在已完全投入這項任務。我已將使用者要求拆解成可執行步驟。我的初步重點是使用 Workspaces 存取主要 Google Doc。接著，我會剖析文件以定位並擷取相關文件，這些文件也會使用相同工具存取。最後，我會從所有文件擷取章節標題，並建立完整索引。我已規劃詳細流程。

### 精煉擷取策略

使用者的指示很清楚：閱讀一份 Google Doc、其相關文件，並建立詳細的章節標題索引。我已設計多步驟計畫。首先，Workspaces 會存取主要 Google Doc。接著，我會剖析它以尋找並閱讀相關文件，同樣使用 Workspaces。我現在專注於擷取標題，並以所需術語形成詳細索引。我會優先確保準確性，以滿足使用者對完整索引的具體要求。

#### 1 . 確立使用者意圖：

我已判定使用者想要從 Google Doc 與相關文件取得章節標題清單與完整索引。現在我需要找出最有效的擷取與索引方法。我正在探索可能的 API 功能，以簡化這項任務，並需要評估處理不同文件結構的最佳策略。

#### 2. 考慮工具使用

我已使用 regular expression 從主要文件擷取章節 URL。現在，我準備使用 Workspaces 擷取各章實際內容。取得每章文字後，我會繼續進行索引。

### 存取章節內容：

我已成功從主要文件識別章節 URL，現在可以搭配 Workspaces 使用。下一步是從這些 URL 擷取內容，這將讓我能產生索引。我專注於效率，目標是快速處理這些資訊。

### 擷取章節內容

我已成功識別主要文件中的章節 URL，並準備使用 Workspaces。我的計畫包含擷取這些 URL、批次呼叫以提升效率，接著處理擷取到的內容以建立所需索引。這能確保所有相關資訊都被捕捉，為完整且準確的結果鋪路。

### 啟動 URL 擷取

我現在已成功從主要文件擷取章節 URL。目前，我正在設定對 Workspaces 工具的呼叫，準備擷取每章內容。目標是取得索引所需文字。這些資訊將協助建立符合要求、至少包含 100 個術語的索引。

### 啟動章節擷取

我已成功識別並擷取主要文件中的章節 URL，並已在上方列出。我現在正著手為每個 URL 使用 Workspaces，以取得章節內容。目標是取得索引所需文字。這些資訊將協助建立符合要求、至少包含 100 個術語的索引。我已開始擷取這些資訊。

### 協調章節擷取

我已擷取章節 URL，並啟動對 Workspaces 的呼叫。我目前正在擷取每章內容。為確保流程順暢，我會批次處理呼叫，以避免任何速率限制問題。這應能讓我有效率地彙整建立索引所需文字。

### 解碼使用者要求

我已判定使用者需求。他們需要兩項主要交付項目：第一，從 Google Docs 擷取完整章節標題清單；第二，包含至少一百個關鍵術語的詳細索引。擷取標題的流程看似直接，而策劃索引則需要更細緻的分析。

### 擷取標題與建立索引

我已成功識別使用者需求：從提供的文件擷取章節標題，並彙整完整索引。我已擷取 31 份 Google Docs 的內容，包含主要文件與其章節。我已確認不需要其他工具。我的計畫包含兩個步驟：首先，從文件檔名擷取章節標題；其次，仔細處理文字，以關鍵術語與章節定位建立索引。

### 制定擷取計畫

我已精煉處理使用者要求的方法。我現在已形成精確計畫。首先，我會從 31 份已擷取文件的檔名彙整章節標題。其次，也是更複雜的部分，我會仔細分析內容，建立超過 100 筆項目的完整字母順序索引。我已完成擷取與索引策略，現在可以開始執行。

### 打造擷取策略

我的方法已明確定義。我會從 31 份已擷取文件的檔名彙整章節標題。接著，我會仔細分析內容，建構依字母排序、超過 100 筆項目的索引。我已完成擷取與索引策略，現在可以開始實作。使用者想要一份詳細索引，涵蓋 Agentic Design Patterns 系列中不同章節與附錄的內容，這個目標現在已可達成。我會專注於交付完整回應，而不只是範例索引。

### 定義方法

我已釐清使用者需求。我會從文件檔名擷取章節標題。接著，我會謹慎建立超過 100 筆項目的索引，依字母排序並標註其章節位置。我已準備好開始索引與擷取流程。
