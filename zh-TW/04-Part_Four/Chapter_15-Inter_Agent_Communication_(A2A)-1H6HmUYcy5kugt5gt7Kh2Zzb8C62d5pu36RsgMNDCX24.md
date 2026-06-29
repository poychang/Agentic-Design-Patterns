# 第 15 章：代理間通訊（A2A）

即使具備進階能力，個別 AI 代理（agent）在處理複雜且多面向的問題時，仍經常受到限制。為了克服這一點，代理間通訊（A2A）讓可能以不同框架建置的各種 AI 代理能有效協作。這種協作包含順暢的協調、任務委派與資訊交換。

Google 的 A2A 協定是一項開放標準，旨在促成這種通用通訊。本章將探討 A2A、其實務應用，以及在 Google ADK 中的實作方式。

## 代理間通訊模式概觀

Agent2Agent（A2A）協定是一項開放標準，旨在讓不同 AI 代理框架之間能夠通訊與協作。它確保互通性，讓使用 LangGraph、CrewAI 或 Google ADK 等技術開發的 AI 代理，無論來源或框架差異為何，都能共同運作。

A2A 獲得多家科技公司與服務提供者支援，包括 Atlassian、Box、LangChain、MongoDB、Salesforce、SAP 與 ServiceNow。Microsoft 計畫將 A2A 整合至 Azure AI Foundry 與 Copilot Studio，展現其對開放協定的承諾。此外，Auth0 與 SAP 也正在將 A2A 支援整合到各自的平台與代理中。

作為開放原始碼協定，A2A 歡迎社群貢獻，以促進其演進與廣泛採用。

## A2A 的核心概念

A2A 協定以多個核心概念為基礎，為代理互動提供結構化方法。對於任何開發或整合 A2A 相容系統的人而言，充分掌握這些概念都至關重要。A2A 的基礎支柱包括核心參與者、Agent Card、代理探索、通訊與任務、互動機制，以及安全性，以下將逐一詳細說明。

**核心參與者：** A2A 涉及三個主要實體：

* User：發起代理協助請求。  
* A2A Client（Client Agent）：代表使用者請求動作或資訊的應用程式或 AI 代理。  
* A2A Server（Remote Agent）：提供 HTTP 端點以處理用戶端請求並回傳結果的 AI 代理或系統。遠端代理以「不透明」系統運作，表示用戶端不需要了解其內部運作細節。

**Agent Card：** 代理的數位身分由其 Agent Card 定義，通常是一個 JSON 檔案。此檔案包含用戶端互動與自動探索所需的關鍵資訊，包括代理的身分、端點 URL 與版本。它也詳細說明支援的能力，例如串流或推播通知、特定技能、預設輸入／輸出模式，以及驗證需求。以下是一個 WeatherBot 的 Agent Card 範例。

```json
{
    "name": "WeatherBot",
    "description": "Provides accurate weather forecasts and historical data.",
    "url": "http://weather-service.example.com/a2a",
    "version": "1.0.0",
    "capabilities": {
        "streaming": true,
        "pushNotifications": false,
        "stateTransitionHistory": true
    },
    "authentication": {
        "schemes": [
            "apiKey"
        ]
    },
    "defaultInputModes": [
        "text"
    ],
    "defaultOutputModes": [
        "text"
    ],
    "skills": [
        {
            "id": "get_current_weather",
            "name": "Get Current Weather",
            "description": "Retrieve real-time weather for any location.",
            "inputModes": [
                "text"
            ],
            "outputModes": [
                "text"
            ],
            "examples": [
                "What's the weather in Paris?",
                "Current conditions in Tokyo"
            ],
            "tags": [
                "weather",
                "current",
                "real-time"
            ]
        },
        {
            "id": "get_forecast",
            "name": "Get Forecast",
            "description": "Get 5-day weather predictions.",
            "inputModes": [
                "text"
            ],
            "outputModes": [
                "text"
            ],
            "examples": [
                "5-day forecast for New York",
                "Will it rain in London this weekend?"
            ],
            "tags": [
                "weather",
                "forecast",
                "prediction"
            ]
        }
    ]
}
```

