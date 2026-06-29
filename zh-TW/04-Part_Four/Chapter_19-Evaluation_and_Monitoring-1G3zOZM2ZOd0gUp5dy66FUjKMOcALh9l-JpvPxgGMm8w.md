# 第 19 章：評估與監控

本章探討可讓代理（agent）有系統地評估自身效能、監控目標進度，並偵測作業異常的方法。第 11 章概述目標設定與監控，第 17 章則討論推理機制；本章聚焦於持續且通常來自外部的量測，用來評估代理的有效性、效率，以及是否符合需求。這包括定義指標、建立回饋迴路，以及實作報告系統，以確保代理在作業環境中的表現符合預期（見圖 1）。

![Monitoring and Evaluating Agent Performance](../assets/Monitoring_and_Evaluating_Agent_Performance.png)

圖 1：評估與監控的最佳實務

## 實務應用與使用案例

最常見的應用與使用案例：

* **即時系統中的效能追蹤：** 持續監控部署於生產環境中的代理之準確率、延遲與資源消耗（例如客服聊天機器人的解決率、回應時間）。  
* **代理改進的 A/B 測試：** 以平行方式系統性比較不同代理版本或策略的表現，以找出最佳方法（例如為物流代理嘗試兩種不同的規劃演算法）。  
* **合規與安全稽核：** 產生自動化稽核報告，長期追蹤代理是否遵循倫理準則、法規要求與安全協定。這些報告可由人類介入或另一個代理驗證，也可在發現問題時產生 KPI 或觸發警示。  
* **企業系統：** 為了在企業系統中治理代理式 AI，需要一種新的控制工具：AI「契約」。這項動態協議會將 AI 委派任務的目標、規則與控制措施編碼化。  
* **漂移偵測：** 長期監控代理輸出的相關性或準確性，偵測其表現是否因輸入資料分布改變（概念漂移）或環境變化而下降。  
* **代理行為中的異常偵測：** 識別代理採取的不尋常或非預期動作，這些動作可能代表錯誤、惡意攻擊，或浮現出的非預期行為。  
* **學習進度評估：** 對設計為會學習的代理，追蹤其學習曲線、特定技能的進步，或在不同任務或資料集上的泛化能力。

## 動手寫程式範例

為 AI 代理開發完整的評估框架是一項艱鉅工作，其複雜度可比擬一門學術領域或一部大型著作。困難來自於必須考量眾多因素，例如模型效能、使用者互動、倫理影響，以及更廣泛的社會影響。不過，在實務實作上，可以將焦點縮小到對 AI 代理高效率且有效運作至關重要的關鍵使用案例。

**代理回應評估：** 這個核心流程對於評估代理輸出的品質與準確性至關重要。它涉及判斷代理是否能針對給定輸入，提供相關、正確、合乎邏輯、不偏頗且準確的資訊。評估指標可包括事實正確性、流暢度、文法精確度，以及是否符合使用者的預期目的。

```python
def evaluate_response_accuracy(agent_output: str, expected_output: str) -> float:
    """Calculates a simple accuracy score for agent responses."""
    # This is a very basic exact match; real-world would use more sophisticated metrics
    return 1.0 if agent_output.strip().lower() == expected_output.strip().lower() else 0.0


# Example usage
agent_response = "The capital of France is Paris."
ground_truth = "Paris is the capital of France."
score = evaluate_response_accuracy(agent_response, ground_truth)
print(f"Response accuracy: {score}")
```

Python 函式 `evaluate_response_accuracy` 會在移除前後空白後，對代理輸出與預期輸出進行不區分大小寫的精確比較，藉此計算 AI 代理回應的基本準確度分數。若完全相符則回傳 1.0，否則回傳 0.0，代表二元的正確或錯誤評估。這種方法雖然很適合簡單檢查，但無法處理改寫或語意等價等變化。

問題在於其比較方法。此函式會對兩個字串執行嚴格的逐字元比較。在所提供的範例中：

* `agent_response`："The capital of France is Paris."  
* `ground_truth`："Paris is the capital of France."

即使移除空白並轉成小寫，這兩個字串仍不相同。因此，該函式會錯誤地回傳準確度分數 `0.0`，即使兩個句子傳達的是相同意思。

