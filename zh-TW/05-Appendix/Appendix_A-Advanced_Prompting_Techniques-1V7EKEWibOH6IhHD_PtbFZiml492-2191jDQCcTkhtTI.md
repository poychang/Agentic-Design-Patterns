# 附錄 A：進階提示技巧

## 提示簡介

提示是與語言模型互動的主要介面，指的是設計輸入內容，引導模型產生所需輸出。這包括組織請求、提供相關上下文、指定輸出格式，以及示範預期的回應類型。設計良好的提示可以最大化語言模型的潛力，產生準確、相關且具創意的回應。相反地，設計不佳的提示可能導致模糊、不相關或錯誤的輸出。

提示工程的目標，是穩定地從語言模型取得高品質回應。這需要理解模型的能力與限制，並有效傳達預期目標。也就是透過學習如何最好地指示 AI，培養與 AI 溝通的專業能力。

本附錄詳細介紹多種超越基本互動方式的提示技巧。內容探討如何組織複雜請求、增強模型的推理能力、控制輸出格式，以及整合外部資訊。這些技巧可用於建構各類應用，從簡單的聊天機器人到複雜的多代理系統，並能提升代理式應用的效能與可靠性。

代理式模式，也就是用來建構智慧系統的架構結構，已在主要章節中詳述。這些模式定義代理如何規劃、使用工具、管理記憶並進行協作。這些代理式系統的成效，取決於它們能否與語言模型進行有意義的互動。

## 核心提示原則

有效提示語言模型的核心原則：

有效提示建立在與語言模型溝通的基本原則之上，適用於各種模型與不同複雜度的任務。掌握這些原則，是穩定產生有用且準確回應的必要基礎。

**清楚且具體**：指示應明確且精準。語言模型會解讀模式；若存在多種解讀，可能導致非預期回應。請定義任務、所需輸出格式，以及任何限制或需求。避免含糊語言或未明說的假設。不充分的提示會產生模糊且不準確的回應，阻礙有意義的輸出。

**簡潔**：雖然具體很重要，但不應犧牲簡潔性。指示應直接明瞭。不必要的措辭或複雜句構，可能讓模型混淆或掩蓋主要指示。提示應保持簡單；對使用者而言令人困惑的內容，對模型也很可能令人困惑。避免複雜語言與多餘資訊。使用直接措辭與主動動詞，清楚界定所需動作。有效的動詞包括：Act、Analyze、Categorize、Classify、Contrast、Compare、Create、Describe、Define、Evaluate、Extract、Find、Generate、Identify、List、Measure、Organize、Parse、Pick、Predict、Provide、Rank、Recommend、Return、Retrieve、Rewrite、Select、Show、Sort、Summarize、Translate、Write。

**使用動詞：** 動詞選擇是關鍵的提示工具。動作動詞指出預期操作。與其說「想想如何摘要這段內容」，不如直接指示「摘要以下文字」，效果更好。精準的動詞會引導模型啟用與特定任務相關的訓練資料與流程。

**指示優於限制：** 正向指示通常比負面限制更有效。指定想要的動作，比列出不要做什麼更理想。雖然限制在安全性或嚴格格式方面有其用途，但過度依賴限制可能讓模型專注於避免錯誤，而非完成目標。應以直接引導模型的方式設計提示。正向指示符合人類偏好的引導方式，也能減少混淆。

**實驗與迭代：** 提示工程是一個迭代流程。找出最有效提示需要多次嘗試。從草稿開始，測試它、分析輸出、找出缺點，然後調整提示。模型差異、設定（例如 temperature 或 top-p），以及細微措辭變化，都可能產生不同結果。記錄嘗試過程對學習與改進至關重要。若要達到所需效能，實驗與迭代是必要的。

這些原則構成與語言模型有效溝通的基礎。透過優先考量清楚性、簡潔性、動作動詞、正向指示與迭代，就能建立穩健框架，用於套用更進階的提示技巧。

## 基本提示技巧

在核心原則之上，基礎技巧會提供不同程度的資訊或範例給語言模型，以引導其回應。這些方法是提示工程的起始階段，並適用於廣泛的應用情境。

### 零樣本提示

零樣本提示是最基本的提示形式，也就是提供語言模型指示與輸入資料，但不提供任何期望輸入輸出配對的範例。它完全仰賴模型的預訓練能力來理解任務並產生相關回應。本質上，零樣本提示包含任務描述，以及用來開始流程的初始文字。

* **使用時機：** 對於模型在訓練期間很可能已大量見過的任務，零樣本提示通常已足夠，例如簡單問答、文字補全，或直接文字的基本摘要。這是最先嘗試的最快方法。  
* **範例：**  
  Translate the following English sentence to French: 'Hello, how are you?'

### 單樣本提示

單樣本提示是在提出實際任務之前，先提供語言模型一個輸入及其對應期望輸出的範例。這個方法作為初始示範，用來說明模型應複製的模式。其目的是讓模型取得一個具體實例，可作為有效執行指定任務的範本。

* **使用時機：** 當所需輸出格式或風格很具體，或較不常見時，單樣本提示很有用。它提供模型一個可學習的具體實例。對於需要特定結構或語氣的任務，相較於零樣本，它可以改善效能。  
* **範例：**  
  Translate the following English sentences to Spanish:  
  English: 'Thank you.'  
  Spanish: 'Gracias.'

  English: 'Please.'  
  Spanish:

### 少樣本提示

少樣本提示透過提供數個輸入輸出配對範例（通常三到五個）來強化單樣本提示。這旨在更清楚地展示預期回應模式，提高模型對新輸入複製該模式的可能性。這個方法提供多個範例，引導模型遵循特定輸出模式。

