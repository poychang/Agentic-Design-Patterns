# 第 5 章：工具使用（Tool Use / Function Calling）

## 工具使用模式概述

到目前為止，我們已經討論了幾種代理式模式，主要涉及協調語言模型之間的互動，以及管理代理（agent）內部工作流程中的資訊流（鏈結、路由、平行化、反思）。然而，若要讓代理真正有用，並能與真實世界或外部系統互動，它們就需要使用工具的能力。

工具使用模式通常透過稱為 Function Calling 的機制實作，使代理能夠與外部 API、資料庫、服務互動，甚至執行程式碼。它讓代理核心的大型語言模型（large language model, LLM）能根據使用者請求或任務目前狀態，決定何時以及如何使用特定外部函式。

這個流程通常包含：

1. **工具定義：** 將外部函式或能力定義並描述給 LLM。這份描述包含函式用途、名稱，以及它接受的參數，連同各參數的型別與說明。  
2. **LLM 決策：** LLM 接收使用者請求與可用的工具定義。根據它對請求與工具的理解，LLM 判斷是否需要呼叫一個或多個工具才能完成請求。  
3. **函式呼叫生成：** 如果 LLM 決定使用工具，它會產生結構化輸出（通常是 JSON 物件），指定要呼叫的工具名稱，以及要傳入的引數（參數）；這些資訊會從使用者請求中擷取。  
4. **工具執行：** 代理式框架或協調層會攔截這個結構化輸出，辨識被要求的工具，並用提供的引數執行實際的外部函式。  
5. **觀察／結果：** 工具執行後的輸出或結果會回傳給代理。  
6. **LLM 處理（選用但常見）：** LLM 接收工具輸出作為上下文，並用它來組成給使用者的最終回應，或決定工作流程中的下一步（可能包含呼叫另一個工具、反思，或提供最終答案）。

這個模式很基礎，因為它突破了 LLM 訓練資料的限制，讓 LLM 能存取最新資訊、執行其內部無法完成的計算、與使用者特定資料互動，或觸發真實世界中的動作。Function Calling 是一種技術機制，用來銜接 LLM 的推理能力與大量可用的外部功能。

雖然「function calling」很貼切地描述了呼叫特定、預先定義的程式碼函式，但從更廣泛的「tool calling」概念來理解也很有幫助。這個較寬廣的術語承認代理的能力可以遠超出單純的函式執行。「工具」可以是傳統函式，也可以是複雜的 API 端點、對資料庫的請求，甚至是交給另一個專門代理的指令。這個觀點讓我們能想像更精密的系統；例如主要代理可以把複雜的資料分析任務委派給專門的「analyst agent」，或透過外部知識庫的 API 查詢資料。用「tool calling」來思考，更能完整呈現代理作為協調者，在多元數位資源與其他智慧實體生態系中運作的潛力。

LangChain、LangGraph 與 Google Agent Developer Kit（ADK）等框架，提供穩健支援來定義工具並將其整合到代理工作流程中，通常也會利用 Gemini 或 OpenAI 系列等現代 LLM 的原生 Function Calling 能力。在這些框架的「canvas」上，你會定義工具，然後設定代理（通常是 LLM Agents），使其知道這些工具並能使用它們。

工具使用是建構強大、互動式且能感知外部環境之代理的基石模式。

## 實務應用與使用案例

只要代理需要超越文字生成，進一步執行動作或擷取特定的動態資訊，工具使用模式幾乎都適用：

### 1. 從外部來源擷取資訊

存取即時資料，或取得不存在於 LLM 訓練資料中的資訊。

* **使用案例：** 天氣代理。  
  * **工具：** 接收地點並回傳目前天氣狀況的天氣 API。  
  * **代理流程：** 使用者詢問「What's the weather in London?」，LLM 判斷需要天氣工具，使用「London」呼叫該工具，工具回傳資料，LLM 再將資料格式化為容易理解的回應。

### 2. 與資料庫和 API 互動

對結構化資料執行查詢、更新或其他操作。

* **使用案例：** 電子商務代理。  
  * **工具：** 用於檢查商品庫存、取得訂單狀態或處理付款的 API 呼叫。  
  * **代理流程：** 使用者詢問「Is product X in stock?」，LLM 呼叫庫存 API，工具回傳庫存數量，LLM 告知使用者庫存狀態。

