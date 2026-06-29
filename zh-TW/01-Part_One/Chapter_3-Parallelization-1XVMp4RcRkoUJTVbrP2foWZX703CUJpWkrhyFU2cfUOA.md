# 第 3 章：平行化（Parallelization）

## 平行化模式概覽

在前幾章中，我們探討了用於循序工作流程的提示鏈（Prompt Chaining），以及用於動態決策與不同路徑之間轉換的路由（Routing）。雖然這些模式都很重要，但許多複雜的代理式任務包含多個子任務，這些子任務可以*同時*執行，而不是一個接著一個執行。這正是**平行化**模式變得關鍵的地方。

平行化是指同時執行多個元件，例如大型語言模型（large language model, LLM）呼叫、工具使用，甚至整個子代理（見圖 1）。平行執行不必等某個步驟完成後才開始下一個步驟，而是讓彼此獨立的任務同時運行；對於能拆解成獨立部分的任務，這能大幅降低整體執行時間。

假設有一個代理（agent）被設計用來研究某個主題並摘要其發現。循序方法可能會：

1. 搜尋來源 A。  
2. 摘要來源 A。  
3. 搜尋來源 B。  
4. 摘要來源 B。  
5. 根據摘要 A 與 B 綜合出最終答案。

平行方法則可以改成：

1. 同時搜尋來源 A *和*搜尋來源 B。  
2. 當兩個搜尋都完成後，同時摘要來源 A *和*摘要來源 B。  
3. 根據摘要 A 與 B 綜合出最終答案（這個步驟通常是循序的，會等待平行步驟完成）。

核心概念是找出工作流程中不依賴其他部分輸出的部分，並讓它們平行執行。當處理具有延遲的外部服務（例如 API 或資料庫）時，這特別有效，因為你可以同時發出多個請求。

實作平行化通常需要支援非同步執行或多執行緒／多處理程序的框架。現代代理式框架在設計時多半已考量非同步操作，讓你可以輕鬆定義能平行運行的步驟。

![Parallelization with Sub-Agents](../assets/Parallelization_with_Sub_Agents.png)

圖 1：使用子代理進行平行化的範例

LangChain、LangGraph 與 Google ADK 等框架都提供平行執行機制。在 LangChain Expression Language (LCEL) 中，你可以透過使用 `|`（用於循序）等運算子組合 runnable 物件，並將鏈或圖結構化為可同時執行的分支，來達成平行執行。LangGraph 以圖結構為基礎，允許你定義多個可由單一狀態轉換執行的節點，實際上就是在工作流程中啟用平行分支。Google ADK 提供強健的原生機制來促成並管理代理的平行執行，顯著提升複雜多代理系統的效率與可擴展性。ADK 框架內建的這項能力，讓開發者能設計並實作多個代理可同時運作的解決方案，而不是只能循序執行。

平行化模式對提升代理式系統的效率與回應能力非常重要，尤其適用於包含多個獨立查詢、計算，或與外部服務互動的任務。它是最佳化複雜代理工作流程效能的關鍵技術。

## 實務應用與使用案例

平行化是可在各種應用中最佳化代理效能的強大模式：

### 1. 資訊蒐集與研究

同時從多個來源收集資訊，是經典的使用案例。

* **使用案例：** 研究某家公司的代理。  
  * **平行任務：** 同時搜尋新聞文章、擷取股票資料、檢查社群媒體提及內容，並查詢公司資料庫。  
  * **效益：** 比循序查詢更快取得全面觀點。

### 2. 資料處理與分析

同時套用不同分析技術，或處理不同資料區段。

* **使用案例：** 分析客戶回饋的代理。  
  * **平行任務：** 對一批回饋項目同時執行情緒分析、擷取關鍵字、分類回饋，並識別緊急問題。  
  * **效益：** 快速提供多面向分析。

### 3. 多 API 或工具互動

呼叫多個彼此獨立的 API 或工具，以收集不同類型的資訊或執行不同動作。

* **使用案例：** 旅遊規劃代理。  
  * **平行任務：** 同時查詢機票價格、搜尋飯店空房、查找當地活動，並尋找餐廳推薦。  
  * **效益：** 更快呈現完整旅遊計畫。

### 4. 多元件內容生成

平行產生複雜內容的不同部分。

* **使用案例：** 建立行銷電子郵件的代理。  
  * **平行任務：** 同時產生主旨、撰寫電子郵件本文、尋找相關圖片，並建立行動呼籲按鈕文字。  
  * **效益：** 更有效率地組合最終電子郵件。

### 5. 驗證與確認

同時執行多個獨立檢查或驗證。

