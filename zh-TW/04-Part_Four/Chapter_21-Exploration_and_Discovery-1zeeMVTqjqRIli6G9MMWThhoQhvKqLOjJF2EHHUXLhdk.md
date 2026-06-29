# 第 21 章：探索與發現（Exploration and Discovery）

本章探討能讓代理（agent）主動尋找新資訊、發掘新可能，並在其運作環境中辨識「未知的未知」的模式。探索與發現不同於反應式行為，也不同於在預先定義的解決方案空間中進行最佳化。它們關注的是代理主動進入不熟悉的領域、嘗試新方法，並產生新的知識或理解。對於在開放式、複雜或快速演變的領域中運作的代理而言，這個模式至關重要，因為靜態知識或預先程式化的解法往往並不足夠。它強調代理擴展自身理解與能力的能力。

## 實務應用與使用案例

AI 代理具備智慧化優先排序與探索的能力，因此能應用於各種領域。透過自主評估並排序潛在行動，這些代理可以在複雜環境中導航、發掘隱藏洞見，並推動創新。這種優先探索的能力，使它們能夠最佳化流程、發現新知識並產生內容。

範例：

* **科學研究自動化：** 代理設計並執行實驗、分析結果，並提出新的假設，以發現新材料、候選藥物或科學原理。  
* **遊戲遊玩與策略生成：** 代理探索遊戲狀態，發現浮現出的策略，或識別遊戲環境中的弱點（例如 AlphaGo）。  
* **市場研究與趨勢偵測：** 代理掃描非結構化資料（社群媒體、新聞、報告），以識別趨勢、消費者行為或市場機會。  
* **安全性弱點發現：** 代理探測系統或程式碼庫，以找出安全缺陷或攻擊向量。  
* **創意內容生成：** 代理探索風格、主題或資料的組合，以生成藝術作品、音樂創作或文學作品。  
* **個人化教育與訓練：** AI 家教會根據學生的進度、學習風格與需要改進的領域，優先安排學習路徑與內容傳遞。

Google Co-Scientist

AI co-scientist 是由 Google Research 開發的 AI 系統，設計目標是作為運算式科學協作者。它協助人類科學家處理研究面向，例如假設生成、提案精修與實驗設計。此系統運作於 Gemini 大型語言模型（large language model, LLM）之上。

AI co-scientist 的開發旨在回應科學研究中的挑戰，包括處理大量資訊、產生可測試的假設，以及管理實驗規劃。AI co-scientist 透過執行涉及大規模資訊處理與綜合的任務來支援研究人員，並可能揭示資料中的關係。它的目的，是透過處理早期研究中對運算要求較高的部分，增強人類的認知流程。

**系統架構與方法論：** AI co-scientist 的架構以多代理（multi-agent）框架為基礎，設計上用來模擬協作與反覆迭代的流程。此設計整合多個專門的 AI 代理，每個代理在研究目標中扮演特定角色。監督代理會在非同步任務執行框架中管理並協調這些個別代理的活動，讓運算資源能夠彈性擴展。

核心代理及其功能包括（見圖 1）：

* **生成代理**：透過文獻探索與模擬科學辯論產生初始假設，藉此啟動流程。  
* **反思代理**：扮演同儕審查者，批判性評估所生成假設的正確性、新穎性與品質。  
* **排名代理**：採用以 Elo 為基礎的競賽，透過模擬科學辯論比較、排名並優先排序假設。  
* **演化代理**：透過簡化概念、綜合想法與探索非傳統推理，持續精修排名靠前的假設。  
* **鄰近代理**：計算鄰近圖，以聚類相似想法，並協助探索假設景觀。  
* **後設審查代理**：綜合所有審查與辯論中的洞見，識別共同模式並提供回饋，使系統能持續改進。

系統的運作基礎仰賴 Gemini，提供語言理解、推理與生成能力。系統納入「test-time compute scaling」機制，配置更多運算資源，以反覆推理並強化輸出。系統會處理並綜合來自多種來源的資訊，包括學術文獻、網路資料與資料庫。

![AI Co-Scientist：從構想到驗證](../assets/AI_Co_Scientist_Ideation_to_Validation.png)

圖 1：（作者提供）AI Co-Scientist：從構想到驗證

系統採取反覆迭代的「生成、辯論與演化」方法，對應科學方法的流程。當人類科學家輸入科學問題後，系統會進入一個自我改進循環，進行假設生成、評估與精修。假設會經過系統化評估，包括代理之間的內部評估，以及以競賽為基礎的排名機制。

