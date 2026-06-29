# 第 1 章：提示鏈（Prompt Chaining）

## 提示鏈模式概覽

提示鏈，有時也稱為管線（Pipeline）模式，是運用大型語言模型（large language model, LLM）處理複雜任務時的一種強大典範。它不是期待 LLM 用單一、龐大的一步解決複雜問題，而是主張採用分而治之的策略。核心概念是將原本令人望而生畏的問題，拆解成一連串更小、更容易管理的子問題。每個子問題都透過特別設計的提示個別處理，而某個提示產生的輸出，會被有策略地送入提示鏈中的下一個提示作為輸入。

這種循序處理技巧，會自然地為與 LLM 的互動帶來模組化與清晰度。將複雜任務分解後，每一個步驟都更容易理解與除錯，讓整體流程更穩健，也更容易解釋。鏈中的每個步驟都可以經過仔細設計與最佳化，專注於較大問題中的特定面向，進而產生更準確、更聚焦的輸出。

讓一個步驟的輸出成為下一個步驟的輸入，是此模式的關鍵。這種資訊傳遞建立了一條相依鏈，也就是名稱的由來；先前操作的上下文與結果會引導後續處理。如此一來，LLM 就能建立在先前工作的基礎上，精煉自己的理解，並逐步接近期望的解決方案。

此外，提示鏈不只是將問題拆解；它也能整合外部知識與工具。在每個步驟中，都可以指示 LLM 與外部系統、API 或資料庫互動，讓它的知識與能力超越內部訓練資料。這項能力大幅擴展了 LLM 的潛力，使其不只是孤立的模型，而是更廣泛、更智慧系統中的整合元件。

提示鏈的重要性不只在於單純的問題解決。它也是建構精密 AI 代理（agent）的基礎技術。這些代理可以利用提示鏈，在動態環境中自主規劃、推理與行動。透過有策略地安排提示順序，代理可以執行需要多步驟推理、規劃與決策的任務。這類代理工作流程能更接近地模擬人類思考過程，讓與複雜領域和系統的互動更自然、更有效。

**單一提示的限制：** 對於多面向任務，對 LLM 使用單一複雜提示可能效率不佳，導致模型難以同時處理限制與指示，進而可能出現指示忽略（提示的部分內容被略過）、上下文漂移（模型忘記初始上下文）、錯誤傳播（早期錯誤被放大）、提示需要更長的上下文視窗（模型取得的資訊不足以回應），以及幻覺（認知負荷增加而提高錯誤資訊的機率）。例如，一個要求分析市場研究報告、摘要發現、以資料點找出趨勢，並起草電子郵件的查詢，就有失敗風險；模型可能摘要得很好，卻無法正確擷取資料或妥善撰寫電子郵件。

**透過循序分解提升可靠性：** 提示鏈透過將複雜任務拆成聚焦的循序工作流程來因應這些挑戰，顯著提升可靠性與控制力。以上述範例來說，管線或鏈式方法可以描述如下：

1. 初始提示（摘要）："請摘要以下市場研究報告的關鍵發現：\[text\]。" 模型唯一的焦點是摘要，因此提高了這個初始步驟的準確性。  
2. 第二個提示（趨勢辨識）："使用摘要，找出前三大新興趨勢，並擷取支持每個趨勢的具體資料點：\[output from step 1\]。" 這個提示現在更受限，並直接建立在已驗證的輸出之上。  
3. 第三個提示（電子郵件撰寫）："請起草一封簡潔的電子郵件給行銷團隊，概述以下趨勢及其支持資料：\[output from step 2\]。"

這種分解讓流程能被更細緻地控制。每個步驟都更簡單、歧義更少，因而降低模型的認知負荷，並產生更準確且可靠的最終輸出。這種模組化類似於運算管線：每個函式先執行特定操作，再將結果傳給下一個函式。為了確保每項特定任務都能得到準確回應，可以在每個階段指派模型不同角色。例如，在上述情境中，初始提示可指定為「Market Analyst」，後續提示可指定為「Trade Analyst」，第三個提示可指定為「Expert Documentation Writer」，以此類推。