### 3. 執行計算與資料分析

使用外部計算器、資料分析函式庫或統計工具。

* **使用案例：** 金融代理。  
  * **工具：** 計算器函式、股市資料 API、試算表工具。  
  * **代理流程：** 使用者詢問「What's the current price of AAPL and calculate the potential profit if I bought 100 shares at $150?」，LLM 呼叫股票 API 取得目前價格，接著呼叫計算器工具取得結果，最後格式化回應。

### 4. 傳送通訊

傳送電子郵件、訊息，或呼叫外部通訊服務的 API。

* **使用案例：** 個人助理代理。  
  * **工具：** 電子郵件傳送 API。  
  * **代理流程：** 使用者說：「Send an email to John about the meeting tomorrow.」，LLM 會用從請求中擷取出的收件者、主旨與內文呼叫電子郵件工具。

### 5. 執行程式碼

在安全環境中執行程式碼片段，以完成特定任務。

* **使用案例：** 程式設計助理代理。  
  * **工具：** 程式碼直譯器。  
  * **代理流程：** 使用者提供 Python 片段並詢問「What does this code do?」，LLM 使用直譯器工具執行程式碼並分析其輸出。

### 6. 控制其他系統或裝置

與智慧家庭裝置、IoT 平台或其他連線系統互動。

* **使用案例：** 智慧家庭代理。  
  * **工具：** 用於控制智慧燈具的 API。  
  * **代理流程：** 使用者說：「Turn off the living room lights.」LLM 會帶著命令與目標裝置呼叫智慧家庭工具。

工具使用讓語言模型從文字生成器，轉變為能在數位或實體世界中感知、推理並採取行動的代理（見圖 1）。

![代理使用工具的一些範例](Some_Examples_of_an_Agent_Using_Tool.png)

圖 1：代理使用工具的一些範例

## 實作程式碼範例（LangChain）

在 LangChain 框架中實作工具使用，是一個兩階段流程。首先，會定義一個或多個工具，通常是將既有 Python 函式或其他可執行元件封裝起來。接著，這些工具會繫結到語言模型，讓模型在判斷需要外部函式呼叫才能回應使用者查詢時，能產生結構化的工具使用請求。

以下實作會示範這項原則：先定義一個簡單函式來模擬資訊擷取工具，接著建構並設定一個代理，使其能根據使用者輸入運用這個工具。執行此範例需要安裝核心 LangChain 函式庫，以及特定模型供應商的套件。此外，也必須完成所選語言模型服務的驗證，通常是透過在本機環境中設定 API key。