**驗證與結果：** AI co-scientist 的實用性已在多項驗證研究中展現，特別是在生物醫學領域，透過自動化基準測試、專家審查與端到端濕實驗來評估其表現。

**自動化與專家評估：** 在具有挑戰性的 GPQA 基準測試中，系統的內部 Elo 評分被證明與其結果準確度一致，在困難的「diamond set」上達到 78.4% 的 top-1 準確率。針對超過 200 個研究目標的分析顯示，擴展 test-time compute 能持續改善假設品質，並可由 Elo 評分衡量。在一組精心整理的 15 個挑戰性問題上，AI co-scientist 的表現優於其他最先進的 AI 模型，以及人類專家提供的「最佳猜測」解法。在一項小規模評估中，生物醫學專家認為，與其他基準模型相比，co-scientist 的輸出更具新穎性與影響力。系統針對藥物重新定位所提出、以 NIH Specific Aims 頁面格式撰寫的提案，也被六位腫瘤學專家組成的小組評定為高品質。

**端到端實驗驗證：**

藥物重新定位：針對急性骨髓性白血病（AML），系統提出了新的候選藥物。其中有些像 KIRA6，是完全新穎的建議，先前沒有用於 AML 的臨床前證據。後續體外實驗證實，KIRA6 與其他建議藥物在多個 AML 細胞株中，能以具臨床相關性的濃度抑制腫瘤細胞存活率。

 新標的發現：系統識別出肝纖維化的新表觀遺傳標的。使用人類肝臟類器官的實驗室實驗驗證了這些發現，顯示針對建議表觀遺傳調節因子的藥物具有顯著抗纖維化活性。其中一種被識別出的藥物已獲 FDA 核准用於另一種疾病，因而開啟重新定位的機會。

抗菌素抗藥性：AI co-scientist 獨立重現了尚未發表的實驗發現。它被指派解釋為何某些可移動遺傳元件（cf-PICIs）會出現在許多細菌物種中。兩天內，系統排名最高的假設是 cf-PICIs 會與多樣化的噬菌體尾部互動，以擴大其宿主範圍。這呼應了一個獨立研究團隊在十多年研究後才達成、且已由實驗驗證的新發現。

**增強與限制：** AI co-scientist 背後的設計哲學強調的是增強，而不是完全自動化人類研究。研究人員透過自然語言與系統互動並引導系統，提供回饋、貢獻自己的想法，並在「scientist-in-the-loop」的協作範式中指導 AI 的探索流程。然而，此系統仍有一些限制。它的知識受限於對開放取用文獻的依賴，可能遺漏付費牆後方的重要先前研究。它也較難取得負面實驗結果；這些結果很少發表，但對有經驗的科學家而言至關重要。此外，系統也承襲底層 LLM 的限制，包括可能出現事實不準確或「幻覺」。

**安全性：** 安全性是關鍵考量，而系統也納入多重防護措施。所有研究目標在輸入時都會經過安全審查，生成的假設也會被檢查，以避免系統被用於不安全或不道德的研究。一項使用 1,200 個對抗性研究目標的初步安全評估發現，系統能穩健地拒絕危險輸入。為確保負責任的開發，系統正透過 Trusted Tester Program 開放給更多科學家使用，以蒐集真實世界的回饋。

## 實作程式碼範例

讓我們看一個代理式（agentic）AI 如何實際用於探索與發現的具體範例：Agent Laboratory，這是 Samuel Schmidgall 依 MIT License 開發的專案。

「Agent Laboratory」是一個自主研究工作流程框架，設計目標是增強人類的科學工作，而不是取代人類。此系統運用專門的 LLM，自動化科學研究流程的各個階段，讓人類研究人員能把更多認知資源投入概念化與批判性分析。

此框架整合了「AgentRxiv」，這是一個為自主研究代理而設的去中心化儲存庫。AgentRxiv 促進研究輸出的存放、檢索與開發

Agent Laboratory 透過不同階段引導研究流程：

1. **文獻回顧：** 在這個初始階段，由 LLM 驅動的專門代理負責自主蒐集並批判性分析相關學術文獻。這涉及運用 arXiv 等外部資料庫，識別、綜合並分類相關研究，進而為後續階段有效建立完整知識基礎。  
2. **實驗：** 此階段涵蓋實驗設計的協作制定、資料準備、實驗執行與結果分析。代理會使用整合工具，例如用於程式碼生成與執行的 Python，以及用於模型存取的 Hugging Face，以進行自動化實驗。系統設計支援反覆精修，讓代理能根據即時結果調整並最佳化實驗程序。  
3. **報告撰寫：** 在最後階段，系統會自動生成完整的研究報告。這包括將實驗階段的發現與文獻回顧中的洞見加以綜合、依照學術慣例組織文件，並整合 LaTeX 等外部工具，以產生專業格式與圖表。  
4. **知識分享**：AgentRxiv 是一個讓自主研究代理能分享、存取並協作推進科學發現的平台。它允許代理建立在既有發現之上，促進累積性的研究進展。