**結構化輸出的角色：** 提示鏈的可靠性高度取決於步驟之間傳遞資料的完整性。如果某個提示的輸出含糊不清或格式不佳，後續提示可能因輸入有缺陷而失敗。為了降低這種風險，指定結構化輸出格式（例如 JSON 或 XML）非常重要。

例如，趨勢辨識步驟的輸出可以格式化為 JSON 物件：

```json
{  "trends": [    
        {      
            "trend_name": "AI-Powered Personalization",      
            "supporting_data": "73% of consumers prefer to do business with brands that use personal information to make their shopping experiences more relevant."    
        },    
        {      
            "trend_name": "Sustainable and Ethical Brands",      
            "supporting_data": "Sales of products with ESG-related claims grew 28% over the last five years, compared to 20% for products without."    
        } 
    ] 
}
```

這種結構化格式確保資料可由機器讀取，並能被精確解析後插入下一個提示，且不產生歧義。這項做法能將自然語言解讀所造成的錯誤降到最低，也是建構穩健、多步驟 LLM 系統的關鍵組成。

## 實際應用與使用案例

在建構代理式系統時，提示鏈是一種可套用於各種情境的多用途模式。它的核心效用在於將複雜問題拆解成循序且可管理的步驟。以下是幾個實際應用與使用案例：

### 1. 資訊處理工作流程

許多任務都涉及將原始資訊經過多次轉換。例如，先摘要文件、擷取關鍵實體，接著使用這些實體查詢資料庫或生成報告。提示鏈可能如下：

* 提示 1：從指定 URL 或文件中擷取文字內容。  
* 提示 2：摘要清理後的文字。  
* 提示 3：從摘要或原始文字中擷取特定實體（例如姓名、日期、地點）。  
* 提示 4：使用這些實體搜尋內部知識庫。  
* 提示 5：產出整合摘要、實體與搜尋結果的最終報告。

這種方法可應用於自動化內容分析、AI 驅動研究助理開發，以及複雜報告生成等領域。

### 2. 複雜查詢回答

回答需要多步驟推理或資訊檢索的複雜問題，是提示鏈的主要使用案例。例如：「1929 年股市崩盤的主要原因是什麼？政府政策又如何回應？」

* 提示 1：辨識使用者查詢中的核心子問題（崩盤原因、政府回應）。  
* 提示 2：專門研究或檢索 1929 年崩盤原因的相關資訊。  
* 提示 3：專門研究或檢索政府對 1929 年股市崩盤的政策回應。  
* 提示 4：將步驟 2 與 3 的資訊綜合成對原始查詢的連貫回答。

這種循序處理方法，是開發具備多步驟推論與資訊綜合能力的 AI 系統不可或缺的一部分。當某個查詢無法從單一資料點回答，而是需要一系列邏輯步驟，或需要整合來自不同來源的資訊時，就會需要這類系統。

例如，設計用來針對特定主題產生完整報告的自動化研究代理，會執行一種混合式運算工作流程。起初，系統會擷取大量相關文章。接下來，從每篇文章中擷取關鍵資訊的任務，可以針對每個來源並行執行。這個階段很適合平行處理，也就是同時執行彼此獨立的子任務，以最大化效率。

然而，一旦個別擷取完成，流程本質上就會變成循序進行。系統必須先彙整擷取出的資料，再將其綜合成連貫草稿，最後審閱並精煉這份草稿以產生最終報告。這些後續階段在邏輯上都依賴前一階段的成功完成。這正是提示鏈的應用所在：彙整後的資料會作為綜合提示的輸入，而綜合產生的文字則會成為最終審閱提示的輸入。因此，複雜操作經常會將平行處理用於獨立資料蒐集，並將提示鏈用於彼此相依的綜合與精煉步驟。

