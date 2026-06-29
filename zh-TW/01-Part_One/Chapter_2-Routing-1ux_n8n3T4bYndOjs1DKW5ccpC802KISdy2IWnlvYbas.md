# 第 2 章：路由（Routing）

## 路由模式概觀

雖然透過提示鏈進行循序處理，是使用語言模型執行確定性、線性工作流程的基礎技術，但在需要自適應回應的情境中，它的適用範圍有限。真實世界的代理式系統，往往必須根據環境狀態、使用者輸入，或前一個操作的結果等條件因素，在多種可能動作之間進行判斷。這種動態決策能力會將控制流程導向不同的專門功能、工具或子程序，而實現這種能力的機制稱為路由。

路由會在代理（agent）的運作框架中引入條件邏輯，讓系統從固定的執行路徑，轉變為由代理動態評估特定條件，並從一組可能的後續動作中做出選擇的模型。這使系統行為更具彈性，也更能感知上下文。

例如，當設計來處理客戶詢問的代理具備路由功能時，它可以先分類傳入查詢，以判斷使用者意圖。接著，它可以根據分類結果，將查詢導向負責直接問答的專門代理、用於取得帳戶資訊的資料庫擷取工具，或處理複雜問題的升級程序，而不是預設走單一、事先決定好的回應路徑。因此，使用路由的較進階代理可以：

1. 分析使用者的查詢。  
2. 根據查詢的*意圖*來**路由**：  
   * 如果意圖是「check order status」，路由至會與訂單資料庫互動的子代理或工具鏈。  
   * 如果意圖是「product information」，路由至會搜尋產品目錄的子代理或鏈。  
   * 如果意圖是「technical support」，路由至會存取疑難排解指南或升級給人類的另一條鏈。  
   * 如果意圖不明確，路由至澄清用的子代理或提示鏈。

路由模式的核心元件，是一個負責執行評估並導引流程的機制。這個機制可以用幾種方式實作：

* **基於大型語言模型（large language model, LLM）的路由：** 可以提示語言模型本身分析輸入，並輸出特定識別碼或指令，用來表示下一步或目的地。例如，提示可以要求 LLM「Analyze the following user query and output only the category: 'Order Status', 'Product Info', 'Technical Support', or 'Other'.」代理式系統接著讀取這個輸出，並據此導引工作流程。  
* **基於嵌入（embedding）的路由：** 輸入查詢可以轉換成向量 embedding（請參見 RAG，第 14 章）。接著，這個 embedding 會與代表不同路由或能力的 embedding 進行比較。查詢會被路由到 embedding 最相似的路徑。這對語意路由很有用，因為決策依據是輸入的意義，而不只是關鍵字。  
* **基於規則的路由：** 這涉及使用預先定義的規則或邏輯（例如 if-else 陳述式、switch cases），依據關鍵字、模式，或從輸入中擷取出的結構化資料進行判斷。這可以比基於 LLM 的路由更快且更具確定性，但在處理細微或新穎輸入時較不彈性。  
* **基於機器學習模型的路由**：它採用判別式模型，例如分類器，該模型已在一小組標記資料語料上接受專門訓練，以執行路由任務。雖然它在概念上與基於 embedding 的方法相似，但其關鍵特徵是監督式微調流程，會調整模型參數以建立專門的路由功能。這項技術不同於基於 LLM 的路由，因為其決策元件不是在推論時執行提示的生成式模型。相反地，路由邏輯被編碼在微調模型所學得的權重中。雖然 LLM 可用於前處理步驟，以產生合成資料來擴充訓練集，但它們不會參與即時路由決策本身。

路由機制可以在代理運作週期中的多個交會點實作。它們可以在一開始用來分類主要任務，在處理鏈的中間節點用來決定後續動作，或在子程序中用來從既定集合中選出最合適的工具。

LangChain、LangGraph 和 Google 的 Agent Developer Kit (ADK) 等運算框架，都提供明確的建構方式來定義與管理這類條件邏輯。憑藉其基於狀態的圖架構，LangGraph 特別適合複雜路由情境，尤其是決策取決於整個系統累積狀態的情況。同樣地，Google 的 ADK 提供用於建構代理能力與互動模型的基礎元件，這些元件可作為實作路由邏輯的基礎。在這些框架提供的執行環境中，開發者會定義可能的運作路徑，以及決定運算圖中節點之間轉換的函式或基於模型的評估。

實作路由能讓系統超越確定性的循序處理。它有助於發展更具適應性的執行流程，能對更廣泛的輸入與狀態變化做出動態且適當的回應。