* **使用時機：** 當所需輸出必須遵循特定格式、風格，或呈現細緻變化時，少樣本提示特別有效。它很適合分類、依特定 schema 擷取資料，或以特定風格產生文字等任務，尤其在零樣本或單樣本無法產生一致結果時。一般經驗法則是至少使用三到五個範例，並依任務複雜度與模型 token 限制調整。  
* **範例品質與多樣性的重要性：** 少樣本提示的成效高度仰賴所提供範例的品質與多樣性。範例應準確、能代表任務，並涵蓋模型可能遇到的變化或邊界案例。高品質且撰寫良好的範例非常重要；即使是小錯誤也可能混淆模型，導致非預期輸出。納入多樣範例能幫助模型更好地泛化到未見過的輸入。  
* **在分類範例中混合類別：** 將少樣本提示用於分類任務時（模型需要將輸入分類到預先定義的類別），最佳實務是打亂不同類別範例的順序。這可避免模型可能過度擬合範例的特定順序，並確保它能獨立學會辨識各類別的關鍵特徵，進而在未見資料上有更穩健且可泛化的效能。  
* **演進到「多樣本」學習：** 隨著 Gemini 等現代 LLM 在長上下文建模方面變得更強，它們正逐漸能有效運用「多樣本」學習。這表示對於複雜任務，現在可以在提示中直接納入更多範例——有時甚至數百個——來達到最佳效能，讓模型學習更複雜的模式。  
* **範例：**  
  Classify the sentiment of the following movie reviews as POSITIVE, NEUTRAL, or NEGATIVE:

  Review: "The acting was superb and the story was engaging."  
  Sentiment: POSITIVE

  Review: "It was okay, nothing special."  
  Sentiment: NEUTRAL

  Review: "I found the plot confusing and the characters unlikable."  
  Sentiment: NEGATIVE

  Review: "The visuals were stunning, but the dialogue was weak."  
  Sentiment:

理解何時應用零樣本、單樣本與少樣本提示技巧，並審慎設計與組織範例，是提升代理式系統成效的關鍵。這些基本方法是各種提示策略的基礎。

## 組織提示

除了提供範例的基本技巧之外，提示的組織方式也在引導語言模型時扮演關鍵角色。組織提示是指在提示中使用不同區段或元素，以清楚、有條理的方式提供不同類型的資訊，例如指示、上下文或範例。這有助於模型正確解析提示，並理解每段文字的特定角色。

### 系統提示

系統提示會為語言模型設定整體上下文與目的，定義其在一次互動或一個工作階段中的預期行為。這包括提供指示或背景資訊，以建立規則、人格或整體行為。不同於特定使用者查詢，系統提示為模型回應提供基礎準則。它會影響整個互動期間模型的語氣、風格與一般處理方式。例如，系統提示可以指示模型始終以簡潔且有幫助的方式回應，或確保回應適合一般讀者。系統提示也可用於安全性與毒性控制，例如納入維持尊重語言的準則。

此外，為了最大化系統提示的成效，可以透過以 LLM 為基礎的迭代式精煉，對系統提示進行自動提示最佳化。Vertex AI Prompt Optimizer 等服務可根據使用者定義的指標與目標資料，系統性地改進提示，確保指定任務達到盡可能高的效能。

* **範例：**  
  You are a helpful and harmless AI assistant. Respond to all queries in a polite and informative manner. Do not generate content that is harmful, biased, or inappropriate

### 角色提示

角色提示會將特定角色、人格或身分指派給語言模型，通常會搭配系統提示或上下文提示使用。這包括指示模型採用與該角色相關的知識、語氣與溝通風格。例如，「Act as a travel guide」或「You are an expert data analyst」這類提示，會引導模型反映指定角色的觀點與專業。定義角色可提供語氣、風格與聚焦專業的框架，目標是提升輸出的品質與相關性。也可以指定角色內的期望風格，例如「幽默且具啟發性的風格」。

* **範例：**  
  Act as a seasoned travel blogger. Write a short, engaging paragraph about the best hidden gem in Rome.

### 使用分隔符