### 3. 資料擷取與轉換

將非結構化文字轉換為結構化格式，通常是透過反覆迭代的流程完成，並需要循序修改以改善輸出的準確性與完整性。

* 提示 1：嘗試從發票文件中擷取特定欄位（例如姓名、地址、金額）。  
* 處理：檢查是否擷取出所有必要欄位，以及它們是否符合格式要求。  
* 提示 2（條件式）：如果欄位缺漏或格式錯誤，設計新的提示，要求模型專門找出缺漏／格式錯誤的資訊，必要時提供先前失敗嘗試的上下文。  
* 處理：再次驗證結果。必要時重複。  
* 輸出：提供已擷取且驗證過的結構化資料。

這種循序處理方法特別適用於從表單、發票或電子郵件等非結構化來源進行資料擷取與分析。例如，解決複雜的 Optical Character Recognition（OCR）問題（如處理 PDF 表單）時，採用分解後的多步驟方法通常更有效。

一開始，會使用大型語言模型從文件影像中執行主要文字擷取。接著，模型會處理原始輸出以正規化資料；在這個步驟中，它可能會將像「one thousand and fifty」這樣的數字文字轉換成對應數值 1050。對 LLM 來說，精確執行數學計算是一項重大挑戰。因此，在後續步驟中，系統可以將任何必要的算術運算委派給外部計算器工具。LLM 會辨識所需計算，將正規化後的數字傳給工具，然後納入精確結果。這一連串文字擷取、資料正規化與外部工具使用的鏈式序列，可以得到最終且準確的結果；若只用單一 LLM 查詢，通常很難可靠地取得這樣的結果。

### 4. 內容生成工作流程

複雜內容的撰寫是一項程序性任務，通常會分解成不同階段，包括初始構思、結構大綱、草稿撰寫，以及後續修訂。

* 提示 1：根據使用者的一般興趣產生 5 個主題構想。  
* 處理：讓使用者選擇一個構想，或自動選出最佳構想。  
* 提示 2：根據所選主題，產生詳細大綱。  
* 提示 3：根據大綱中的第一個重點撰寫草稿段落。  
* 提示 4：根據大綱中的第二個重點撰寫草稿段落，並提供前一段作為上下文。對所有大綱重點持續此流程。  
* 提示 5：審閱並精煉完整草稿的連貫性、語氣與文法。

這種方法可用於一系列自然語言生成任務，包括自動撰寫創意敘事、技術文件，以及其他形式的結構化文字內容。

### 5. 具備狀態的對話代理

雖然完整的狀態管理架構會採用比循序連結更複雜的方法，但提示鏈提供了保留對話連續性的基礎機制。這項技術透過將每一次對話輪次建構成新的提示來維持上下文，並系統化地納入對話序列中先前互動的資訊或擷取出的實體。

* 提示 1：處理使用者話語 1，辨識意圖與關鍵實體。  
* 處理：使用意圖與實體更新對話狀態。  
* 提示 2：根據目前狀態產生回應，並／或辨識下一個需要取得的資訊。  
* 對後續輪次重複此流程；每個新的使用者話語都會啟動一條鏈，運用持續累積的對話歷史（狀態）。

這項原則是開發對話代理的基礎，讓它們能在長時間、多輪對話中維持上下文與連貫性。透過保留對話歷史，系統可以理解並適當回應依賴先前交換資訊的使用者輸入。

### 6. 程式碼生成與精煉

產生可運作的程式碼通常是一個多階段流程，需要將問題分解成一連串離散的邏輯操作，並逐步執行。

* 提示 1：理解使用者對程式碼函式的要求。產生虛擬碼或大綱。  
* 提示 2：根據大綱撰寫初始程式碼草稿。  
* 提示 3：辨識程式碼中潛在錯誤或可改善之處（可能使用靜態分析工具或另一個 LLM 呼叫）。  
* 提示 4：根據已辨識的問題重寫或精煉程式碼。  
* 提示 5：新增文件或測試案例。

