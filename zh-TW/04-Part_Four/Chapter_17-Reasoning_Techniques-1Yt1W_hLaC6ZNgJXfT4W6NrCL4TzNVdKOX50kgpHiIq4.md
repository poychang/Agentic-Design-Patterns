# 第 17 章：推理技術

本章深入探討代理（agent）的進階推理方法，聚焦於多步驟邏輯推論與問題解決。這些技術超越簡單的序列式操作，讓代理的內部推理變得明確。如此一來，代理就能拆解問題、考量中間步驟，並得出更穩健且準確的結論。這些進階方法的核心原則之一，是在推論期間分配更多運算資源。這意味著給予代理或底層大型語言模型（large language model, LLM）更多處理查詢與產生回應的時間或步驟。代理不再只是快速、單次通過地回答，而是能進行反覆精修、探索多條解決路徑，或使用外部工具。這種在推論期間延長的處理時間，通常能顯著提升準確性、一致性與穩健性，尤其適用於需要深入分析與審慎思考的複雜問題。

## 實務應用與使用案例

實務應用包括：

* **複雜問答：** 協助解決多跳查詢，這類查詢需要整合來自不同來源的資料並執行邏輯推演，可能還要檢視多條推理路徑，並受益於延長推論時間以綜合資訊。  
* **數學問題求解：** 讓數學問題得以拆分成較小且可解的組件，呈現逐步求解過程，並使用程式碼執行精確計算；延長推論可支援更複雜的程式碼生成與驗證。  
* **程式碼偵錯與生成：** 支援代理說明其生成或修正程式碼的理由、依序指出潛在問題，並根據測試結果反覆精修程式碼（自我修正），利用延長的推論時間進行更完整的偵錯循環。  
* **策略規劃：** 協助透過推理不同選項、後果與前提條件來制定完整計畫，並根據即時回饋調整計畫（ReAct）；較長的審慎思考可帶來更有效且可靠的計畫。  
* **醫療診斷：** 協助代理有系統地評估症狀、檢查結果與病史，以做出診斷，並在每個階段清楚說明其推理，也可能使用外部工具進行資料擷取（ReAct）。增加推論時間能支援更完整的鑑別診斷。  
* **法律分析：** 支援分析法律文件與判例，以形成論點或提供指引，詳細說明採取的邏輯步驟，並透過自我修正確保邏輯一致性。增加推論時間能支援更深入的法律研究與論證建構。

## 推理技術

首先，讓我們深入了解用來強化 AI 模型問題解決能力的核心推理技術。

**思維鏈（Chain-of-Thought, CoT）** 提示會模仿逐步思考流程，顯著提升 LLM 的複雜推理能力（見圖 1）。CoT 提示不直接提供答案，而是引導模型產生一系列中間推理步驟。這種明確拆解讓 LLM 能將複雜問題分解為更小、更易處理的子問題。對於需要多步驟推理的任務，例如算術、常識推理與符號操作，這項技術能明顯改善模型表現。CoT 的主要優勢在於，它能將困難的單步驟問題轉換成一連串較簡單的步驟，進而提高 LLM 推理流程的透明度。這種方法不只提升準確性，也提供模型決策方式的寶貴洞察，有助於偵錯與理解。CoT 可透過多種策略實作，包括提供少樣本範例來示範逐步推理，或直接要求模型「一步一步思考」。它的有效性來自能引導模型的內部處理走向更審慎且合乎邏輯的推進。因此，Chain-of-Thought 已成為當代 LLM 啟用進階推理能力的基石技術。這種更高的透明度，以及將複雜問題拆成可處理子問題的能力，對自主代理尤其重要，因為它能讓代理在複雜環境中執行更可靠且可稽核的動作。  

![COT: Chain of Thought](../assets/COT_Chain_of_Thought.png)  

圖 1：CoT 提示，以及代理產生的詳細逐步回應。

讓我們看一個範例。它以一組指示開始，告訴 AI 如何思考，定義其角色，並提供一個清楚的五步驟流程。這就是啟動結構化思考的提示。

接著，範例展示 CoT 流程的實際運作。標示為「Agent's Thought Process」的區段是內部獨白，模型會在其中執行被指示的步驟。這就是字面意義上的「思維鏈」。最後，「Agent's Final Answer」是經過這個謹慎、逐步推理流程後產生的精修完整輸出。