直接比較不足以評估語意相似度，只有在代理回應與預期輸出完全相同時才會成功。更有效的評估需要進階自然語言處理（Natural Language Processing, NLP）技術來辨識句子之間的意義。若要在真實情境中完整評估 AI 代理，通常不可或缺更精細的指標。這些指標可包含字串相似度量測，例如 Levenshtein distance 與 Jaccard similarity；用來判斷特定關鍵字是否存在的關鍵字分析；使用 embedding 模型與 cosine similarity 的語意相似度；LLM-as-a-Judge 評估（稍後討論，用於評估細緻的正確性與有用性）；以及 RAG 特定指標，例如忠實度與相關性。

**延遲監控：** 針對代理動作的延遲監控，對於 AI 代理回應或動作速度是關鍵因素的應用至關重要。此流程會量測代理處理請求並產生輸出所需的時間。延遲升高可能對使用者體驗與代理整體有效性造成負面影響，尤其是在即時或互動式環境中。在實務應用中，僅將延遲資料列印到主控台並不足夠。建議將這些資訊記錄到持久化儲存系統。可選項包括結構化記錄檔（例如 JSON）、時間序列資料庫（例如 InfluxDB、Prometheus）、資料倉儲（例如 Snowflake、BigQuery、PostgreSQL），或可觀測性平台（例如 Datadog、Splunk、Grafana Cloud）。

**追蹤 LLM 互動的 token 使用量：** 對於由 LLM 驅動的代理，追蹤 token 使用量對於管理成本與最佳化資源配置至關重要。LLM 互動的計費通常取決於處理的 token 數量（輸入與輸出）。因此，具效率的 token 使用會直接降低營運費用。此外，監控 token 數量也有助於找出提示工程或回應生成流程中可改進的潛在區域。

```python
# This is conceptual as actual token counting depends on the LLM API
class LLMInteractionMonitor:
    def __init__(self):
        self.total_input_tokens = 0
        self.total_output_tokens = 0

    def record_interaction(self, prompt: str, response: str):
        # In a real scenario, use LLM API's token counter or a tokenizer
        input_tokens = len(prompt.split())  # Placeholder
        output_tokens = len(response.split())  # Placeholder
        self.total_input_tokens += input_tokens
        self.total_output_tokens += output_tokens
        print(f"Recorded interaction: Input tokens={input_tokens}, Output tokens={output_tokens}")

    def get_total_tokens(self):
        return self.total_input_tokens, self.total_output_tokens


# Example usage
monitor = LLMInteractionMonitor()
monitor.record_interaction("What is the capital of France?", "The capital of France is Paris.")
monitor.record_interaction("Tell me a joke.", "Why don't scientists trust atoms? Because they make up everything!")
input_t, output_t = monitor.get_total_tokens()
print(f"Total input tokens: {input_t}, Total output tokens: {output_t}")
```

本節介紹一個概念性的 Python 類別 `LLMInteractionMonitor`，用來追蹤大型語言模型（large language model, LLM）互動中的 token 使用量。此類別包含輸入與輸出 token 的計數器。其 `record_interaction` 方法透過分割提示與回應字串來模擬 token 計數。在實務實作中，會使用特定 LLM API 的 tokenizer 來取得精確的 token 數量。隨著互動發生，監控器會累計輸入與輸出 token 的總數。`get_total_tokens` 方法可存取這些累計總數，這對 LLM 使用的成本管理與最佳化至關重要。

**使用 LLM-as-a-Judge 的「有用性」自訂指標：** 評估 AI 代理的「有用性」等主觀品質，會遇到超出標準客觀指標的挑戰。一種可行框架是使用 LLM 作為評估者。這種 LLM-as-a-Judge 方法會根據預先定義的「有用性」準則，評估另一個 AI 代理的輸出。藉由運用 LLM 的進階語言能力，這種方法能提供細緻、類似人類的主觀品質評估，超越簡單的關鍵字比對或規則式評估。雖然仍在發展中，這項技術已展現出自動化與擴展質性評估的潛力。