```python
import os
import getpass
import asyncio
import nest_asyncio
from typing import List
from dotenv import load_dotenv
import logging

from langchain_google_genai import ChatGoogleGenerativeAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.tools import tool as langchain_tool
from langchain.agents import create_tool_calling_agent, AgentExecutor


# UNCOMMENT
# Prompt the user securely and set API keys as environment variables
os.environ["GOOGLE_API_KEY"] = getpass.getpass("Enter your Google API key: ")
os.environ["OPENAI_API_KEY"] = getpass.getpass("Enter your OpenAI API key: ")

try:
    # A model with function/tool calling capabilities is required.
    llm = ChatGoogleGenerativeAI(model="gemini-2.0-flash", temperature=0)
    print(f"✅ Language model initialized: {llm.model}")
except Exception as e:
    print(f"🛑 Error initializing language model: {e}")
    llm = None


# --- Define a Tool ---
@langchain_tool
def search_information(query: str) -> str:
    """
    Provides factual information on a given topic. Use this tool to find answers to phrases
    like 'capital of France' or 'weather in London?'.
    """
    print(f"\n--- 🛠️ Tool Called: search_information with query: '{query}' ---")

    # Simulate a search tool with a dictionary of predefined results.
    simulated_results = {
        "weather in london": "The weather in London is currently cloudy with a temperature of 15°C.",
        "capital of france": "The capital of France is Paris.",
        "population of earth": "The estimated population of Earth is around 8 billion people.",
        "tallest mountain": "Mount Everest is the tallest mountain above sea level.",
        "default": f"Simulated search result for '{query}': No specific information found, but the topic seems interesting.",
    }
    result = simulated_results.get(query.lower(), simulated_results["default"])
    print(f"--- TOOL RESULT: {result} ---")
    return result


tools = [search_information]


# --- Create a Tool-Calling Agent ---
if llm:
    # This prompt template requires an `agent_scratchpad` placeholder for the agent's internal steps.
    agent_prompt = ChatPromptTemplate.from_messages([
        ("system", "You are a helpful assistant."),
        ("human", "{input}"),
        ("placeholder", "{agent_scratchpad}"),
    ])

    # Create the agent, binding the LLM, tools, and prompt together.
    agent = create_tool_calling_agent(llm, tools, agent_prompt)

    # AgentExecutor is the runtime that invokes the agent and executes the chosen tools.
    # The 'tools' argument is not needed here as they are already bound to the agent.
    agent_executor = AgentExecutor(agent=agent, verbose=True, tools=tools)


async def run_agent_with_tool(query: str):
    """Invokes the agent executor with a query and prints the final response."""
    print(f"\n--- 🏃 Running Agent with Query: '{query}' ---")
    try:
        response = await agent_executor.ainvoke({"input": query})
        print("\n--- ✅ Final Agent Response ---")
        print(response["output"])
    except Exception as e:
        print(f"\n🛑 An error occurred during agent execution: {e}")


async def main():
    """Runs all agent queries concurrently."""
    tasks = [
        run_agent_with_tool("What is the capital of France?"),
        run_agent_with_tool("What's the weather like in London?"),
        run_agent_with_tool("Tell me something about dogs."),  # Should trigger the default tool response
    ]
    await asyncio.gather(*tasks)


nest_asyncio.apply()
asyncio.run(main())

```

這段程式碼使用 LangChain 函式庫與 Google Gemini 模型建立一個工具呼叫代理。它定義了 `search_information` 工具，用來模擬針對特定查詢提供事實性答案。此工具對「weather in london」、「capital of france」和「population of earth」有預先定義的回應，其他查詢則使用預設回應。程式會初始化具有工具呼叫能力的 ChatGoogleGenerativeAI 模型，並建立 ChatPromptTemplate 來引導代理互動。接著使用 `create_tool_calling_agent` 函式，將語言模型、工具與提示組合成代理。然後設定 AgentExecutor 來管理代理執行與工具呼叫。`run_agent_with_tool` 非同步函式會以給定查詢呼叫代理並列印結果。主要非同步函式準備多個查詢並同時執行；這些查詢用來測試 `search_information` 工具的特定回應與預設回應。最後，asyncio.run(main()) 呼叫會執行所有代理任務。程式碼在進行代理設定與執行前，也包含確認 LLM 是否成功初始化的檢查。

# 實作程式碼範例（CrewAI）

這段程式碼提供一個實務範例，示範如何在 CrewAI 框架中實作 Function Calling（工具）。它設定一個簡單情境，讓代理配備可查詢資訊的工具。此範例特別展示如何透過這個代理與工具取得模擬的股價。