```markdown
You are an Information Retrieval Agent. Your goal is to answer the user's question comprehensively and accurately by thinking step-by-step. 

Here's the process you must follow: 
1.  **Analyze the Query:** Understand the core subject and specific requirements of the user's question. Identify key entities, keywords, and the type of information being sought. 
2.  **Formulate Search Queries (for Knowledge Base):** Based on your analysis, generate a list of precise search queries that you would use to retrieve relevant information from a knowledge base or external tools. 
3.  **Simulate Information Retrieval (Self-Correction/Reasoning):** For each search query, mentally consider what kind of information you expect to find. If you were to retrieve the content, what would be the most relevant snippets? Think about potential ambiguities or missing pieces. 
4.  **Synthesize Information:** Based on the simulated retrieval and your understanding of the user's original query, synthesize the gathered information into a coherent and complete answer. Ensure all aspects of the query are addressed. 
5.  **Review and Refine:** Before finalizing, critically evaluate your answer. Is it accurate? Is it comprehensive? Is it easy to understand? Is it concise? If not, identify what needs to be improved and how. 

**User Query:** "Explain the main differences between classical computers and quantum computers, and briefly describe a potential application of quantum computing." 

**Agent's Thought Process (Internal CoT Output - encouraged by the prompt):** 

**Thought 1 (Analyze Query):** 
The user wants two things: 
    1.  Main differences between classical and quantum computers. 
    2.  One potential application of quantum computing. Keywords: classical computers, quantum computers, differences, potential application. 

**Thought 2 (Formulate Search Queries):** 
    - "differences classical vs quantum computers" 
    - "how quantum computers work" 
    - "how classical computers work" 
    - "applications of quantum computing" 
    
**Thought 3 (Simulate Information Retrieval & Identify Key Concepts):** 
    - From "differences": Expect concepts like bits vs. qubits, superposition, entanglement, probabilistic nature, computational power for specific problems. 
    - From "how they work": Classical uses transistors/binary. Quantum uses quantum mechanics. 
    - From "applications": Expect drug discovery, material science, cryptography, optimization. 

**Thought 4 (Synthesize Information):** 
Classical computers use bits (0 or 1), processing information sequentially. Quantum computers use qubits, which can be 0, 1, or both simultaneously (superposition), and can be entangled, allowing for vastly different computational approaches. This enables them to solve certain problems (like factoring large numbers or simulating complex molecules) far more efficiently than classical computers. A potential application is in drug discovery, where quantum computers could simulate molecular interactions with unprecedented accuracy, accelerating the development of new medicines. 

**Thought 5 (Review and Refine):** 
    - Differences are covered (bits vs qubits, superposition, entanglement, processing). 
    - Application is given (drug discovery). - Is it brief? Yes. - Is it clear? Yes. - Looks good. 

**Agent's Final Answer (Output to User):** 
Classical computers process information using bits, which can be either a 0 or a 1 at any given time, performing operations sequentially. In contrast, quantum computers utilize qubits, which can exist in multiple states simultaneously through superposition and can be linked through entanglement, allowing for exponentially more complex computations. This fundamental difference enables quantum machines to tackle certain problems, such as simulating molecular structures or breaking complex encryption, that are intractable for even the most powerful classical supercomputers. A significant potential application of quantum computing lies in drug discovery, where its ability to precisely model molecular behavior could revolutionize the development of new pharmaceuticals.
```

**思維樹（Tree-of-Thought, ToT）** 是建立在思維鏈（CoT）之上的推理技術。它允許大型語言模型透過分岔到不同中間步驟，探索多條推理路徑，形成樹狀結構（見圖 2）。這種方法支援複雜問題解決，因為它能進行回溯、自我修正並探索替代解法。維持一棵可能性之樹，讓模型能在定案答案前評估各種推理軌跡。這種反覆流程提升了模型處理需要策略規劃與決策之挑戰性任務的能力。  

![TOT: Tree of Thought](../assets/TOT_Tree_of_Thought.png)

圖 2：Tree of Thoughts 範例

**自我修正**，也稱為自我精修，是代理推理流程中的關鍵面向，尤其是在 Chain-of-Thought 提示中。它涉及代理對自身生成內容與中間思考流程的內部評估。這種批判性檢視讓代理能識別其理解或解法中的模糊之處、資訊缺口或不準確之處。這個反覆的檢視與精修循環，使代理能調整方法、改善回應品質，並在交付最終輸出前確保準確性與完整性。這種內部批判提升了代理產生可靠且高品質結果的能力，如專門的第 4 章範例所示。

這個範例展示了一個系統化的自我修正流程，對精修 AI 生成內容至關重要。它包含一個反覆循環：起草、對照原始需求檢視，並實作具體改善。示例一開始先概述 AI 作為「Self-Correction Agent」的功能，並定義五步驟分析與修訂工作流程。接著，呈現一篇品質不佳的社群媒體貼文「Initial Draft」。「Self-Correction Agent's Thought Process」構成示範的核心。在這裡，代理根據指示批判性評估草稿，指出互動吸引力不足、行動呼籲模糊等弱點。然後它提出具體改善，包括使用更有力的動詞與 emoji。流程最後產出「Final Revised Content」，也就是整合自我識別調整後的精修且明顯改善版本。