* **使用案例：** 驗證使用者輸入的代理。  
  * **平行任務：** 同時檢查電子郵件格式、驗證電話號碼、對照資料庫驗證地址，並檢查是否含有髒話。  
  * **效益：** 更快提供輸入有效性的回饋。

### 6. 多模態處理

同時處理同一輸入的不同模態（文字、影像、音訊）。

* **使用案例：** 分析含有文字與圖片之社群媒體貼文的代理。  
  * **平行任務：** 同時分析文字的情緒與關鍵字，*並*分析圖片中的物件與場景描述。  
  * **效益：** 更快整合來自不同模態的洞察。

### 7. A/B 測試或多選項生成

平行產生回應或輸出的多個變體，以便選出最佳版本。

* **使用案例：** 產生不同創意文字選項的代理。  
  * **平行任務：** 使用略有不同的提示或模型，同時為一篇文章產生三個不同標題。  
  * **效益：** 可快速比較並選擇最佳選項。

平行化是代理式設計中的基本最佳化技術，讓開發者能利用獨立任務的並行執行，建構效能更好、回應更快的應用程式。

## 實作程式碼範例 (LangChain)

LangChain 框架中的平行執行由 LangChain Expression Language (LCEL) 促成。主要方法是將多個 runnable 元件組織在字典或清單結構中。當這個集合作為輸入傳遞給鏈中的後續元件時，LCEL 執行階段會同時執行其中包含的 runnables。

在 LangGraph 的上下文中，這個原則會套用到圖的拓撲。平行工作流程的定義方式，是設計圖結構，讓多個沒有直接循序依賴關係的節點能從同一個共同節點啟動。這些平行路徑會獨立執行，然後在圖中後續的匯合點彙總其結果。

以下實作示範了一個以 LangChain 框架建構的平行處理工作流程。這個工作流程設計為針對單一使用者查詢，同時執行兩個獨立操作。這些平行程序會被實例化為不同的鏈或函式，並在之後將各自的輸出彙總為單一結果。

此實作的先決條件包括安裝必要的 Python 套件，例如 langchain、langchain-community，以及像 langchain-openai 這樣的模型供應商程式庫。此外，必須在本機環境中設定所選語言模型的有效 API 金鑰，以供驗證使用。

```python
import os
import asyncio
from typing import Optional

from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import Runnable, RunnableParallel, RunnablePassthrough


# --- Configuration ---
# Ensure your API key environment variable is set (e.g., OPENAI_API_KEY)
try:
    llm: Optional[ChatOpenAI] = ChatOpenAI(model="gpt-4o-mini", temperature=0.7)
except Exception as e:
    print(f"Error initializing language model: {e}")
    llm = None


# --- Define Independent Chains ---
# These three chains represent distinct tasks that can be executed in parallel.
summarize_chain: Runnable = (
    ChatPromptTemplate.from_messages([
        ("system", "Summarize the following topic concisely:"),
        ("user", "{topic}"),
    ])
    | llm
    | StrOutputParser()
)

questions_chain: Runnable = (
    ChatPromptTemplate.from_messages([
        ("system", "Generate three interesting questions about the following topic:"),
        ("user", "{topic}"),
    ])
    | llm
    | StrOutputParser()
)

terms_chain: Runnable = (
    ChatPromptTemplate.from_messages([
        ("system", "Identify 5-10 key terms from the following topic, separated by commas:"),
        ("user", "{topic}"),
    ])
    | llm
    | StrOutputParser()
)


# --- Build the Parallel + Synthesis Chain ---
# 1. Define the block of tasks to run in parallel. The results of these,
#    along with the original topic, will be fed into the next step.
map_chain = RunnableParallel(
    {
        "summary": summarize_chain,
        "questions": questions_chain,
        "key_terms": terms_chain,
        "topic": RunnablePassthrough(),  # Pass the original topic through
    }
)

# 2. Define the final synthesis prompt which will combine the parallel results.
synthesis_prompt = ChatPromptTemplate.from_messages([
    (
        "system",
        """Based on the following information:
        Summary: {summary}
        Related Questions: {questions}
        Key Terms: {key_terms}
        Synthesize a comprehensive answer."""
    ),
    ("user", "Original topic: {topic}"),
])

# 3. Construct the full chain by piping the parallel results directly
#    into the synthesis prompt, followed by the LLM and output parser.
full_parallel_chain = map_chain | synthesis_prompt | llm | StrOutputParser()


# --- Run the Chain ---
async def run_parallel_example(topic: str) -> None:
    """
    Asynchronously invokes the parallel processing chain with a specific topic
    and prints the synthesized result.

    Args:
        topic: The input topic to be processed by the LangChain chains.
    """
    if not llm:
        print("LLM not initialized. Cannot run example.")
        return

    print(f"\n--- Running Parallel LangChain Example for Topic: '{topic}' ---")
    try:
        # The input to `ainvoke` is the single 'topic' string,
        # then passed to each runnable in the `map_chain`.
        response = await full_parallel_chain.ainvoke(topic)
        print("\n--- Final Response ---")
        print(response)
    except Exception as e:
        print(f"\nAn error occurred during chain execution: {e}")


if __name__ == "__main__":
    test_topic = "The history of space exploration"
    # In Python 3.7+, asyncio.run is the standard way to run an async function.
    asyncio.run(run_parallel_example(test_topic))
```