Agent Laboratory 的模組化架構確保了運算彈性。其目標是在維持人類研究人員參與的同時，透過自動化任務提升研究生產力。

**程式碼分析：** 雖然完整的程式碼分析超出本書範圍，但我想提供一些關鍵洞見，並鼓勵你自行深入研究程式碼。

**判斷：** 為了模擬人類評估流程，系統採用三方代理式判斷機制來評估輸出。這涉及部署三個不同的自主代理，每個代理都被設定為從特定觀點評估產出，藉此共同模擬人類判斷細緻且多面向的特性。這種方法不只依賴單一指標，而是能捕捉更豐富的質性評估，進而形成更穩健且全面的評價。

```python
class ReviewersAgent:
    def __init__(self, model="gpt-4o-mini", notes=None, openai_api_key=None):
        if notes is None:
            self.notes = []
        else:
            self.notes = notes
        self.model = model
        self.openai_api_key = openai_api_key

    def inference(self, plan, report):
        reviewer_1 = "You are a harsh but fair reviewer and expect good experiments that lead to insights for the research topic."
        review_1 = get_score(
            outlined_plan=plan,
            latex=report,
            reward_model_llm=self.model,
            reviewer_type=reviewer_1,
            openai_api_key=self.openai_api_key
        )

        reviewer_2 = "You are a harsh and critical but fair reviewer who is looking for an idea that would be impactful in the field."
        review_2 = get_score(
            outlined_plan=plan,
            latex=report,
            reward_model_llm=self.model,
            reviewer_type=reviewer_2,
            openai_api_key=self.openai_api_key
        )

        reviewer_3 = "You are a harsh but fair open-minded reviewer that is looking for novel ideas that have not been proposed before."
        review_3 = get_score(
            outlined_plan=plan,
            latex=report,
            reward_model_llm=self.model,
            reviewer_type=reviewer_3,
            openai_api_key=self.openai_api_key
        )

        return f"Reviewer #1:\n{review_1}, \nReviewer #2:\n{review_2}, \nReviewer #3:\n{review_3}"
```

判斷代理會搭配特定提示來設計，此提示緊密模擬人類審查者通常採用的認知框架與評估標準。這個提示引導代理以類似人類專家的視角分析輸出，考量相關性、連貫性、事實準確性與整體品質等因素。透過設計這些提示以對應人類審查流程，系統旨在達到接近人類辨識能力的評估精細度。

````python
def get_score(outlined_plan, latex, reward_model_llm, reviewer_type=None, attempts=3, openai_api_key=None):
   e = str()
   for _attempt in range(attempts):
       try:
          
           template_instructions = """
           Respond in the following format:

           THOUGHT:
           <THOUGHT>

           REVIEW JSON:
           ```json
           <JSON>
           ```

           In <THOUGHT>, first briefly discuss your intuitions 
           and reasoning for the evaluation.
           Detail your high-level arguments, necessary choices 
           and desired outcomes of the review.
           Do not make generic comments here, but be specific 
           to your current paper.
           Treat this as the note-taking phase of your review.

           In <JSON>, provide the review in JSON format with 
           the following fields in the order:
           - "Summary": A summary of the paper content and 
           its contributions.
           - "Strengths": A list of strengths of the paper.
           - "Weaknesses": A list of weaknesses of the paper.
           - "Originality": A rating from 1 to 4 
             (low, medium, high, very high).
           - "Quality": A rating from 1 to 4 
             (low, medium, high, very high).
           - "Clarity": A rating from 1 to 4 
             (low, medium, high, very high).
           - "Significance": A rating from 1 to 4 
             (low, medium, high, very high).
           - "Questions": A set of clarifying questions to be
              answered by the paper authors.
           - "Limitations": A set of limitations and potential
              negative societal impacts of the work.
           - "Ethical Concerns": A boolean value indicating 
              whether there are ethical concerns.
           - "Soundness": A rating from 1 to 4 
              (poor, fair, good, excellent).
           - "Presentation": A rating from 1 to 4 
              (poor, fair, good, excellent).
           - "Contribution": A rating from 1 to 4 
             (poor, fair, good, excellent).
           - "Overall": A rating from 1 to 10 
             (very strong reject to award quality).
           - "Confidence": A rating from 1 to 5 
             (low, medium, high, very high, absolute).
           - "Decision": A decision that has to be one of the
             following: Accept, Reject.

           For the "Decision" field, don't use Weak Accept,   
           Borderline Accept, Borderline Reject, or Strong Reject.  
           Instead, only use Accept or Reject.
           This JSON will be automatically parsed, so ensure 
           the format is precise.
           """