在 AI 輔助軟體開發等應用中，提示鏈的實用性來自於它能將複雜的編碼任務分解成一系列可管理的子問題。這種模組化結構會降低大型語言模型在每個步驟中的操作複雜度。更重要的是，這種方法也允許在模型呼叫之間插入確定性邏輯，使工作流程能執行中間資料處理、輸出驗證與條件式分支。透過這種方法，原本可能導致不可靠或不完整結果的單一多面向請求，會被轉換成由底層執行框架管理的結構化操作序列。

### 7. 多模態與多步驟推理

分析具有多種模態的資料集，需要將問題拆成更小、以提示為基礎的任務。例如，解讀一張包含內嵌文字的圖片、標示特定文字片段的標籤，以及說明每個標籤的表格資料，就需要這樣的方法。

* 提示 1：從使用者的影像請求中擷取並理解文字。  
* 提示 2：將擷取出的影像文字與對應標籤連結。  
* 提示 3：使用表格解讀蒐集到的資訊，以判斷所需輸出。

# 實作程式碼範例

實作提示鏈的方式，範圍可以從腳本中直接、循序的函式呼叫，到使用專為管理控制流程、狀態與元件整合而設計的專門框架。LangChain、LangGraph、Crew AI 與 Google Agent Development Kit（ADK）等框架，提供了建構與執行這些多步驟流程的結構化環境，對複雜架構尤其有利。

為了示範，LangChain 與 LangGraph 是合適的選擇，因為它們的核心 API 明確設計用來組合操作鏈與圖。LangChain 提供線性序列的基礎抽象，而 LangGraph 則擴展這些能力，以支援具狀態與循環式運算，這是實作更精密代理式行為所必需的。本範例將聚焦於基本的線性序列。

以下程式碼實作了一個兩步驟提示鏈，功能相當於資料處理管線。初始階段設計用來解析非結構化文字並擷取特定資訊。後續階段接收這個擷取輸出，並將其轉換為結構化資料格式。

若要重現此程序，必須先安裝所需的程式庫。可使用以下命令完成：

```bash
pip install langchain langchain-community langchain-openai langgraph
```

請注意，langchain-openai 可替換為適用於不同模型供應商的套件。接著，必須使用所選語言模型供應商所需的 API 認證來設定執行環境，例如 OpenAI、Google Gemini 或 Anthropic。

```python
import os 
from langchain_openai import ChatOpenAI 
from langchain_core.prompts import ChatPromptTemplate 
from langchain_core.output_parsers import StrOutputParser 

# For better security, load environment variables from a .env file 
# from dotenv import load_dotenv 
# load_dotenv() 
# Make sure your OPENAI_API_KEY is set in the .env file 

# Initialize the Language Model (using ChatOpenAI is recommended) 

llm = ChatOpenAI(temperature=0) 

# --- Prompt 1: Extract Information ---

prompt_extract = ChatPromptTemplate.from_template(
    "Extract the technical specifications from the following text:\n\n{text_input}" 
) 

# --- Prompt 2: Transform to JSON --- 

prompt_transform = ChatPromptTemplate.from_template(
    "Transform the following specifications into a JSON object with 'cpu', 'memory', and 'storage' as keys:\n\n{specifications}" 
) 

# --- Build the Chain using LCEL --- 
# The StrOutputParser() converts the LLM's message output to a simple string. 
extraction_chain = prompt_extract | llm | StrOutputParser() 

# The full chain passes the output of the extraction chain into the 'specifications' 
# variable for the transformation prompt. 
full_chain = (    
    {"specifications": extraction_chain}
        | 
    prompt_transform
        | 
    llm
        | 
    StrOutputParser() 
) 

# --- Run the Chain --- 

input_text = "The new laptop model features a 3.5 GHz octa-core processor, 16GB of RAM, and a 1TB NVMe SSD." 

# Execute the chain with the input text dictionary. 
final_result = full_chain.invoke({"text_input": input_text})
print("\n--- Final JSON Output ---")
print(final_result)
```