```markdown
You are a highly critical and detail-oriented Self-Correction Agent. Your task is to review a previously generated piece of content against its original requirements and identify areas for improvement. Your goal is to refine the content to be more accurate, comprehensive, engaging, and aligned with the prompt. 

Here's the process you must follow for self-correction: 

1.  **Understand Original Requirements:** Review the initial prompt/requirements that led to the content's creation. What was the *original intent*? What were the key constraints or goals? 
2.  **Analyze Current Content:** Read the provided content carefully. 
3.  **Identify Discrepancies/Weaknesses:** Compare the current content against the original requirements. Look for:    
    * **Accuracy Issues:** Are there any factual errors or misleading statements?    
    * **Completeness Gaps:** Does it fully address all aspects of the original prompt? Is anything missing?    
    * **Clarity & Coherence:** Is the language clear, concise, and easy to understand? Does it flow logically?    
    * **Tone & Style:** Does it match the desired tone and style (e.g., professional, engaging, concise)?    
    * **Engagement:** Is it captivating? Does it hold the reader's attention?    
    * **Redundancy/Verbosity:** Can any parts be condensed or removed without losing meaning? 
4.  **Propose Specific Improvements:** For each identified weakness, suggest concrete and actionable changes. Do not just state the problem; propose a solution. 
5.  **Generate Revised Content:** Based on your proposed improvements, rewrite the original content to incorporate all the necessary changes. Ensure the revised content is polished and ready for final use. 

**Original Prompt/Requirements:** "Write a short, engaging social media post (max 150 characters) announcing a new eco-friendly product line: 'GreenTech Gadgets'." 

**Initial Draft (Content to be corrected):** "We have new products. They are green and techy. Buy GreenTech Gadgets now!" 

**Self-Correction Agent's Thought Process (Internal CoT Output - encouraged by the prompt):** 

**Thought 1 (Understand Original Requirements):** 
    - Short, engaging social media post. 
    - Max 150 characters. 
    - Announce "GreenTech Gadgets" - a new eco-friendly product line. 

**Thought 2 (Analyze Current Content):** 
"We have new products. They are green and techy. Buy GreenTech Gadgets now!" (64 characters) 

**Thought 3 (Identify Discrepancies/Weaknesses):** - **Engaging:** Not very engaging. Sounds generic. - **Eco-friendly:** Implied by "green," but not explicitly highlighted as a benefit. - **Call to Action:** "Buy now!" is okay, but could be more compelling. - **Conciseness:** It's short, but could use stronger verbs/phrasing. - **Impact:** Lacks excitement for a new product launch. 

**Thought 4 (Propose Specific Improvements):** - Make it more dynamic and exciting. - Emphasize "eco-friendly" and "innovation." - Use stronger verbs and more active voice. - Add a clear, engaging call to action. - Ensure it stays under 150 characters. **Thought 5 (Generate Revised Content):** Original: "We have new products. They are green and techy. Buy GreenTech Gadgets now!" Revised idea 1: "🌱 Discover GreenTech Gadgets! Our new eco-friendly line blends innovation with sustainability. Go green, go smart! Shop now! #EcoFriendly #GreenTech" (148 characters) This revision directly addresses eco-friendliness, uses emojis for engagement, adds a stronger call to action, and includes relevant hashtags while staying within the character limit. **Self-Correction Agent's Final Revised Content (Output to User):** 🌱 Discover GreenTech Gadgets! Our new eco-friendly line blends innovation with sustainability. Go green, go smart! Shop now! #EcoFriendly #GreenTech
```

從根本上說，這項技術將品質控制措施直接整合進代理的內容生成流程，產出更精修、更精確且更優質的結果，也更能有效滿足複雜的使用者需求。

**程式輔助語言模型（Program-Aided Language Models, PALMs）** 將 LLM 與符號推理能力整合。這種整合讓 LLM 能在問題解決流程中生成並執行程式碼，例如 Python。PALMs 會將複雜計算、邏輯運算與資料操作卸載到確定性的程式設計環境。這種方法運用傳統程式設計的強項，處理 LLM 可能在準確性或一致性上受限的任務。面對符號式挑戰時，模型可以產生程式碼、執行程式碼，並將結果轉換成自然語言。這種混合方法結合 LLM 的理解與生成能力，以及精確計算能力，使模型能處理更廣泛的複雜問題，並可能提高可靠性與準確性。這對代理很重要，因為代理能在理解與生成能力之外，運用精確計算來執行更準確且可靠的動作。範例之一是在 Google 的 ADK 中使用外部工具生成程式碼。