有效提示需要清楚區分給語言模型的指示、上下文、範例與輸入。分隔符，例如三個反引號 (```)、XML 標籤 (<instruction>, <context>)，或標記 (---)，可用於在視覺與程式層面分隔這些區段。這項做法廣泛用於提示工程，可降低模型誤解的機率，確保它清楚理解提示中各部分的角色。

* **範例：**  
  <instruction>Summarize the following article, focusing on the main arguments presented by the author.</instruction>  
  <article>  
  [Insert the full text of the article here]  
  </article>

## 上下文工程

上下文工程不同於靜態系統提示，它會動態提供對任務與對話至關重要的背景資訊。這些不斷變化的資訊可幫助模型掌握細微差異、回想過去互動，並整合相關細節，進而產生有根據的回應與更順暢的交流。範例包括先前對話、相關文件（如檢索增強生成），或特定操作參數。舉例來說，在討論日本旅行時，使用者可能會根據既有對話上下文，要求推薦三個東京適合家庭的活動。在代理式系統中，上下文工程是記憶持久化、決策制定與跨子任務協調等核心代理行為的基礎。具備動態上下文管線的代理能長時間維持目標、調整策略，並與其他代理或工具無縫協作——這些都是長期自主性不可或缺的特質。這套方法認為，模型輸出的品質更多取決於所提供上下文的豐富程度，而非模型架構本身。它代表從傳統提示工程的一項重大演進；傳統提示工程主要聚焦於最佳化即時使用者查詢的措辭，而上下文工程則將範圍擴展到多層資訊。

這些層次包括：

* **系統提示：** 定義 AI 操作參數的基礎指示（例如：「You are a technical writer; your tone must be formal and precise」）。  
* **外部資料：**  
  * **檢索文件：** 從知識庫主動擷取、用來支援回應的資訊（例如擷取技術規格）。  
  * **工具輸出：** AI 使用外部 API 取得即時資料的結果（例如查詢行事曆可用時段）。  
* **隱含資料：** 使用者身分、互動歷史與環境狀態等關鍵資訊。納入隱含上下文會帶來與隱私及倫理資料管理相關的挑戰。因此，穩健的治理對上下文工程至關重要，尤其是在企業、醫療保健與金融等領域。

核心原則是：即使是進階模型，若對其操作環境的觀察有限或建構不佳，也會表現不佳。這項做法將任務從單純回答問題，重新框定為為代理建立完整的操作圖像。例如，一個經過上下文工程設計的代理，在回覆查詢之前，會整合使用者的行事曆可用性（工具輸出）、與電子郵件收件人的專業關係（隱含資料），以及先前會議的筆記（檢索文件）。這讓模型能產生高度相關、個人化且實用的輸出。「工程」層面則涉及建立穩健管線，在執行期間擷取並轉換這些資料，並建立回饋迴路以持續改善上下文品質。

若要實作這一點，Google 的 Vertex AI prompt optimizer 等專門調校系統，可以大規模自動化改善流程。透過依樣本輸入與預先定義的指標系統性評估回應，這些工具能提升模型效能，並在不同模型之間調整提示與系統指示，而不需要大量手動改寫。向最佳化器提供範例提示、系統指示與範本，可讓它以程式方式精煉上下文輸入，為實作複雜 Context Engineering 所需的回饋迴路提供結構化方法。  
這種結構化方法區分了基礎 AI 工具與更複雜、具上下文感知能力的系統。它將上下文視為主要元件，強調代理知道什麼、何時知道，以及如何使用該資訊。這項做法確保模型對使用者意圖、歷史與目前環境有全面理解。最終，上下文工程是將無狀態聊天機器人轉化為高度能幹、具情境感知能力系統的關鍵方法。

## 結構化輸出

提示的目標通常不只是取得自由格式的文字回應，而是以特定、機器可讀格式擷取或產生資訊。要求結構化輸出，例如 JSON、XML、CSV 或 Markdown 表格，是一項關鍵的結構化技巧。透過明確要求以特定格式輸出，並可進一步提供所需結構的 schema 或範例，你可以引導模型用容易被代理式系統或應用程式其他部分解析與使用的方式組織回應。回傳用於資料擷取的 JSON 物件很有幫助，因為它會迫使模型建立結構，也能限制幻覺。建議實驗不同輸出格式，尤其是擷取或分類資料等非創意型任務。

* **範例：**  
  Extract the following information from the text below and return it as a JSON object with keys `name`, `address`, and `phone.number`.

  Text: "Contact John Smith at 123 Main St, Anytown, CA or call (555) 123-4567."

有效運用系統提示、角色指派、上下文資訊、分隔符與結構化輸出，能顯著提升與語言模型互動時的清楚度、控制力與實用性，並為開發可靠的代理式系統奠定堅實基礎。要求結構化輸出對建立管線至關重要，因為語言模型的輸出會作為後續系統或處理步驟的輸入。

**運用 Pydantic 建立物件導向 Facade：** 強制結構化輸出並增強互通性的一項強大技巧，是使用 LLM 產生的資料來填入 Pydantic 物件執行個體。Pydantic 是一個 Python 函式庫，使用 Python 型別註解進行資料驗證與設定管理。透過定義 Pydantic model，你可以為所需資料結構建立清楚且可強制執行的 schema。這種方法實際上為提示輸出提供物件導向 facade，將原始文字或半結構化資料轉換為經驗證、具型別提示的 Python 物件。

你可以使用 `model.validate.json` method，直接將來自 LLM 的 JSON 字串解析為 Pydantic 物件。這特別有用，因為它在單一步驟中結合解析與驗證。

```python
from pydantic import BaseModel, EmailStr, Field, ValidationError
from typing import List, Optional
from datetime import date


# --- Pydantic Model Definition (from above) ---
class User(BaseModel):
    name: str = Field(..., description="The full name of the user.")
    email: EmailStr = Field(..., description="The user's email address.")
    date_of_birth: Optional[date] = Field(None, description="The user's date of birth.")
    interests: List[str] = Field(default_factory=list, description="A list of the user's interests.")


# --- Hypothetical LLM Output ---
llm_output_json = """
{
    "name": "Alice Wonderland",
    "email": "alice.w@example.com",
    "date_of_birth": "1995-07-21",
    "interests": [
        "Natural Language Processing",
        "Python Programming",
        "Gardening"
    ]
}
"""


# --- Parsing and Validation ---
try:
    # Use the model_validate_json class method to parse the JSON string.
    # This single step parses the JSON and validates the data against the User model.
    user_object = User.model_validate_json(llm_output_json)

    # Now you can work with a clean, type-safe Python object.
    print("Successfully created User object!")
    print(f"Name: {user_object.name}")
    print(f"Email: {user_object.email}")
    print(f"Date of Birth: {user_object.date_of_birth}")
    print(f"First Interest: {user_object.interests[0]}")

    # You can access the data like any other Python object attribute.
    # Pydantic has already converted the 'date_of_birth' string to a datetime.date object.
    print(f"Type of date_of_birth: {type(user_object.date_of_birth)}")
except ValidationError as e:
    # If the JSON is malformed or the data doesn't match the model's types,
    # Pydantic will raise a ValidationError.
    print("Failed to validate JSON from LLM.")
    print(e)
```

這段 Python 程式碼示範如何使用 Pydantic 函式庫定義資料模型並驗證 JSON 資料。它定義一個 User model，包含姓名、電子郵件、出生日期與興趣等欄位，並加入型別提示與描述。接著，程式碼使用 User model 的 `model.validate.json` method，解析來自大型語言模型（large language model, LLM）的假想 JSON 輸出。此 method 會依模型的結構與型別，同時處理 JSON 解析與資料驗證。最後，程式碼會從產生的 Python 物件存取已驗證資料，並包含 ValidationError 的錯誤處理，以因應 JSON 無效的情況。

對於 XML 資料，可以使用 xmltodict 函式庫將 XML 轉換為 dictionary，然後傳給 Pydantic model 進行解析。透過在 Pydantic model 中使用 Field aliases，你可以無縫地將 XML 常見的冗長或屬性繁多結構，對應到物件欄位。

這套方法對於確保以 LLM 為基礎的元件能與大型系統其他部分互通極具價值。當 LLM 輸出被封裝在 Pydantic 物件中時，就能可靠地傳遞給其他函式、API 或資料處理管線，並確保資料符合預期結構與型別。這種在系統元件邊界「parse, don't validate」的做法，能產生更穩健且更易維護的應用程式。

有效運用系統提示、角色指派、上下文資訊、分隔符與結構化輸出，能顯著提升與語言模型互動時的清楚度、控制力與實用性，並為開發可靠的代理式系統奠定堅實基礎。要求結構化輸出對建立管線至關重要，因為語言模型的輸出會作為後續系統或處理步驟的輸入。

組織提示 除了提供範例的基本技巧之外，提示的組織方式也在引導語言模型時扮演關鍵角色。組織提示是指在提示中使用不同區段或元素，以清楚、有條理的方式提供不同類型的資訊，例如指示、上下文或範例。這有助於模型正確解析提示，並理解每段文字的特定角色。

# 推理與思考流程技巧

大型語言模型擅長模式辨識與文字生成，但在需要複雜、多步驟推理的任務上通常會面臨挑戰。本附錄聚焦於旨在增強這些推理能力的技巧，方法是鼓勵模型揭露其內部思考流程。具體而言，它涵蓋改善邏輯演繹、數學計算與規劃的方法。

## 思維鏈（CoT）

思維鏈（Chain of Thought, CoT）提示技巧是一種強大方法，透過明確提示模型在得出最終答案之前產生中間推理步驟，來提升語言模型的推理能力。你不是只要求結果，而是指示模型「逐步思考」。這個流程類似人類將問題拆解為較小、較易處理的部分，並依序解決。

CoT 能幫助 LLM 產生更準確的答案，特別是對需要某種計算或邏輯演繹的任務；若沒有 CoT，模型可能會在這些任務上吃力並產生錯誤結果。透過產生這些中間步驟，模型更可能維持方向並正確執行必要操作。

CoT 有兩種主要變體：

* **零樣本 CoT：** 這只需要在提示中加入「Let's think step by step」（或類似措辭），不提供任何推理流程範例。令人意外的是，對許多任務而言，這個簡單的加入就能顯著提升模型效能，因為它會觸發模型揭露內部推理軌跡的能力。  
  * **範例（零樣本 CoT）：**  
    If a train travels at 60 miles per hour and covers a distance of 240 miles, how long did the journey take? Let's think step by step.

* **少樣本 CoT：** 這會結合 CoT 與少樣本提示。你提供模型數個範例，其中同時展示輸入、逐步推理流程與最終輸出。這會給模型更清楚的範本，說明如何執行推理並組織回應；相較於零樣本 CoT，在更複雜任務上通常能帶來更好的結果。  
  * **範例（少樣本 CoT）：**  
    Q: The sum of three consecutive integers is 36. What are the integers?  
    A: Let the first integer be x. The next consecutive integer is x+1, and the third is x+2. The sum is x + (x+1) + (x+2) \= 3x + 3. We know the sum is 36, so 3x + 3 \= 36. Subtract 3 from both sides: 3x \= 33. Divide by 3: x \= 11. The integers are 11, 11+1=12, and 11+2=13. The integers are 11, 12, and 13.

    Q: Sarah has 5 apples, and she buys 8 more. She eats 3 apples. How many apples does she have left? Let's think step by step.  
    A: Let's think step by step. Sarah starts with 5 apples. She buys 8 more, so she adds 8 to her initial amount: 5 + 8 \= 13 apples. Then, she eats 3 apples, so we subtract 3 from the total: 13 - 3 \= 10. Sarah has 10 apples left. The answer is 10.

CoT 有多項優點。它實作起來相對低成本，並且可直接搭配現成 LLM 使用，無需微調即可非常有效。一項重要好處是模型輸出的可解釋性提高；你可以看到模型遵循的推理步驟，有助於理解它為何得出特定答案，並在出錯時進行除錯。此外，CoT 似乎能提升提示在不同語言模型版本間的穩健性，這表示當模型更新時，效能較不容易下降。主要缺點是產生推理步驟會增加輸出長度，導致 token 使用量更高，進而增加成本與回應時間。

CoT 的最佳實務包括確保最終答案出現在推理步驟*之後*，因為推理的生成會影響後續答案 token 的預測。此外，對於只有單一正確答案的任務（例如數學問題），使用 CoT 時建議將模型 temperature 設為 0（greedy decoding），以確保每一步都以決定性方式選取最可能的下一個 token。

## 自我一致性

自我一致性技巧建立在思維鏈概念之上，目標是利用語言模型的機率性質來提升推理可靠性。它不依賴單一貪婪推理路徑（如基本 CoT），而是為同一問題產生多條多樣化推理路徑，然後從中選出最一致的答案。

自我一致性包含三個主要步驟：

1. **產生多樣化推理路徑：** 將相同提示（通常是 CoT 提示）多次送給 LLM。透過使用較高的 temperature 設定，鼓勵模型探索不同推理方法，並產生多樣化的逐步說明。  
2. **擷取答案：** 從每條生成的推理路徑中擷取最終答案。  
3. **選擇最常見答案：** 對擷取出的答案進行多數決。最常出現在多樣化推理路徑中的答案，會被選為最終且最一致的答案。

這種方法可改善回應的準確性與連貫性，特別適用於可能存在多種有效推理路徑，或模型單次嘗試容易出錯的任務。其好處是能提供答案正確性的類機率可能性，提升整體準確度。然而，顯著代價是必須針對同一查詢多次執行模型，導致計算量與費用大幅增加。

* **範例（概念性）：**  
  * *提示：* "Is the statement 'All birds can fly' true or false? Explain your reasoning."  
  * *模型執行 1（高 Temp）：* 依據多數鳥類會飛進行推理，結論為 True。  
  * *模型執行 2（高 Temp）：* 依據企鵝與鴕鳥進行推理，結論為 False。  
  * *模型執行 3（高 Temp）：* 依據鳥類*一般而言*進行推理，簡短提及例外，結論為 True。  
  * *自我一致性結果：* 根據多數決（True 出現兩次），最終答案為 "True"。（註：更精緻的方法會對推理品質加權）。

## 退一步提示

退一步提示會先要求語言模型在處理具體細節之前，思考與任務相關的一般原則或概念，藉此增強推理。對這個較廣泛問題的回應，接著會被用作解決原始問題的上下文。

這個流程讓語言模型啟用相關背景知識與更廣泛的推理策略。透過聚焦於底層原則或較高層次抽象，模型能產生更準確且有洞察力的答案，也較不受表面元素影響。先思考一般因素，可以為產生具體創意輸出提供更強基礎。退一步提示鼓勵批判性思考與知識應用，並可能透過強調一般原則來減輕偏見。

* **範例：**  
  * *提示 1（退一步）：* "What are the key factors that make a good detective story?"  
  * *模型回應 1：*（列出紅鯡魚線索、具說服力的動機、有缺陷的主角、合乎邏輯的線索、令人滿意的結局等元素）。  
  * *提示 2（原始任務 + 退一步上下文）：* "Using the key factors of a good detective story [insert Model Response 1 here], write a short plot summary for a new mystery novel set in a small town."

## 思維樹（ToT）

思維樹（Tree of Thoughts, ToT）是一項進階推理技巧，延伸自思維鏈方法。它讓語言模型能同時探索多條推理路徑，而不是遵循單一線性進程。這項技巧使用樹狀結構，其中每個節點代表一個「thought」——作為中間步驟的一段連貫語言序列。模型可以從每個節點分支，探索替代推理路線。

ToT 特別適合需要探索、回溯，或在得出解答前評估多種可能性的複雜問題。雖然相較於線性的思維鏈方法，ToT 在運算上更昂貴且實作更複雜，但在需要審慎且探索式問題解決的任務上，能取得更優異的結果。它允許代理考量不同觀點，並可能透過檢查「thought tree」中的替代分支，從初始錯誤中恢復。

* **範例（概念性）：** 對於「根據這些情節點，為故事發展三種不同可能結局」這類複雜創意寫作任務，ToT 會讓模型從關鍵轉折點探索不同敘事分支，而不是只產生一條線性延續。

這些推理與思考流程技巧，對建構能處理超越簡單資訊檢索或文字生成任務的代理至關重要。透過提示模型揭露推理、考量多種觀點，或退一步思考一般原則，我們可以顯著增強它們在代理式系統中執行複雜認知任務的能力。

# 動作與互動技巧

智慧代理具備主動與環境互動的能力，不僅限於生成文字。這包括使用工具、執行外部函式，以及參與觀察、推理與行動的迭代循環。本節檢視旨在啟用這些主動行為的提示技巧。

## 工具使用（Tool Use / Function Calling）

代理的一項關鍵能力，是使用外部工具或呼叫函式，以執行超出其內部能力的動作。這些動作可能包括網路搜尋、資料庫存取、傳送電子郵件、執行計算，或與外部 API 互動。針對工具使用進行有效提示，需要設計能指示模型在適當時機與以適當方法使用工具的提示。

現代語言模型通常會針對「function calling」或「tool use」進行微調。這讓它們能解讀可用工具的描述，包括工具用途與參數。收到使用者請求後，模型可以判斷是否需要使用工具、識別適當工具，並格式化呼叫該工具所需的參數。模型不會直接執行工具。相反地，它會產生結構化輸出，通常是 JSON 格式，指定工具及其參數。代理式系統接著會處理此輸出、執行工具，並將工具結果回傳給模型，整合進持續進行的互動中。

* **範例：**  
  You have access to a weather tool that can get the current weather for a specified city. The tool is called '`get.current.weather`' and takes a '`city`' parameter (string).

  User: What's the weather like in London right now?

  * *預期模型輸出（Function Call）：*  
    {  
      "tool.code": "get.current.weather",  
      "tool.name": "get.current.weather",  
      "parameters": {  
        "city": "London"  
      }  
    }

## ReAct（Reason & Act）

ReAct 是 Reason and Act 的縮寫，是一種提示範式，會將思維鏈式推理與使用工具執行動作的能力，以交錯方式結合。ReAct 模仿人類的運作方式——我們會用語言推理，並採取行動來蒐集更多資訊或朝目標推進。

ReAct 模式包含一個循環：

1. **Thought：** 模型產生思考流程，說明目前理解與計畫。  
2. **Action：** 根據 thought，模型決定執行一個動作，通常會使用工具（例如 Search、Calculator、API call）。它會輸出工具名稱與所需輸入。  
3. **Observation：** 代理式系統執行工具，並將結果（Observation）回傳給模型。  
4. 這個循環會持續進行，模型根據最新的「Observation」產生新的「Thought」，進一步導向更多「Actions」與「Observations」，直到任務完成並由模型輸出「Final Answer」。

這種思考與行動交錯的流程，讓代理能動態蒐集資訊、回應工具輸出並調整方法，因此特別適合需要與動態環境或外部知識來源互動的任務。

* **範例（說明性 ReAct Trace）：**  
  User: What is the capital of France and its current population?

  Thought: The user is asking for two pieces of information: the capital of France and its population. I can use a search tool to find this information. I will start by confirming the capital, although I am likely to know it.  
  Action: Search({"query": "capital of France"})  
  Observation: Paris

  Thought: Okay, the capital is Paris. Now I need to find the population of Paris.  
  Action: Search({"query": "current population of Paris"})  
  Observation: The population of Paris was estimated to be around 2.1 million people in 2023.

  Thought: I have found both pieces of information. I can now provide the final answer to the user.  
  Final Answer: The capital of France is Paris, and its estimated population in 2023 was around 2.1 million people.

這些技巧對建構能主動與世界互動、擷取即時資訊，並執行需要與外部系統互動任務的代理至關重要。

## 進階技巧

除了基礎、結構與推理模式之外，還有數種其他提示技巧能進一步增強代理式系統的能力與效率。這些技巧涵蓋從使用 AI 最佳化提示，到納入外部知識，以及根據使用者特徵客製化回應。

### 自動提示工程（APE）

有效提示的設計可能是複雜且迭代的流程；自動提示工程（Automatic Prompt Engineering, APE）正是基於這項認知，探索如何使用語言模型本身來產生、評估與精煉提示。此方法旨在自動化提示撰寫流程，可能在不需要大量人工提示設計工作的情況下提升模型效能。

一般概念是讓「meta-model」或某個流程接收任務描述，並產生多個候選提示。接著根據這些提示在指定輸入集合上產生的輸出品質來評估它們（可能使用 BLEU 或 ROUGE 等指標，或人工評估）。表現最佳的提示可以被選出，必要時再進一步精煉，並用於目標任務。使用 LLM 產生使用者查詢變體來訓練聊天機器人，就是一個例子。

* **範例（概念性）：** 開發者提供描述：「I need a prompt that can extract the date and sender from an email.」APE 系統產生數個候選提示。這些提示會在範例電子郵件上測試，並選出能穩定擷取正確資訊的提示。

當然。以下是使用 DSPy 等框架進行程式化提示最佳化的改寫並稍微擴充的說明：

另一項強大的提示最佳化技巧，特別由 DSPy 框架所推廣，是將提示視為可自動最佳化的程式化模組，而非靜態文字。這種方法超越手動試誤，進入更系統化、資料驅動的方法。

這項技巧的核心仰賴兩個關鍵元件：

1. **Goldset（或高品質資料集）：** 這是一組具代表性的高品質輸入與輸出配對。它作為「ground truth」，定義指定任務中成功回應的樣貌。  
2. **目標函式（或評分指標）：** 這是一個函式，會自動將 LLM 輸出與資料集中對應的「golden」輸出進行比較。它會回傳分數，表示回應的品質、準確性或正確性。

使用這些元件，最佳化器（例如 Bayesian optimizer）會系統性地精煉提示。這個流程通常包含兩種主要策略，可獨立使用，也可搭配使用：

* **少樣本範例最佳化：** 最佳化器不讓開發者手動為少樣本提示選擇範例，而是以程式方式從 goldset 抽樣不同的範例組合。接著測試這些組合，以找出最能有效引導模型產生所需輸出的特定範例集合。

* **指令式提示最佳化：** 在這種方法中，最佳化器會自動精煉提示的核心指示。它使用 LLM 作為「meta-model」，迭代式地變異並改寫提示文字——調整措辭、語氣或結構——以發現哪種措辭能從目標函式取得最高分。

這兩種策略的最終目標，都是最大化目標函式的分數，實際上是在「訓練」提示，使其產生的結果持續更接近高品質 goldset。透過結合這兩種方法，系統可以同時最佳化要給模型的*指示內容*，以及要展示給它的*範例*，進而得到針對特定任務由機器最佳化、高效且穩健的提示。

### 迭代式提示／精煉

這項技巧是從簡單、基本的提示開始，然後根據模型初始回應反覆精煉。如果模型輸出不夠理想，就分析缺點並修改提示以解決問題。這較少是自動化流程（如 APE），更像是由人類驅動的迭代式設計循環。

* **範例：**  
  * *嘗試 1：* "Write a product description for a new type of coffee maker."（結果太泛泛而談）。  
  * *嘗試 2：* "Write a product description for a new type of coffee maker. Highlight its speed and ease of cleaning."（結果較好，但缺少細節）。  
  * *嘗試 3：* "Write a product description for the 'SpeedClean Coffee Pro'. Emphasize its ability to brew a pot in under 2 minutes and its self-cleaning cycle. Target busy professionals."（結果更接近預期）。

### 提供負面範例

雖然「指示優於限制」原則通常成立，但在某些情況下，提供負面範例可能有幫助，不過必須謹慎使用。負面範例會向模型展示一個輸入及其*不想要的*輸出，或一個輸入及*不應該*生成的輸出。這有助於釐清邊界，或防止特定類型的錯誤回應。

* **範例：**  
  Generate a list of popular tourist attractions in Paris. Do NOT include the Eiffel Tower.

  Example of what NOT to do:  
  Input: List popular landmarks in Paris.  
  Output: The Eiffel Tower, The Louvre, Notre Dame Cathedral.

### 使用類比

使用類比來框定任務，有時能透過將任務連結到熟悉事物，幫助模型理解所需輸出或流程。這對創意任務或解釋複雜角色特別有用。

* **範例：**  
  Act as a "data chef". Take the raw ingredients (data points) and prepare a "summary dish" (report) that highlights the key flavors (trends) for a business audience.

### 分解式認知／拆解

對於非常複雜的任務，將整體目標拆解成較小、較易管理的子任務，並分別針對每個子任務提示模型，會很有效。接著再結合子任務結果，以達成最終成果。這與提示鏈和規劃相關，但強調有意識地拆解問題。

* **範例：** 撰寫研究論文：  
  * Prompt 1: "Generate a detailed outline for a paper on the impact of AI on the job market."  
  * Prompt 2: "Write the introduction section based on this outline: [insert outline intro]."  
  * Prompt 3: "Write the section on 'Impact on White-Collar Jobs' based on this outline: [insert outline section]."（對其他章節重複）。  
  * Prompt N: "Combine these sections and write a conclusion."

### 檢索增強生成（RAG）

RAG 是一項強大技巧，會在提示流程中讓語言模型存取外部、最新或特定領域資訊，藉此增強模型能力。當使用者提出問題時，系統會先從知識庫（例如資料庫、一組文件或網路）擷取相關文件或資料。接著，這些擷取到的資訊會作為上下文納入提示，讓語言模型能生成以外部知識為基礎的回應。這能減輕幻覺等問題，並提供模型未受訓過或非常近期的資訊。對需要處理動態或專有資訊的代理式系統而言，這是一項關鍵模式。

* **範例：**  
  * *使用者查詢：* "What are the new features in the latest version of the Python library 'X'?"  
  * *系統動作：* 在文件資料庫搜尋 "Python library X latest features"。  
  * *給 LLM 的提示：* "Based on the following documentation snippets: [insert retrieved text], explain the new features in the latest version of Python library 'X'."

### 人格模式（使用者 Persona）

角色提示會將人格指派給*模型*，而人格模式則描述模型輸出的使用者或目標受眾。這有助於模型在語言、複雜度、語氣與所提供資訊類型上調整回應。

* **範例：**  
  You are explaining quantum physics. The target audience is a high school student with no prior knowledge of the subject. Explain it simply and use analogies they might understand.

  Explain quantum physics: [Insert basic explanation request]

這些進階與補充技巧為提示工程師提供更多工具，可在代理式工作流程中最佳化模型行為、整合外部資訊，並為特定使用者與任務客製化互動。

## 使用 Google Gems

Google 的 AI「Gems」（見圖 1）代表其大型語言模型架構中的使用者可設定功能。每個「Gem」都作為核心 Gemini AI 的專門化執行個體，針對特定、可重複的任務客製化。使用者透過提供一組明確指示來建立 Gem，這會確立其操作參數。這組初始指示會定義 Gem 的指定目的、回應風格與知識領域。底層模型被設計為在整段對話中持續遵循這些預先定義的指令。

這讓使用者可以為聚焦應用建立高度專門化的 AI 代理。例如，可以將 Gem 設定為程式碼解譯器，且只參考特定程式設計函式庫。另一個 Gem 可被指示分析資料集，並產生不含推測性評論的摘要。另一個不同的 Gem 可能作為遵循特定正式風格指南的翻譯器。這個流程為人工智慧建立持久且特定任務的上下文。

因此，使用者不必在每次新查詢時重新建立相同的上下文資訊。這套方法降低對話冗餘，並提升任務執行效率。產生的互動更聚焦，輸出也會持續符合使用者初始需求。這個框架允許對通用 AI 模型套用細緻且持久的使用者指引。最終，Gems 促成從通用互動轉向專門化、預先定義 AI 功能的轉變。

![Example of Google Gem Usage](../assets/Example_of_Google_Gem_Usage.png)

圖 1：Google Gem 使用範例。

## 使用 LLM 精煉提示（Meta 方法）

我們已探索多種設計有效提示的技巧，強調清楚性、結構，以及提供上下文或範例。然而，這個流程可能需要反覆迭代，有時也具有挑戰性。如果我們可以運用大型語言模型本身的強大能力，例如 Gemini，來幫助我們*改善*提示，會如何？這正是使用 LLM 精煉提示的核心——一種「meta」應用，由 AI 協助最佳化給 AI 的指示。

這項能力特別「酷」，因為它代表一種 AI 自我改進，或至少是 AI 協助人類改善與 AI 互動的形式。我們不必只依賴人類直覺與試誤，而可以利用 LLM 對語言、模式，甚至常見提示陷阱的理解，取得改善提示的建議。它把 LLM 變成提示工程流程中的協作夥伴。

實務上這如何運作？你可以提供語言模型一個想要改善的現有提示、你希望它完成的任務，或甚至目前得到的輸出範例（以及為何不符合期待）。接著提示 LLM 分析該提示並提出改善建議。

像 Gemini 這樣具備強大推理與語言生成能力的模型，可以分析現有提示中潛在的模糊處、具體性不足，或低效率措辭。它可以建議納入我們討論過的技巧，例如加入分隔符、釐清所需輸出格式、建議更有效的人格，或建議納入少樣本範例。

這種 meta-prompting 方法的好處包括：

* **加速迭代：** 相較於純手動試誤，更快取得改善建議。  
* **識別盲點：** LLM 可能發現你忽略的提示模糊處或潛在誤解。  
* **學習機會：** 透過觀察 LLM 提出的建議類型，你可以更了解有效提示的要素，並提升自己的提示工程技能。  
* **可擴展性：** 可能自動化部分提示最佳化流程，尤其是在處理大量提示時。

需要注意的是，LLM 的建議不一定總是完美，應像任何手動設計的提示一樣接受評估與測試。不過，它提供了強大的起點，並能顯著簡化精煉流程。

* **精煉範例提示：**  
  Analyze the following prompt for a language model and suggest ways to improve it to consistently extract the main topic and key entities (people, organizations, locations) from news articles. The current prompt sometimes misses entities or gets the main topic wrong.

  Existing Prompt:  
  "Summarize the main points and list important names and places from this article: [insert article text]"

  Suggestions for Improvement:

在這個範例中，我們使用 LLM 來評論並強化另一個提示。這種 meta 層級互動展示了這些模型的彈性與力量，讓我們能先最佳化代理式系統接收的基本指示，進而建構更有效的代理式系統。這是一個迷人的循環：AI 幫助我們更好地與 AI 對話。

## 針對特定任務的提示

雖然目前討論的技巧都具有廣泛適用性，但某些任務會受益於特定提示考量。這些在程式碼與多模態輸入領域中特別相關。

### 程式碼提示

語言模型，尤其是以大型程式碼資料集訓練的模型，可以成為開發者的強大助理。程式碼提示是指使用 LLM 生成、解釋、翻譯或除錯程式碼。其使用案例有多種：

* **撰寫程式碼的提示：** 根據所需功能描述，要求模型產生程式碼片段或函式。  
  * **範例：** "Write a Python function that takes a list of numbers and returns the average."  
* **解釋程式碼的提示：** 提供程式碼片段，並要求模型逐行或摘要式說明它的作用。  
  * **範例：** "Explain the following JavaScript code snippet: [insert code]."  
* **翻譯程式碼的提示：** 要求模型將程式碼從一種程式語言翻譯成另一種。  
  * **範例：** "Translate the following Java code to C++: [insert code]."  
* **除錯與審查程式碼的提示：** 提供有錯誤或可改進的程式碼，並要求模型識別問題、建議修正方式，或提供重構建議。  
  * **範例：** "The following Python code is giving a 'NameError'. What is wrong and how can I fix it? [insert code and traceback]."

有效的程式碼提示通常需要提供足夠上下文、指定所需語言與版本，並清楚說明功能或問題。

### 多模態提示

雖然本附錄及目前多數 LLM 互動都以文字為主，但此領域正快速朝向能跨不同模態（文字、影像、音訊、影片等）處理與生成資訊的多模態模型發展。多模態提示是指使用多種輸入組合來引導模型。這表示使用多種輸入格式，而不只是文字。

* **範例：** 提供一張圖表影像，並要求模型解釋圖中展示的流程（Image Input + Text Prompt）。或提供一張影像，並要求模型產生描述性 caption（Image Input + Text Prompt -> Text Output）。

隨著多模態能力變得更成熟，提示技巧也將演進，以有效運用這些組合式輸入與輸出。

## 最佳實務與實驗

成為熟練的提示工程師是一個涉及持續學習與實驗的迭代流程。以下幾項有價值的最佳實務值得再次強調：

* **提供範例：** 提供單樣本或少樣本範例，是引導模型最有效的方法之一。  
* **以簡單性設計：** 讓提示保持簡潔、清楚且易於理解。避免不必要的術語或過度複雜的措辭。  
* **明確指定輸出：** 清楚定義模型回應的所需格式、長度、風格與內容。  
* **使用指示而非限制：** 著重告訴模型你希望它做什麼，而不是不要做什麼。  
* **控制最大 token 長度：** 使用模型設定或明確提示指示，管理生成輸出的長度。  
* **在提示中使用變數：** 對於應用程式中使用的提示，使用變數讓它們具動態性且可重複使用，避免硬編碼特定值。  
* **實驗輸入格式與寫作風格：** 嘗試不同提示措辭方式（問題、陳述、指示），並實驗不同語氣或風格，以觀察何者產生最佳結果。  
* **針對分類任務使用少樣本提示時，混合類別：** 隨機化不同類別範例的順序，以避免過度擬合。  
* **適應模型更新：** 語言模型會持續更新。應準備好在新模型版本上測試既有提示，並調整它們以利用新能力或維持效能。  
* **實驗輸出格式：** 尤其對非創意型任務，應實驗要求 JSON 或 XML 等結構化輸出。  
* **與其他提示工程師一起實驗：** 與他人協作可以提供不同觀點，並發現更有效的提示。  
* **CoT 最佳實務：** 記住思維鏈的特定做法，例如將答案放在推理之後，並對只有單一正確答案的任務將 temperature 設為 0。  
* **記錄各種提示嘗試：** 這對追蹤哪些有效、哪些無效以及原因至關重要。維護提示、設定與結果的結構化紀錄。  
* **將提示儲存在程式碼庫中：** 將提示整合進應用程式時，將它們儲存在獨立且組織良好的檔案中，以便維護與版本控制。  
* **依賴自動化測試與評估：** 對於生產系統，實作自動化測試與評估程序，以監控提示效能並確保可泛化到新資料。

提示工程是一項會隨練習而進步的技能。透過套用這些原則與技巧，並維持系統化的實驗與文件記錄方法，你可以顯著提升建構有效代理式系統的能力。

## 結論

本附錄全面概述提示，將其重新框定為一項有紀律的工程實務，而不是單純提出問題。其核心目的，是展示如何將通用語言模型轉化為針對特定任務的專門化、可靠且高能力工具。這段旅程始於不可妥協的核心原則，例如清楚性、簡潔性與迭代式實驗；這些是與 AI 有效溝通的基石。這些原則很關鍵，因為它們能降低自然語言固有的模糊性，幫助將模型的機率式輸出導向單一、正確的意圖。在這個基礎之上，零樣本、單樣本與少樣本提示等基本技巧，成為透過範例展示預期行為的主要方法。這些方法提供不同程度的上下文引導，強而有力地塑造模型的回應風格、語氣與格式。不只範例，使用明確角色、系統層級指示與清楚分隔符來組織提示，也為細緻控制模型提供必要的架構層。

在建構自主代理的脈絡中，這些技巧的重要性變得至關重要，因為它們提供複雜、多步驟操作所需的控制力與可靠性。代理若要有效建立並執行計畫，就必須運用思維鏈與思維樹等進階推理模式。這些精緻方法迫使模型外化其邏輯步驟，系統性地將複雜目標拆解成一系列可管理的子任務。整個代理式系統的操作可靠性，取決於每個元件輸出的可預測性。這正是為什麼要求 JSON 等結構化資料，並使用 Pydantic 等工具以程式方式驗證它，不只是方便而已，而是穩健自動化的絕對必要條件。若缺乏這項紀律，代理的內部認知元件就無法可靠溝通，導致自動化工作流程中的災難性失敗。最終，正是這些結構化與推理技巧，成功將模型的機率式文字生成，轉換為代理可使用的決定性且可信賴的認知引擎。

此外，這些提示賦予代理感知並作用於環境的關鍵能力，銜接數位思考與真實世界互動之間的落差。ReAct 與原生 function calling 等以動作為導向的框架，是作為代理雙手的重要機制，讓代理能使用工具、查詢 API 並操作資料。同時，檢索增強生成（RAG）與更廣泛的上下文工程，則作為代理的感官。它們會主動從外部知識庫擷取相關即時資訊，確保代理的決策奠基於當前且符合事實的現實。這項關鍵能力可防止代理在真空中運作，避免它受限於靜態且可能過時的訓練資料。因此，掌握完整提示光譜，是將通用語言模型從簡單文字生成器，提升為真正精密代理的決定性技能；這樣的代理能以自主性、感知力與智慧執行複雜任務。

## 參考資料

以下是進一步閱讀與深入探索提示工程技巧的資源清單：

1. Prompt Engineering, [https://www.kaggle.com/whitepaper-prompt-engineering](https://www.kaggle.com/whitepaper-prompt-engineering)
2. Chain-of-Thought Prompting Elicits Reasoning in Large Language Models, [https://arxiv.org/abs/2201.11903](https://arxiv.org/abs/2201.11903)
3. Self-Consistency Improves Chain of Thought Reasoning in Language Models,  [https://arxiv.org/pdf/2203.11171](https://arxiv.org/pdf/2203.11171)
4. ReAct: Synergizing Reasoning and Acting in Language Models, [https://arxiv.org/abs/2210.03629](https://arxiv.org/abs/2210.03629)  
5. Tree of Thoughts: Deliberate Problem Solving with Large Language Models,  [https://arxiv.org/pdf/2305.10601](https://arxiv.org/pdf/2305.10601)
6. Take a Step Back: Evoking Reasoning via Abstraction in Large Language Models, [https://arxiv.org/abs/2310.06117](https://arxiv.org/abs/2310.06117)
7. DSPy: Programming—not prompting—Foundation Models [https://github.com/stanfordnlp/dspy](https://github.com/stanfordnlp/dspy)
