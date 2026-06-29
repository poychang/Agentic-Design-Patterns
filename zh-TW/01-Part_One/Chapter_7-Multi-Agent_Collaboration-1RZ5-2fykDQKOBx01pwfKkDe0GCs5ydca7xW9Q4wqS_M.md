# 第 7 章：多代理協作

雖然單體代理（agent）架構對定義明確的問題可能很有效，但面對複雜、跨領域任務時，其能力往往會受到限制。多代理協作模式透過將系統建構為一組彼此合作、各自獨立且專精的代理，來處理這些限制。這種方法以任務分解原則為基礎，也就是將高階目標拆解為離散的子問題。接著，每個子問題都會指派給最適合該任務、具備特定工具、資料存取能力或推理能力的代理。

例如，一個複雜的研究查詢可以被分解並指派給 Research Agent 負責資訊檢索、Data Analysis Agent 負責統計處理，以及 Synthesis Agent 負責產生最終報告。這類系統的效能不只是來自分工，更關鍵地取決於代理間通訊（A2A）的機制。這需要標準化的通訊協定與共享本體，讓代理能交換資料、委派子任務，並協調彼此行動，以確保最終輸出一致且連貫。

這種分散式架構帶來多項優點，包括更高的模組化、可擴充性與強健性，因為單一代理失效不一定會導致整個系統失效。這種協作能產生綜效，使多代理系統的整體表現超越群組中任何單一代理可能達到的能力。

## 多代理協作模式概觀

多代理協作模式涉及設計由多個獨立或半獨立代理共同達成共同目標的系統。每個代理通常都有明確角色、與整體目標一致的特定目標，並且可能能存取不同工具或知識庫。此模式的力量在於這些代理之間的互動與綜效。

協作可以採取多種形式：

* **循序交接：** 一個代理完成任務後，將其輸出傳遞給另一個代理，作為管線中下一步的輸入（類似於規劃模式，但明確涉及不同代理）。  
* **平行處理：** 多個代理同時處理問題的不同部分，之後再合併其結果。  
* **辯論與共識：** 多代理協作中，具有不同觀點與資訊來源的代理會進行討論以評估選項，最後達成共識或形成更充分的決策。  
* **階層式結構：** 管理者代理可能根據工作者代理的工具存取權或外掛能力，動態委派任務並整合其結果。每個代理也可以處理相關的一組工具，而不是由單一代理處理所有工具。  
* **專家團隊：** 在不同領域具備專門知識的代理（例如研究員、寫作者、編輯）共同產出複雜成果。

* **批評者－審查者：** 代理會先建立計畫、草稿或答案等初步輸出。接著，第二組代理會從政策遵循、安全性、合規性、正確性、品質，以及與組織目標的一致性等面向，批判性地評估該輸出。原始建立者或最終代理會根據這些回饋修訂輸出。此模式對程式碼生成、研究寫作、邏輯檢查，以及確保倫理一致性特別有效。此方法的優點包括提高強健性、改善品質，並降低幻覺或錯誤發生的可能性。

多代理系統（見圖 1）基本上包含代理角色與職責的劃分、建立供代理交換資訊的通訊管道，以及制定引導其協作行動的任務流程或互動協定。

![Multi-Agent System](../assets/Multi_Agent_System.png)

圖 1：多代理系統範例

Crew AI 與 Google ADK 等框架，是為了促進這種典範而設計，提供用來指定代理、任務及其互動程序的結構。這種方法特別適合需要多種專門知識、包含多個離散階段，或能受益於並行處理與跨代理資訊相互佐證的挑戰。

## 實務應用與使用案例

多代理協作是一種強大的模式，可應用於許多領域：

* **複雜研究與分析：** 一組代理可以協作進行研究專案。一個代理可能專門搜尋學術資料庫，另一個代理負責摘要發現，第三個代理辨識趨勢，第四個代理則將資訊整合成報告。這反映了人類研究團隊可能採取的運作方式。  
* **軟體開發：** 想像代理協作建構軟體。一個代理可以是需求分析師，另一個是程式碼產生器，第三個是測試人員，第四個是文件撰寫者。他們可以彼此傳遞輸出，以建構並驗證元件。  
* **創意內容生成：** 建立行銷活動可能涉及市場研究代理、文案代理、平面設計代理（使用影像生成工具），以及社群媒體排程代理共同合作。  
* **財務分析：** 多代理系統可以分析金融市場。代理可能分別專精於擷取股票資料、分析新聞情緒、執行技術分析，以及產生投資建議。  
* **客戶支援升級：** 第一線支援代理可以處理初始查詢，並在需要時將複雜問題升級給專家代理（例如技術專家或帳務專家），展現基於問題複雜度的循序交接。  
* **供應鏈最佳化：** 代理可以代表供應鏈中的不同節點（供應商、製造商、經銷商），並協作最佳化庫存水位、物流與排程，以回應需求變化或中斷。  
* **網路分析與修復**：自主營運非常受益於代理式架構，尤其是在故障定位方面。多個代理可以協作分類與修復問題，並建議最佳行動。這些代理也能與傳統機器學習模型和工具整合，在運用既有系統的同時，也提供生成式 AI 的優勢。