```python
# pip install crewai langchain-openai

import os
from crewai import Agent, Task, Crew
from crewai.tools import tool
import logging


# --- Best Practice: Configure Logging ---
# A basic logging setup helps in debugging and tracking the crew's execution.
logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')


# --- Set up your API Key ---
# For production, it's recommended to use a more secure method for key management
# like environment variables loaded at runtime or a secret manager.
#
# Set the environment variable for your chosen LLM provider (e.g., OPENAI_API_KEY)
# os.environ["OPENAI_API_KEY"] = "YOUR_API_KEY"
# os.environ["OPENAI_MODEL_NAME"] = "gpt-4o"


# --- 1. Refactored Tool: Returns Clean Data ---
# The tool now returns raw data (a float) or raises a standard Python error.
# This makes it more reusable and forces the agent to handle outcomes properly.
@tool("Stock Price Lookup Tool")
def get_stock_price(ticker: str) -> float:
    """
    Fetches the latest simulated stock price for a given stock ticker symbol.
    Returns the price as a float. Raises a ValueError if the ticker is not found.
    """
    logging.info(f"Tool Call: get_stock_price for ticker '{ticker}'")
    simulated_prices = {
        "AAPL": 178.15,
        "GOOGL": 1750.30,
        "MSFT": 425.50,
    }
    price = simulated_prices.get(ticker.upper())
    if price is not None:
        return price
    else:
        # Raising a specific error is better than returning a string.
        # The agent is equipped to handle exceptions and can decide on the next action.
        raise ValueError(f"Simulated price for ticker '{ticker.upper()}' not found.")


# --- 2. Define the Agent ---
# The agent definition remains the same, but it will now leverage the improved tool.
financial_analyst_agent = Agent(
    role='Senior Financial Analyst',
    goal='Analyze stock data using provided tools and report key prices.',
    backstory="You are an experienced financial analyst adept at using data sources to find stock information. You provide clear, direct answers.",
    verbose=True,
    tools=[get_stock_price],
    # Allowing delegation can be useful, but is not necessary for this simple task.
    allow_delegation=False,
)


# --- 3. Refined Task: Clearer Instructions and Error Handling ---
# The task description is more specific and guides the agent on how to react
# to both successful data retrieval and potential errors.
analyze_aapl_task = Task(
    description=(
        "What is the current simulated stock price for Apple (ticker: AAPL)? "
        "Use the 'Stock Price Lookup Tool' to find it. "
        "If the ticker is not found, you must report that you were unable to retrieve the price."
    ),
    expected_output=(
        "A single, clear sentence stating the simulated stock price for AAPL. "
        "For example: 'The simulated stock price for AAPL is $178.15.' "
        "If the price cannot be found, state that clearly."
    ),
    agent=financial_analyst_agent,
)


# --- 4. Formulate the Crew ---
# The crew orchestrates how the agent and task work together.
financial_crew = Crew(
    agents=[financial_analyst_agent],
    tasks=[analyze_aapl_task],
    verbose=True  # Set to False for less detailed logs in production
)


# --- 5. Run the Crew within a Main Execution Block ---
# Using a __name__ == "__main__": block is a standard Python best practice.
def main():
    """Main function to run the crew."""
    # Check for API key before starting to avoid runtime errors.
    if not os.environ.get("OPENAI_API_KEY"):
        print("ERROR: The OPENAI_API_KEY environment variable is not set.")
        print("Please set it before running the script.")
        return

    print("\n## Starting the Financial Crew...")
    print("---------------------------------")

    # The kickoff method starts the execution.
    result = financial_crew.kickoff()

    print("\n---------------------------------")
    print("## Crew execution finished.")
    print("\nFinal Result:\n", result)


if __name__ == "__main__":
    main()
```

這段程式碼示範如何使用 Crew.ai 函式庫建立一個簡單應用，模擬金融分析任務。它定義自訂工具 `get_stock_price`，用來模擬查詢預先定義股票代號的股價。此工具設計為對有效股票代號回傳浮點數，對無效代號則拋出 ValueError。程式建立名為 `financial_analyst_agent` 的 Crew.ai Agent，角色是 Senior Financial Analyst，並提供 `get_stock_price` 工具讓此代理互動。接著定義 `analyze_aapl_task` 這個 Task，明確指示代理使用該工具尋找 AAPL 的模擬股價。任務描述也包含清楚指示，說明使用工具成功或失敗時該如何處理。之後組成一個 Crew，包含 `financial_analyst_agent` 與 `analyze_aapl_task`。代理與 crew 都啟用 verbose 設定，以便在執行期間提供詳細記錄。腳本的主要部分會在標準的 `if __name__ \== "__main__":` 區塊中使用 kickoff() 方法執行 crew 的任務。啟動 crew 前，程式會檢查是否已設定代理運作所需的 `OPENAI_API_KEY` 環境變數。crew 執行結果，也就是任務輸出，接著會列印到主控台。程式碼也包含基本 logging 設定，方便追蹤 crew 的動作與工具呼叫。它使用環境變數管理 API key，但也指出正式環境建議使用更安全的方法。簡而言之，核心邏輯展示了如何定義工具、代理與任務，以建立 Crew.ai 中的協作工作流程。

## 實作程式碼（Google ADK）

Google Agent Developer Kit（ADK）包含一組原生整合的工具函式庫，可直接納入代理的能力中。