```python
import os
import json
import logging
from typing import Optional

import google.generativeai as genai

# --- Configuration ---
logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')

# Set your API key as an environment variable to run this script
# For example, in your terminal: export GOOGLE_API_KEY='your_key_here'
try:
    genai.configure(api_key=os.environ["GOOGLE_API_KEY"])
except KeyError:
    logging.error("Error: GOOGLE_API_KEY environment variable not set.")
    exit(1)

# --- LLM-as-a-Judge Rubric for Legal Survey Quality ---
LEGAL_SURVEY_RUBRIC = """
 You are an expert legal survey methodologist and a critical legal reviewer. Your task is to evaluate the quality of a given legal survey question. Provide a score from 1 to 5 for overall quality, along with a detailed rationale and specific feedback.

 Focus on the following criteria:

 1.  **Clarity & Precision (Score 1-5):**
    * 1: Extremely vague, highly ambiguous, or confusing.
    * 3: Moderately clear, but could be more precise.
    * 5: Perfectly clear, unambiguous, and precise in its legal terminology (if applicable) and intent.

 2.  **Neutrality & Bias (Score 1-5):**
    * 1: Highly leading or biased, clearly influencing the respondent towards a specific answer.
    * 3: Slightly suggestive or could be interpreted as leading.
    * 5: Completely neutral, objective, and free from any leading language or loaded terms.

 3.  **Relevance & Focus (Score 1-5):**
    * 1: Irrelevant to the stated survey topic or out of scope.
    * 3: Loosely related but could be more focused.
    * 5: Directly relevant to the survey's objectives and well-focused on a single concept.

 4.  **Completeness (Score 1-5):**
    * 1: Omits critical information needed to answer accurately or provides insufficient context.
    * 3: Mostly complete, but minor details are missing.
    * 5: Provides all necessary context and information for the respondent to answer thoroughly.

 5.  **Appropriateness for Audience (Score 1-5):**
    * 1: Uses jargon inaccessible to the target audience or is overly simplistic for experts.
    * 3: Generally appropriate, but some terms might be challenging or oversimplified.
    * 5: Perfectly tailored to the assumed legal knowledge and background of the target survey audience.

 **Output Format:**
 Your response MUST be a JSON object with the following keys:
 * `overall_score`: An integer from 1 to 5 (average of criterion scores, or your holistic judgment).
 * `rationale`: A concise summary of why this score was given, highlighting major strengths and weaknesses.
 * `detailed_feedback`: A bullet-point list detailing feedback for each criterion (Clarity, Neutrality, Relevance, Completeness, Audience Appropriateness). Suggest specific improvements.
 * `concerns`: A list of any specific legal, ethical, or methodological concerns.
 * `recommended_action`: A brief recommendation (e.g., "Revise for neutrality", "Approve as is", "Clarify scope").
"""

class LLMJudgeForLegalSurvey:
    """A class to evaluate legal survey questions using a generative AI model."""

    def __init__(self, model_name: str = 'gemini-1.5-flash-latest', temperature: float = 0.2):
        """
        Initializes the LLM Judge.

        Args:
            model_name (str): The name of the Gemini model to use.
                              'gemini-1.5-flash-latest' is recommended for speed and cost.
                              'gemini-1.5-pro-latest' offers the highest quality.
            temperature (float): The generation temperature. Lower is better for deterministic evaluation.
        """
        self.model = genai.GenerativeModel(model_name)
        self.temperature = temperature

    def _generate_prompt(self, survey_question: str) -> str:
        """Constructs the full prompt for the LLM judge."""
        return f"{LEGAL_SURVEY_RUBRIC}\n\n---\n**LEGAL SURVEY QUESTION TO EVALUATE:**\n{survey_question}\n---"

    def judge_survey_question(self, survey_question: str) -> Optional[dict]:
        """
        Judges the quality of a single legal survey question using the LLM.

        Args:
            survey_question (str): The legal survey question to be evaluated.

        Returns:
            Optional[dict]: A dictionary containing the LLM's judgment, or None if an error occurs.
        """
        full_prompt = self._generate_prompt(survey_question)

        try:
            logging.info(f"Sending request to '{self.model.model_name}' for judgment...")
            response = self.model.generate_content(
                full_prompt,
                generation_config=genai.types.GenerationConfig(
                    temperature=self.temperature,
                    response_mime_type="application/json"
                )
            )

            # Check for content moderation or other reasons for an empty response.
            if not response.parts:
                safety_ratings = response.prompt_feedback.safety_ratings
                logging.error(f"LLM response was empty or blocked. Safety Ratings: {safety_ratings}")
                return None

            return json.loads(response.text)
        except json.JSONDecodeError:
            logging.error(f"Failed to decode LLM response as JSON. Raw response: {response.text}")
            return None
        except Exception as e:
            logging.error(f"An unexpected error occurred during LLM judgment: {e}")
            return None


# --- Example Usage ---
if __name__ == "__main__":
    judge = LLMJudgeForLegalSurvey()

    # --- Good Example ---
    good_legal_survey_question = """
    To what extent do you agree or disagree that current intellectual property laws in Switzerland adequately protect emerging AI-generated content, assuming the content meets the originality criteria established by the Federal Supreme Court?
    (Select one: Strongly Disagree, Disagree, Neutral, Agree, Strongly Agree)
    """
    print("\n--- Evaluating Good Legal Survey Question ---")
    judgment_good = judge.judge_survey_question(good_legal_survey_question)
    if judgment_good:
        print(json.dumps(judgment_good, indent=2))

    # --- Biased/Poor Example ---
    biased_legal_survey_question = """
    Don't you agree that overly restrictive data privacy laws like the FADP are hindering essential technological innovation and economic growth in Switzerland?
    (Select one: Yes, No)
    """
    print("\n--- Evaluating Biased Legal Survey Question ---")
    judgment_biased = judge.judge_survey_question(biased_legal_survey_question)
    if judgment_biased:
        print(json.dumps(judgment_biased, indent=2))

    # --- Ambiguous/Vague Example ---
    vague_legal_survey_question = """
    What are your thoughts on legal tech?
    """
    print("\n--- Evaluating Vague Legal Survey Question ---")
    judgment_vague = judge.judge_survey_question(vague_legal_survey_question)
    if judgment_vague:
        print(json.dumps(judgment_vague, indent=2))
```