```python
from google.adk.tools import agent_tool
from google.adk.agents import Agent
from google.adk.tools import google_search
from google.adk.code_executors import BuiltInCodeExecutor


search_agent = Agent(
    model="gemini-2.0-flash",
    name="SearchAgent",
    instruction="""
    You're a specialist in Google Search
    """,
    tools=[google_search],
)

coding_agent = Agent(
    model="gemini-2.0-flash",
    name="CodeAgent",
    instruction="""
    You're a specialist in Code Execution
    """,
    code_executor=BuiltInCodeExecutor(),
)

root_agent = Agent(
    name="RootAgent",
    model="gemini-2.0-flash",
    description="Root Agent",
    tools=[
        agent_tool.AgentTool(agent=search_agent),
        agent_tool.AgentTool(agent=coding_agent),
    ],
)
```

**具可驗證獎勵的強化學習（Reinforcement Learning with Verifiable Rewards, RLVR）：** 雖然有效，但許多 LLM 使用的標準思維鏈（CoT）提示，是一種相對基礎的推理方法。它會生成單一、預先決定的思路，而不會依問題複雜度調整。為了克服這些限制，已經發展出一類新的專門「推理模型」。這些模型的運作方式不同：它們會在提供答案前投入可變長度的「思考」時間。這個「思考」流程會產生更長、更動態的思維鏈，長度可達數千個 token。這種延伸推理允許更複雜的行為，例如自我修正與回溯，模型也會對較困難的問題投入更多心力。啟用這些模型的關鍵創新，是一種稱為 Reinforcement Learning from Verifiable Rewards（RLVR）的訓練策略。透過在已知正確答案的問題（例如數學或程式碼）上訓練模型，模型能透過試誤學會產生有效的長篇推理。這讓模型能在沒有直接人類監督的情況下，演化其問題解決能力。最終，這些推理模型不只是產生答案；它們會生成一條「推理軌跡」，展現規劃、監控與評估等進階技能。這種強化的推理與策略能力，是自主 AI 代理發展的根本，因為它們能在極少人類介入下拆解並解決複雜任務。

**ReAct**（Reasoning and Acting，見圖 3，其中 KB 代表 Knowledge Base）是一種典範，將思維鏈（CoT）提示與代理透過工具和外部環境互動的能力整合在一起。不同於生成式模型直接產生最終答案，ReAct 代理會推理應採取哪些動作。這個推理階段包含類似 CoT 的內部規劃流程，代理會決定下一步、考量可用工具，並預期結果。接著，代理會透過執行工具或函式呼叫來採取行動，例如查詢資料庫、進行計算，或與 API 互動。

![REACT: Reasoning and Act](../assets/REACT_Reasoning_and_Act.png)

圖 3：Reasoning and Act

ReAct 以交錯方式運作：代理執行動作、觀察結果，並將此觀察納入後續推理。這個「思考、動作、觀察、思考……」的反覆迴圈，讓代理能動態調整計畫、修正錯誤，並達成需要與環境多次互動的目標。相較於線性 CoT，這提供了更穩健且彈性的問題解決方法，因為代理會回應即時回饋。透過結合語言模型的理解與生成，以及使用工具的能力，ReAct 讓代理能執行同時需要推理與實際執行的複雜任務。這種方法對代理至關重要，因為它讓代理不只能推理，也能實際執行步驟並與動態環境互動。

**CoD**（Chain of Debates）是 Microsoft 提出的正式 AI 框架，讓多個不同模型協作與辯論來解決問題，超越單一 AI 的「思維鏈」。這個系統的運作方式就像 AI 委員會會議，不同模型會提出初始想法、批判彼此的推理，並交換反駁論點。主要目標是運用集體智慧來提升準確性、降低偏誤，並改善最終答案的整體品質。這種方法像是 AI 版同儕審查，會建立一份透明且可信的推理流程紀錄。最終，它代表從單一代理提供答案，轉向由代理協作團隊共同尋找更穩健且經驗證解法的轉變。

**GoD**（Graph of Debates）是一種進階代理式框架，它將討論重新想像為動態、非線性的網路，而不是簡單鏈條。在這個模型中，論點是個別節點，並由邊連接；這些邊代表「支持」或「反駁」等關係，反映真實辯論的多執行緒本質。這種結構讓新的探究路線能動態分支、獨立演進，甚至隨時間合併。結論不是在序列末端得出，而是透過識別整張圖中最穩健、最有充分支持的論點群集來形成。在此脈絡中，「有充分支持」指的是已穩固建立且可驗證的知識。這可以包含被視為 ground truth 的資訊，也就是本質上正確且被廣泛接受為事實的資訊。此外，它也涵蓋透過搜尋 grounding 取得的事實證據，也就是將資訊與外部來源及真實世界資料進行驗證。最後，它也指多個模型在辯論期間達成的共識，表示對所呈現資訊有高度一致與信心。這種全面方法確保被討論資訊具備更穩健且可靠的基礎。這種方法為複雜、協作式 AI 推理提供了更整體且更符合現實的模型。