**Google search：** 這類元件的一個主要範例是 Google Search 工具。此工具作為 Google Search 引擎的直接介面，讓代理具備執行網路搜尋並擷取外部資訊的功能。

```python
from google.adk.agents import Agent as ADKAgent
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService
from google.adk.tools import google_search
from google.genai import types
import nest_asyncio
import asyncio


# Define variables required for Session setup and Agent execution
APP_NAME = "Google Search Agent"
USER_ID = "user1234"
SESSION_ID = "1234"


# Define Agent with access to search tool
root_agent = ADKAgent(
    name="basic_search_agent",
    model="gemini-2.0-flash-exp",
    description="Agent to answer questions using Google Search.",
    instruction="I can answer your questions by searching the internet. Just ask me anything!",
    tools=[google_search],  # Google Search is a pre-built tool to perform Google searches.
)


# Agent Interaction
async def call_agent(query: str):
    """
    Helper function to call the agent with a query.
    """
    # Session and Runner
    session_service = InMemorySessionService()
    await session_service.create_session(
        app_name=APP_NAME,
        user_id=USER_ID,
        session_id=SESSION_ID,
    )

    runner = Runner(agent=root_agent, app_name=APP_NAME, session_service=session_service)

    content = types.Content(role='user', parts=[types.Part(text=query)])
    events = runner.run(user_id=USER_ID, session_id=SESSION_ID, new_message=content)

    for event in events:
        if event.is_final_response() and event.content:
            # Safely extract text from the final response
            if hasattr(event.content, "text") and event.content.text:
                final_response = event.content.text
            elif event.content.parts:
                final_response = "".join(
                    part.text for part in event.content.parts if getattr(part, "text", None)
                )
            else:
                final_response = ""
            print("Agent Response:", final_response)


nest_asyncio.apply()
asyncio.run(call_agent("what's the latest ai news?"))
```

這段程式碼示範如何建立並使用由 Google ADK for Python 驅動的基本代理。此代理設計為透過 Google Search 作為工具來回答問題。首先匯入 IPython、google.adk 與 google.genai 的必要函式庫，並定義應用程式名稱、使用者 ID 與工作階段 ID 的常數。接著建立名為 `basic_search_agent` 的 Agent 執行個體，並透過描述與指令指出其用途。它被設定為使用 Google Search 工具，這是 ADK 提供的預建工具。程式初始化 InMemorySessionService（見第 8 章），用來管理代理的工作階段。然後為指定的應用程式、使用者與工作階段 ID 建立新的工作階段。接著實例化 Runner，將建立好的代理與工作階段服務連結起來。此 runner 負責在工作階段中執行代理互動。程式也定義輔助函式 `call_agent`，簡化向代理傳送查詢並處理回應的流程。在 `call_agent` 內，使用者查詢會格式化為 role 為 'user' 的 types.Content 物件。程式使用使用者 ID、工作階段 ID 與新訊息內容呼叫 runner.run 方法。runner.run 方法會回傳一系列事件，代表代理的動作與回應。程式會逐一檢查這些事件，以找出最終回應。如果某個事件被辨識為最終回應，便會擷取該回應的文字內容。接著將擷取出的代理回應列印到主控台。最後，以查詢「what's the latest ai news?」呼叫 `call_agent` 函式，示範代理的實際運作。

**Code execution：** Google ADK 提供專門任務的整合元件，包括用於動態程式碼執行的環境。`built_in_code_execution` 工具為代理提供沙盒化的 Python 直譯器。這讓模型可以撰寫並執行程式碼，以完成計算任務、操作資料結構，以及執行程序式腳本。這項功能對於處理需要確定性邏輯與精確計算的問題非常關鍵，因為這些問題超出了單靠機率式語言生成所能處理的範圍。