````

在這個多代理系統中，研究流程圍繞專門角色建構，並映照典型的學術層級，以簡化工作流程並最佳化輸出。

**Professor Agent：** Professor Agent 作為主要研究主持人，負責建立研究議程、定義研究問題，並將任務委派給其他代理。這個代理設定策略方向，並確保與專案目標一致。

````python
class ProfessorAgent(BaseAgent):
   def __init__(self, model="gpt4omini", notes=None, max_steps=100, openai_api_key=None):
       super().__init__(model, notes, max_steps, openai_api_key)
       self.phases = ["report writing"]

   def generate_readme(self):
       sys_prompt = f"""You are {self.role_description()} \n Here is the written paper \n{self.report}. Task instructions: Your goal is to integrate all of the knowledge, code, reports, and notes provided to you and generate a readme.md for a github repository."""
       history_str = "\n".join([_[1] for _ in self.history])
       prompt = (
           f"""History: {history_str}\n{'~' * 10}\n"""
           f"Please produce the readme below in markdown:\n")
       model_resp = query_model(model_str=self.model, system_prompt=sys_prompt, prompt=prompt, openai_api_key=self.openai_api_key)
       return model_resp.replace("```markdown", "")
````

**PostDoc Agent：** PostDoc Agent 的角色是執行研究。這包括進行文獻回顧、設計並實作實驗，以及產出論文等研究輸出。重要的是，PostDoc Agent 具備撰寫與執行程式碼的能力，使實驗流程與資料分析能夠實際落地。這個代理是研究成果的主要產出者。

```python
class PostdocAgent(BaseAgent):
    def __init__(self, model="gpt4omini", notes=None, max_steps=100, openai_api_key=None):
        super().__init__(model, notes, max_steps, openai_api_key)
        self.phases = ["plan formulation", "results interpretation"]

    def context(self, phase):
        sr_str = str()
        if self.second_round:
            sr_str = (
                f"The following are results from the previous experiments\n",
                f"Previous Experiment code: {self.prev_results_code}\n"
                f"Previous Results: {self.prev_exp_results}\n"
                f"Previous Interpretation of results: {self.prev_interpretation}\n"
                f"Previous Report: {self.prev_report}\n"
                f"{self.reviewer_response}\n\n\n"
            )

        if phase == "plan formulation":
            return (
                sr_str,
                f"Current Literature Review: {self.lit_review_sum}",
            )
        elif phase == "results interpretation":
            return (
                sr_str,
                f"Current Literature Review: {self.lit_review_sum}\n"
                f"Current Plan: {self.plan}\n"
                f"Current Dataset code: {self.dataset_code}\n"
                f"Current Experiment code: {self.results_code}\n"
                f"Current Results: {self.exp_results}"
            )

        return ""
```

**Reviewer Agents：** Reviewer agents 會對 PostDoc Agent 的研究輸出進行批判性評估，評估論文與實驗結果的品質、有效性與科學嚴謹度。這個評估階段模擬學術環境中的同儕審查流程，以確保研究輸出在定稿前達到高標準。

**ML Engineering Agents**：Machine Learning Engineering Agents 扮演機器學習工程師，與博士生進行對話式協作以開發程式碼。其核心功能是根據所提供的文獻回顧與實驗流程產生簡單的資料前處理程式碼。這可確保資料已針對指定實驗進行適當格式化與準備。

```markdown
"You are a machine learning engineer being directed by a PhD student who will help you write the code, and you can interact with them through dialogue.\n"
"Your goal is to produce code that prepares the data for the provided experiment. You should aim for simple code to prepare the data, not complex code. You should integrate the provided literature review and the plan and come up with code to prepare data for this experiment.\n"
```

**SWEngineerAgents：** Software Engineering Agents 會引導 Machine Learning Engineer Agents。它們的主要目的是協助 Machine Learning Engineer Agent 為特定實驗建立直接明瞭的資料準備程式碼。Software Engineer Agent 會整合所提供的文獻回顧與實驗計畫，確保生成的程式碼簡單，且與研究目標直接相關。

```markdown
"You are a software engineer directing a machine learning engineer, where the machine learning engineer will be writing the code, and you can interact with them through dialogue.\n"
"Your goal is to help the ML engineer produce code that prepares the data for the provided experiment. You should aim for very simple code to prepare the data, not complex code. You should integrate the provided literature review and the plan and come up with code to prepare data for this experiment.\n"
```

