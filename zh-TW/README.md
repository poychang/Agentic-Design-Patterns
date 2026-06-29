# Agentic Design Patterns

本 repository 收錄 Antonio Gulli 與 Mauro Sauco 所著《Agentic Design Patterns》一書全文。內容由 Tom Mathews 編整，方便社群存取與參考。

![Agentic Design Patterns - Book Cover](../assets/Agentic_Design_Patterns_Book_Cover.png)

## 作者與致謝

- **作者：** [Antonio Gulli](https://www.linkedin.com/in/searchguy/) 與 [Mauro Sauco](https://www.linkedin.com/in/maurosauco/)
- **編整：** [Tom Mathews](https://www.linkedin.com/in/mathews-tom/)

### 這本書有何特別之處？

這本 424 頁的指南探討我們在建構智慧型、自主式 AI 系統時面臨的實際挑戰。它銜接理論與實作之間的落差，正是目前這個領域所需要的內容。對於認真建構真實 AI 系統的人來說，這是很好的資源。如果你是工程師、研究人員或產品經理，已準備好超越基本的大型語言模型（large language model, LLM）應用，並建構真正穩健的 AI 代理（agent），這本書就是為你而寫。

本書涵蓋必要的代理式模式，包括提示鏈（Prompt Chaining）、路由（Routing）、規劃（Planning）與多代理系統，並搭配實用、以程式碼為基礎的範例。你也會看到對工具使用（Tool Use / Function Calling）、記憶管理與 RAG 實作的完整介紹，以及推理技術與代理間通訊（A2A）等進階主題。

書中包含：

- **真實程式碼範例：** 不只是理論，而是可運作的實作。
- **經驗證的模式：** 記憶處理、例外邏輯、資源控制、安全護欄。
- **進階技術：** 多代理協調、代理間訊息傳遞、人類介入。
- **Model Context Protocol（MCP）完整章節：** 這是將工具與代理整合的關鍵框架。

全書以 4 個部分涵蓋 21 個核心模式：

1. 基礎模式（提示鏈、路由、工具使用）
2. 進階系統（記憶、學習、監控）
3. 生產環境關注事項（錯誤處理、安全、評估）
4. 多代理架構

多數 AI 內容只停留在「如何呼叫 API」。但在真實世界系統中，你需要思考：

- 如果代理在任務中途卡住怎麼辦？
- 如何在長時間 session 中保留記憶？
- 當你執行 10 個以上代理時，如何避免混亂？

本書用你能實際套用的模式回答這些問題。光是超過 70 頁的附錄就很值得投入，其中包含進階提示技術，以及代理式框架概覽。

## 目錄

### 引言

- [獻詞](00-Introduction/01-Dedication-1cQ61mNpiWn6eSORmWjEjF44vN2Lpba8kyKmNwIC60ig.md)
- [致謝](00-Introduction/02-Acknowledgment-1u2y6tY48bw8nriDUuwWEf9s8g66vyIqBKSKZDOS-n0s.md)
- [前言](00-Introduction/03-Foreword-18Q9kfZuCTL37ztrSjLxwf8Elr5UfAiAavmnj0IqSpbU.md)
- [思想領袖觀點：權力與責任](00-Introduction/04-A_Thought_Leaders_Perspective_Power_and_Responsibility-1PWhaXD_UNKgJaxYe3JBxRFRt3_B8Wm67CFxtSBQ4LkU.md)
- [引言](00-Introduction/05-Introduction-1K5jwqB6jh20uHL0TTWxqWOxFk-dzFxRvHzrRRV79hrg.md)
- [什麼讓 AI 系統成為代理？](00-Introduction/06-What_makes_an_AI_system_an_Agent-1Nw6hRa7ItdLr_Tj5hF2q-OH8B_uPKb--RLn8SXZKA94.md)

### 第一部：基礎模式

- [第 1 章：提示鏈（Prompt Chaining）](01-Part_One/Chapter_1-Prompt_Chaining-1flxKGrbnF2g8yh3F-oVD5Xx7ZumId56HbFpIiPdkqLI.md)
- [第 2 章：路由（Routing）](01-Part_One/Chapter_2-Routing-1ux_n8n3T4bYndOjs1DKW5ccpC802KISdy2IWnlvYbas.md)
- [第 3 章：平行化（Parallelization）](01-Part_One/Chapter_3-Parallelization-1XVMp4RcRkoUJTVbrP2foWZX703CUJpWkrhyFU2cfUOA.md)
- [第 4 章：反思（Reflection）](01-Part_One/Chapter_4-Reflection-1HXXJOQIMWowtLw4WMiSR360caDAlZPtl5dPPgvq9IT4.md)
- [第 5 章：工具使用](01-Part_One/Chapter_5-Tool_Use_(Function_Calling)-1bE4iMljhppqGY1p48gQWtZvk6MfRuJRCiba1yRykGNE.md)
- [第 6 章：規劃（Planning）](01-Part_One/Chapter_6-Planning-18vvNESEwHnVUREzIipuaDNCnNAREGqEfy9MQYC9wb4o.md)
- [第 7 章：多代理協作](01-Part_One/Chapter_7-Multi-Agent_Collaboration-1RZ5-2fykDQKOBx01pwfKkDe0GCs5ydca7xW9Q4wqS_M.md)

### 第二部：進階系統

- [第 8 章：記憶管理](02-Part_Two/Chapter_8-Memory_Management-1asVTObtzIye0I9ypAztaeeI_sr_Hx2TORE02uUuqH_c.md)
- [第 9 章：學習與適應](02-Part_Two/Chapter_9-Learning_and_Adaptation-1UHTEDCmSM1nwB-iyMoHuYzVcu_B_4KkJ2ITGGUKqo8s.md)
- [第 10 章：Model Context Protocol（MCP）](02-Part_Two/Chapter_10-Model_Context_Protocol_(MCP)-1e6XimYczKmhX9zpqEyxLFWPQgGuG0brp7Hic2sFl_qw.md)
- [第 11 章：目標設定與監控](02-Part_Two/Chapter_11-Goal_Setting_and_Monitoring-10ndlCB39BWjyFRWKpcoKib4vuPD1ojD-x0-ynMaf5uw.md)

### 第三部：生產環境關注事項

- [第 12 章：例外處理與復原](03-Part_Three/Chapter_12-Exception_Handling_and_Recovery-1C07AuMur6-infwE0viCp4QtAy_wWI-uceFm6MaYHQGk.md)
- [第 13 章：人類介入（Human in the Loop）](03-Part_Three/Chapter_13-Human_in_the_Loop-1ImOZcw6yeb7a-uRBMNP1VdovYfyip4IdsAcLu9yue-0.md)
- [第 14 章：知識檢索（RAG）](03-Part_Three/Chapter_14-Knowledge_Retrieval_(RAG)-1v96Oobio6xDOqbK8ejsXjmOc4Dp2uoLMo5_gfJgi-NE.md)

### 第四部：多代理架構

- [第 15 章：代理間通訊（A2A）](04-Part_Four/Chapter_15-Inter_Agent_Communication_(A2A)-1H6HmUYcy5kugt5gt7Kh2Zzb8C62d5pu36RsgMNDCX24.md)
- [第 16 章：資源感知最佳化](04-Part_Four/Chapter_16-Resource_Aware_Optimization-1nAN58l6JjqEJHk43126uh7xgdEblCpcbsNUHXgtBmJQ.md)
- [第 17 章：推理技術](04-Part_Four/Chapter_17-Reasoning_Techniques-1Yt1W_hLaC6ZNgJXfT4W6NrCL4TzNVdKOX50kgpHiIq4.md)
- [第 18 章：護欄與安全模式](04-Part_Four/Chapter_18-Guardrails_Safety_Patterns-1Gpc5af_okze1kprRLohP6-81e1KwL6HggjeLvxQyIuk.md)
- [第 19 章：評估與監控](04-Part_Four/Chapter_19-Evaluation_and_Monitoring-1G3zOZM2ZOd0gUp5dy66FUjKMOcALh9l-JpvPxgGMm8w.md)
- [第 20 章：優先排序](04-Part_Four/Chapter_20-Prioritization-1qyXxGM2hNqW_qjXuBFxrEUeoYVO79BoW1ogKu1bfdCY.md)
- [第 21 章：探索與發現](04-Part_Four/Chapter_21-Exploration_and_Discovery-1zeeMVTqjqRIli6G9MMWThhoQhvKqLOjJF2EHHUXLhdk.md)

### 附錄

- [附錄 A：進階提示技術](05-Appendix/Appendix_A-Advanced_Prompting_Techniques-1V7EKEWibOH6IhHD_PtbFZiml492-2191jDQCcTkhtTI.md)
- [附錄 B：AI 代理式互動：從 GUI 到真實世界環境](05-Appendix/Appendix_B-AI_Agentic_Interactions_From_GUI_to_Real_World_Environment-11pma_tCoC7uZ2SFKjcR5KyIq0_ooMGSoadI6f9mxG2I.md)
- [附錄 C：代理式框架快速概覽](05-Appendix/Appendix_C-Quick_Overview_of_Agentic_Frameworks-151rGsiEYOkXUcNDRus_N8TxxuvjoyTDViBhzt9z0Mfw.md)
- [附錄 D：使用 AgentSpace 建構代理（僅線上版）](05-Appendix/Appendix_D-Building_an_Agent_with_AgentSpace_(on_line_only)-1bDRJ8mKtLTeWNC-cGD0Cr8pEJQgJHNcjqz5ekloAjaE.md)
- [附錄 E：CLI 上的 AI 代理](05-Appendix/Appendix_E-AI_Agents_on_the_CLI-1W4znto0a8Ikajw5a4tEyRAaB2nJPJw_iFc4w4qNnjho.md)
- [附錄 F：底層機制：代理推理引擎內部觀察](05-Appendix/Appendix_F-Under_the_Hood_An_Inside_Look_at_the_Agents_Reasoning_Engines-14q3fQ-FZmDgiughno_WLSILMWkURvUgR7mlGiFtvwd4.md)
- [附錄 G：Coding Agents](05-Appendix/Appendix_G-Coding_Agents-1tVyhgwrD4fu_D_pHUrwhNxoguRG3tLc1KObXFxrxE_s.md)

## 授權

本 repository 採用 [MIT License](../LICENSE) 授權。

![Agentic Design Patterns](../assets/Agentic_Design_Patterns.png)