## 實際應用與使用案例

路由模式是設計自適應代理式系統時的關鍵控制機制，讓系統能因應不同輸入與內部狀態，動態改變其執行路徑。它透過提供必要的條件邏輯層，展現出跨多個領域的效用。

在人機互動中，例如虛擬助理或 AI 驅動的家教系統，路由會用來解讀使用者意圖。對自然語言查詢的初步分析，會決定最合適的後續動作：可能是叫用特定資訊擷取工具、升級給人類操作員，或根據使用者表現選擇課程中的下一個模組。這讓系統能超越線性對話流程，並依上下文回應。

在自動化資料與文件處理管線中，路由扮演分類與分配功能。傳入資料，例如電子郵件、支援票證或 API payload，會根據內容、中繼資料或格式進行分析。接著，系統會將每個項目導向對應的工作流程，例如銷售線索匯入流程、針對 JSON 或 CSV 格式的特定資料轉換函式，或緊急問題升級路徑。

在涉及多個專門工具或代理的複雜系統中，路由扮演高階分派器的角色。由不同代理組成、分別負責搜尋、摘要與分析資訊的研究系統，會使用路由器根據目前目標，將任務指派給最合適的代理。同樣地，AI 程式碼助理會先使用路由來識別程式語言與使用者意圖——例如偵錯、解釋或翻譯——再將程式碼片段傳遞給正確的專門工具。

最終，路由提供建立功能多樣且能感知上下文的系統所必需的邏輯判斷能力。它會將代理從預先定義序列的靜態執行者，轉變為能在條件變化下，就完成任務的最有效方法做出決策的動態系統。

## 實作程式碼範例 (LangChain)

在程式碼中實作路由，涉及定義可能路徑，以及決定要採取哪條路徑的邏輯。LangChain 和 LangGraph 等框架提供了對應的元件與結構。LangGraph 基於狀態的圖結構，特別直覺地支援路由邏輯的視覺化與實作。

這段程式碼示範一個使用 LangChain 與 Google Generative AI 的簡單類代理系統。它設定一個「協調者」，根據請求意圖（預訂、資訊或不明確），將使用者請求路由至不同的模擬「子代理」處理常式。系統使用語言模型分類請求，再將其委派給適當的處理函式，模擬多代理架構中常見的基本委派模式。

首先，請確認已安裝必要的程式庫：

```bash
pip install langchain langgraph google-cloud-aiplatform langchain-google-genai google-adk deprecated pydantic
```

你也需要使用所選語言模型的 API key 設定環境（例如 OpenAI、Google Gemini、Anthropic）。