提供的 Python 程式碼實作了一個 LangChain 應用程式，透過平行執行有效率地處理指定主題。請注意，asyncio 提供的是並行性，而不是平行性。它在單一執行緒上透過事件迴圈達成這一點：當某個任務閒置時（例如等待網路請求），事件迴圈會智慧地切換到其他任務。這會產生多個任務同時推進的效果，但程式碼本身仍只由一個執行緒執行，並受到 Python Global Interpreter Lock (GIL) 的限制。

程式碼一開始會從 `langchain_openai` 與 `langchain_core` 匯入必要模組，包括語言模型、提示、輸出解析與 runnable 結構相關元件。程式碼接著嘗試初始化 ChatOpenAI 實例，明確使用 "gpt-4o-mini" 模型，並設定用於控制創造性的 temperature。為了讓語言模型初始化更穩健，這裡使用 try-except 區塊。接著定義三個獨立的 LangChain「chains」，每個鏈都設計為對輸入主題執行不同任務。第一個鏈用於簡潔摘要主題，使用系統訊息與包含主題佔位符的使用者訊息。第二個鏈設定為產生三個與主題相關的有趣問題。第三個鏈設定為從輸入主題中識別 5 到 10 個關鍵詞，並要求以逗號分隔。每個獨立鏈都包含針對其特定任務調整的 ChatPromptTemplate，後面接著已初始化的語言模型，以及將輸出格式化為字串的 StrOutputParser。

接著建構一個 RunnableParallel 區塊，將這三個鏈包在一起，使它們能同時執行。這個平行 runnable 也包含 RunnablePassthrough，以確保後續步驟可以使用原始輸入主題。最後的綜合步驟則定義了一個獨立的 ChatPromptTemplate，接收摘要、問題、關鍵詞與原始主題作為輸入，用來產生完整答案。名為 `full_parallel_chain` 的完整端到端處理鏈，是透過將 `map_chain`（平行區塊）接到綜合提示，再接上語言模型與輸出解析器而建立。範例提供了非同步函式 `run_parallel_example`，示範如何叫用這個 `full_parallel_chain`。此函式接收主題作為輸入，並使用 invoke 執行非同步鏈。最後，標準 Python `if __name__ \== "__main__":` 區塊示範如何使用 asyncio.run 管理非同步執行，並以範例主題 "The history of space exploration" 執行 `run_parallel_example`。

本質上，這段程式碼建立了一個工作流程：針對指定主題，同時發生多個 LLM 呼叫（摘要、問題與詞彙），接著由最後一次 LLM 呼叫合併它們的結果。這展示了使用 LangChain 在代理式工作流程中實作平行化的核心概念。

## 實作程式碼範例 (Google ADK)

好的，現在讓我們把注意力轉向一個具體範例，說明這些概念在 Google ADK 框架中的運作方式。我們會檢視 ParallelAgent 與 SequentialAgent 等 ADK 基本元件，如何套用於建構利用並行執行提升效率的代理流程。

