# 第 6 章：規劃（Planning）

智慧行為通常不只是回應當下輸入而已。它需要預見能力，能將複雜任務拆解成更小且可管理的步驟，並制定策略以達成期望結果。這正是規劃模式發揮作用的地方。從根本上說，規劃是代理（agent）或代理系統擬定一連串行動的能力，用來從初始狀態推進到目標狀態。

## 規劃模式概觀

在 AI 的上下文中，把規劃代理想成一位你可以委派複雜目標的專家，會很有幫助。當你要求它「安排一次團隊外部活動」時，你是在定義「要做什麼」——也就是目標與限制條件——但不是定義「怎麼做」。代理的核心任務，是自主規劃通往該目標的路徑。它必須先理解初始狀態（例如預算、參與人數、理想日期）與目標狀態（成功預訂外部活動），接著找出連接兩者的最佳行動順序。這個計畫並非事先已知；它是為了回應請求而建立的。

這個過程的一項特徵是適應性。初始計畫只是一個起點，而不是僵固的腳本。代理真正的力量，在於能納入新資訊，並引導專案繞過障礙。例如，如果偏好的場地已無法預訂，或選定的餐飲供應商已額滿，有能力的代理不會只是失敗收場。它會適應情況，記錄新的限制條件，重新評估選項，並制定新的計畫，例如建議替代場地或日期。

然而，必須認清彈性與可預測性之間的取捨。動態規劃是一種特定工具，而不是萬用解法。當問題的解法已經充分理解且可重複時，將代理限制在預先決定的固定工作流程中會更有效。這種方法會限制代理的自主性，以降低不確定性與不可預測行為的風險，確保結果可靠且一致。因此，要使用規劃代理還是簡單的任務執行代理，取決於一個問題：「怎麼做」是否需要被探索出來，還是已經明確知道？

## 實際應用與使用案例

規劃模式是自主系統中的核心運算流程，使代理能合成一連串行動以達成指定目標，尤其適用於動態或複雜環境。這個流程會將高階目標轉換成由離散、可執行步驟組成的結構化計畫。

在程序式任務自動化等領域中，規劃可用來協調複雜工作流程。例如，新員工到職這類業務流程，可以拆解成一連串有方向性的子任務，例如建立系統帳號、指派訓練模組，以及與不同部門協調。代理會產生一份計畫，以邏輯順序執行這些步驟，並呼叫必要工具或與各種系統互動來管理相依性。

在機器人與自主導航中，規劃是狀態空間遍歷的基礎。無論是實體機器人還是虛擬實體，系統都必須產生路徑或一連串行動，才能從初始狀態轉移到目標狀態。這涉及在遵守環境限制的同時，最佳化時間或能源消耗等指標，例如避開障礙物或遵守交通規則。

這個模式對結構化資訊合成也很關鍵。當任務是產生研究報告等複雜輸出時，代理可以制定一份計畫，其中包含資訊蒐集、資料摘要、內容結構化與反覆精煉等不同階段。同樣地，在需要多步驟問題解決的客戶支援情境中，代理可以建立並遵循一套系統化計畫，進行診斷、解決方案實作與升級處理。

本質上，規劃模式讓代理能超越簡單的反應式行動，進入目標導向的行為。它提供必要的邏輯框架，用來解決需要一連串相互依賴操作的問題。

## 實作程式碼（Crew AI）

以下章節將示範如何使用 Crew AI 框架實作 Planner pattern。這個模式包含一個代理，它會先制定多步驟計畫來處理複雜查詢，然後依序執行該計畫。

```python
import os
from dotenv import load_dotenv
from crewai import Agent, Task, Crew, Process
from langchain_openai import ChatOpenAI


# Load environment variables from .env file for security
load_dotenv()


# 1. Explicitly define the language model for clarity
llm = ChatOpenAI(model="gpt-4-turbo")


# 2. Define a clear and focused agent
planner_writer_agent = Agent(
    role='Article Planner and Writer',
    goal='Plan and then write a concise, engaging summary on a specified topic.',
    backstory=(
        'You are an expert technical writer and content strategist. '
        'Your strength lies in creating a clear, actionable plan before writing, '
        'ensuring the final summary is both informative and easy to digest.'
    ),
    verbose=True,
    allow_delegation=False,
    llm=llm,  # Assign the specific LLM to the agent
)


# 3. Define a task with a more structured and specific expected output
topic = "The importance of Reinforcement Learning in AI"

high_level_task = Task(
    description=(
        f"1. Create a bullet-point plan for a summary on the topic: '{topic}'.\n"
        f"2. Write the summary based on your plan, keeping it around 200 words."
    ),
    expected_output=(
        "A final report containing two distinct sections:\n\n"
        "### Plan\n"
        "- A bulleted list outlining the main points of the summary.\n\n"
        "### Summary\n"
        "- A concise and well-structured summary of the topic."
    ),
    agent=planner_writer_agent,
)


# Create the crew with a clear process
crew = Crew(
    agents=[planner_writer_agent],
    tasks=[high_level_task],
    process=Process.sequential,
)


# Execute the task
print("## Running the planning and writing task ##")
result = crew.kickoff()

print("\n\n---\n## Task Result ##\n---")
print(result)
```