**MASS（選讀進階主題）：** 深入分析多代理系統的設計可以發現，其成效高度取決於兩項因素：用來設定個別代理的提示品質，以及決定代理互動方式的拓撲。設計這類系統相當複雜，因為它牽涉龐大且錯綜複雜的搜尋空間。為了解決這項挑戰，研究者開發了一個稱為 Multi-Agent System Search（MASS）的新框架，用於自動化並最佳化 MAS 的設計。

MASS 採用多階段最佳化策略，透過交錯進行提示與拓撲最佳化，有系統地巡覽複雜設計空間（見圖 4）。

**1. 區塊層級提示最佳化：** 流程從個別代理類型或「區塊」的提示局部最佳化開始，確保每個組件在整合到更大系統前，都能有效執行其角色。這個初始步驟很重要，因為它確保後續拓撲最佳化是建立在表現良好的代理之上，而不是受到設定不良代理的複合影響。例如，針對 HotpotQA 資料集進行最佳化時，「Debator」代理的提示會被創意地設計為要求它扮演「大型出版物的專家事實查核員」。其最佳化任務是仔細審查其他代理提出的答案、將其與提供的上下文段落交叉比對，並識別任何不一致或缺乏支持的主張。這個在區塊層級最佳化期間發現的專門角色扮演提示，旨在讓 debator 代理在被放入更大工作流程前，就能高度有效地綜合資訊。

**2. 工作流程拓撲最佳化：** 完成局部最佳化後，MASS 會透過從可自訂設計空間中選擇並排列不同代理互動，來最佳化工作流程拓撲。為了讓搜尋更有效率，MASS 採用影響力加權方法。此方法會衡量各拓撲相對於基準代理的效能提升，計算其「增量影響力」，並使用這些分數引導搜尋朝向更有前景的組合。例如，針對 MBPP 程式設計任務進行最佳化時，拓撲搜尋發現某個特定混合工作流程最有效。找到的最佳拓撲不是簡單結構，而是結合外部工具使用的反覆精修流程。具體而言，它包含一個 predictor 代理進行數輪反思，而其程式碼由一個 executor 代理針對測試案例執行並驗證。這個被發現的工作流程顯示，對程式設計而言，結合反覆自我修正與外部驗證的結構，優於較簡單的 MAS 設計。

![MASS: Multi-Agent System Search](../assets/MASS_Multi_Agent_System_Search.png)

圖 4：（作者提供）：Multi-Agent System Search（MASS）框架是一個三階段最佳化流程，會巡覽一個搜尋空間，該空間涵蓋可最佳化的提示（指示與示範）以及可設定的代理建構區塊（Aggregate、Reflect、Debate、Summarize 與 Tool-use）。第一階段「區塊層級提示最佳化」會獨立最佳化每個代理模組的提示。第二階段「工作流程拓撲最佳化」會從影響力加權設計空間中取樣有效的系統組態，並整合已最佳化的提示。最後階段「工作流程層級提示最佳化」則是在識別第二階段的最佳工作流程後，對整個多代理系統進行第二輪提示最佳化。

**3. 工作流程層級提示最佳化：** 最後階段涉及整個系統提示的全域最佳化。在識別表現最佳的拓撲後，提示會作為單一整合實體進行微調，以確保它們適合協調，並使代理之間的相依關係最佳化。舉例來說，在找到 DROP 資料集的最佳拓撲後，最終最佳化階段會精修「Predictor」代理的提示。最終最佳化提示非常詳細，一開始會提供代理資料集本身的摘要，指出其重點在於「擷取式問答」與「數值資訊」。接著，它包含正確問答行為的少樣本範例，並將核心指示設定為高風險情境：「你是一個高度專精的 AI，負責為緊急新聞報導擷取關鍵數值資訊。即時廣播仰賴你的準確性與速度」。這個多面向提示結合後設知識、範例與角色扮演，並專門針對最終工作流程調校，以最大化準確性。

主要發現與原則：實驗顯示，由 MASS 最佳化的 MAS，在一系列任務中顯著優於既有手動設計系統與其他自動化設計方法。此研究歸納出有效 MAS 的三項關鍵設計原則：

* 先使用高品質提示最佳化個別代理，再將它們組合起來。  
* 透過組合具影響力的拓撲來建構 MAS，而不是探索不受限制的搜尋空間。  
* 透過最後的工作流程層級聯合最佳化，建模並最佳化代理之間的相依關係。