```python
from google.adk.agents import LlmAgent, ParallelAgent, SequentialAgent
from google.adk.tools import google_search

GEMINI_MODEL = "gemini-2.0-flash"


# --- 1. Define Researcher Sub-Agents (to run in parallel) ---

# Researcher 1: Renewable Energy
researcher_agent_1 = LlmAgent(
    name="RenewableEnergyResearcher",
    model=GEMINI_MODEL,
    instruction="""You are an AI Research Assistant specializing in energy. Research the latest advancements in 'renewable energy sources'. Use the Google Search tool provided. Summarize your key findings concisely (1-2 sentences). Output *only* the summary. """,
    description="Researches renewable energy sources.",
    tools=[google_search],
    # Store result in state for the merger agent
    output_key="renewable_energy_result",
)

# Researcher 2: Electric Vehicles
researcher_agent_2 = LlmAgent(
    name="EVResearcher",
    model=GEMINI_MODEL,
    instruction="""You are an AI Research Assistant specializing in transportation. Research the latest developments in 'electric vehicle technology'. Use the Google Search tool provided. Summarize your key findings concisely (1-2 sentences). Output *only* the summary. """,
    description="Researches electric vehicle technology.",
    tools=[google_search],
    # Store result in state for the merger agent
    output_key="ev_technology_result",
)

# Researcher 3: Carbon Capture
researcher_agent_3 = LlmAgent(
    name="CarbonCaptureResearcher",
    model=GEMINI_MODEL,
    instruction="""You are an AI Research Assistant specializing in climate solutions. Research the current state of 'carbon capture methods'. Use the Google Search tool provided. Summarize your key findings concisely (1-2 sentences). Output *only* the summary. """,
    description="Researches carbon capture methods.",
    tools=[google_search],
    # Store result in state for the merger agent
    output_key="carbon_capture_result",
)


# --- 2. Create the ParallelAgent (Runs researchers concurrently) ---
# This agent orchestrates the concurrent execution of the researchers.
# It finishes once all researchers have completed and stored their results in state.
parallel_research_agent = ParallelAgent(
    name="ParallelWebResearchAgent",
    sub_agents=[researcher_agent_1, researcher_agent_2, researcher_agent_3],
    description="Runs multiple research agents in parallel to gather information.",
)


# --- 3. Define the Merger Agent (Runs after the parallel agents) ---
# This agent takes the results stored in the session state by the parallel agents
# and synthesizes them into a single, structured response with attributions.
merger_agent = LlmAgent(
    name="SynthesisAgent",
    model=GEMINI_MODEL,  # Or potentially a more powerful model if needed for synthesis
    instruction="""You are an AI Assistant responsible for combining research findings into a structured report. Your primary task is to synthesize the following research summaries, clearly attributing findings to their source areas. Structure your response using headings for each topic. Ensure the report is coherent and integrates the key points smoothly.

**Crucially:** Your entire response MUST be grounded *exclusively* on the information provided in the 'Input Summaries' below. Do NOT add any external knowledge, facts, or details not present in these specific summaries.

**Input Summaries:**
*   **Renewable Energy:**
    {renewable_energy_result}
*   **Electric Vehicles:**
    {ev_technology_result}
*   **Carbon Capture:**
    {carbon_capture_result}

**Output Format:**
## Summary of Recent Sustainable Technology Advancements

### Renewable Energy Findings (Based on RenewableEnergyResearcher's findings)
[Synthesize and elaborate *only* on the renewable energy input summary provided above.]

### Electric Vehicle Findings (Based on EVResearcher's findings)
[Synthesize and elaborate *only* on the EV input summary provided above.]

### Carbon Capture Findings (Based on CarbonCaptureResearcher's findings)
[Synthesize and elaborate *only* on the carbon capture input summary provided above.]

### Overall Conclusion
[Provide a brief (1-2 sentence) concluding statement that connects *only* the findings presented above.]

Output *only* the structured report following this format. Do not include introductory or concluding phrases outside this structure, and strictly adhere to using only the provided input summary content.
""",
    description="Combines research findings from parallel agents into a structured, cited report, strictly grounded on provided inputs.",
    # No tools needed for merging
    # No output_key needed here, as its direct response is the final output of the sequence
)


# --- 4. Create the SequentialAgent (Orchestrates the overall flow) ---
# This is the main agent that will be run. It first executes the ParallelAgent
# to populate the state, and then executes the MergerAgent to produce the final output.
sequential_pipeline_agent = SequentialAgent(
    name="ResearchAndSynthesisPipeline",
    # Run parallel research first, then merge
    sub_agents=[parallel_research_agent, merger_agent],
    description="Coordinates parallel research and synthesizes the results.",
)

root_agent = sequential_pipeline_agent
```

這段程式碼定義了一個多代理系統，用於研究並綜合永續技術進展的資訊。它設定了三個 LlmAgent 實例，讓它們擔任專門的研究者。`ResearcherAgent_1` 聚焦於再生能源來源，`ResearcherAgent_2` 研究電動車技術，`ResearcherAgent_3` 則調查碳捕捉方法。每個研究代理都設定為使用 `GEMINI_MODEL` 與 `google_search` 工具。它們被指示簡潔摘要研究發現（1 到 2 句），並使用 `output_key` 將這些摘要儲存在工作階段狀態中。