```python
# Copyright (c) 2025 Marco Fago
# https://www.linkedin.com/in/marco-fago/
#
# This code is licensed under the MIT License.
# See the LICENSE file in the repository for the full license text.

from langchain_google_genai import ChatGoogleGenerativeAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough, RunnableBranch


# --- Configuration ---
# Ensure your API key environment variable is set (e.g., GOOGLE_API_KEY)
try:
    llm = ChatGoogleGenerativeAI(model="gemini-2.5-flash", temperature=0)
    print(f"Language model initialized: {llm.model}")
except Exception as e:
    print(f"Error initializing language model: {e}")
    llm = None


# --- Define Simulated Sub-Agent Handlers (equivalent to ADK sub_agents) ---
def booking_handler(request: str) -> str:
    """Simulates the Booking Agent handling a request."""
    print("\n--- DELEGATING TO BOOKING HANDLER ---")
    return f"Booking Handler processed request: '{request}'. Result: Simulated booking action."


def info_handler(request: str) -> str:
    """Simulates the Info Agent handling a request."""
    print("\n--- DELEGATING TO INFO HANDLER ---")
    return f"Info Handler processed request: '{request}'. Result: Simulated information retrieval."


def unclear_handler(request: str) -> str:
    """Handles requests that couldn't be delegated."""
    print("\n--- HANDLING UNCLEAR REQUEST ---")
    return f"Coordinator could not delegate request: '{request}'. Please clarify."


# --- Define Coordinator Router Chain (equivalent to ADK coordinator's instruction) ---
# This chain decides which handler to delegate to.
coordinator_router_prompt = ChatPromptTemplate.from_messages([
    (
        "system",
        """Analyze the user's request and determine which specialist handler should process it.
        - If the request is related to booking flights or hotels,
           output 'booker'.
        - For all other general information questions, output 'info'.
        - If the request is unclear or doesn't fit either category,
           output 'unclear'.
        ONLY output one word: 'booker', 'info', or 'unclear'."""
    ),
    ("user", "{request}")
])

if llm:
    coordinator_router_chain = coordinator_router_prompt | llm | StrOutputParser()


# --- Define the Delegation Logic (equivalent to ADK's Auto-Flow based on sub_agents) ---
# Use RunnableBranch to route based on the router chain's output.

# Define the branches for the RunnableBranch
branches = {
    "booker": RunnablePassthrough.assign(
        output=lambda x: booking_handler(x['request']['request'])
    ),
    "info": RunnablePassthrough.assign(
        output=lambda x: info_handler(x['request']['request'])
    ),
    "unclear": RunnablePassthrough.assign(
        output=lambda x: unclear_handler(x['request']['request'])
    ),
}

# Create the RunnableBranch. It takes the output of the router chain
# and routes the original input ('request') to the corresponding handler.
delegation_branch = RunnableBranch(
    (lambda x: x['decision'].strip() == 'booker', branches["booker"]),  # Added .strip()
    (lambda x: x['decision'].strip() == 'info', branches["info"]),      # Added .strip()
    branches["unclear"]  # Default branch for 'unclear' or any other output
)

# Combine the router chain and the delegation branch into a single runnable
# The router chain's output ('decision') is passed along with the original input ('request')
# to the delegation_branch.
coordinator_agent = {
    "decision": coordinator_router_chain,
    "request": RunnablePassthrough()
} | delegation_branch | (lambda x: x['output'])  # Extract the final output


# --- Example Usage ---
def main():
    if not llm:
        print("\nSkipping execution due to LLM initialization failure.")
        return

    print("--- Running with a booking request ---")
    request_a = "Book me a flight to London."
    result_a = coordinator_agent.invoke({"request": request_a})
    print(f"Final Result A: {result_a}")

    print("\n--- Running with an info request ---")
    request_b = "What is the capital of Italy?"
    result_b = coordinator_agent.invoke({"request": request_b})
    print(f"Final Result B: {result_b}")

    print("\n--- Running with an unclear request ---")
    request_c = "Tell me about quantum physics."
    result_c = coordinator_agent.invoke({"request": request_c})
    print(f"Final Result C: {result_c}")


if __name__ == "__main__":
    main()
```

如前所述，這段 Python 程式碼使用 LangChain 程式庫與 Google Generative AI 模型（具體來說是 gemini-2.5-flash），建構了一個簡單的類代理系統。詳細來說，它定義了三個模擬子代理處理常式：`booking_handler`、`info_handler` 和 `unclear_handler`，各自設計用來處理特定類型的請求。

核心元件是 `coordinator_router_chain`，它使用 ChatPromptTemplate 指示語言模型將傳入的使用者請求分類為三種之一：`booker`、`info` 或 `unclear`。接著，RunnableBranch 會使用這個路由器鏈的輸出，將原始請求委派給對應的處理函式。RunnableBranch 會檢查語言模型做出的決策，並將請求資料導向 `booking_handler`、`info_handler` 或 `unclear_handler`。`coordinator_agent` 會組合這些元件，先路由請求以取得決策，再將請求傳遞給選定的處理常式。最終輸出會從處理常式的回應中擷取。

main 函式透過三個範例請求示範系統用法，展示不同輸入如何被模擬代理路由與處理。程式碼也包含語言模型初始化的錯誤處理，以確保穩健性。程式碼結構模擬了基本多代理框架，其中中央協調者會根據意圖，將任務委派給專門代理。

## 實作程式碼範例 (Google ADK)

Agent Development Kit (ADK) 是用於工程化代理式系統的框架，提供結構化環境來定義代理的能力與行為。相較於基於明確運算圖的架構，ADK 範式中的路由通常是透過定義一組離散的「工具」來實作，這些工具代表代理的功能。框架的內部邏輯會管理對使用者查詢所採取的適當工具選擇，並利用底層模型將使用者意圖對應至正確的功能處理常式。

這段 Python 程式碼示範一個使用 Google ADK 程式庫的 Agent Development Kit (ADK) 應用程式範例。它設定一個「Coordinator」代理，根據定義好的指示，將使用者請求路由至專門子代理（「Booker」負責預訂，「Info」負責一般資訊）。接著，子代理會使用特定工具來模擬處理請求，展示代理系統中的基本委派模式。