這段 Python 程式碼示範如何使用 LangChain 程式庫處理文字。它使用兩個獨立提示：一個用來從輸入字串擷取技術規格，另一個用來將這些規格格式化為 JSON 物件。ChatOpenAI 模型用於語言模型互動，而 StrOutputParser 確保輸出是可用的字串格式。LangChain Expression Language（LCEL）用來優雅地將這些提示與語言模型串接在一起。第一條鏈 `extraction_chain` 會擷取規格。接著，`full_chain` 取得擷取結果並將其作為轉換提示的輸入。範例提供了一段描述筆電的輸入文字。`full_chain` 會以這段文字被叫用，並透過兩個步驟處理。最後結果是一個包含已擷取且格式化規格的 JSON 字串，然後會被印出。

## 上下文工程與提示工程

上下文工程（Context Engineering，見圖 1）是一門系統化學科，負責在 token 生成之前，為 AI 模型設計、建構並提供完整的資訊環境。這種方法主張，模型輸出的品質較少取決於模型架構本身，而更取決於所提供上下文的豐富程度。

![上下文工程](../assets/context_engineering.png)

圖 1：上下文工程是一門為 AI 建立豐富且完整資訊環境的學科，因為這種上下文的品質，是促成進階代理式效能的主要因素。

它代表了從傳統提示工程而來的重要演進；傳統提示工程主要聚焦於最佳化使用者即時查詢的措辭。上下文工程將範圍擴展到多個資訊層次，例如 **系統提示（system prompt）**，也就是一組定義 AI 操作參數的基礎指示；例如：*「你是一位技術寫作者；你的語氣必須正式且精確。」* 上下文還會透過外部資料進一步豐富。這包括檢索到的文件，也就是 AI 主動從知識庫擷取資訊以支援其回應，例如拉取專案的技術規格。它也納入工具輸出，也就是 AI 使用外部 API 取得即時資料後得到的結果，例如查詢行事曆以判斷使用者是否有空。這些明確資料會與關鍵的隱含資料結合，例如使用者身分、互動歷史與環境狀態。核心原則是，即使是先進模型，若只取得有限或建構不佳的操作環境視圖，也會表現不佳。

因此，這項實務會將任務從單純回答問題，重新定義為替代理建立完整的操作圖像。例如，一個經過上下文工程設計的代理，不會只是回應查詢，而會先整合使用者的行事曆可用性（工具輸出）、與電子郵件收件人的專業關係（隱含資料），以及先前會議的筆記（檢索到的文件）。這讓模型能產生高度相關、個人化且實務上有用的輸出。「工程」這個組成部分，涉及建立穩健的管線，以便在執行期間擷取並轉換這些資料，同時建立回饋迴路來持續改善上下文品質。

若要實作這點，可以使用專門的調校系統，大規模自動化改善流程。例如，Google 的 Vertex AI prompt optimizer 等工具，可以透過根據一組範例輸入與預先定義的評估指標系統化評估回應，來提升模型效能。這種方法能有效調整不同模型上的提示與系統指示，而不需要大量人工重寫。提供這類最佳化器範例提示、系統指示與範本後，它就能以程式化方式精煉上下文輸入，為實作精密上下文工程所需的回饋迴路提供結構化方法。

這種結構化方法，正是區分初階 AI 工具與更精密、具上下文感知能力系統的關鍵。它將上下文本身視為主要元件，並高度重視代理知道什麼、何時知道，以及如何使用這些資訊。這項實務確保模型能全面理解使用者的意圖、歷史與當前環境。最終，上下文工程是將無狀態聊天機器人推進為高度能幹、具情境感知能力系統的關鍵方法。