這段 Python 程式碼定義了 LLMJudgeForLegalSurvey 類別，旨在使用生成式 AI 模型評估法律問卷題目的品質。它使用 google.`generativeai` 程式庫與 Gemini 模型互動。

其核心功能是將問卷題目連同詳細的評分規準傳送給模型進行評估。該規準指定五項用於評判問卷題目的標準：清晰度與精確度、中立性與偏誤、相關性與焦點、完整性，以及對受眾的適切性。每項標準都會給予 1 到 5 分，並要求在輸出中提供詳細理由與回饋。程式碼會建構一個包含評分規準與待評估問卷題目的提示。

`judge_survey_question` 方法會將此提示傳送給已設定的 Gemini 模型，要求依照定義好的結構回傳 JSON 回應。預期輸出的 JSON 包含整體分數、理由摘要、各項標準的詳細回饋、疑慮清單，以及建議採取的行動。此類別會處理 AI 模型互動期間可能發生的錯誤，例如 JSON 解碼問題或空白回應。此腳本透過評估法律問卷題目的範例來示範其運作，說明 AI 如何根據預先定義的標準評估品質。

在結束之前，讓我們檢視各種評估方法，並考量它們的優點與弱點。

| 評估方法 | 優點 | 弱點 |
| :---- | :---- | :---- |
| 人類評估 | 能捕捉細微行為 | 難以擴展、成本高且耗時，因為它會納入主觀的人類因素。 |
| LLM-as-a-Judge | 一致、高效率且可擴展。 | 可能忽略中間步驟。受限於 LLM 能力。 |
| 自動化指標 | 可擴展、高效率且客觀 | 在捕捉完整能力方面可能有所限制。 |

## 代理軌跡

評估代理的軌跡相當重要，因為傳統軟體測試並不足夠。標準程式碼會產生可預測的通過／失敗結果，而代理以機率方式運作，因此必須對最終輸出與代理軌跡進行質性評估，也就是評估代理為了達成解決方案所採取的一連串步驟。評估多代理系統很具挑戰，因為它們會持續變動。這需要開發更精細的指標，超越個別表現，進一步量測溝通與團隊合作的有效性。此外，環境本身也不是靜態的，因此包括測試案例在內的評估方法也必須隨時間調整。

這涉及檢視決策品質、推理流程與整體結果。實作自動化評估很有價值，尤其適用於超越原型階段的開發。分析軌跡與工具使用包括評估代理為達成目標所採用的步驟，例如工具選擇、策略與任務效率。舉例來說，處理客戶產品查詢的代理，理想上可能會遵循包含意圖判定、使用資料庫搜尋工具、檢視結果與產生報告的軌跡。代理的實際動作會與這條預期或真實基準軌跡比較，以找出錯誤與低效率之處。比較方法包括精確匹配（要求與理想序列完全一致）、依序匹配（正確動作依序出現，允許額外步驟）、任意順序匹配（正確動作以任意順序出現，允許額外步驟）、精確率（量測預測動作的相關性）、召回率（量測捕捉到多少必要動作），以及單一工具使用（檢查是否執行特定動作）。指標選擇取決於特定代理需求；高風險情境可能要求精確匹配，而較有彈性的情境則可能使用依序匹配或任意順序匹配。

