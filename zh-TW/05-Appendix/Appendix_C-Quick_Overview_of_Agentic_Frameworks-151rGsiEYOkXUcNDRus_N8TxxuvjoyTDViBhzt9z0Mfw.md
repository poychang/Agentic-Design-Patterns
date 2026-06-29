# 附錄 C - 代理式框架快速概覽

## LangChain

LangChain 是用於開發由 LLM 驅動之應用程式的框架。它的核心強項在於 LangChain Expression Language (LCEL)，讓你可以將元件「pipe」成一條鏈。這會建立清楚的線性序列，其中一個步驟的輸出會成為下一個步驟的輸入。它是為 Directed Acyclic Graphs (DAGs) 形式的工作流程而建，代表流程會單向前進且沒有迴圈。

適用於：

* 簡單 RAG：擷取文件、建立提示，並從 LLM 取得答案。  
* 摘要：取得使用者文字，將其送入摘要提示，並回傳輸出。  
* 擷取：從一段文字中擷取結構化資料（例如 JSON）。

Python

```python
# A simple LCEL chain conceptually # (This is not runnable code, just illustrates the flow) 
chain = prompt | model | output_parse
```

## LangGraph

LangGraph 是建立在 LangChain 之上的程式庫，用於處理更進階的代理式系統。它讓你能將工作流程定義為圖，其中包含節點（函式或 LCEL 鏈）與邊（條件邏輯）。它的主要優勢是能建立循環，允許應用程式反覆執行、重試，或以彈性的順序呼叫工具，直到任務完成。它會明確管理應用程式狀態，而狀態會在節點之間傳遞，並在整個過程中更新。

適用於：

* 多代理系統：監督代理將任務路由給專門的工作代理，並可能反覆執行直到達成目標。  
* Plan-and-Execute 代理：代理建立計畫、執行一個步驟，然後根據結果回頭更新計畫。  
* Human-in-the-Loop：圖可以等待人類輸入，再決定接下來要前往哪個節點。

| 功能 | LangChain | LangGraph |
| :---- | :---- | :---- |
| 核心抽象 | 鏈（使用 LCEL） | 節點圖 |
| 工作流程類型 | 線性（Directed Acyclic Graph） | 循環式（具有迴圈的圖） |
| 狀態管理 | 通常每次執行都無狀態 | 明確且持久的狀態物件 |
| 主要用途 | 簡單、可預測的序列 | 複雜、動態、具狀態的代理 |

### 應該使用哪一個？

* 當你的應用程式具有清楚、可預測且線性的步驟流程時，請選擇 LangChain。如果你可以定義從 A 到 B 到 C 的流程，而不需要回頭循環，搭配 LCEL 的 LangChain 就是理想工具。  
* 當你需要應用程式進行推理、規劃或在迴圈中運作時，請選擇 LangGraph。如果你的代理需要使用工具、反思結果，並可能以不同方法再次嘗試，你就需要 LangGraph 的循環式與具狀態特性。

```python
# Graph state
class State(TypedDict):
    topic: str
    joke: str
    story: str
    poem: str
    combined_output: str


# Nodes
def call_llm_1(state: State):
    """First LLM call to generate initial joke"""
    msg = llm.invoke(f"Write a joke about {state['topic']}")
    return {"joke": msg.content}


def call_llm_2(state: State):
    """Second LLM call to generate story"""
    msg = llm.invoke(f"Write a story about {state['topic']}")
    return {"story": msg.content}


def call_llm_3(state: State):
    """Third LLM call to generate poem"""
    msg = llm.invoke(f"Write a poem about {state['topic']}")
    return {"poem": msg.content}


def aggregator(state: State):
    """Combine the joke and story into a single output"""
    combined = f"Here's a story, joke, and poem about {state['topic']}!\n\n"
    combined += f"STORY:\n{state['story']}\n\n"
    combined += f"JOKE:\n{state['joke']}\n\n"
    combined += f"POEM:\n{state['poem']}"
    return {"combined_output": combined}


# Build workflow
parallel_builder = StateGraph(State)

# Add nodes
parallel_builder.add_node("call_llm_1", call_llm_1)
parallel_builder.add_node("call_llm_2", call_llm_2)
parallel_builder.add_node("call_llm_3", call_llm_3)
parallel_builder.add_node("aggregator", aggregator)

# Add edges to connect nodes
parallel_builder.add_edge(START, "call_llm_1")
parallel_builder.add_edge(START, "call_llm_2")
parallel_builder.add_edge(START, "call_llm_3")
parallel_builder.add_edge("call_llm_1", "aggregator")
parallel_builder.add_edge("call_llm_2", "aggregator")
parallel_builder.add_edge("call_llm_3", "aggregator")
parallel_builder.add_edge("aggregator", END)

parallel_workflow = parallel_builder.compile()

# Show workflow
display(Image(parallel_workflow.get_graph().draw_mermaid_png()))

# Invoke
state = parallel_workflow.invoke({"topic": "cats"})
print(state["combined_output"])
```