接著建立名為 ParallelWebResearchAgent 的 ParallelAgent，用來同時執行這三個研究代理。這讓研究工作可以平行進行，可能節省時間。當所有子代理（也就是研究者）都完成並填入狀態後，ParallelAgent 就會完成執行。

接下來定義一個 MergerAgent（同樣是 LlmAgent）來綜合研究結果。這個代理會以平行研究者儲存在工作階段狀態中的摘要作為輸入。它的指令強調輸出必須嚴格只根據提供的輸入摘要，禁止加入外部知識。MergerAgent 的設計目標，是將合併後的發現組織成一份報告，針對每個主題提供標題，並附上簡短的整體結論。

最後，建立名為 ResearchAndSynthesisPipeline 的 SequentialAgent，用來協調整個工作流程。作為主要控制器，這個主代理會先執行 ParallelAgent 進行研究。當 ParallelAgent 完成後，SequentialAgent 再執行 MergerAgent 來綜合收集到的資訊。`sequential_pipeline_agent` 被設定為 `root_agent`，代表執行這個多代理系統的進入點。整體流程的設計，是為了有效率地平行從多個來源收集資訊，然後將其合併為單一結構化報告。

## 快速總覽

**是什麼：** 許多代理式工作流程都包含多個必須完成的子任務，才能達成最終目標。純粹循序的執行方式，也就是每個任務都等待前一個任務完成，通常效率不佳且速度緩慢。當任務依賴外部 I/O 操作時，例如呼叫不同 API 或查詢多個資料庫，這種延遲會成為顯著瓶頸。如果沒有並行執行機制，總處理時間就是所有個別任務時長的總和，進而妨礙系統整體效能與回應能力。

**為什麼：** 平行化模式透過讓獨立任務同時執行，提供標準化解法。它的作法是識別工作流程中不依賴彼此即時輸出的元件，例如工具使用或 LLM 呼叫。LangChain 與 Google ADK 等代理式框架提供內建結構，用於定義並管理這些並行操作。例如，主要程序可以叫用數個平行運行的子任務，並等待它們全部完成後再進入下一個步驟。透過讓這些獨立任務同時執行，而不是一個接著一個執行，此模式能大幅降低總執行時間。

**經驗法則：** 當工作流程包含多個可同時運行的獨立操作時，就使用此模式，例如從多個 API 擷取資料、處理不同資料區塊，或產生多個內容片段以便後續綜合。

**視覺摘要：**

![Parallelization Design Pattern](../assets/Parallelization_Design_Pattern.png)

圖 2：平行化設計模式

## 重點整理

以下是重點整理：

* 平行化是一種同時執行獨立任務以提升效率的模式。  
* 當任務需要等待外部資源（例如 API 呼叫）時，它特別有用。  
* 採用並行或平行架構會引入相當程度的複雜度與成本，影響設計、除錯與系統記錄等關鍵開發階段。  
* LangChain 與 Google ADK 等框架提供內建支援，用於定義並管理平行執行。  
* 在 LangChain Expression Language (LCEL) 中，RunnableParallel 是讓多個 runnables 並排運行的關鍵結構。  
* Google ADK 可透過 LLM-Driven Delegation 促成平行執行，其中 Coordinator 代理的 LLM 會識別獨立子任務，並觸發專門子代理同時處理它們。  
* 平行化有助於降低整體延遲，並讓代理式系統在面對複雜任務時更具回應能力。

## 結論

平行化模式是一種透過同時執行獨立子任務來最佳化計算工作流程的方法。這種作法能降低整體延遲，尤其適用於涉及多次模型推論或呼叫外部服務的複雜操作。

不同框架提供不同機制來實作此模式。在 LangChain 中，RunnableParallel 這類結構用於明確定義並同時執行多個處理鏈。相較之下，Google Agent Developer Kit (ADK) 等框架可以透過多代理委派達成平行化，由主要協調者模型將不同子任務指派給可同時運作的專門代理。

透過將平行處理與循序（鏈接）及條件式（路由）控制流程整合，就能建構精密且高效能的計算系統，有效管理各種多樣且複雜的任務。

## 參考資料

以下是一些可進一步閱讀平行化模式與相關概念的資源：

1. LangChain Expression Language (LCEL) 文件（平行性）： [https://python.langchain.com/docs/concepts/lcel/](https://python.langchain.com/docs/concepts/lcel/)
2. Google Agent Developer Kit (ADK) 文件（多代理系統）： [https://google.github.io/adk-docs/agents/multi-agents/](https://google.github.io/adk-docs/agents/multi-agents/)  
3. Python `asyncio` 文件： [https://docs.python.org/3/library/asyncio.html](https://docs.python.org/3/library/asyncio.html)