**代理探索：** 代理探索讓用戶端找到 Agent Cards，這些卡片描述可用 A2A Servers 的能力。此流程有幾種策略：

* Well-Known URI：代理將其 Agent Card 託管在標準化路徑（例如 /.well-known/agent.json）。這種方法能為公開或特定網域用途提供廣泛且通常可自動化的可存取性。  
* 策展式註冊表（Curated Registries）**：** 這些註冊表提供集中式目錄，用於發布 Agent Cards，並可依特定條件查詢。這非常適合需要集中管理與存取控制的企業環境。  
* 直接設定（Direct Configuration）**：** Agent Card 資訊會被嵌入或私下分享。此方法適用於緊密耦合或私有系統，也就是動態探索並非關鍵需求的情境。

無論選擇哪一種方法，保護 Agent Card 端點都很重要。這可以透過存取控制、相互 TLS（mTLS）或網路限制來達成，特別是當卡片包含敏感但非機密資訊時。

**通訊與任務：** 在 A2A 框架中，通訊是圍繞非同步任務而結構化的，這些任務代表長時間執行流程的基本工作單位。每個任務都會被指派唯一識別碼，並經過一系列狀態，例如 submitted、working 或 completed；這樣的設計支援複雜作業中的平行處理。代理之間的通訊透過 Message 進行。

此通訊包含屬性，也就是描述訊息的鍵值中繼資料（例如優先順序或建立時間），以及一個或多個 parts，用來承載實際傳遞的內容，例如純文字、檔案或結構化 JSON 資料。代理在任務期間產生的具體輸出稱為 artifacts。與 messages 類似，artifacts 也由一個或多個 parts 組成，並可在結果可用時以增量方式串流傳送。A2A 框架內的所有通訊都透過 HTTP(S) 進行，負載使用 JSON-RPC 2.0 協定。為了在多次互動之間維持連續性，伺服器產生的 contextId 會用來群組相關任務並保留上下文。

**互動機制**：Request/Response（Polling）Server-Sent Events（SSE）。A2A 提供多種互動方法，以符合各種 AI 應用需求，每種方法都有不同機制：

* Synchronous Request/Response：適用於快速、即時的作業。在此模型中，用戶端送出請求，並主動等待伺服器處理後，在單一同步交換中回傳完整回應。  
* Asynchronous Polling：適用於需要較長處理時間的任務。用戶端送出請求後，伺服器會立即以「working」狀態與任務 ID 確認收到。接著用戶端可自由執行其他動作，並能定期透過傳送新的請求向伺服器輪詢任務狀態，直到任務被標示為「completed」或「failed」。  
* Streaming Updates（Server-Sent Events - SSE）：非常適合接收即時、增量的結果。此方法會建立從伺服器到用戶端的持久單向連線，讓遠端代理能持續推送更新，例如狀態變更或部分結果，而不需要用戶端發出多次請求。  
* Push Notifications（Webhooks）：專為非常長時間執行或資源密集的任務而設計，因為在這些情境中維持固定連線或頻繁輪詢效率不佳。用戶端可以註冊 webhook URL，伺服器會在任務狀態發生重大變化時（例如完成時），向該 URL 傳送非同步通知（「push」）。

Agent Card 會指定代理是否支援串流或推播通知能力。此外，A2A 不受模態限制，表示它不只可促成文字的這些互動模式，也可用於音訊與影片等其他資料類型，進而支援豐富的多模態 AI 應用。串流與推播通知能力都會在 Agent Card 中指定。

```json
# Synchronous Request Example 
{
    "jsonrpc": "2.0",
    "id": "1",
    "method": "sendTask",
    "params": {
        "id": "task-001",
        "sessionId": "session-001",
        "message": {
            "role": "user",
            "parts": [
                {
                    "type": "text",
                    "text": "What is the exchange rate from USD to EUR?"
                }
            ]
        },
        "acceptedOutputModes": [
            "text/plain"
        ],
        "historyLength": 5
    }
}
```