定義專精代理並細緻協調其相互關係的能力，讓開發者能建構具備更高模組化、可擴充性，並能處理單一整合式代理難以克服之複雜性的系統。

## 多代理協作：探索相互關係與通訊結構

了解代理互動與通訊的複雜方式，是設計有效多代理系統的基礎。如圖 2 所示，相互關係與通訊模型形成一個光譜，從最簡單的單一代理情境，到複雜且自訂設計的協作框架皆包含在內。每種模型都有獨特優勢與挑戰，並會影響多代理系統的整體效率、強健性與適應性。

### 1. 單一代理

在最基本的層級，「單一代理」會自主運作，不與其他實體直接互動或通訊。雖然此模型易於實作與管理，但其能力本質上受限於個別代理的範圍與資源。它適用於可分解為獨立子問題的任務，其中每個子問題都能由單一、自給自足的代理解決。

### 2. 網路

「網路」模型代表邁向協作的重要一步，其中多個代理以去中心化方式彼此直接互動。通訊通常以對等方式進行，允許共享資訊、資源，甚至任務。此模型能促進韌性，因為一個代理失效不一定會癱瘓整個系統。然而，在大型、非結構化網路中，管理通訊負擔並確保決策一致可能具有挑戰性。

### 3. 監督者

在「監督者」模型中，一個專門代理，也就是「監督者」，會監看並協調一組從屬代理的活動。監督者作為通訊、任務分配與衝突解決的中心樞紐。這種階層式結構提供明確的權責線，並能簡化管理與控制。然而，它也引入單點故障（監督者），而且當監督者被大量從屬代理或複雜任務壓垮時，可能成為瓶頸。

### 4. 監督者作為工具

此模型是「監督者」概念的細緻延伸，其中監督者的角色較少是直接命令與控制，更多是向其他代理提供資源、指引或分析支援。監督者可能提供工具、資料或運算服務，讓其他代理能更有效地執行任務，而不一定要指示它們的每個動作。此方法旨在運用監督者的能力，同時避免施加僵硬的由上而下控制。

### 5. 階層式

「階層式」模型擴展監督者概念，建立多層級的組織結構。這涉及多個層級的監督者，由較高層級監督者監看較低層級監督者，最終在最低層則是一組操作型代理。此結構非常適合可分解為子問題的複雜問題，而且每個子問題都由階層中的特定層級管理。它提供可擴充性與複雜度管理的結構化方法，允許在定義好的邊界內進行分散式決策。

![Agents Communicate and Interact in Various Ways](../assests/Agents_Communicate_and_Interact_in_Various_Ways.png)

圖 2：代理以各種方式通訊與互動。

### 6. 自訂

「自訂」模型代表多代理系統設計中的最高彈性。它允許建立獨特的相互關係與通訊結構，精準貼合特定問題或應用的需求。這可以包含混合式方法，結合前述模型中的元素；也可以是完全新穎的設計，源自環境中的獨特限制與機會。自訂模型通常來自以下需求：針對特定效能指標最佳化、處理高度動態的環境，或將領域專屬知識納入系統架構。設計與實作自訂模型通常需要深入理解多代理系統原則，並仔細考量通訊協定、協調機制與湧現行為。

總結來說，為多代理系統選擇相互關係與通訊模型，是一項關鍵設計決策。每種模型都有不同優缺點，最佳選擇取決於任務複雜度、代理數量、期望的自主程度、強健性需求，以及可接受的通訊負擔等因素。多代理系統未來的進展，很可能會持續探索並精煉這些模型，同時發展協作智慧的新典範。

## 實作程式碼（Crew AI）