AI 代理評估主要涉及兩種方法：使用 test files 與使用 evalset files。Test files 採 JSON 格式，代表單一、簡單的代理與模型互動或 session，適合在積極開發期間進行單元測試，重點在於快速執行與簡單的 session 複雜度。每個 test file 都包含一個具有多個 turns 的單一 session，其中 turn 是一次使用者與代理互動，包含使用者查詢、預期工具使用軌跡、中間代理回應，以及最終回應。例如，test file 可能詳細描述使用者要求「Turn off `device_2` in the Bedroom」，並指定代理使用 `set_device_info` 工具及其參數，例如 location: Bedroom、`device_id: device_2` 與 status: OFF，以及預期最終回應「I have set the `device_2` status to off.」。Test files 可整理到資料夾中，並可包含 `test_config`.json 檔案來定義評估標準。Evalset files 使用名為「evalset」的資料集來評估互動，其中包含多個可能較長的 sessions，適合模擬複雜的多輪對話與整合測試。Evalset file 由多個「evals」組成，每個 eval 代表一個不同的 session，且包含一個或多個「turns」，其中包含使用者查詢、預期工具使用、中間回應，以及參考最終回應。範例 evalset 可能包含一個 session：使用者先問「What can you do?」，接著說「Roll a 10 sided dice twice and then check if 9 is a prime or not」，並定義預期的 `roll_die` 工具呼叫與 `check_prime` 工具呼叫，以及總結擲骰結果與質數檢查的最終回應。

**多代理**：評估具有多個代理的複雜 AI 系統，很像評估一個團隊專案。由於其中有許多步驟與交接，這種複雜性反而是一項優勢，讓你能檢查每個階段的工作品質。你可以檢視每個個別「代理」在其特定工作上的表現，但也必須評估整個系統作為整體的表現。

為了做到這一點，你會針對團隊動態提出關鍵問題，並搭配具體範例：

* 這些代理是否有效合作？例如，在「Flight-Booking Agent」訂好航班後，是否能成功將正確日期與目的地傳給「Hotel-Booking Agent」？合作失敗可能導致飯店被訂在錯誤的週次。  
* 它們是否建立了良好計畫並遵循該計畫？想像計畫是先訂航班，再訂飯店。如果「Hotel Agent」在航班確認前就嘗試訂房，就偏離了計畫。你也會檢查代理是否卡住，例如不斷搜尋「完美」租車選項，卻永遠不進入下一步。  
* 是否為正確任務選擇了正確代理？如果使用者詢問旅程中的天氣，系統應使用能提供即時資料的專門「Weather Agent」。若它改用「General Knowledge Agent」並給出「夏天通常很暖」之類的通用答案，就代表為這項工作選錯了工具。  
* 最後，加入更多代理是否能提升效能？如果你把新的「Restaurant-Reservation Agent」加入團隊，它是否讓整體旅程規劃變得更好、更有效率？還是它造成衝突並拖慢系統，顯示可擴展性出現問題？

## 從代理到進階承包者

最近，有人提出（Agent Companion, gulli et al.）從簡單 AI 代理演進為進階「承包者」的概念，從機率式且往往不可靠的系統，轉向更具確定性與責任性的系統，專為複雜且高風險的環境而設計（見圖 2）。

今日常見的 AI 代理仰賴簡短且規格不足的指令，因此適合簡單展示，但在生產環境中相當脆弱，因為模糊性會導致失敗。「承包者」模型透過在使用者與 AI 之間建立嚴謹且形式化的關係來解決這個問題，其基礎是清楚定義且雙方同意的條款，類似人類世界中的法律服務協議。這項轉變由四項關鍵支柱支撐，共同確保清晰度、可靠性，以及對過去超出自主系統範圍之任務的穩健執行能力。