這段程式碼定義並執行一個平行運作的 LangGraph 工作流程。它的主要目的，是同時針對指定主題產生一則笑話、一篇故事和一首詩，然後將它們合併為單一、格式化的文字輸出。

## Google's ADK

Google's Agent Development Kit（ADK）提供高階、結構化的框架，用於建置與部署由多個互動式 AI 代理組成的應用程式。它與 LangChain 和 LangGraph 的差異在於，它提供更具主張且偏向生產環境的系統，用來協調代理協作，而不是提供代理內部邏輯的基本建構區塊。

LangChain 位於最基礎的層級，提供元件與標準化介面，用來建立操作序列，例如呼叫模型並剖析其輸出。LangGraph 則在此基礎上延伸，引入更具彈性且更強大的控制流程；它將代理的工作流程視為具狀態的圖。使用 LangGraph 時，開發者會明確定義節點，也就是函式或工具，以及邊，也就是決定執行路徑的邏輯。這種圖結構允許複雜、循環式的推理，系統可以根據在節點間傳遞且被明確管理的狀態物件，進行迴圈、重試任務並做出決策。它讓開發者能細緻控制單一代理的思考流程，或從第一原理建構多代理系統。

Google's ADK 抽象化了許多這類低階圖建構工作。它不是要求開發者定義每一個節點與邊，而是提供多代理互動的預建架構模式。例如，ADK 內建 SequentialAgent 或 ParallelAgent 等代理類型，能自動管理不同代理之間的控制流程。它的架構圍繞著代理「團隊」的概念，通常由一個主要代理將任務委派給專門的子代理。狀態與 session 管理則由框架以較隱含的方式處理；相較於 LangGraph 明確傳遞狀態，這種方法更具整體性，但顆粒度較低。因此，若 LangGraph 提供的是設計單一機器人或團隊複雜配線的詳細工具，Google's ADK 提供的則是一條工廠組裝線，專為建置與管理一支已經知道如何協同工作的機器人艦隊而設計。

```python
from google.adk.agents import LlmAgent
from google.adk.tools import google_Search

dice_agent = LlmAgent(
    model="gemini-2.0-flash-exp",
    name="question_answer_agent",
    description="A helpful assistant agent that can answer questions.",
    instruction="""Respond to the query using google search""",
    tools=[google_search],
)
```

這段程式碼建立了一個具搜尋增強能力的代理。當此代理收到問題時，它不只依賴既有知識。相反地，它會依照指令使用 Google Search 工具，從網路尋找相關的即時資訊，然後運用這些資訊建構答案。

## Crew.AI

CrewAI 提供一個用於建置多代理系統的協調框架，重點放在協作角色與結構化流程。它的抽象層級高於基礎工具包，提供一個映照人類團隊的概念模型。開發者不是把細粒度邏輯流程定義為圖，而是定義行動者與其任務分配，CrewAI 則管理它們之間的互動。

此框架的核心元件是 Agents、Tasks 與 Crew。Agent 的定義不只是其功能，也包含 persona，例如特定角色、目標與背景故事，這些會引導其行為與溝通風格。Task 是一個離散工作單元，具有清楚描述與預期輸出，並指派給特定 Agent。Crew 則是包含 Agents 與 Tasks 清單的整合單位，並執行預先定義的 Process。此流程決定工作流程，通常可以是序列式，也就是一項任務的輸出成為下一項任務的輸入；或是階層式，也就是由類似管理者的代理委派任務，並在其他代理之間協調工作流程。

與其他框架相比，CrewAI 具有明確定位。它不同於 LangGraph 的低階、明確狀態管理與控制流程；在 LangGraph 中，開發者需要將每個節點與條件邊串接起來。開發者不是建構狀態機，而是設計團隊章程。雖然 Googlés ADK 為整個代理生命週期提供完整、偏向生產環境的平台，CrewAI 則特別專注於代理協作邏輯，以及模擬專家團隊。

```python
@crew
def crew(self) -> Crew:
   """Creates the research crew"""
   return Crew(
     agents=self.agents,
     tasks=self.tasks,
     process=Process.sequential,
     verbose=True,
   )
```

這段程式碼為一組 AI 代理設定序列式工作流程，使它們按照特定順序處理任務清單，並啟用詳細記錄以監控進度。

## 其他代理開發框架

**Microsoft AutoGen**：AutoGen 是一個以協調多個代理為核心的框架，這些代理會透過對話來解決任務。其架構讓具備不同能力的代理能彼此互動，從而進行複雜問題分解與協作式解決。AutoGen 的主要優勢是彈性、由對話驅動的方法，可支援動態且複雜的多代理互動。不過，這種對話式典範可能導致較不可預測的執行路徑，也可能需要精密的提示工程，才能確保任務有效收斂。