這段 Python 程式碼使用 CrewAI 框架定義一個由 AI 驅動的 crew，用來產生一篇關於 AI 趨勢的部落格文章。它一開始會設定環境，從 .env 檔載入 API keys。應用程式核心包含定義兩個代理：一個研究員用來找出並摘要 AI 趨勢，另一個寫作者則根據研究內容建立部落格文章。

程式碼也相應定義了兩個任務：一個負責研究趨勢，另一個負責撰寫部落格文章，其中寫作任務依賴研究任務的輸出。接著，這些代理與任務會被組合成 Crew，並指定循序流程，讓任務依序執行。Crew 會以代理、任務與語言模型（特別是 "gemini-2.0-flash" 模型）初始化。main 函式使用 kickoff() 方法執行這個 crew，協調代理之間的協作以產生所需輸出。最後，程式碼會印出 crew 執行後的最終結果，也就是生成的部落格文章。

```python
import os
from dotenv import load_dotenv
from crewai import Agent, Task, Crew, Process
from langchain_google_genai import ChatGoogleGenerativeAI


def setup_environment():
    """Loads environment variables and checks for the required API key."""
    load_dotenv()
    if not os.getenv("GOOGLE_API_KEY"):
        raise ValueError("GOOGLE_API_KEY not found. Please set it in your .env file.")


def main():
    """
    Initializes and runs the AI crew for content creation using the latest Gemini model.
    """
    setup_environment()

    # Define the language model to use.
    # Updated to a model from the Gemini 2.0 series for better performance and features.
    # For cutting-edge (preview) capabilities, you could use "gemini-2.5-flash".
    llm = ChatGoogleGenerativeAI(model="gemini-2.0-flash")

    # Define Agents with specific roles and goals
    researcher = Agent(
        role='Senior Research Analyst',
        goal='Find and summarize the latest trends in AI.',
        backstory="You are an experienced research analyst with a knack for identifying key trends and synthesizing information.",
        verbose=True,
        allow_delegation=False,
    )

    writer = Agent(
        role='Technical Content Writer',
        goal='Write a clear and engaging blog post based on research findings.',
        backstory="You are a skilled writer who can translate complex technical topics into accessible content.",
        verbose=True,
        allow_delegation=False,
    )

    # Define Tasks for the agents
    research_task = Task(
        description="Research the top 3 emerging trends in Artificial Intelligence in 2024-2025. Focus on practical applications and potential impact.",
        expected_output="A detailed summary of the top 3 AI trends, including key points and sources.",
        agent=researcher,
    )

    writing_task = Task(
        description="Write a 500-word blog post based on the research findings. The post should be engaging and easy for a general audience to understand.",
        expected_output="A complete 500-word blog post about the latest AI trends.",
        agent=writer,
        context=[research_task],
    )

    # Create the Crew
    blog_creation_crew = Crew(
        agents=[researcher, writer],
        tasks=[research_task, writing_task],
        process=Process.sequential,
        llm=llm,
        verbose=2,  # Set verbosity for detailed crew execution logs
    )

    # Execute the Crew
    print("## Running the blog creation crew with Gemini 2.0 Flash... ##")
    try:
        result = blog_creation_crew.kickoff()
        print("\n------------------\n")
        print("## Crew Final Output ##")
        print(result)
    except Exception as e:
        print(f"\nAn unexpected error occurred: {e}")


if __name__ == "__main__":
    main()
```

接下來，我們將深入探討 Google ADK 框架中的更多範例，特別聚焦於階層式、平行與循序協調典範，以及將代理實作為操作工具的方式。

## 實作程式碼（Google ADK）

以下程式碼範例示範如何在 Google ADK 中透過建立父子關係，建構階層式代理結構。程式碼定義了兩種代理：LlmAgent，以及衍生自 BaseAgent 的自訂 TaskExecutor 代理。TaskExecutor 設計用於特定的非 LLM 任務；在這個範例中，它只會產生一個 "Task finished successfully" 事件。名為 greeter 的 LlmAgent 會以指定模型與指示初始化，扮演友善的問候者。自訂 TaskExecutor 會實例化為 `task_doer`。接著建立名為 coordinator 的父 LlmAgent，同樣帶有模型與指示。coordinator 的指示會引導它將問候委派給 greeter，並將任務執行委派給 `task_doer`。greeter 與 `task_doer` 會被加入 coordinator 作為子代理，藉此建立父子關係。程式碼接著會斷言此關係已正確建立。最後，它會印出訊息，表示代理階層已成功建立。