```python
import os
import getpass
import asyncio
import nest_asyncio
from typing import List
from dotenv import load_dotenv
import logging

from google.adk.agents import Agent as ADKAgent, LlmAgent
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService
from google.adk.tools import google_search
from google.adk.code_executors import BuiltInCodeExecutor
from google.genai import types


# Define variables required for Session setup and Agent execution
APP_NAME = "calculator"
USER_ID = "user1234"
SESSION_ID = "session_code_exec_async"


# Agent Definition
code_agent = LlmAgent(
    name="calculator_agent",
    model="gemini-2.0-flash",
    code_executor=BuiltInCodeExecutor(),
    instruction="""You are a calculator agent.
    When given a mathematical expression, write and execute Python code to calculate the result.
    Return only the final numerical result as plain text, without markdown or code blocks.
    """,
    description="Executes Python code to perform calculations.",
)


# Agent Interaction (Async)
async def call_agent_async(query: str):
    # Session and Runner
    session_service = InMemorySessionService()
    await session_service.create_session(app_name=APP_NAME, user_id=USER_ID, session_id=SESSION_ID)

    runner = Runner(agent=code_agent, app_name=APP_NAME, session_service=session_service)

    content = types.Content(role='user', parts=[types.Part(text=query)])
    print(f"\n--- Running Query: {query} ---")

    try:
        # Use run_async
        async for event in runner.run_async(user_id=USER_ID, session_id=SESSION_ID, new_message=content):
            print(f"Event ID: {event.id}, Author: {event.author}")

            if event.content and event.content.parts and event.is_final_response():
                for part in event.content.parts:  # Iterate through all parts
                    if getattr(part, "executable_code", None):
                        # Access the actual code string via .code
                        print(f"  Debug: Agent generated code:\n```python\n{part.executable_code.code}\n```")
                    elif getattr(part, "code_execution_result", None):
                        # Access outcome and output correctly
                        print(
                            "  Debug: Code Execution Result: "
                            f"{part.code_execution_result.outcome} - Output:\n{part.code_execution_result.output}"
                        )
                    elif getattr(part, "text", None) and not part.text.isspace():
                        # Also print any text parts found in any event for debugging
                        print(f"  Text: '{part.text.strip()}'")

                # --- Check for final response AFTER specific parts ---
                text_parts = [part.text for part in event.content.parts if getattr(part, "text", None)]
                final_result = "".join(text_parts)
                print(f"==> Final Agent Response: {final_result}")

    except Exception as e:
        print(f"ERROR during agent run: {e}")

    print("-" * 30)


# Main async function to run the examples
async def main():
    await call_agent_async("Calculate the value of (5 + 7) * 3")
    await call_agent_async("What is 10 factorial?")


# Execute the main async function
try:
    nest_asyncio.apply()
    asyncio.run(main())
except RuntimeError as e:
    # Handle specific error when running asyncio.run in an already running loop (like Jupyter/Colab)
    if "cannot be called from a running event loop" in str(e):
        print("\nRunning in an existing event loop (like Colab/Jupyter).")
        print("Please run `await main()` in a notebook cell instead.")
        # If in an interactive environment like a notebook, you might need to run:
        # await main()
    else:
        raise e  # Re-raise other runtime errors
```

這個腳本使用 Google Agent Development Kit（ADK）建立一個代理，透過撰寫並執行 Python 程式碼來解決數學問題。它定義了一個 LlmAgent，明確指示其扮演計算器，並配備 `built_in_code_execution` 工具。主要邏輯位於 `call_agent_async` 函式中，該函式會把使用者查詢傳送給代理的 runner，並處理產生的事件。在這個函式內，非同步迴圈逐一處理事件，並為了除錯列印代理產生的 Python 程式碼及其執行結果。程式碼會仔細區分這些中間步驟，以及包含數值答案的最終事件。最後，main 函式會以兩個不同數學運算式執行代理，示範其進行計算的能力。

**Enterprise search：** 這段程式碼使用 Python 中的 google.adk 函式庫定義一個 Google ADK 應用程式。它特別使用 VSearchAgent，設計目的是透過搜尋指定的 Vertex AI Search datastore 來回答問題。程式碼初始化名為 `q2_strategy_vsearch_agent` 的 VSearchAgent，並提供描述、要使用的模型（"gemini-2.0-flash-exp"），以及 Vertex AI Search datastore 的 ID。`DATASTORE_ID` 預期會被設定為環境變數。接著，它使用 InMemorySessionService 管理對話歷史，為代理設定 Runner。程式定義非同步函式 `call_vsearch_agent_async` 來與代理互動。此函式接收查詢、建構訊息內容物件，並呼叫 runner 的 `run_async` 方法將查詢送給代理。然後，函式會在代理回應抵達時串流輸出到主控台。它也會列印最終回應的資訊，包括 datastore 的任何來源歸因。程式包含錯誤處理，以便在代理執行期間捕捉例外，並針對 datastore ID 錯誤或權限不足等潛在問題提供資訊。另一個非同步函式 `run_vsearch_example` 則示範如何用範例查詢呼叫代理。主要執行區塊會檢查 `DATASTORE_ID` 是否已設定，然後使用 asyncio.run 執行範例。它也包含檢查，用來處理程式在已有執行中事件迴圈的環境（如 Jupyter notebook）中執行的情況。