這段程式碼使用 CrewAI 程式庫建立一個 AI 代理，負責針對指定主題進行規劃並撰寫摘要。它一開始匯入必要的程式庫，包括 Crew.ai 和 `langchain_openai`，並從 .env 檔載入環境變數。接著明確定義一個 ChatOpenAI 語言模型供代理使用。程式建立名為 `planner_writer_agent` 的 Agent，並指定特定角色與目標：先規劃，再撰寫精簡摘要。代理的 backstory 強調其在規劃與技術寫作方面的專業。Task 以清楚的 description 定義，要求先建立計畫，再針對主題 "The importance of Reinforcement Learning in AI" 撰寫摘要，並為預期輸出指定特定格式。Crew 由代理與任務組成，並設定為依序處理。最後，呼叫 crew.kickoff() 方法來執行已定義的任務，並印出結果。

## Google DeepResearch

Google Gemini DeepResearch（見圖 1）是一個以代理為基礎的系統，專為自主資訊檢索與合成而設計。它透過多步驟代理式管線運作，動態且反覆地查詢 Google Search，以系統化方式探索複雜主題。該系統被設計用來處理大量網路來源語料，評估蒐集到的資料在相關性與知識缺口上的狀況，並執行後續搜尋以補足缺口。最終輸出會將經過審核的資訊整合成結構化、多頁摘要，並附上原始來源的引用。

進一步來看，這個系統的運作不是單次查詢與回應事件，而是一個受管理、長時間執行的流程。它會先將使用者的提示拆解成多點研究計畫（見圖 1），再呈現給使用者審閱與修改。如此一來，使用者可在執行前共同塑造研究路徑。計畫核准後，代理式管線就會啟動反覆的搜尋與分析迴圈。這不只是執行一連串預先定義的搜尋；代理會根據蒐集到的資訊，動態制定並精煉查詢，主動找出知識缺口、佐證資料點，並解決不一致之處。

![Google Deep Research 代理產生使用 Google Search 作為工具的執行計畫](../assets/Google_Deep_Research_Agent_Generating_an_execution_plan_for_using_Google_Search_as_a_Tool.png)

圖 1：Google Deep Research 代理產生使用 Google Search 作為工具的執行計畫。

一個關鍵架構元件，是系統能以非同步方式管理這個流程。這項設計確保調查過程即使涉及分析數百個來源，也能承受單點失敗，並允許使用者暫時離開，待完成後再收到通知。系統也能整合使用者提供的文件，將私有來源資訊與網路研究結合。最終輸出不只是把發現串接成清單，而是一份結構化、多頁報告。在合成階段，模型會對蒐集到的資訊進行關鍵評估，找出主要主題，並將內容組織成具有邏輯區段的連貫敘事。報告被設計成可互動形式，通常包含音訊概覽、圖表，以及通往原始引用來源的連結，讓使用者能進行驗證與進一步探索。除了合成結果之外，模型也會明確回傳其搜尋與參考過的完整來源清單（見圖 2）。這些來源會以引用形式呈現，提供完整透明度，並可直接存取主要資訊。整個流程會將簡單查詢轉換成完整且經合成的知識體。

![Deep Research 計畫執行範例，結果是使用 Google Search 作為工具搜尋各種網路來源](../assests/Example_of_Deep_Research_Plan_Being_Executed_Resulting_in_Google_Search_being_used_as_a_Tool_to_Search_Various_Web_Sources.png)

圖 2：Deep Research 計畫執行範例，結果是使用 Google Search 作為工具搜尋各種網路來源。

Gemini DeepResearch 減少了手動資料取得與合成所需的大量時間與資源投入，因此提供了一種更有結構且更完整的資訊發掘方法。系統的價值在各領域複雜且多面向的研究任務中特別明顯。

例如，在競爭分析中，可以指示代理系統化蒐集並彙整市場趨勢、競爭者產品規格、來自多元線上來源的公眾情緒，以及行銷策略等資料。這個自動化流程取代了手動追蹤多個競爭者的繁重工作，讓分析師能專注於更高層次的策略解讀，而不是資料蒐集（見圖 3）。

![Google Deep Research 代理產生的最終輸出，代表我們分析使用 Google Search 作為工具取得的來源](../assets/Final_Output_Generated_by_Google_Deep_Research_Agent_Analyzing_on_our_Behalf_Sources_Obtained_using_Google_Search_as_a_Tool.png)