```python
from typing import AsyncGenerator

from google.adk.agents import LlmAgent, BaseAgent
from google.adk.agents.invocation_context import InvocationContext
from google.adk.events import Event


# Correctly implement a custom agent by extending BaseAgent
class TaskExecutor(BaseAgent):
    """A specialized agent with custom, non-LLM behavior."""
    name: str = "TaskExecutor"
    description: str = "Executes a predefined task."

    async def _run_async_impl(self, context: InvocationContext) -> AsyncGenerator[Event, None]:
        """Custom implementation logic for the task."""
        # This is where your custom logic would go.
        # For this example, we'll just yield a simple event.
        yield Event(author=self.name, content="Task finished successfully.")


# Define individual agents with proper initialization
# LlmAgent requires a model to be specified.
greeter = LlmAgent(
    name="Greeter",
    model="gemini-2.0-flash-exp",
    instruction="You are a friendly greeter.",
)

# Instantiate our concrete custom agent
task_doer = TaskExecutor()

# Create a parent agent and assign its sub-agents
# The parent agent's description and instructions should guide its delegation logic.
coordinator = LlmAgent(
    name="Coordinator",
    model="gemini-2.0-flash-exp",
    description="A coordinator that can greet users and execute tasks.",
    instruction="When asked to greet, delegate to the Greeter. When asked to perform a task, delegate to the TaskExecutor.",
    sub_agents=[
        greeter,
        task_doer,
    ],
)

# The ADK framework automatically establishes the parent-child relationships.
# These assertions will pass if checked after initialization.
assert greeter.parent_agent == coordinator
assert task_doer.parent_agent == coordinator

print("Agent hierarchy created successfully.")
```

這段程式碼摘錄說明如何在 Google ADK 框架中使用 LoopAgent 建立迭代式工作流程。程式碼定義了兩個代理：ConditionChecker 與 ProcessingStep。ConditionChecker 是自訂代理，會檢查工作階段狀態中的 "status" 值。如果 "status" 是 "completed"，ConditionChecker 會升級事件以停止迴圈。否則，它會產生事件以繼續迴圈。`ProcessingStep` 是使用 "gemini-2.0-flash-exp" 模型的 LlmAgent。其指示是執行任務，並在它是最後一步時將工作階段的 `status` 設為 "completed"。名為 StatusPoller 的 LoopAgent 會被建立。StatusPoller 設定為 `max_iterations=10`。StatusPoller 同時包含 ProcessingStep 與 ConditionChecker 的實例作為子代理。LoopAgent 會最多執行子代理 10 次，且每次依序執行；如果 ConditionChecker 發現狀態為 "completed"，便會停止。

```python
import asyncio
from typing import AsyncGenerator

from google.adk.agents import LoopAgent, LlmAgent, BaseAgent
from google.adk.events import Event, EventActions
from google.adk.agents.invocation_context import InvocationContext


# Best Practice: Define custom agents as complete, self-describing classes.
class ConditionChecker(BaseAgent):
    """A custom agent that checks for a 'completed' status in the session state."""
    name: str = "ConditionChecker"
    description: str = "Checks if a process is complete and signals the loop to stop."

    async def _run_async_impl(
        self, context: InvocationContext
    ) -> AsyncGenerator[Event, None]:
        """Checks state and yields an event to either continue or stop the loop."""
        status = context.session.state.get("status", "pending")
        is_done = status == "completed"

        if is_done:
            # Escalate to terminate the loop when the condition is met.
            yield Event(author=self.name, actions=EventActions(escalate=True))
        else:
            # Yield a simple event to continue the loop.
            yield Event(author=self.name, content="Condition not met, continuing loop.")


# Correction: The LlmAgent must have a model and clear instructions.
process_step = LlmAgent(
    name="ProcessingStep",
    model="gemini-2.0-flash-exp",
    instruction=(
        "You are a step in a longer process. Perform your task. "
        "If you are the final step, update session state by setting 'status' to 'completed'."
    ),
)


# The LoopAgent orchestrates the workflow.
poller = LoopAgent(
    name="StatusPoller",
    max_iterations=10,
    sub_agents=[
        process_step,
        ConditionChecker(),  # Instantiating the well-defined custom agent.
    ],
)

# This poller will now execute 'process_step'
# and then 'ConditionChecker' repeatedly until the status is 'completed'
# or 10 iterations have passed.
```