第一項是形式化契約支柱，也就是作為任務單一事實來源的詳細規格。它遠遠超越簡單提示。例如，財務分析任務的契約不會只寫「分析上一季銷售額」；它會要求「一份 20 頁 PDF 報告，分析 2025 年第 1 季歐洲市場銷售，包含五個特定資料視覺化、與 2024 年第 1 季的比較分析，以及根據所附供應鏈中斷資料集進行的風險評估。」這份契約會明確定義所需交付項目、精確規格、可接受資料來源、工作範圍，甚至預期運算成本與完成時間，使結果能被客觀驗證。

第二項是協商與回饋的動態生命週期支柱。契約不是靜態命令，而是對話的開始。承包者代理可以分析初始條款並進行協商。例如，如果契約要求使用代理無法存取的特定專有資料來源，代理可以回傳回饋：「指定的 XYZ 資料庫無法存取。請提供憑證，或核准使用替代的公開資料庫；這可能會稍微改變資料粒度。」這個協商階段也允許代理標示模糊處或潛在風險，能在執行開始前解決誤解，避免代價高昂的失敗，並確保最終輸出完全符合使用者的實際意圖。

![Contract Execution Example Among Agents](../assets/Contract_Execution_Example_Among_Agents.png)

圖 2：代理之間的契約執行範例

第三項支柱是以品質為焦點的迭代執行。不同於設計目標是低延遲回應的代理，承包者會優先考量正確性與品質。它依循自我驗證與修正的原則運作。以程式碼生成契約為例，代理不會只是撰寫程式碼；它會產生多種演算法方法，編譯並針對契約中定義的一組單元測試執行它們，依效能、安全性與可讀性等指標為每個解決方案評分，並只提交通過所有驗證標準的版本。這種在內部循環中持續生成、檢視並改進自身工作，直到符合契約規格為止的做法，對於建立輸出可信度至關重要。

最後，第四項支柱是透過子契約進行階層式分解。對於高度複雜的任務，主要承包者代理可以扮演專案經理，將主要目標拆解成更小、更容易管理的子任務。它透過產生新的形式化「子契約」來達成這一點。例如，一份「建立電子商務行動應用程式」的主契約，可由主要代理分解為「設計 UI/UX」、「開發使用者驗證模組」、「建立產品資料庫綱要」與「整合支付閘道」等子契約。每份子契約都是完整且獨立的契約，擁有自己的交付項目與規格，並可指派給其他專門代理。這種結構化分解讓系統能以高度組織化且可擴展的方式處理龐大、多面向的專案，標誌著 AI 從簡單工具轉變為真正自主且可靠的問題解決引擎。

最終，這個承包者框架透過將形式化規格、協商與可驗證執行的原則直接嵌入代理核心邏輯，重新想像 AI 互動。這種有條理的方法，將人工智慧從有前景但往往難以預測的助理，提升為能以可稽核精確度自主管理複雜專案的可靠系統。透過解決模糊性與可靠性這些關鍵挑戰，此模型為在信任與責任性至關重要的任務關鍵領域部署 AI 鋪平道路。

## Google 的 ADK

在總結之前，讓我們看看一個支援評估的具體框架範例。使用 Google 的 ADK 進行代理評估（見圖 3）可透過三種方法執行：用於互動式評估與資料集生成的網頁式 UI（adk web）、使用 pytest 納入測試管線的程式化整合，以及適合一般建置生成與驗證流程之自動化評估的直接命令列介面（adk eval）。

![Evaluation Support for Google ADK](../assets/Evaluation_Support_for_Google_ADK.png)

圖 3：Google ADK 的評估支援

網頁式 UI 可建立互動式 session，並將其儲存到既有或新的 eval sets 中，同時顯示評估狀態。Pytest 整合可透過呼叫 AgentEvaluator.evaluate，指定代理模組與 test file 路徑，讓 test files 作為整合測試的一部分執行。

命令列介面可透過提供代理模組路徑與 eval set file 來促成自動化評估，並提供指定設定檔或列印詳細結果的選項。在較大的 eval set 中，可以在 eval set 檔名後方以逗號分隔列出特定 evals，以選擇要執行的項目。

## 概覽

**內容：** 代理式系統與 LLM 在複雜且動態的環境中運作，其效能可能隨時間下降。其機率性與非決定性本質代表傳統軟體測試不足以確保可靠性。評估動態多代理系統是一項重大挑戰，因為它們與其環境都會不斷變化，因此需要開發自適應測試方法與精細指標，量測超越個別表現的協作成功。部署後可能出現資料漂移、非預期互動、工具呼叫，以及偏離預定目標等問題。因此，持續評估是必要的，用以衡量代理的有效性、效率，以及是否遵循作業與安全要求。