**LlamaIndex**：LlamaIndex 本質上是一個資料框架，設計目的是將大型語言模型連接到外部與私有資料來源。它擅長建立精密的資料擷取與檢索管線，而這些管線對於建置能執行 RAG 的知識型代理至關重要。雖然它的資料索引與查詢能力在建立具上下文感知能力的代理方面極為強大，但與代理優先的框架相比，其原生工具在複雜代理式控制流程與多代理協調方面發展較少。當核心技術挑戰在於資料檢索與合成時，LlamaIndex 是最佳選擇。

**Haystac**k：Haystack 是一個開放原始碼框架，專為建置由語言模型驅動、可擴展且可投入生產環境的搜尋系統而設計。其架構由模組化、可互通的節點組成，這些節點形成用於文件檢索、問答與摘要的管線。Haystack 的主要強項是聚焦於大規模資訊檢索任務的效能與可擴展性，使其適合企業級應用程式。一個可能的取捨是，它針對搜尋管線最佳化的設計，在實作高度動態且具創意的代理式行為時可能較為僵硬。

**MetaGPT**：MetaGPT 透過根據一組預先定義的 Standard Operating Procedures (SOPs) 指派角色與任務，實作多代理系統。此框架結構化代理協作，以模擬軟體開發公司，讓代理承擔產品經理或工程師等角色來完成複雜任務。這種 SOP 驅動的方法會產生高度結構化且一致的輸出，對程式碼產生等專門領域而言是顯著優勢。此框架的主要限制在於其高度專門化，使其在核心設計以外的一般用途代理式任務中較不具適應性。

**SuperAGI**：SuperAGI 是一個開放原始碼框架，旨在為自主代理提供完整生命週期管理系統。它包含代理佈建、監控與圖形介面等功能，目標是提升代理執行的可靠性。其關鍵優點是聚焦於生產就緒性，具備內建機制來處理迴圈等常見失敗模式，並提供對代理效能的可觀測性。一個可能缺點是，其完整平台式方法可能比更輕量、以程式庫為基礎的框架帶來更多複雜度與額外負擔。

**Semantic Kernel**：Semantic Kernel 由 Microsoft 開發，是一套 SDK，透過「plugins」與「planners」系統將大型語言模型與傳統程式碼整合。它允許 LLM 呼叫 native functions 並協調工作流程，實際上是把模型視為較大型軟體應用程式中的推理引擎。它的主要強項是能與現有企業程式碼庫無縫整合，尤其是在 .NET 與 Python 環境中。相較於較直接的代理框架，其 plugin 與 planner 架構的概念負擔可能帶來更陡峭的學習曲線。

**Strands Agents:** AWS 的輕量且彈性 SDK，使用模型驅動方法來建置與執行 AI 代理。它設計上簡單且可擴展，支援從基本對話助理到複雜多代理自主系統的各種情境。此框架不綁定特定模型，廣泛支援各種 LLM 提供者，並原生整合 MCP，方便存取外部工具。它的核心優勢是簡潔與彈性，並提供可自訂且容易上手的代理迴圈。一個可能的取捨是，它的輕量設計代表開發者可能需要自行建構更多周邊營運基礎設施，例如進階監控或生命週期管理系統，而更完整的框架可能已開箱提供這些能力。

## 結論

代理式框架的版圖提供多樣化的工具光譜，從用於定義代理邏輯的低階程式庫，到用於協調多代理協作的高階平台皆有。在基礎層級，LangChain 支援簡單、線性的工作流程，而 LangGraph 則引入具狀態、循環式的圖，以支援更複雜的推理。CrewAI 與 Google's ADK 等較高階框架，則把重點轉向協調具有預定義角色的代理團隊；其他像 LlamaIndex 則專精於資料密集型應用。這種多樣性讓開發者面臨一項核心取捨：是選擇圖式系統的細粒度控制，還是選擇更具主張平台的精簡開發。因此，選擇合適框架取決於應用程式需要的是簡單序列、動態推理迴圈，還是受管理的專家團隊。最終，這個持續演進的生態系讓開發者能依據專案所需的精確抽象層級，建構越來越精密的 AI 系統。

參考資料

1. LangChain, [https://www.langchain.com/](https://www.langchain.com/)
2. LangGraph, [https://www.langchain.com/langgraph](https://www.langchain.com/langgraph)
3. Google's ADK, [https://google.github.io/adk-docs/](https://google.github.io/adk-docs/)
4. Crew.AI, [https://docs.crewai.com/en/introduction](https://docs.crewai.com/en/introduction)