這段程式碼摘錄闡明 Google ADK 中的 SequentialAgent 模式，該模式設計用於建構線性工作流程。這段程式碼使用 google.adk.agents 函式庫定義一條循序代理管線。此管線包含兩個代理：step1 與 step2。step1 名為 `Step1_Fetch`，其輸出會儲存在工作階段狀態的 `data` 鍵底下。step2 名為 `Step2_Process`，並被指示分析儲存在 `session.state["data"]` 中的資訊並提供摘要。名為 "MyPipeline" 的 SequentialAgent 會協調這些子代理的執行。當管線以初始輸入執行時，step1 會先執行。step1 的回應會被儲存在工作階段狀態的 "data" 鍵底下。接著，step2 會執行，並依照其指示使用 step1 放入狀態中的資訊。這種結構可用於建立一個代理輸出成為下一個代理輸入的工作流程。這是建立多步驟 AI 或資料處理管線時常見的模式。

```python
from google.adk.agents import SequentialAgent, Agent


# This agent's output will be saved to session.state["data"]
step1 = Agent(
    name="Step1_Fetch",
    output_key="data",
)

# This agent will use the data from the previous step.
# We instruct it on how to find and use this data.
step2 = Agent(
    name="Step2_Process",
    instruction="Analyze the information found in state['data'] and provide a summary.",
)

pipeline = SequentialAgent(
    name="MyPipeline",
    sub_agents=[step1, step2],
)

# When the pipeline is run with an initial input, Step1 will execute,
# its response will be stored in session.state["data"], and then
# Step2 will execute, using the information from the state as instructed.
```

以下程式碼範例說明 Google ADK 中的 ParallelAgent 模式，該模式有助於同時執行多個代理任務。`data_gatherer` 設計為同時執行兩個子代理：`weather_fetcher` 與 `news_fetcher`。`weather_fetcher` 代理被指示取得指定地點的天氣，並將結果儲存在 `session.state["weather_data"]`。同樣地，`news_fetcher` 代理被指示擷取指定主題的頭條新聞，並將結果儲存在 `session.state["news_data"]`。每個子代理都設定為使用 "gemini-2.0-flash-exp" 模型。ParallelAgent 會協調這些子代理的執行，讓它們能平行工作。來自 `weather_fetcher` 與 `news_fetcher` 的結果會被蒐集並儲存在工作階段狀態中。最後，此範例展示在代理執行完成後，如何從 `final_state` 存取已蒐集的天氣與新聞資料。

```python
from google.adk.agents import Agent, ParallelAgent


# It's better to define the fetching logic as tools for the agents.
# For simplicity in this example, we'll embed the logic in the agent's instruction.
# In a real-world scenario, you would use tools.

# Define the individual agents that will run in parallel
weather_fetcher = Agent(
    name="weather_fetcher",
    model="gemini-2.0-flash-exp",
    instruction="Fetch the weather for the given location and return only the weather report.",
    output_key="weather_data",  # The result will be stored in session.state["weather_data"]
)

news_fetcher = Agent(
    name="news_fetcher",
    model="gemini-2.0-flash-exp",
    instruction="Fetch the top news story for the given topic and return only that story.",
    output_key="news_data",  # The result will be stored in session.state["news_data"]
)

# Create the ParallelAgent to orchestrate the sub-agents
data_gatherer = ParallelAgent(
    name="data_gatherer",
    sub_agents=[
        weather_fetcher,
        news_fetcher,
    ],
)
```

提供的程式碼片段示範 Google ADK 中的「Agent as a Tool」典範，讓一個代理能以類似函式呼叫的方式使用另一個代理的能力。具體而言，程式碼使用 Google 的 LlmAgent 與 AgentTool 類別定義一個影像生成系統。它包含兩個代理：父代理 `artist_agent` 與子代理 `image_generator_agent`。`generate_image` 函式是一個簡單工具，會模擬影像建立並回傳模擬影像資料。`image_generator_agent` 負責根據它收到的文字提示使用此工具。`artist_agent` 的角色是先發想一個有創意的影像提示。接著，它會透過 AgentTool 包裝器呼叫 `image_generator_agent`。AgentTool 扮演橋接角色，允許一個代理將另一個代理作為工具使用。當 `artist_agent` 呼叫 `image_tool` 時，AgentTool 會以 artist 發想的提示叫用 `image_generator_agent`。接著，`image_generator_agent` 會使用該提示呼叫 `generate_image` 函式。最後，生成的影像（或模擬資料）會沿著代理回傳回來。此架構示範了分層代理系統，其中較高層級代理會協調較低層級的專精代理來執行任務。