同步請求使用 sendTask 方法；在此情況下，用戶端提出查詢，並預期取得單一且完整的答案。相較之下，串流請求使用 sendTaskSubscribe 方法建立持久連線，讓代理能隨時間回傳多個增量更新或部分結果。

```json
# Streaming Request Example 
{
    "jsonrpc": "2.0",
    "id": "2",
    "method": "sendTaskSubscribe",
    "params": {
        "id": "task-002",
        "sessionId": "session-001",
        "message": {
            "role": "user",
            "parts": [
                {
                    "type": "text",
                    "text": "What's the exchange rate for JPY to GBP today?"
                }
            ]
        },
        "acceptedOutputModes": [
            "text/plain"
        ],
        "historyLength": 5
    }
}
```

**安全性：** 代理間通訊（A2A）：代理間通訊（A2A）是系統架構的重要元件，可讓代理之間安全且順暢地交換資料。它透過多種內建機制確保穩健性與完整性。

相互傳輸層安全性（Mutual Transport Layer Security, TLS）：建立加密且經過驗證的連線，以防止未授權存取與資料攔截，確保通訊安全。

完整稽核記錄（Comprehensive Audit Logs）：所有代理間通訊都會被仔細記錄，詳細描述資訊流、涉及的代理與動作。這項稽核軌跡對於究責、疑難排解與安全性分析至關重要。

Agent Card 宣告（Agent Card Declaration）：驗證需求會在 Agent Card 中明確宣告；Agent Card 是描述代理身分、能力與安全性政策的設定成品。這能集中並簡化驗證管理。

憑證處理（Credential Handling）：代理通常會使用 OAuth 2.0 token 或 API 金鑰等安全憑證進行驗證，並透過 HTTP 標頭傳遞。此方法可避免憑證暴露於 URL 或訊息本文中，進而強化整體安全性。

## A2A 與 MCP

A2A 是一個與 Anthropic 的 Model Context Protocol（MCP）互補的協定（見圖 1）。MCP 著重於為代理及其與外部資料和工具的互動建構上下文，而 A2A 則促成代理之間的協調與通訊，支援任務委派與協作。

![A2A 與 MCP 協定比較](../assets/Comparison_A2A_and_MCP_Protocols.png)

圖 1：A2A 與 MCP 協定比較

A2A 的目標是在開發複雜、多代理 AI 系統時提升效率、降低整合成本，並促進創新與互通性。因此，若要有效設計、實作並應用 A2A 來建構協作且可互通的 AI 代理系統，就必須深入理解其核心元件與運作方式。

## 實務應用與使用案例

代理間通訊對於在各種領域建置精密的 AI 解決方案不可或缺，因為它能支援模組化、可擴充性，並提升智慧程度。

* **多框架協作：** A2A 的主要使用案例，是讓獨立 AI 代理無論其底層框架為何（例如 ADK、LangChain、CrewAI），都能彼此通訊與協作。這是建置複雜多代理系統的基礎，在這類系統中，不同代理會專精於問題的不同面向。  
* **自動化工作流程協調：** 在企業環境中，A2A 可讓代理委派並協調任務，進而促成複雜工作流程。例如，一個代理可能負責初始資料收集，接著委派另一個代理進行分析，最後再交由第三個代理產生報告，而所有代理都透過 A2A 協定通訊。  
* **動態資訊擷取：** 代理可透過通訊來擷取並交換即時資訊。主要代理可能會向專門的「data fetching agent」請求即時市場資料；該代理再使用外部 APIs 收集資訊並回傳。

## 實作程式碼範例