在討論完關鍵推理技術後，讓我們先檢視一項核心效能原則：LLM 的推論擴展定律（Scaling Inference Law）。這項定律指出，隨著分配給模型的運算資源增加，模型效能會以可預期的方式改善。我們可以在 Deep Research 這類複雜系統中看到這項原則的實際運作：AI 代理會運用這些資源，自主地將主題拆解成子問題、使用 Web 搜尋作為工具，並綜合其發現來調查一個主題。

**Deep Research。** 「Deep Research」一詞描述一類代理式 AI 工具，這些工具旨在扮演不知疲倦且有條理的研究助理。此領域的主要平台包括 Perplexity AI、Google 的 Gemini 研究能力，以及 OpenAI 在 ChatGPT 中的進階功能（見圖 5）。

![Google Deep Research for Information Gathering](../assets/Google_Deep_Research_for_Information_Gathering.png)

圖 5：Google Deep Research 用於資訊蒐集

這些工具帶來的根本轉變，是搜尋流程本身的改變。標準搜尋會立即提供連結，將綜合整理的工作留給你。Deep Research 採用不同模型。在這裡，你交給 AI 一個複雜查詢，並給它一段「時間預算」——通常是幾分鐘。作為耐心等待的回報，你會收到一份詳細報告。

在這段時間裡，AI 會以代理式方式代表你工作。它會自主執行一系列精密步驟，而這些步驟若由人來做會極為耗時：

1. 初始探索：它會根據你的初始提示執行多個具目標性的搜尋。  
2. 推理與精修：它會閱讀並分析第一波結果、綜合發現，並批判性地識別缺口、矛盾，或需要更多細節的區域。  
3. 後續探詢：根據其內部推理，它會進行新的、更細緻的搜尋，以填補那些缺口並加深理解。  
4. 最終綜合：經過數輪反覆搜尋與推理後，它會將所有經驗證的資訊彙整成單一、連貫且結構化的摘要。

這種系統化方法確保回應全面且經過充分推理，顯著提升資訊蒐集的效率與深度，進而促進更代理式的決策。

## 推論擴展定律

這項關鍵原則規範 LLM 效能與其在作業階段（稱為推論）所分配運算資源之間的關係。推論擴展定律不同於較為人熟悉的訓練擴展定律；後者聚焦於模型建立期間，模型品質如何隨資料量與運算能力增加而改善。相對地，這項定律專門檢視 LLM 主動生成輸出或答案時所發生的動態取捨。

這項定律的基石，是揭示相對較小的 LLM 經常可透過增加推論時間的運算投入，達成更好的結果。這不一定代表使用更強大的 GPU，而是採用更精密或更耗資源的推論策略。這類策略的一個典型例子，是指示模型產生多個潛在答案——也許透過 diverse beam search 或 self-consistency 等技術——再使用選擇機制找出最佳輸出。這種反覆精修或多候選生成流程需要更多運算週期，但能顯著提升最終回應品質。

這項原則為部署代理系統時做出明智且具經濟效益的決策，提供了關鍵框架。它挑戰了「較大模型一定會帶來較好效能」的直覺觀念。這項定律主張，較小模型若在推論期間獲得更充足的「思考預算」，有時能超越仰賴較簡單、較低運算密集生成流程的大型模型。這裡的「思考預算」指的是推論期間額外套用的運算步驟或複雜演算法，使較小模型能探索更廣泛的可能性，或在定案答案前套用更嚴格的內部檢查。

因此，推論擴展定律成為建構高效率且具成本效益之代理式系統的基礎。它提供一種方法，用於仔細平衡幾個相互關聯的因素：

* **模型大小：** 較小模型本質上對記憶體與儲存空間的需求較低。  
* **回應延遲：** 雖然增加推論期間的運算可能提高延遲，但這項定律有助於識別效能增益超過延遲增加的臨界點，或說明如何策略性地套用運算以避免過度延遲。  
* **營運成本：** 部署並執行較大型模型通常會因耗電與基礎設施需求增加，而產生較高的持續營運成本。這項定律展示如何在不必要地推高成本的情況下最佳化效能。

透過理解並套用推論擴展定律，開發者與組織可以做出策略性選擇，為特定代理式應用帶來最佳效能，確保運算資源被分配到最能影響 LLM 輸出品質與效用的地方。這讓 AI 部署能採取更細緻且經濟可行的方法，超越單純「越大越好」的典範。

## 動手寫程式範例

Google 開源的 DeepSearch 程式碼可透過 gemini-fullstack-langgraph-quickstart 儲存庫取得（圖 6）。此儲存庫提供一個範本，讓開發者使用 Gemini 2.5 與 LangGraph 協調框架建構全端 AI 代理。這個開源技術堆疊便於實驗代理式架構，也可與 Gemma 等本機 LLM 整合。它使用 Docker 與模組化專案鷹架來快速原型開發。需要注意的是，此版本是一個結構良好的示範，不是可直接用於生產環境的後端。