```python
from google.adk.agents import LlmAgent
from google.adk.tools import agent_tool
from google.genai import types


# 1. A simple function tool for the core capability.
# This follows the best practice of separating actions from reasoning.
def generate_image(prompt: str) -> dict:
    """
    Generates an image based on a textual prompt.

    Args:
        prompt: A detailed description of the image to generate.

    Returns:
        A dictionary with the status and the generated image bytes.
    """
    print(f"TOOL: Generating image for prompt: '{prompt}'")
    # In a real implementation, this would call an image generation API.
    # For this example, we return mock image data.
    mock_image_bytes = b"mock_image_data_for_a_cat_wearing_a_hat"
    return {
        "status": "success",
        # The tool returns the raw bytes, the agent will handle the Part creation.
        "image_bytes": mock_image_bytes,
        "mime_type": "image/png",
    }


# 2. Refactor the ImageGeneratorAgent into an LlmAgent.
# It now correctly uses the input passed to it.
image_generator_agent = LlmAgent(
    name="ImageGen",
    model="gemini-2.0-flash",
    description="Generates an image based on a detailed text prompt.",
    instruction=(
        "You are an image generation specialist. Your task is to take the user's request "
        "and use the `generate_image` tool to create the image. "
        "The user's entire request should be used as the 'prompt' argument for the tool. "
        "After the tool returns the image bytes, you MUST output the image."
    ),
    tools=[generate_image],
)


# 3. Wrap the corrected agent in an AgentTool.
# The description here is what the parent agent sees.
image_tool = agent_tool.AgentTool(
    agent=image_generator_agent,
    description="Use this tool to generate an image. The input should be a descriptive prompt of the desired image.",
)


# 4. The parent agent remains unchanged. Its logic was correct.
artist_agent = LlmAgent(
    name="Artist",
    model="gemini-2.0-flash",
    instruction=(
        "You are a creative artist. First, invent a creative and descriptive prompt for an image. "
        "Then, use the `ImageGen` tool to generate the image using your prompt."
    ),
    tools=[image_tool],
)
```

## 重點一覽

**What：** 複雜問題通常會超出單一、單體且基於大型語言模型（large language model, LLM）的代理能力。單一代理可能缺乏多樣且專精的技能，或缺少處理多面向任務所有部分所需的特定工具存取能力。這項限制會形成瓶頸，降低系統的整體效果與可擴充性。因此，處理精密的跨領域目標會變得沒有效率，並可能導致不完整或次佳的結果。

**Why：** 多代理協作模式透過建立由多個協作代理組成的系統，提供標準化解決方案。複雜問題會被拆解為更小、更容易管理的子問題。接著，每個子問題都會指派給具備精確工具與能力的專精代理來解決。這些代理會透過定義好的通訊協定與互動模型共同工作，例如循序交接、平行工作流，或階層式委派。這種代理式、分散式方法會產生綜效，讓團隊達成任何單一代理都不可能完成的成果。

**Rule of thumb：** 當任務對單一代理而言過於複雜，且可分解為需要專門技能或工具的不同子任務時，請使用此模式。此模式非常適合能受益於多元專業知識、平行處理，或具有多個階段之結構化工作流程的問題，例如複雜研究與分析、軟體開發或創意內容生成。

**Visual summary：**

![Multi-Agent Design Pattern](../assets/Multi_Agent_Design_Pattern.png)

圖 3：多代理設計模式

## 關鍵重點

* 多代理協作涉及多個代理共同努力達成共同目標。  
* 此模式運用專門角色、分散式任務，以及代理間通訊。  
* 協作可以採取循序交接、平行處理、辯論或階層式結構等形式。  
* 此模式非常適合需要多元專業知識或多個明確階段的複雜問題。

## 結論

本章探討了多代理協作模式，展示在系統中協調多個專精代理所帶來的好處。我們檢視了各種協作模型，強調此模式在處理不同領域中複雜、多面向問題時的關鍵作用。理解代理協作，自然會引出對它們如何與外部環境互動的探問。

## 參考資料

1. Multi-Agent Collaboration Mechanisms: A Survey of LLMs, [https://arxiv.org/abs/2501.06322](https://arxiv.org/abs/2501.06322)
2. Multi-Agent System — The Power of Collaboration, [https://aravindakumar.medium.com/introducing-multi-agent-frameworks-the-power-of-collaboration-e9db31bba1b6](https://aravindakumar.medium.com/introducing-multi-agent-frameworks-the-power-of-collaboration-e9db31bba1b6)