讓我們檢視 A2A 協定的實務應用。[https://github.com/google-a2a/a2a-samples/tree/main/samples](https://github.com/google-a2a/a2a-samples/tree/main/samples) 的儲存庫提供 Java、Go 與 Python 範例，說明 LangGraph、CrewAI、Azure AI Foundry 與 AG2 等各種代理框架如何使用 A2A 進行通訊。此儲存庫中的所有程式碼皆以 Apache 2.0 授權條款發布。為了進一步說明 A2A 的核心概念，我們將檢視幾段程式碼摘錄，重點放在使用具備 Google 驗證工具、以 ADK 為基礎的代理來設定 A2A Server。請參閱 [https://github.com/google-a2a/a2a-samples/blob/main/samples/python/agents/birthday_planner_adk/calendar_agent/adk_agent.py](https://github.com/google-a2a/a2a-samples/blob/main/samples/python/agents/birthday_planner_adk/calendar_agent/adk_agent.py)

```python
import datetime

from google.adk.agents import LlmAgent  # type: ignore[import-untyped]
from google.adk.tools.google_api_tool import CalendarToolset  # type: ignore[import-untyped]


async def create_agent(client_id: str, client_secret: str) -> LlmAgent:
    """Constructs the ADK agent."""
    toolset = CalendarToolset(client_id=client_id, client_secret=client_secret)
    return LlmAgent(
        model="gemini-2.0-flash-001",
        name="calendar_agent",
        description="An agent that can help manage a user's calendar",
        instruction=(
            f""" You are an agent that can help manage a user's calendar. Users will request information about the state of their calendar """
            f""" or to make changes to their calendar. Use the provided tools for interacting with the calendar API. """
            f""" If not specified, assume the calendar the user wants is the 'primary' calendar. """
            f""" When using the Calendar API tools, use well-formed RFC3339 timestamps. Today is {datetime.datetime.now()}. """
        ),
        tools=await toolset.get_tools(),
    )
```

這段 Python 程式碼定義了一個非同步函式 `create_agent`，用來建構 ADK LlmAgent。它一開始使用提供的用戶端憑證 初始化 `CalendarToolset`，以存取 Google Calendar API。接著，程式會建立一個 `LlmAgent` 實例，並以指定的 Gemini 模型、描述性名稱，以及管理使用者行事曆的指令 進行設定。此代理會取得來自 `CalendarToolset` 的行事曆工具，使其能與 Calendar API 互動，並回應使用者關於行事曆狀態或修改的查詢。代理的指令 會動態納入目前日期，作為時間上下文。為了說明代理如何建構，讓我們檢視 GitHub 上 A2A samples 中 `calendar_agent` 的關鍵段落。

以下程式碼顯示代理如何以其特定指令 與工具定義。請注意，這裡只顯示說明此功能所需的程式碼；你可以在此存取完整檔案：[https://github.com/a2aproject/a2a-samples/blob/main/samples/python/agents/birthday_planner_adk/calendar_agent/__main__.py](https://github.com/a2aproject/a2a-samples/blob/main/samples/python/agents/birthday_planner_adk/calendar_agent/__main__.py)

```python
def main(host: str = "0.0.0.0", port: int = 8000):
    # Verify an API key is set.
    # Not required if using Vertex AI APIs.
    if os.getenv("GOOGLE_GENAI_USE_VERTEXAI") != "TRUE" and not os.getenv("GOOGLE_API_KEY"):
        raise ValueError(
            "GOOGLE_API_KEY environment variable not set and "
            "GOOGLE_GENAI_USE_VERTEXAI is not TRUE."
        )

    skill = AgentSkill(
        id="check_availability",
        name="Check Availability",
        description="Checks a user's availability for a time using their Google Calendar",
        tags=["calendar"],
        examples=["Am I free from 10am to 11am tomorrow?"],
    )

    agent_card = AgentCard(
        name="Calendar Agent",
        description="An agent that can manage a user's calendar",
        url=f"http://{host}:{port}/",
        version="1.0.0",
        defaultInputModes=["text"],
        defaultOutputModes=["text"],
        capabilities=AgentCapabilities(streaming=True),
        skills=[skill],
    )

    adk_agent = asyncio.run(
        create_agent(
            client_id=os.getenv("GOOGLE_CLIENT_ID"),
            client_secret=os.getenv("GOOGLE_CLIENT_SECRET"),
        )
    )

    runner = Runner(
        app_name=agent_card.name,
        agent=adk_agent,
        artifact_service=InMemoryArtifactService(),
        session_service=InMemorySessionService(),
        memory_service=InMemoryMemoryService(),
    )
    agent_executor = ADKAgentExecutor(runner, agent_card)

    async def handle_auth(request: Request) -> PlainTextResponse:
        await agent_executor.on_auth_callback(
            str(request.query_params.get("state")),
            str(request.url),
        )
        return PlainTextResponse("Authentication successful.")

    request_handler = DefaultRequestHandler(
        agent_executor=agent_executor,
        task_store=InMemoryTaskStore(),
    )

    a2a_app = A2AStarletteApplication(
        agent_card=agent_card,
        http_handler=request_handler,
    )
    routes = a2a_app.routes()
    routes.append(
        Route(
            path="/authenticate",
            methods=["GET"],
            endpoint=handle_auth,
        )
    )
    app = Starlette(routes=routes)

    uvicorn.run(app, host=host, port=port)


if __name__ == "__main__":
    main()
```

這段 Python 程式碼示範如何設定符合 A2A 的「Calendar Agent」，用 Google Calendar 檢查使用者是否有空。它會驗證 API 金鑰或 Vertex AI 設定，以用於驗證。代理的能力（包括「check_availability」技能）定義在 AgentCard 中，而 AgentCard 也會指定代理的網路位址。接著會建立 ADK 代理，並以記憶體內服務設定，用來管理 artifacts、工作階段與記憶。程式碼接著初始化 Starlette Web 應用程式，納入驗證回呼與 A2A 協定處理常式，並使用 Uvicorn 執行，以透過 HTTP 對外提供該代理。

這些範例說明了建置符合 A2A 的代理流程，從定義其能力到將其作為 Web 服務執行。透過使用 Agent Cards 與 ADK，開發者可以建立可互通的 AI 代理，並能與 Google Calendar 等工具整合。這種實務方法展示了 A2A 在建立多代理生態系中的應用。

建議透過 [https://www.trickle.so/blog/how-to-build-google-a2a-project](https://www.trickle.so/blog/how-to-build-google-a2a-project) 的程式碼示範進一步探索 A2A。此連結提供的資源包括 Python 與 JavaScript 的 A2A 用戶端與伺服器、多代理 Web 應用程式、命令列介面，以及各種代理框架的實作範例。

## 快速總覽

**內容：** 個別 AI 代理，尤其是建立在不同框架上的代理，往往難以獨自處理複雜且多面向的問題。主要挑戰在於缺乏共同語言或協定，讓它們能有效通訊與協作。這種孤立狀態阻礙了精密系統的建立；在這類系統中，多個專門化代理可以結合各自獨特技能，解決更大型的任務。若沒有標準化方法，整合這些各自不同的代理會成本高昂、耗時，並阻礙更強大且內聚的 AI 解決方案發展。

**原因：** 代理間通訊（A2A）協定為這個問題提供開放且標準化的解決方案。它是基於 HTTP 的協定，可實現互通性，讓不同 AI 代理無論其底層技術為何，都能順暢地協調、委派任務並分享資訊。核心元件之一是 Agent Card，也就是描述代理能力、技能與通訊端點的數位身分檔案，可促進探索與互動。A2A 定義多種互動機制，包括同步與非同步通訊，以支援多樣化的使用案例。透過為代理協作建立通用標準，A2A 促成一個可用於建構複雜、多代理代理式系統的模組化且可擴充生態系。

**經驗法則：** 當你需要協調兩個或更多 AI 代理之間的協作時，請使用此模式，尤其是在它們使用不同框架建置時（例如 Google ADK、LangGraph、CrewAI）。它非常適合建置複雜且模組化的應用程式，其中專門化代理會處理工作流程中的特定部分，例如將資料分析委派給一個代理，再將報告產生委派給另一個代理。當代理需要動態探索並使用其他代理的能力以完成任務時，此模式也很重要。

**視覺摘要：**

![A2A 代理間通訊模式](../assets/A2A_Inter-Agent_Communication_Pattern.png)

圖 2：A2A 代理間通訊模式

## 重點整理

重點整理：

* Google A2A 協定是一項基於 HTTP 的開放標準，可促進以不同框架建置的 AI 代理之間的通訊與協作。  
* AgentCard 是代理的數位識別碼，可讓其他代理自動探索並理解其能力。  
* A2A 同時提供同步請求／回應 互動（使用 `tasks/send`）與串流更新（使用 `tasks/sendSubscribe`），以滿足不同通訊需求。  
* 此協定支援多輪對話，包括 `input-required` 狀態，讓代理能在互動期間請求其他資訊並維持上下文。  
* A2A 鼓勵模組化架構，讓專門化代理可在不同連接埠上獨立運作，進而支援系統可擴充性與分散式部署。  
* Trickle AI 等工具有助於視覺化並追蹤 A2A 通訊，協助開發者監控、除錯並最佳化多代理系統。  
* 雖然 A2A 是用於管理不同代理之間任務與工作流程的高階協定，Model Context Protocol（MCP）則提供標準化介面，讓 LLM 能與外部資源介接

## 結論

代理間通訊（A2A）協定建立了一項重要的開放標準，用來克服個別 AI 代理固有的孤立性。透過提供共同且基於 HTTP 的框架，它確保以 Google ADK、LangGraph 或 CrewAI 等不同平台建置的代理之間，能順暢協作並具備互通性。核心元件之一是 Agent Card，它作為數位身分，清楚定義代理的能力，並讓其他代理能動態探索。此協定的彈性支援各種互動模式，包括同步請求、非同步輪詢與即時串流，可滿足廣泛的應用需求。

這使得開發者能建立模組化且可擴充的架構，將專門化代理組合起來，協調複雜的自動化工作流程。安全性是基本面向，內建 mTLS 與明確驗證需求等機制，以保護通訊。A2A 雖然與 MCP 等其他標準互補，但其獨特重點在於代理之間的高階協調與任務委派。主要科技公司的強力支持，以及實務實作的可用性，都突顯其日益提升的重要性。此協定為開發者建置更精密、分散式且智慧的多代理系統鋪路。最終，A2A 是促成創新且可互通的協作式 AI 生態系的基礎支柱。

## 參考資料

1. Chen, B. (2025, April 22). *How to Build Your First Google A2A Project: A Step-by-Step Tutorial*. Trickle.so Blog. [https://www.trickle.so/blog/how-to-build-google-a2a-project](https://www.trickle.so/blog/how-to-build-google-a2a-project)
2. Google A2A GitHub Repository. [https://github.com/google-a2a/A2A](https://github.com/google-a2a/A2A)
3. Google Agent Development Kit (ADK) [https://google.github.io/adk-docs/](https://google.github.io/adk-docs/)
4. Getting Started with Agent-to-Agent (A2A) Protocol: [https://codelabs.developers.google.com/intro-a2a-purchasing-concierge\#0](https://codelabs.developers.google.com/intro-a2a-purchasing-concierge#0)
5. Google AgentDiscovery - [https://a2a-protocol.org/latest/](https://a2a-protocol.org/latest/)
6. Communication between different AI frameworks such as LangGraph, CrewAI, and Google ADK [https://www.trickle.so/blog/how-to-build-google-a2a-project](https://www.trickle.so/blog/how-to-build-google-a2a-project#setting-up-your-a2a-development-environment)
7. Designing Collaborative Multi-Agent Systems with the A2A Protocol [https://www.oreilly.com/radar/designing-collaborative-multi-agent-systems-with-the-a2a-protocol/](https://www.oreilly.com/radar/designing-collaborative-multi-agent-systems-with-the-a2a-protocol/)