總結來說，「Agent Laboratory」代表了一個成熟的自主科學研究框架。它旨在透過自動化關鍵研究階段，並促進由 AI 驅動的協作式知識生成，來增強人類研究能力。系統的目標是在維持人類監督的同時管理例行任務，進而提升研究效率。

## 概覽

**是什麼：** AI 代理通常在預先定義的知識範圍內運作，這會限制它們處理新情境或開放式問題的能力。在複雜且動態的環境中，這種靜態、預先程式化的資訊不足以支撐真正的創新或發現。根本挑戰在於讓代理超越單純最佳化，主動尋找新資訊並識別「未知的未知」。這需要從純粹反應式行為轉向主動的代理式探索，藉此擴展系統自身的理解與能力。

**為什麼：** 標準化的解法，是建構專為自主探索與發現而設計的代理式 AI 系統。這些系統通常使用多代理框架，讓專門的 LLM 協作，以模擬科學方法等流程。例如，不同代理可以分別負責生成假設、批判性審查假設，以及演化最有前景的概念。這種結構化、協作式的方法，讓系統能智慧地探索龐大的資訊景觀、設計並執行實驗，並產生真正的新知識。透過自動化探索中耗費大量人力的部分，這些系統能增強人類智慧，並大幅加快發現的速度。

**經驗法則：** 當你在開放式、複雜或快速演變的領域中運作，且解決方案空間尚未完全定義時，請使用探索與發現模式。它非常適合需要生成新假設、新策略或新洞見的任務，例如科學研究、市場分析與創意內容生成。當目標是揭示「未知的未知」，而不只是最佳化已知流程時，這個模式就十分關鍵。

**視覺摘要：**

![探索與發現設計模式](../assets/Exploration_and_Discovery_Design_Pattern.png)

圖 2：探索與發現設計模式

## 重點整理

* AI 中的探索與發現讓代理能主動追求新資訊與新可能，這對於在複雜且不斷演變的環境中導航至關重要。  
* Google Co-Scientist 等系統展示了代理如何自主生成假設並設計實驗，以補充人類的科學研究。  
* 以 Agent Laboratory 的專門角色為代表的多代理框架，透過自動化文獻回顧、實驗與報告撰寫來改善研究流程。  
* 最終，這些代理的目標是透過管理運算密集型任務來強化人類創造力與問題解決能力，進而加速創新與發現。

## 結論

總結而言，探索與發現模式是真正代理式系統的核心本質，定義了系統超越被動遵循指令、主動探索其環境的能力。這種內在的代理式驅動力，使 AI 能在複雜領域中自主運作，不只是執行任務，也能獨立設定子目標以發掘新資訊。這種進階的代理式行為，最能透過多代理框架發揮威力；在其中，每個代理都在更大的協作流程中承擔特定且主動的角色。例如，Google Co-scientist 這個高度代理式系統中的代理，能自主生成、辯論並演化科學假設。

Agent Laboratory 等框架進一步透過建立模仿人類研究團隊的代理式層級來組織這件事，使系統能自我管理整個發現生命週期。此模式的核心，在於協調浮現的代理式行為，讓系統能在最低限度的人類介入下追求長期、開放式目標。這提升了人類與 AI 的夥伴關係，使 AI 成為真正的代理式協作者，負責自主執行探索性任務。透過將這類主動發現工作委派給代理式系統，人類智慧能獲得顯著增強，進而加速創新。這類強大代理式能力的發展，也需要對安全性與倫理監督有堅定承諾。最終，這個模式提供了建立真正代理式 AI 的藍圖，將運算工具轉化為追求知識時獨立且以目標為導向的夥伴。

## 參考資料

1. Exploration-Exploitation Dilemma**：** 強化學習與不確定性下決策中的一個根本問題。[https://en.wikipedia.org/wiki/Exploration%E2%80%93exploitation_dilemma](https://en.wikipedia.org/wiki/Exploration%E2%80%93exploitation_dilemma)
2. Google Co-Scientist：[https://research.google/blog/accelerating-scientific-breakthroughs-with-an-ai-co-scientist/](https://research.google/blog/accelerating-scientific-breakthroughs-with-an-ai-co-scientist/)
3. Agent Laboratory: Using LLM Agents as Research Assistants [https://github.com/SamuelSchmidgall/AgentLaboratory](https://github.com/SamuelSchmidgall/AgentLaboratory)
4. AgentRxiv: Towards Collaborative Autonomous Research: [https://agentrxiv.github.io/](https://agentrxiv.github.io/)