```python
import asyncio
import os

from google.genai import types
from google.adk import agents
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService


# --- Configuration ---
# Ensure you have set your GOOGLE_API_KEY and DATASTORE_ID environment variables
# For example:
# os.environ["GOOGLE_API_KEY"] = "YOUR_API_KEY"
# os.environ["DATASTORE_ID"] = "YOUR_DATASTORE_ID"
DATASTORE_ID = os.environ.get("DATASTORE_ID")


# --- Application Constants ---
APP_NAME = "vsearch_app"
USER_ID = "user_123"  # Example User ID
SESSION_ID = "session_456"  # Example Session ID


# --- Agent Definition (Updated with the newer model from the guide) ---
vsearch_agent = agents.VSearchAgent(
    name="q2_strategy_vsearch_agent",
    description="Answers questions about Q2 strategy documents using Vertex AI Search.",
    model="gemini-2.0-flash-exp",  # Updated model based on the guide's examples
    datastore_id=DATASTORE_ID,
    model_parameters={"temperature": 0.0},
)


# --- Runner and Session Initialization ---
runner = Runner(
    agent=vsearch_agent,
    app_name=APP_NAME,
    session_service=InMemorySessionService(),
)


# --- Agent Invocation Logic ---
async def call_vsearch_agent_async(query: str):
    """Initializes a session and streams the agent's response."""
    print(f"User: {query}")
    print("Agent: ", end="", flush=True)
    try:
        # Construct the message content correctly
        content = types.Content(role='user', parts=[types.Part(text=query)])

        # Process events as they arrive from the asynchronous runner
        async for event in runner.run_async(
            user_id=USER_ID,
            session_id=SESSION_ID,
            new_message=content,
        ):
            # For token-by-token streaming of the response text
            if hasattr(event, "content_part_delta") and event.content_part_delta:
                print(event.content_part_delta.text, end="", flush=True)

            # Process the final response and its associated metadata
            if event.is_final_response():
                print()  # Newline after the streaming response
                if getattr(event, "grounding_metadata", None):
                    print(
                        f"  (Source Attributions: "
                        f"{len(event.grounding_metadata.grounding_attributions)} sources found)"
                    )
                else:
                    print("  (No grounding metadata found)")
                print("-" * 30)
    except Exception as e:
        print(f"\nAn error occurred: {e}")
        print("Please ensure your datastore ID is correct and that the service account has the necessary permissions.")
        print("-" * 30)


# --- Run Example ---
async def run_vsearch_example():
    # Replace with a question relevant to YOUR datastore content
    await call_vsearch_agent_async("Summarize the main points about the Q2 strategy document.")
    await call_vsearch_agent_async("What safety procedures are mentioned for lab X?")


# --- Execution ---
if __name__ == "__main__":
    if not DATASTORE_ID:
        print("Error: DATASTORE_ID environment variable is not set.")
    else:
        try:
            asyncio.run(run_vsearch_example())
        except RuntimeError as e:
            # This handles cases where asyncio.run is called in an environment
            # that already has a running event loop (like a Jupyter notebook).
            if "cannot be called from a running event loop" in str(e):
                print("Skipping execution in a running event loop. Please run this script directly.")
            else:
                raise e
```

整體而言，這段程式碼提供一個基本框架，用於建構會利用 Vertex AI Search，根據 datastore 中儲存資訊回答問題的對話式 AI 應用程式。它示範如何定義代理、設定 runner，以及在串流回應的同時以非同步方式與代理互動。重點在於從特定 datastore 擷取並綜整資訊，以回答使用者查詢。