## 快速總覽

**是什麼：** 複雜任務若在單一提示中處理，往往會使 LLM 不堪負荷，並導致顯著的效能問題。模型的認知負荷會提高出錯的可能性，例如忽略指示、失去上下文，以及生成錯誤資訊。單體式提示難以有效管理多項限制與循序推理步驟。結果是輸出不可靠且不準確，因為 LLM 無法處理多面向請求的所有層面。

**為什麼：** 提示鏈提供了一種標準化解法，將複雜問題拆成一連串較小且相互連結的子任務。鏈中的每個步驟都使用聚焦提示執行特定操作，顯著提升可靠性與控制力。某個提示的輸出會作為下一個提示的輸入，建立逐步邁向最終解決方案的邏輯工作流程。這種模組化、分而治之的策略，使流程更容易管理、更容易除錯，也允許在步驟之間整合外部工具或結構化資料格式。這個模式是開發精密、多步驟代理式系統的基礎，讓系統能規劃、推理並執行複雜工作流程。

**經驗法則：** 當任務對單一提示來說過於複雜、涉及多個不同處理階段、需要在步驟之間與外部工具互動，或是在建構需要執行多步驟推理並維持狀態的代理式系統時，請使用此模式。

**視覺摘要：**

![提示鏈模式](../assets/Prompt_Chaining_Pattern.png)

圖 2：提示鏈模式：代理會從使用者接收一系列提示，而每個代理的輸出會作為鏈中下一個代理的輸入。

## 重點整理

以下是幾個重點：

* 提示鏈會將複雜任務拆解成一連串較小且聚焦的步驟。有時也稱為管線模式。  
* 鏈中的每個步驟都涉及一次 LLM 呼叫或處理邏輯，並使用前一步驟的輸出作為輸入。  
* 這個模式能提高與語言模型進行複雜互動時的可靠性與可管理性。  
* LangChain/LangGraph 與 Google ADK 等框架提供了穩健的工具，可用來定義、管理與執行這些多步驟序列。

## 結論

透過將複雜問題拆解成一連串更簡單、更容易管理的子任務，提示鏈提供了一個用來引導大型語言模型的穩健框架。這種「分而治之」策略透過讓模型一次專注於一個特定操作，顯著提升輸出的可靠性與控制力。作為基礎模式，它讓我們能開發具備多步驟推理、工具整合與狀態管理能力的精密 AI 代理。最終，掌握提示鏈對於建構穩健、具上下文感知能力的系統至關重要，這些系統能執行遠超出單一提示能力範圍的複雜工作流程。

## 參考資料

1. LangChain Documentation on LCEL: [https://python.langchain.com/v0.2/docs/core_modules/expression_language/](https://python.langchain.com/v0.2/docs/core_modules/expression_language/)
2. LangGraph Documentation: [https://langchain-ai.github.io/langgraph/](https://langchain-ai.github.io/langgraph/)  
3. Prompt Engineering Guide \- Chaining Prompts: [https://www.promptingguide.ai/techniques/chaining](https://www.promptingguide.ai/techniques/chaining)
4. OpenAI API Documentation (General Prompting Concepts): [https://platform.openai.com/docs/guides/gpt/prompting](https://platform.openai.com/docs/guides/gpt/prompting)
5. Crew AI Documentation (Tasks and Processes): [https://docs.crewai.com/](https://docs.crewai.com/)
6. Google AI for Developers (Prompting Guides): [https://cloud.google.com/discover/what-is-prompt-engineering?hl=en](https://cloud.google.com/discover/what-is-prompt-engineering?hl=en)
7. Vertex Prompt Optimizer [https://cloud.google.com/vertex-ai/generative-ai/docs/learn/prompts/prompt-optimizer](https://cloud.google.com/vertex-ai/generative-ai/docs/learn/prompts/prompt-optimizer)