```python
# Copyright (c) 2025 Marco Fago
#
# This code is licensed under the MIT License.
# See the LICENSE file in the repository for the full license text.

import uuid
from typing import Dict, Any, Optional

from google.adk.agents import Agent
from google.adk.runners import InMemoryRunner
from google.adk.tools import FunctionTool
from google.genai import types
from google.adk.events import Event


# --- Define Tool Functions ---
# These functions simulate the actions of the specialist agents.
def booking_handler(request: str) -> str:
    """
    Handles booking requests for flights and hotels.

    Args:
        request: The user's request for a booking.

    Returns:
        A confirmation message that the booking was handled.
    """
    print("-------------------------- Booking Handler Called ----------------------------")
    return f"Booking action for '{request}' has been simulated."


def info_handler(request: str) -> str:
    """
    Handles general information requests.

    Args:
        request: The user's question.

    Returns:
        A message indicating the information request was handled.
    """
    print("-------------------------- Info Handler Called ----------------------------")
    return f"Information request for '{request}'. Result: Simulated information retrieval."


def unclear_handler(request: str) -> str:
    """Handles requests that couldn't be delegated."""
    return f"Coordinator could not delegate request: '{request}'. Please clarify."


# --- Create Tools from Functions ---
booking_tool = FunctionTool(booking_handler)
info_tool = FunctionTool(info_handler)

# Define specialized sub-agents equipped with their respective tools
booking_agent = Agent(
    name="Booker",
    model="gemini-2.0-flash",
    description="A specialized agent that handles all flight "
                "and hotel booking requests by calling the booking tool.",
    tools=[booking_tool],
)

info_agent = Agent(
    name="Info",
    model="gemini-2.0-flash",
    description="A specialized agent that provides general information "
                "and answers user questions by calling the info tool.",
    tools=[info_tool],
)

# Define the parent agent with explicit delegation instructions
coordinator = Agent(
    name="Coordinator",
    model="gemini-2.0-flash",
    instruction=(
        "You are the main coordinator. Your only task is to analyze "
        "incoming user requests "
        "and delegate them to the appropriate specialist agent. Do not try to answer the user directly.\n"
        "- For any requests related to booking flights or hotels, delegate to the 'Booker' agent.\n"
        "- For all other general information questions, delegate to the 'Info' agent."
    ),
    description="A coordinator that routes user requests to the correct specialist agent.",
    # The presence of sub_agents enables LLM-driven delegation (Auto-Flow) by default.
    sub_agents=[booking_agent, info_agent],
)


# --- Execution Logic ---
async def run_coordinator(runner: InMemoryRunner, request: str):
    """Runs the coordinator agent with a given request and delegates."""
    print(f"\n--- Running Coordinator with request: '{request}' ---")
    final_result = ""
    try:
        user_id = "user_123"
        session_id = str(uuid.uuid4())

        await runner.session_service.create_session(
            app_name=runner.app_name,
            user_id=user_id,
            session_id=session_id,
        )

        for event in runner.run(
            user_id=user_id,
            session_id=session_id,
            new_message=types.Content(
                role='user',
                parts=[types.Part(text=request)],
            ),
        ):
            if event.is_final_response() and event.content:
                # Try to get text directly from event.content to avoid iterating parts
                if hasattr(event.content, 'text') and event.content.text:
                    final_result = event.content.text
                elif event.content.parts:
                    # Fallback: Iterate through parts and extract text (might trigger warning)
                    text_parts = [part.text for part in event.content.parts if getattr(part, "text", None)]
                    final_result = "".join(text_parts)
                # Assuming the loop should break after the final response
                break

        print(f"Coordinator Final Response: {final_result}")
        return final_result

    except Exception as e:
        print(f"An error occurred while processing your request: {e}")
        return f"An error occurred while processing your request: {e}"


async def main():
    """Main function to run the ADK example."""
    print("--- Google ADK Routing Example (ADK Auto-Flow Style) ---")
    print("Note: This requires Google ADK installed and authenticated.")

    runner = InMemoryRunner(coordinator)

    # Example Usage
    result_a = await run_coordinator(runner, "Book me a hotel in Paris.")
    print(f"Final Output A: {result_a}")

    result_b = await run_coordinator(runner, "What is the highest mountain in the world?")
    print(f"Final Output B: {result_b}")

    result_c = await run_coordinator(runner, "Tell me a random fact.")  # Should go to Info
    print(f"Final Output C: {result_c}")

    result_d = await run_coordinator(runner, "Find flights to Tokyo next month.")  # Should go to Booker
    print(f"Final Output D: {result_d}")


if __name__ == "__main__":
    import nest_asyncio

    nest_asyncio.apply()
    await main()
```