**Vertex Extensions：** Vertex AI extension 是一種結構化 API wrapper，可讓模型連接外部 API，以進行即時資料處理與動作執行。Extensions 提供企業級安全性、資料隱私與效能保證。它們可用於產生並執行程式碼、查詢網站，以及分析私人 datastore 中資訊等任務。Google 針對 Code Interpreter 與 Vertex AI Search 等常見使用案例提供預建 extensions，也能建立自訂 extensions。Extensions 的主要優點包含強大的企業控制，以及與其他 Google 產品的無縫整合。Extensions 與 Function Calling 的關鍵差異在於執行方式：Vertex AI 會自動執行 extensions，而 function calls 則需要由使用者或用戶端手動執行。

## 一覽

**內容：** LLM 是強大的文字生成器，但本質上與外部世界脫節。它們的知識是靜態的，受限於訓練資料，而且缺乏執行動作或擷取即時資訊的能力。這個內在限制使它們無法完成需要與外部 API、資料庫或服務互動的任務。若沒有通往這些外部系統的橋樑，它們在解決真實世界問題上的實用性會受到嚴重限制。

**原因：** 工具使用模式通常透過 Function Calling 實作，為這個問題提供標準化解法。它的運作方式，是用 LLM 能理解的方式描述可用的外部函式，或稱「工具」。根據使用者請求，代理式 LLM 可以判斷是否需要工具，並產生結構化資料物件（例如 JSON），指定要呼叫哪個函式以及使用哪些引數。協調層會執行這個函式呼叫、取回結果，並將結果回饋給 LLM。這讓 LLM 能把最新外部資訊或動作結果納入最終回應，實質上賦予它行動能力。

**經驗法則：** 只要代理需要跳脫 LLM 內部知識並與外部世界互動，就使用工具使用模式。對於需要即時資料（例如查詢天氣、股價）、存取私人或專有資訊（例如查詢公司資料庫）、執行精確計算、執行程式碼，或在其他系統中觸發動作（例如傳送電子郵件、控制智慧裝置）的任務，這都是必要的。

**視覺摘要：**

![工具使用設計模式](../assets/Tool_Use_Design_Pattern.png)

圖 2：工具使用設計模式

## 重點整理

* 工具使用（Tool Use / Function Calling）讓代理能與外部系統互動並存取動態資訊。  
* 它涉及以清楚描述與參數定義工具，讓 LLM 能理解。  
* LLM 會決定何時使用工具，並產生結構化函式呼叫。  
* 代理式框架會執行實際工具呼叫，並將結果回傳給 LLM。  
* 工具使用對於建構能執行真實世界動作並提供最新資訊的代理至關重要。  
* LangChain 使用 @tool decorator 簡化工具定義，並提供 `create_tool_calling_agent` 與 AgentExecutor 來建構會使用工具的代理。  
* Google ADK 具有許多非常實用的預建工具，例如 Google Search、Code Execution 和 Vertex AI Search Tool。

## 結論

工具使用模式是一項關鍵架構原則，可將 LLM 的功能範圍延伸到其固有文字生成能力之外。透過讓模型具備與外部軟體和資料來源介接的能力，這個典範使代理能執行動作、進行運算，並從其他系統擷取資訊。此流程涉及模型在判斷必須呼叫外部工具才能完成使用者查詢時，產生結構化請求來呼叫該工具。LangChain、Google ADK 與 Crew AI 等框架提供結構化抽象與元件，協助整合這些外部工具。這些框架會管理向模型揭露工具規格，以及解析後續工具使用請求的流程。這簡化了複雜代理式系統的開發，讓系統能在外部數位環境中互動並採取行動。

## 參考資料

1. LangChain Documentation (Tools): [https://python.langchain.com/docs/integrations/tools/](https://python.langchain.com/docs/integrations/tools/)
2. Google Agent Developer Kit (ADK) Documentation (Tools): [https://google.github.io/adk-docs/tools/](https://google.github.io/adk-docs/tools/)
3. OpenAI Function Calling Documentation: [https://platform.openai.com/docs/guides/function-calling](https://platform.openai.com/docs/guides/function-calling)
4. CrewAI Documentation (Tools): [https://docs.crewai.com/concepts/tools](https://docs.crewai.com/concepts/tools)