![Example of DeepSearch with multiple Reflection Steps](../assets/Example_of_DeepSearch_with_multiple_Reflection_Steps.png)


圖 6：（作者提供）具多個反思步驟的 DeepSearch 範例

此專案提供一個全端應用程式，包含 React 前端與 LangGraph 後端，專為進階研究與對話式 AI 設計。LangGraph 代理會使用 Google Gemini 模型動態生成搜尋查詢，並透過 Google Search API 整合 Web 研究。系統採用反思式推理來識別知識缺口、反覆精修搜尋，並產生附引用的答案。前端與後端支援 hot-reloading。專案結構包含獨立的 frontend/ 與 backend/ 目錄。設定需求包括 Node.js、npm、Python 3.8+ 與 Google Gemini API key。在後端的 .env 檔中設定 API key 後，可以安裝後端（使用 pip install .）與前端（npm install）的相依套件。開發伺服器可使用 make dev 同時執行，也可個別執行。後端代理定義於 backend/src/agent/graph.py，會生成初始搜尋查詢、進行 Web 研究、執行知識缺口分析、反覆精修查詢，並使用 Gemini 模型綜合附引用的答案。生產部署包含由後端伺服器提供靜態前端建置，並需要 Redis 串流即時輸出，以及 Postgres 資料庫管理資料。可使用 docker-compose up 建置並執行 Docker image；docker-compose.yml 範例也需要 LangSmith API key。此應用程式使用 React 搭配 Vite、Tailwind CSS、Shadcn UI、LangGraph 與 Google Gemini。專案採用 Apache License 2.0 授權。

| ``# Create our Agent Graph builder = StateGraph(OverallState, config_schema=Configuration) # Define the nodes we will cycle between builder.add_node("generate_query", generate_query) builder.add_node("web_research", web_research) builder.add_node("reflection", reflection) builder.add_node("finalize_answer", finalize_answer) # Set the entrypoint as `generate_query` # This means that this node is the first one called builder.add_edge(START, "generate_query") # Add conditional edge to continue with search queries in a parallel branch builder.add_conditional_edges(    "generate_query", continue_to_web_research, ["web_research"] ) # Reflect on the web research builder.add_edge("web_research", "reflection") # Evaluate the research builder.add_conditional_edges(    "reflection", evaluate_research, ["web_research", "finalize_answer"] ) # Finalize the answer builder.add_edge("finalize_answer", END) graph = builder.compile(name="pro-search-agent")`` |
| :---- |

圖 4：使用 LangGraph 的 DeepSearch 範例（程式碼來自 backend/src/agent/graph.py）

## 那麼，代理會思考什麼？

總結來說，代理的思考流程是一種結合推理與行動來解決問題的結構化方法。這種方法讓代理能明確規劃步驟、監控進度，並與外部工具互動以蒐集資訊。

在核心上，代理的「思考」由強大的 LLM 促成。這個 LLM 會產生一系列想法，引導代理後續動作。流程通常遵循思考—動作—觀察迴圈：

1. **思考：** 代理首先生成一段文字形式的想法，用來拆解問題、制定計畫，或分析目前情境。這種內部獨白讓代理的推理流程透明且可引導。  
2. **動作：** 根據想法，代理會從預先定義的離散選項集合中選擇一個動作。例如在問答情境中，動作空間可能包括線上搜尋、從特定網頁擷取資訊，或提供最終答案。  
3. **觀察：** 接著，代理會根據已採取的動作，從環境接收回饋。這可能是 Web 搜尋結果，也可能是網頁內容。

這個循環會重複進行，每一次觀察都會影響下一個想法，直到代理判定已達成最終解法並執行「finish」動作。

這種方法的有效性取決於底層 LLM 的進階推理與規劃能力。為了引導代理，ReAct 框架通常採用少樣本學習，提供 LLM 類似人類問題解決軌跡的範例。這些範例示範如何有效結合思考與動作，以解決類似任務。

代理思考的頻率可依任務調整。對於像事實查核這類知識密集型推理任務，思考通常會與每個動作交錯，以確保資訊蒐集與推理具備邏輯流。相較之下，對於需要許多動作的決策任務，例如在模擬環境中導航，思考可能會較少使用，讓代理自行決定何時需要思考。

## 一覽

**是什麼**：複雜問題解決通常不只需要單一、直接的答案，這對 AI 構成重大挑戰。核心問題在於如何讓 AI 代理處理需要邏輯推論、拆解與策略規劃的多步驟任務。若沒有結構化方法，代理可能無法處理複雜細節，導致結論不準確或不完整。這些進階推理方法旨在讓代理的內部「思考」流程明確化，使其能有系統地處理挑戰。