這個腳本由一個主要 Coordinator 代理與兩個專門的 `sub_agents` 組成：Booker 和 Info。每個專門代理都配備一個 FunctionTool，用來包裝模擬動作的 Python 函式。`booking_handler` 函式模擬處理航班與飯店預訂，而 `info_handler` 函式模擬擷取一般資訊。`unclear_handler` 會作為協調者無法委派請求時的後備處理常式，雖然目前的協調者邏輯並未在主要 `run_coordinator` 函式中明確用它處理委派失敗。

如其指示所定義，Coordinator 代理的主要角色是分析傳入的使用者訊息，並將其委派給 Booker 或 Info 代理。由於 Coordinator 定義了 `sub_agents`，這項委派會由 ADK 的 Auto-Flow 機制自動處理。`run_coordinator` 函式會設定 InMemoryRunner、建立使用者與 session ID，然後使用 runner 透過 coordinator 代理處理使用者請求。runner.run method 會處理請求並產生事件，程式碼則從 event.content 擷取最終回應文字。

main 函式透過不同請求執行 coordinator 來示範系統用法，展示它如何將預訂請求委派給 Booker，並將資訊請求委派給 Info 代理。

## 快速總覽

**是什麼**：代理式系統往往必須回應各式各樣的輸入與情境，而這些輸入與情境無法由單一線性流程處理。簡單的循序工作流程缺乏依上下文做出決策的能力。若沒有能為特定任務選擇正確工具或子程序的機制，系統就會保持僵硬，無法自適應。這項限制會讓開發能管理真實世界使用者請求之複雜性與變異性的進階應用變得困難。

**為什麼：** 路由模式透過將條件邏輯引入代理的運作框架，提供標準化解法。它讓系統先分析傳入查詢，以判斷其意圖或性質。代理會根據這項分析，將控制流程動態導向最合適的專門工具、函式或子代理。這項決策可以由多種方法驅動，包括提示 LLM、套用預先定義的規則，或使用基於 embedding 的語意相似度。最終，路由會將靜態、預先決定好的執行路徑，轉變成彈性且能感知上下文的工作流程，並能選擇最佳可能動作。

**經驗法則：** 當代理必須根據使用者輸入或目前狀態，在多個不同工作流程、工具或子代理之間做出決策時，就使用路由模式。對於需要分流或分類傳入請求以處理不同任務類型的應用程式而言，路由至關重要，例如客戶支援機器人需要區分銷售詢問、技術支援與帳戶管理問題。

**視覺摘要：**

![Router Pattern, using LLM as a Router](../assets/Router_Pattern_Using_LLM_as_a_Router.png)

圖 1：Router pattern, using an LLM as a Router

## 重點整理

* 路由讓代理能根據條件，對工作流程中的下一步做出動態決策。  
* 它讓代理能處理多樣化輸入並調整其行為，超越線性執行。  
* 路由邏輯可以使用 LLM、基於規則的系統，或 embedding 相似度來實作。  
* LangGraph 和 Google ADK 等框架提供結構化方式，可在代理工作流程中定義與管理路由，不過其架構方法有所不同。

## 結論

路由模式是建立真正動態且反應靈敏的代理式系統時的關鍵一步。透過實作路由，我們能超越簡單、線性的執行流程，並讓代理有能力針對如何處理資訊、回應使用者輸入，以及運用可用工具或子代理，做出智慧型決策。

我們已看到路由可應用於各種領域，從客服聊天機器人到複雜資料處理管線皆然。分析輸入並依條件導引工作流程的能力，是建立能處理真實世界任務固有變異性的代理之基礎。

使用 LangChain 與 Google ADK 的程式碼範例，展示了兩種不同但有效的路由實作方法。LangGraph 基於圖的結構，提供一種視覺化且明確的方式來定義狀態與轉換，因此非常適合具有複雜路由邏輯的多步驟工作流程。另一方面，Google ADK 通常著重於定義不同能力（Tools），並仰賴框架將使用者請求路由至適當工具處理常式的能力；對於具有明確離散動作集合的代理而言，這種方式可能更簡單。

掌握路由模式，是建立能智慧地瀏覽不同情境，並根據上下文提供量身打造回應或動作的代理所必需的。它是建立多用途且穩健的代理式應用程式的關鍵元件。

## 參考資料

1. LangGraph Documentation: [https://www.langchain.com/](https://www.langchain.com/)
2. Google Agent Developer Kit Documentation: [https://google.github.io/adk-docs/](https://google.github.io/adk-docs/)