圖 3：Google Deep Research 代理產生的最終輸出，代表我們分析使用 Google Search 作為工具取得的來源。

同樣地，在學術探索中，該系統可作為進行廣泛文獻回顧的強大工具。它能識別並摘要基礎論文，追蹤概念在眾多出版物中的發展，並描繪特定領域中正在興起的研究前沿，從而加速學術探究最初且最耗時的階段。

這種方法的效率，來自於將反覆搜尋與篩選循環自動化；而這正是手動研究中的核心瓶頸。系統能處理的資訊來源數量與種類，通常比人類研究者在相同時間內可行的範圍更大，因此能達成完整性。更廣的分析範圍有助於降低選擇偏誤的可能性，並提高發現不明顯但可能關鍵資訊的機率，進而對主題形成更穩健且有充分依據的理解。

## OpenAI Deep Research API

OpenAI Deep Research API 是一項專門用來自動化複雜研究任務的工具。它使用進階的代理式模型，能獨立推理、規劃，並從真實世界來源合成資訊。不同於簡單的 Q\&A 模型，它會接收高階查詢並自主拆解成子問題，使用內建工具進行網路搜尋，最後交付一份結構化且引用豐富的報告。此 API 提供對整個流程的直接程式化存取；在撰寫本文時，可使用 o3-deep-research-2025-06-26 等模型進行高品質合成，也可使用較快的 o4-mini-deep-research-2025-06-26 來支援對延遲敏感的應用。

Deep Research API 很有用，因為它能自動化原本可能需要數小時的手動研究，交付專業等級、資料驅動的報告，適合用來支援商業策略、投資決策或政策建議。其主要優點包括：

* **結構化、附引用的輸出：** 它會產生組織良好的報告，並以連結至來源中繼資料的行內引用支援內容，確保主張可驗證且有資料依據。  
* **透明度：** 不同於 ChatGPT 中較抽象的流程，API 會揭露所有中間步驟，包括代理的推理、它執行的特定網路搜尋查詢，以及它執行過的任何程式碼。這有助於詳細除錯、分析，並更深入理解最終答案是如何建構出來的。  
* **可擴充性：** 它支援 Model Context Protocol（MCP），讓開發者能將代理連接到私有知識庫與內部資料來源，將公開網路研究與專有資訊結合。

若要使用 API，你會將請求傳送到 client.responses.create endpoint，指定模型、輸入提示，以及代理可使用的工具。輸入通常包含 `system_message`，用來定義代理的人格設定與期望輸出格式，並搭配 `user_query`。你也必須包含 `web_search_preview` 工具，並可選擇加入其他工具，例如 `code_interpreter` 或自訂 MCP 工具（見第 10 章），以使用內部資料。

```python
from openai import OpenAI


# Initialize the client with your API key
client = OpenAI(api_key="YOUR_OPENAI_API_KEY")


# Define the agent's role and the user's research question
system_message = """
You are a professional researcher preparing a structured, data-driven report.
Focus on data-rich insights, use reliable sources, and include inline citations.
"""

user_query = "Research the economic impact of semaglutide on global healthcare systems."


# Create the Deep Research API call
response = client.responses.create(
    model="o3-deep-research-2025-06-26",
    input=[
        {
            "role": "developer",
            "content": [{"type": "input_text", "text": system_message}],
        },
        {
            "role": "user",
            "content": [{"type": "input_text", "text": user_query}],
        },
    ],
    reasoning={"summary": "auto"},
    tools=[{"type": "web_search_preview"}],
)


# Access and print the final report from the response
final_report = response.output[-1].content[0].text
print(final_report)


# --- ACCESS INLINE CITATIONS AND METADATA ---
print("--- CITATIONS ---")
annotations = response.output[-1].content[0].annotations

if not annotations:
    print("No annotations found in the report.")
else:
    for i, citation in enumerate(annotations):
        # The text span the citation refers to
        cited_text = final_report[citation.start_index : citation.end_index]
        print(f"Citation {i + 1}:")
        print(f"  Cited Text: {cited_text}")
        print(f"  Title: {citation.title}")
        print(f"  URL: {citation.url}")
        print(f"  Location: chars {citation.start_index}–{citation.end_index}")

print("\n" + "=" * 50 + "\n")


# --- INSPECT INTERMEDIATE STEPS ---
print("--- INTERMEDIATE STEPS ---")

# 1. Reasoning Steps: Internal plans and summaries generated by the model.
try:
    reasoning_step = next(item for item in response.output if item.type == "reasoning")
    print("\n[Found a Reasoning Step]")
    for summary_part in reasoning_step.summary:
        print(f"  - {summary_part.text}")
except StopIteration:
    print("\nNo reasoning steps found.")

# 2. Web Search Calls: The exact search queries the agent executed.
try:
    search_step = next(item for item in response.output if item.type == "web_search_call")
    print("\n[Found a Web Search Call]")
    print(f"  Query Executed: '{search_step.action['query']}'")
    print(f"  Status: {search_step.status}")
except StopIteration:
    print("\nNo web search steps found.")

# 3. Code Execution: Any code run by the agent using the code interpreter.
try:
    code_step = next(item for item in response.output if item.type == "code_interpreter_call")
    print("\n[Found a Code Execution Step]")
    print("  Code Input:")
    print(f"  ```python\n{code_step.input}\n  ```")
    print("  Code Output:")
    print(f"  {code_step.output}")
except StopIteration:
    print("\nNo code execution steps found.")
```