**為什麼：** 標準化解法是一套推理技術，為代理的問題解決流程提供結構化框架。像 Chain-of-Thought（CoT）與 Tree-of-Thought（ToT）這類方法，會引導 LLM 拆解問題並探索多條解決路徑。自我修正允許答案反覆精修，確保更高準確性。ReAct 等代理式框架將推理與動作整合，讓代理能與外部工具和環境互動，以蒐集資訊並調整計畫。這種明確推理、探索、精修與工具使用的組合，會創造更穩健、透明且有能力的 AI 系統。

**經驗法則：** 當問題過於複雜，無法用單次通過答案解決，並需要拆解、多步驟邏輯、與外部資料來源或工具互動，或需要策略規劃與調適時，就使用這些推理技術。對於「展示工作過程」或思考流程與最終答案同樣重要的任務，它們尤其理想。

**視覺摘要：**

![Reasoning Design Pattern](../assets/Reasoning_Design_Pattern.png)

圖 7：推理設計模式

## 重點整理

* 透過讓推理明確化，代理可以制定透明的多步驟計畫，而這是自主行動與使用者信任的基礎能力。  
* ReAct 框架為代理提供核心操作迴圈，使它們能超越單純推理，並與外部工具互動，在環境中動態行動與調適。  
* 推論擴展定律意味著，代理的效能不只取決於底層模型大小，也取決於分配到的「思考時間」，進而支援更審慎且更高品質的自主動作。  
* Chain-of-Thought（CoT）作為代理的內部獨白，提供一種結構化方式，將複雜目標拆成一連串可管理動作，以制定計畫。  
* Tree-of-Thought 與自我修正賦予代理審慎思考的關鍵能力，讓它們能評估多種策略、從錯誤中回溯，並在執行前改善自己的計畫。  
* Chain of Debates（CoD）等協作框架，標誌著從單一代理轉向多代理系統的轉變；在多代理系統中，代理團隊可以共同推理，以處理更複雜的問題並降低個別偏誤。  
* Deep Research 等應用展示了這些技術如何匯聚成能代表使用者完全自主執行複雜、長時間任務（例如深入調查）的代理。  
* 若要建立有效的代理團隊，MASS 等框架會自動化最佳化個別代理的指示方式，以及它們彼此互動的方式，確保整個多代理系統發揮最佳表現。  
* 透過整合這些推理技術，我們建立的代理不只是自動化，而是真正自主，能被信任在沒有直接監督的情況下規劃、行動並解決複雜問題。

## 結論

現代 AI 正從被動工具演進為自主代理，能透過結構化推理處理複雜目標。這種代理式行為始於內部獨白，並由 Chain-of-Thought（CoT）等技術驅動，使代理能在行動前制定連貫計畫。真正的自主性需要審慎思考，而代理透過自我修正與 Tree-of-Thought（ToT）達成這一點，使它們能評估多種策略並獨立改善自己的工作。邁向完全代理式系統的關鍵躍進來自 ReAct 框架，它賦予代理使用外部工具、超越思考並開始行動的能力。這建立了思考、動作與觀察的核心代理式迴圈，讓代理能根據環境回饋動態調整策略。

代理進行深度審慎思考的能力，由推論擴展定律提供動能；在這項定律中，更多運算「思考時間」會直接轉化為更穩健的自主動作。下一個前沿是多代理系統，Chain of Debates（CoD）等框架會建立協作式代理社群，讓它們共同推理以達成共同目標。這不是理論；Deep Research 等代理式應用已經展示自主代理如何代表使用者執行複雜的多步驟調查。整體目標是打造可靠且透明的自主代理，使其能被信任獨立管理並解決錯綜複雜的問題。最終，透過結合明確推理與行動能力，這些方法正在完成 AI 向真正代理式問題解決者的轉型。

## 參考資料

相關研究包括：

1. "Chain-of-Thought Prompting Elicits Reasoning in Large Language Models" by Wei et al. (2022)  
2. "Tree of Thoughts: Deliberate Problem Solving with Large Language Models" by Yao et al. (2023)  
3. "Program-Aided Language Models" by Gao et al. (2023)  
4. "ReAct: Synergizing Reasoning and Acting in Language Models" by Yao et al. (2023)  
5. Inference Scaling Laws: An Empirical Analysis of Compute-Optimal Inference for LLM Problem-Solving, 2024  
6. Multi-Agent Design: Optimizing Agents with Better Prompts and Topologies, [https://arxiv.org/abs/2502.02533](https://arxiv.org/abs/2502.02533)