**原因：** 標準化的評估與監控框架提供一種系統化方式，用來評估並確保代理的持續表現。這涉及為準確率、延遲與資源消耗定義清楚指標，例如 LLM 的 token 使用量。它也包含進階技術，例如分析代理式軌跡以理解推理流程，以及採用 LLM-as-a-Judge 進行細緻的質性評估。透過建立回饋迴路與報告系統，此框架可支援持續改進、A/B 測試，以及異常或效能漂移偵測，確保代理持續與其目標保持一致。

**經驗法則：** 當你在即時、生產環境中部署代理，且即時效能與可靠性至關重要時，使用此模式。此外，當需要系統性比較代理不同版本或其底層模型以推動改進時，以及在受監管或高風險領域中運作、需要合規、安全與倫理稽核時，也應使用此模式。若代理效能可能因資料或環境變化（漂移）而隨時間下降，或需要評估複雜代理式行為，包括動作序列（軌跡）與「有用性」等主觀輸出品質，此模式也很適合。

**視覺摘要：**

![Evaluation and Monitoring Design Pattern](../assets/Evaluation_and_Monitoring_Design_Pattern.png)

圖 4：評估與監控設計模式

## 重點摘要

* 評估代理不只限於傳統測試，而是要在真實環境中持續衡量其有效性、效率，以及是否遵循需求。  
* 代理評估的實務應用包括即時系統中的效能追蹤、用於改進的 A/B 測試、合規稽核，以及偵測行為中的漂移或異常。  
* 基本代理評估包含評估回應準確性，而真實情境需要更精細的指標，例如延遲監控，以及針對 LLM 驅動代理的 token 使用量追蹤。  
* 代理軌跡，也就是代理採取的一連串步驟，對評估至關重要；它會將實際動作與理想的真實基準路徑比較，以找出錯誤與低效率之處。  
* ADK 透過個別 test files 進行單元測試，以及透過完整 evalset files 進行整合測試，提供結構化評估方法，兩者都會定義預期的代理行為。  
* 代理評估可透過網頁式 UI 進行互動式測試、以 pytest 程式化整合至 CI/CD，或透過命令列介面執行自動化工作流程。  
* 為了讓 AI 能可靠處理複雜且高風險的任務，我們必須從簡單提示轉向能精確定義可驗證交付項目與範圍的形式化「契約」。這種結構化協議允許代理協商、釐清模糊處，並迭代驗證自身工作，將其從不可預測的工具轉變為負責任且值得信賴的系統。

## 結論

總結來說，要有效評估 AI 代理，需要超越簡單的準確性檢查，轉向在動態環境中對其表現進行持續且多面向的評估。這包括對延遲與資源消耗等指標進行實務監控，也包括透過代理軌跡對其決策流程進行精細分析。對於「有用性」等細緻品質，LLM-as-a-Judge 這類創新方法正變得不可或缺，而 Google 的 ADK 等框架則為單元測試與整合測試提供結構化工具。多代理系統會讓挑戰更加嚴峻，因為評估焦點會轉向協作成功與有效合作。

為了確保關鍵應用中的可靠性，典範正從由簡單提示驅動的代理，轉向受形式化協議約束的進階「承包者」。這些承包者代理根據明確且可驗證的條款運作，使它們能協商、分解任務，並自我驗證其工作，以符合嚴格的品質標準。這種結構化方法會將代理從不可預測的工具轉變為能處理複雜、高風險任務的負責任系統。最終，這項演進對於建立在任務關鍵領域部署精密代理式 AI 所需的信任至關重要。

## 參考資料

相關研究包括：

1. ADK Web: [https://github.com/google/adk-web](https://github.com/google/adk-web)
2. ADK Evaluate: [https://google.github.io/adk-docs/evaluate/](https://google.github.io/adk-docs/evaluate/)  
3. Survey on Evaluation of LLM-based Agents, [https://arxiv.org/abs/2503.16416](https://arxiv.org/abs/2503.16416)
4. Agent-as-a-Judge: Evaluate Agents with Agents, [https://arxiv.org/abs/2410.10934](https://arxiv.org/abs/2410.10934)
5. Agent Companion, gulli et al: [https://www.kaggle.com/whitepaper-agent-companion](https://www.kaggle.com/whitepaper-agent-companion)