這段程式碼片段使用 OpenAI API 執行一項「Deep Research」任務。它一開始使用你的 API key 初始化 OpenAI client，這對驗證身分非常重要。接著，它將 AI 代理的角色定義為專業研究員，並設定使用者關於 semaglutide 對全球醫療保健系統經濟影響的研究問題。程式碼建構對 o3-deep-research-2025-06-26 模型的 API 呼叫，並提供已定義的 system message 與 user query 作為輸入。它也要求自動產生推理摘要，並啟用網路搜尋能力。完成 API 呼叫後，它會擷取並印出最終產生的報告。

接著，它會嘗試從報告的 annotations 存取並顯示行內引用與中繼資料，包括被引用文字、標題、URL，以及在報告中的位置。最後，它會檢查並印出模型所採取中間步驟的詳細資訊，例如推理步驟、網路搜尋呼叫（包含執行的查詢），以及如果使用了 code interpreter，則顯示任何程式碼執行步驟。

## 快速總覽

**是什麼：** 複雜問題往往無法透過單一行動解決，而需要預見能力才能達成期望結果。若沒有結構化方法，代理式系統會難以處理包含多個步驟與相依性的多面向請求。這使得系統難以將高階目標拆解成可管理的一系列較小、可執行任務。因此，系統無法有效制定策略，面對錯綜複雜的目標時，可能產生不完整或不正確的結果。

**為什麼：** 規劃模式提供一種標準化解法，讓代理式系統先建立連貫計畫來處理目標。它會將高階目標拆解成一連串較小且可行動的步驟或子目標。這使系統能管理複雜工作流程、協調各種工具，並以邏輯順序處理相依性。大型語言模型（large language model, LLM）特別適合這件事，因為它們能根據龐大的訓練資料產生合理且有效的計畫。這種結構化方法會將簡單的反應式代理轉變成策略型執行者，能主動朝複雜目標推進，必要時甚至能調整計畫。

**經驗法則：** 當使用者請求太複雜，無法由單一行動或工具處理時，就使用這個模式。它非常適合自動化多步驟流程，例如產生詳細研究報告、新員工到職，或執行競爭分析。只要任務需要一連串相互依賴的操作，才能達到最終合成結果，就適合套用規劃模式。

**視覺摘要**  

![規劃設計模式](../assets/Planning_Design_Pattern.png)

圖 4；規劃設計模式

## 重點整理

* 規劃讓代理能將複雜目標拆解成可行動、依序執行的步驟。  
* 它對處理多步驟任務、工作流程自動化，以及在複雜環境中導航至關重要。  
* LLM 能根據任務描述產生逐步方法來進行規劃。  
* 在代理框架中，明確提示或設計任務要求規劃步驟，可以鼓勵這種行為。  
* Google Deep Research 是一個代理，代表我們分析使用 Google Search 作為工具取得的來源。它會反思、規劃並執行

## 結論

總而言之，規劃模式是一項基礎元件，能將代理式系統從簡單的反應式回應者提升為具策略性、目標導向的執行者。現代 LLM 提供了這項核心能力，能自主將高階目標拆解成連貫且可行動的步驟。這個模式的規模可以從直接、依序的任務執行開始，例如 CrewAI 代理建立並遵循寫作計畫的示範，也能擴展到更複雜且動態的系統。Google DeepResearch 代理體現了這種進階應用，會建立反覆式研究計畫，並根據持續蒐集到的資訊進行調整與演進。最終，對於複雜問題而言，規劃提供了人類意圖與自動化執行之間的關鍵橋樑。透過建構問題解決方法，這個模式讓代理能管理複雜工作流程，並交付完整且經合成的結果。

## 參考資料

1. Google DeepResearch (Gemini Feature): [gemini.google.com](http://gemini.google.com)
2. OpenAI ,Introducing deep research  [https://openai.com/index/introducing-deep-research/](https://openai.com/index/introducing-deep-research/)
3. Perplexity, Introducing Perplexity Deep Research, [https://www.perplexity.ai/hub/blog/introducing-perplexity-deep-research](https://www.perplexity.ai/hub/blog/introducing-perplexity-deep-research)
