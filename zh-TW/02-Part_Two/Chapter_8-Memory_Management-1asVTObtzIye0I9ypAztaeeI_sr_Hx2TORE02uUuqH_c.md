# 第 8 章：記憶管理

有效的記憶管理對代理保留資訊至關重要。代理和人類很像，需要不同類型的記憶才能有效運作。本章將深入探討記憶管理，特別聚焦於代理對即時（短期）記憶與持久（長期）記憶的需求。

在代理系統中，記憶是指代理保留並運用過往互動、觀察與學習經驗中資訊的能力。這項能力讓代理能做出有根據的決策、維持對話上下文，並隨時間改善。代理記憶通常分為兩大類：

* **短期記憶（上下文記憶）：** 類似工作記憶，用來保存目前正在處理或最近存取的資訊。對使用大型語言模型（large language model, LLM）的代理而言，短期記憶主要存在於上下文視窗中。這個視窗包含目前互動中的近期訊息、代理回覆、工具使用結果，以及代理反思，這些內容都會影響 LLM 後續的回應與行動。上下文視窗容量有限，限制了代理能直接存取的近期資訊量。有效的短期記憶管理，意味著要把最相關的資訊保留在這個有限空間中，可能透過摘要較早的對話片段或強調關鍵細節等技術來達成。「長上下文」視窗模型的出現，只是擴大了這種短期記憶的容量，讓單次互動中可以容納更多資訊。然而，這種上下文仍是暫時性的，工作階段結束後就會消失，而且每次處理都可能成本高昂且效率不佳。因此，代理需要不同的記憶類型，才能達成真正的持久性、回想過往互動中的資訊，並建立長久的知識庫。  
* **長期記憶（持久記憶）：** 這類記憶作為代理在不同互動、任務或較長時間內需要保留資訊的儲存庫，類似長期知識庫。資料通常儲存在代理即時處理環境之外，常見於資料庫、知識圖譜或向量資料庫。在向量資料庫中，資訊會轉換成數值向量並儲存，使代理能根據語意相似度而非精確關鍵字比對來擷取資料，這個過程稱為語意搜尋。當代理需要長期記憶中的資訊時，會查詢外部儲存、擷取相關資料，並將其整合到短期上下文中供立即使用，藉此結合既有知識與當前互動。

## 實務應用與使用案例

記憶管理對代理追蹤資訊並隨時間展現智慧行為非常重要。這是讓代理超越基本問答能力的關鍵。應用包括：

* **聊天機器人與對話式 AI：** 維持對話流程仰賴短期記憶。聊天機器人需要記住先前的使用者輸入，才能提供連貫回應。長期記憶則讓聊天機器人能回想使用者偏好、過去問題或先前討論，提供個人化且連續的互動。  
* **任務導向代理：** 管理多步驟任務的代理需要短期記憶來追蹤先前步驟、目前進度與整體目標。這些資訊可能存在於任務的上下文或暫存空間中。長期記憶則對存取不在即時上下文中的特定使用者相關資料至關重要。  
* **個人化體驗：** 提供客製化互動的代理會使用長期記憶來儲存並擷取使用者偏好、過往行為與個人資訊。這讓代理能調整其回應與建議。  
* **學習與改善：** 代理可以透過從過往互動中學習來精進表現。成功策略、錯誤與新資訊會儲存在長期記憶中，協助未來調整。強化學習代理會以這種方式儲存學到的策略或知識。  
* **資訊擷取（RAG）：** 設計用來回答問題的代理會存取知識庫，也就是其長期記憶；這通常實作於檢索增強生成（retrieval-augmented generation, RAG）中。代理會擷取相關文件或資料來支援其回應。  
* **自主系統：** 機器人或自駕車需要地圖、路線、物件位置與已學會行為的記憶。這包含用於即時周遭環境的短期記憶，以及用於一般環境知識的長期記憶。

記憶讓代理能維持歷史紀錄、學習、個人化互動，並管理複雜且與時間相關的問題。

## 實作程式碼：Google Agent Developer Kit (ADK) 中的記憶管理

Google Agent Developer Kit (ADK) 提供一套結構化方法來管理上下文與記憶，並包含可用於實務應用的元件。若要建構需要保留資訊的代理，充分理解 ADK 的 Session、State 與 Memory 十分重要。

就像人類互動一樣，代理也需要能回想先前交流，才能進行連貫且自然的對話。ADK 透過三個核心概念及其相關服務簡化上下文管理。

與代理的每次互動都可以視為一條獨立的對話執行緒。代理可能需要存取較早互動中的資料。ADK 將其結構化如下：

* **Session：** 單一聊天執行緒，會記錄該次互動的訊息與動作（Events），也會儲存與該對話相關的暫存資料（State）。  
* **State (`session.state`)：** 儲存在 Session 內的資料，包含只與目前作用中的聊天執行緒相關的資訊。  
* **Memory：** 來自各種過往聊天或外部來源的資訊所形成的可搜尋儲存庫，可作為擷取即時對話之外資料的資源。

ADK 提供專用服務來管理建構複雜、有狀態且具上下文感知能力的代理所需的重要元件。SessionService 透過處理聊天執行緒（Session 物件）的啟動、記錄與終止來管理它們，而 MemoryService 則負責長期知識（Memory）的儲存與擷取。

SessionService 與 MemoryService 都提供多種設定選項，讓使用者能依應用需求選擇儲存方法。記憶體內選項可用於測試，但資料在重新啟動後不會持久保留。若需要持久儲存與可擴充性，ADK 也支援資料庫與雲端服務。

### Session：追蹤每一次聊天

ADK 中的 Session 物件設計用來追蹤並管理單一聊天執行緒。當與代理開始對話時，SessionService 會產生一個 Session 物件，表示為 `google.adk.sessions.Session`。這個物件封裝與特定對話執行緒相關的所有資料，包括唯一識別碼（`id`、`app\_name`、`user\_id`）、以 Event 物件呈現的時間順序事件紀錄、稱為 state 的工作階段專用暫存資料儲存區，以及表示最後更新時間的時間戳記（`last\_update\_time`）。開發者通常透過 SessionService 間接與 Session 物件互動。SessionService 負責管理對話工作階段的生命週期，包括啟動新工作階段、恢復先前工作階段、記錄工作階段活動（包含狀態更新）、識別作用中的工作階段，以及管理工作階段資料的移除。ADK 提供多種 SessionService 實作，具備不同的工作階段歷史與暫存資料儲存機制，例如 InMemorySessionService；它適合測試，但不會在應用程式重新啟動後提供資料持久性。

```python
# Example: Using InMemorySessionService 
# This is suitable for local development and testing where data 
# persistence across application restarts is not required. 
from google.adk.sessions import InMemorySessionService
session_service = InMemorySessionService()
```

接著，如果你想可靠地儲存到自己管理的資料庫，則可以使用 DatabaseSessionService。

```python
# Example: Using DatabaseSessionService 
# This is suitable for production or development requiring persistent storage. 
# You need to configure a database URL (e.g., for SQLite, PostgreSQL, etc.). 
# Requires: pip install google-adk[sqlalchemy] and a database driver (e.g., psycopg2 for PostgreSQL) 
from google.adk.sessions import DatabaseSessionService 
# Example using a local SQLite file: 
db_url = "sqlite:///./my_agent_data.db"
session_service = DatabaseSessionService(db_url=db_url)
```

此外，還有 VertexAiSessionService，它會使用 Vertex AI 基礎架構，在 Google Cloud 上支援可擴充的正式環境部署。

```python
# Example: Using VertexAiSessionService
# This is suitable for scalable production on Google Cloud Platform, leveraging
# Vertex AI infrastructure for session management.
# Requires: pip install google-adk[vertexai] and GCP setup/authentication

from google.adk.sessions import VertexAiSessionService


PROJECT_ID = "your-gcp-project-id"  # Replace with your GCP project ID
LOCATION = "us-central1"  # Replace with your desired GCP location

# The app_name used with this service should correspond to the Reasoning Engine ID or name
REASONING_ENGINE_APP_NAME = (
    "projects/your-gcp-project-id/locations/us-central1/reasoningEngines/your-engine-id"
)  # Replace with your Reasoning Engine resource name

session_service = VertexAiSessionService(project=PROJECT_ID, location=LOCATION)

# When using this service, pass REASONING_ENGINE_APP_NAME to service methods:
# session_service.create_session(app_name=REASONING_ENGINE_APP_NAME, ...)
# session_service.get_session(app_name=REASONING_ENGINE_APP_NAME, ...)
# session_service.append_event(session, event, app_name=REASONING_ENGINE_APP_NAME)
# session_service.delete_session(app_name=REASONING_ENGINE_APP_NAME, ...)
```

選擇合適的 SessionService 很重要，因為它決定代理互動歷史與暫存資料如何儲存，以及它們是否能持久保留。

每次訊息交換都包含一個循環流程：收到訊息後，Runner 使用 SessionService 擷取或建立 Session；代理使用 Session 的上下文（狀態與歷史互動）處理訊息；代理產生回應且可能更新狀態；Runner 將其封裝成 Event；`session\_service.append\_event` 方法記錄新事件並更新儲存中的狀態。接著 Session 等待下一則訊息。理想情況下，互動結束時會使用 `delete\_session` 方法終止工作階段。這個流程說明 SessionService 如何透過管理 Session 專用的歷史與暫存資料來維持連續性。

### State：Session 的暫存筆記區

在 ADK 中，每個代表聊天執行緒的 Session 都包含一個 state 元件，類似代理在該特定對話期間的暫時工作記憶。雖然 session.events 會記錄完整聊天歷史，但 session.state 會儲存並更新與作用中聊天相關的動態資料點。

基本上，session.state 以字典形式運作，將資料儲存為鍵值對。它的核心功能是讓代理能保留並管理對連貫對話不可或缺的細節，例如使用者偏好、任務進度、逐步收集的資料，或會影響後續代理行動的條件旗標。

state 的結構由字串鍵與可序列化 Python 型別的值組成，包括字串、數字、布林值、列表，以及包含這些基本型別的字典。State 是動態的，會在整個對話中演進。這些變更能否永久保留，取決於設定的 SessionService。

State 組織可使用鍵前綴來定義資料範圍與持久性。沒有前綴的鍵屬於特定工作階段。

* `user:` 前綴會將資料與所有工作階段中的某個使用者 ID 關聯起來。
* `app:` 前綴表示應用程式所有使用者之間共享的資料。
* `temp:` 前綴表示只在目前處理回合有效、且不會持久儲存的資料。

代理會透過單一 session.state 字典存取所有 state 資料。SessionService 會處理資料擷取、合併與持久化。State 應在透過 `session\_service.append\_event()` 將 Event 加入工作階段歷史時更新。這可確保追蹤準確、在持久化服務中正確儲存，並安全處理 state 變更。

#### 1. 簡單方式：使用 `output\_key`（用於代理文字回覆）

如果你只是想把代理的最終文字回應直接存入 state，這是最簡單的方法。設定 LlmAgent 時，只要告訴它你想使用的 output\_key。Runner 會看到這項設定，並在附加事件時自動建立必要動作，將回應儲存到 state。以下來看一個透過 `output\_key` 示範 state 更新的程式碼範例。

```python
# Import necessary classes from the Google Agent Developer Kit (ADK)
from google.adk.agents import LlmAgent
from google.adk.sessions import InMemorySessionService, Session
from google.adk.runners import Runner
from google.genai.types import Content, Part


# Define an LlmAgent with an output_key.
greeting_agent = LlmAgent(
 name="Greeter",
 model="gemini-2.0-flash",
 instruction="Generate a short, friendly greeting.",
 output_key="last_greeting",
)


# --- Setup Runner and Session ---
app_name, user_id, session_id = "state_app", "user1", "session1"

session_service = InMemorySessionService()

runner = Runner(
    agent=greeting_agent,
    app_name=app_name,
    session_service=session_service,
)

session = session_service.create_session(
    app_name=app_name,
    user_id=user_id,
    session_id=session_id,
)

print(f"Initial state: {session.state}")


# --- Run the Agent ---
user_message = Content(parts=[Part(text="Hello")])

print("\n--- Running the agent ---")
for event in runner.run(
    user_id=user_id,
    session_id=session_id,
    new_message=user_message,
):
    if event.is_final_response():
        print("Agent responded.")


# --- Check Updated State ---
# Correctly check the state after the runner has finished processing all events.
updated_session = session_service.get_session(app_name, user_id, session_id)
print(f"\nState after agent run: {updated_session.state}")
```

在幕後，Runner 會看到你的 `output\_key`，並在呼叫 `append\_event` 時，自動建立包含 `state\_delta` 的必要 actions。

#### 2. 標準方式：使用 `EventActions.state\_delta`（用於更複雜的更新）

當你需要執行更複雜的事情時，例如一次更新多個鍵、儲存不只是文字的內容、鎖定特定範圍（如 `user:` 或 `app:`），或進行不綁定代理最終文字回覆的更新，你會手動建立一個包含 state 變更的字典（也就是 `state\_delta`），並把它放入你要附加的 Event 的 EventActions 中。以下來看一個範例：

```python
import time

from google.adk.tools.tool_context import ToolContext
from google.adk.sessions import InMemorySessionService


# --- Define the Recommended Tool-Based Approach ---
def log_user_login(tool_context: ToolContext) -> dict:
    """
    Updates the session state upon a user login event.
    This tool encapsulates all state changes related to a user login.

    Args:
        tool_context: Automatically provided by ADK, gives access to session state.

    Returns:
        A dictionary confirming the action was successful.
    """
    # Access the state directly through the provided context.
    state = tool_context.state

    # Get current values or defaults, then update the state.
    # This is much cleaner and co-locates the logic.
    login_count = state.get("user:login_count", 0) + 1
    state["user:login_count"] = login_count
    state["task_status"] = "active"
    state["user:last_login_ts"] = time.time()
    state["temp:validation_needed"] = True

    print("State updated from within the `log_user_login` tool.")

    return {
        "status": "success",
        "message": f"User login tracked. Total logins: {login_count}.",
    }


# --- Demonstration of Usage ---
# In a real application, an LLM Agent would decide to call this tool.
# Here, we simulate a direct call for demonstration purposes.

# 1. Setup
session_service = InMemorySessionService()
app_name, user_id, session_id = "state_app_tool", "user3", "session3"

session = session_service.create_session(
    app_name=app_name,
    user_id=user_id,
    session_id=session_id,
    state={"user:login_count": 0, "task_status": "idle"},
)

print(f"Initial state: {session.state}")

# 2. Simulate a tool call (in a real app, the ADK Runner does this)
# We create a ToolContext manually just for this standalone example.
from google.adk.tools.tool_context import InvocationContext

mock_context = ToolContext(
    invocation_context=InvocationContext(
        app_name=app_name,
        user_id=user_id,
        session_id=session_id,
        session=session,
        session_service=session_service,
    )
)

# 3. Execute the tool
log_user_login(mock_context)

# 4. Check the updated state
updated_session = session_service.get_session(app_name, user_id, session_id)
print(f"State after tool execution: {updated_session.state}")

# Expected output will show the same state change as the "Before" case,
# but the code organization is significantly cleaner and more robust.
```

這段程式碼示範了在應用程式中管理使用者工作階段狀態的工具式方法。它定義了一個名為 *log\_user\_login* 的函式，作為工具使用。這個工具負責在使用者登入時更新工作階段狀態。  
該函式接收由 ADK 提供的 ToolContext 物件，以存取並修改工作階段的 state 字典。在工具內部，它會遞增 *user:login\_count*、將 *task\_status* 設為 "active"、記錄 *user:last\_login\_ts（timestamp）*，並加入暫時旗標 temp:validation\_needed。

程式碼的示範部分模擬這個工具會如何被使用。它設定一個記憶體內工作階段服務，並以一些預先定義的 state 建立初始工作階段。接著手動建立 ToolContext，以模擬 ADK Runner 執行工具時所在的環境。`log\_user\_login` 函式會使用這個模擬 context 呼叫。最後，程式碼再次擷取工作階段，以顯示 state 已經由工具執行更新。其目標是說明，把 state 變更封裝在工具內，會比在工具外直接操作 state 讓程式碼更乾淨、更有組織。

請注意，強烈不建議在擷取工作階段後直接修改 `session.state` 字典，因為這會繞過標準事件處理機制。這類直接變更不會記錄在工作階段的事件歷史中，可能不會被所選的 `SessionService` 持久化，可能導致並行問題，也不會更新時間戳記等必要中繼資料。更新工作階段 state 的建議方法，是在 `LlmAgent` 上使用 `output\_key` 參數（專門用於代理的最終文字回應），或在透過 `session\_service.append\_event()` 附加事件時，將 state 變更包含在 `EventActions.state\_delta` 中。`session.state` 主要應用於讀取既有資料。

總結來說，設計 state 時請保持簡單、使用基本資料型別、為鍵取清楚名稱並正確使用前綴、避免深層巢狀結構，並一律透過 append\_event 流程更新 state。

## Memory：使用 MemoryService 管理長期知識

在代理系統中，Session 元件會維護單一對話專屬的目前聊天歷史（events）與暫存資料（state）紀錄。然而，若要讓代理在多次互動之間保留資訊或存取外部資料，就需要長期知識管理。這由 MemoryService 提供支援。

```python
# Example: Using InMemoryMemoryService
# This is suitable for local development and testing where data
# persistence across application restarts is not required.
# Memory content is lost when the app stops.

from google.adk.memory import InMemoryMemoryService

memory_service = InMemoryMemoryService()
```

Session 與 State 可以被概念化為單一聊天工作階段的短期記憶，而由 MemoryService 管理的長期知識則作為持久且可搜尋的儲存庫。這個儲存庫可能包含多次過往互動或外部來源的資訊。MemoryService 如 BaseMemoryService 介面所定義，建立了管理這種可搜尋長期知識的標準。它的主要功能包含新增資訊，也就是使用 add\_session\_to\_memory 方法從工作階段擷取內容並儲存；以及擷取資訊，也就是讓代理使用 search\_memory 方法查詢儲存區並取得相關資料。

ADK 提供數種實作來建立這個長期知識儲存區。InMemoryMemoryService 提供適合測試用途的暫時儲存方案，但資料不會在應用程式重新啟動後保留。正式環境通常會使用 VertexAiRagMemoryService。這項服務運用 Google Cloud 的檢索增強生成（Retrieval Augmented Generation, RAG）服務，提供可擴充、持久且具語意搜尋能力的功能（另請參考第 14 章 RAG）。

```python
# Example: Using VertexAiRagMemoryService
# This is suitable for scalable production on GCP, leveraging
# Vertex AI RAG (Retrieval Augmented Generation) for persistent,
# searchable memory.
# Requires: pip install google-adk[vertexai], GCP
# setup/authentication, and a Vertex AI RAG Corpus.

from google.adk.memory import VertexAiRagMemoryService


# The resource name of your Vertex AI RAG Corpus
RAG_CORPUS_RESOURCE_NAME = (
    "projects/your-gcp-project-id/locations/us-central1/ragCorpora/your-corpus-id"
)  # Replace with your Corpus resource name

# Optional configuration for retrieval behavior
SIMILARITY_TOP_K = 5  # Number of top results to retrieve
VECTOR_DISTANCE_THRESHOLD = 0.7  # Threshold for vector similarity

memory_service = VertexAiRagMemoryService(
    rag_corpus=RAG_CORPUS_RESOURCE_NAME,
    similarity_top_k=SIMILARITY_TOP_K,
    vector_distance_threshold=VECTOR_DISTANCE_THRESHOLD,
)

# When using this service, methods like add_session_to_memory
# and search_memory will interact with the specified Vertex AI
# RAG Corpus.
```

## 實作程式碼：LangChain 與 LangGraph 中的記憶管理

在 LangChain 與 LangGraph 中，Memory 是建立智慧且感覺自然的對話式應用程式的關鍵元件。它讓 AI 代理能記住過往互動中的資訊、從回饋中學習，並適應使用者偏好。LangChain 的記憶功能透過參考已儲存的歷史來豐富目前提示，並記錄最新交流供未來使用，為此奠定基礎。隨著代理處理更複雜的任務，這項能力對效率與使用者滿意度都變得不可或缺。

**短期記憶：** 這是執行緒範圍內的記憶，意思是它會追蹤單一工作階段或執行緒中的進行中對話。它提供即時上下文，但完整歷史可能對 LLM 的上下文視窗造成挑戰，進而可能導致錯誤或效能不佳。LangGraph 會將短期記憶作為代理 state 的一部分進行管理，並透過 checkpointer 持久化，讓執行緒能隨時恢復。

**長期記憶：** 這會跨工作階段儲存使用者特定或應用程式層級的資料，並在對話執行緒之間共享。它會儲存在自訂「namespaces」中，並可在任何執行緒中隨時回想。LangGraph 提供 stores 來儲存並回想長期記憶，讓代理能無限期保留知識。

LangChain 提供數種管理對話歷史的工具，從手動控制到 chain 內的自動整合都有。

**ChatMessageHistory：手動記憶管理。** 若要在正式 chain 之外直接且簡單地控制對話歷史，ChatMessageHistory class 是理想選擇。它允許手動追蹤對話交換。

```python
from langchain.memory import ChatMessageHistory


# Initialize the history object
history = ChatMessageHistory()

# Add user and AI messages
history.add_user_message("I'm heading to New York next week.")
history.add_ai_message("Great! It's a fantastic city.")

# Access the list of messages
print(history.messages)
```

**ConversationBufferMemory：Chains 的自動化記憶**。若要將記憶直接整合到 chains 中，ConversationBufferMemory 是常見選擇。它會保存對話緩衝區，並讓你的提示可以使用。它的行為可透過兩個關鍵參數自訂：

* `memory\_key`：指定提示中用來保存聊天歷史的變數名稱的字串。預設值為 "history"。  
* `return\_messages`：決定歷史格式的布林值。  
  * 若為 `False`（預設值），會回傳單一格式化字串，適合標準 LLM。  
  * 若為 `True`，會回傳訊息物件列表，這是 Chat Models 的建議格式。

```python
from langchain.memory import ConversationBufferMemory


# Initialize memory
memory = ConversationBufferMemory()

# Save a conversation turn
memory.save_context(
    {"input": "What's the weather like?"},
    {"output": "It's sunny today."},
)

# Load the memory as a string
print(memory.load_memory_variables({}))
```

將這個 memory 整合到 LLMChain 中，能讓模型存取對話歷史並提供符合上下文的回應

```python
from langchain_openai import OpenAI
from langchain.chains import LLMChain
from langchain.prompts import PromptTemplate
from langchain.memory import ConversationBufferMemory


# 1. Define LLM and Prompt
llm = OpenAI(temperature=0)

template = """You are a helpful travel agent.
Previous conversation: {history}
New question: {question}
Response:"""
prompt = PromptTemplate.from_template(template)

# 2. Configure Memory
# The memory_key "history" matches the variable in the prompt
memory = ConversationBufferMemory(memory_key="history")

# 3. Build the Chain
conversation = LLMChain(llm=llm, prompt=prompt, memory=memory)

# 4. Run the Conversation
response = conversation.predict(question="I want to book a flight.")
print(response)

response = conversation.predict(question="My name is Sam, by the way.")
print(response)

response = conversation.predict(question="What was my name again?")
print(response)
```

若要提升 chat models 的效果，建議設定 \`return\_messages=True\`，使用結構化的訊息物件列表。

```python
from langchain_openai import ChatOpenAI
from langchain.chains import LLMChain
from langchain.memory import ConversationBufferMemory
from langchain_core.prompts import (
    ChatPromptTemplate,
    MessagesPlaceholder,
    SystemMessagePromptTemplate,
    HumanMessagePromptTemplate,
)


# 1. Define Chat Model and Prompt
llm = ChatOpenAI()

prompt = ChatPromptTemplate(
    messages=[
        SystemMessagePromptTemplate.from_template("You are a friendly assistant."),
        MessagesPlaceholder(variable_name="chat_history"),
        HumanMessagePromptTemplate.from_template("{question}"),
    ]
)

# 2. Configure Memory
# return_messages=True is essential for chat models
memory = ConversationBufferMemory(memory_key="chat_history", return_messages=True)

# 3. Build the Chain
conversation = LLMChain(llm=llm, prompt=prompt, memory=memory)

# 4. Run the Conversation
response = conversation.predict(question="Hi, I'm Jane.")
print(response)

response = conversation.predict(question="Do you remember my name?")
print(response)
```

**長期記憶的類型**：長期記憶讓系統能在不同對話之間保留資訊，提供更深層的上下文與個人化。它可分為三種類似人類記憶的類型：

* **語意記憶：記住事實：** 這涉及保留特定事實與概念，例如使用者偏好或領域知識。它用來為代理的回應提供依據，產生更個人化且相關的互動。這類資訊可以作為持續更新的使用者「profile」（JSON 文件）管理，或作為個別事實文件的「collection」管理。  
* **情節記憶：記住經驗：** 這涉及回想過去事件或行動。對 AI 代理而言，情節記憶常用於記住如何完成任務。實務上，它經常透過 few-shot 範例提示實作，代理會從過往成功的互動序列中學習，以正確執行任務。  
* **程序記憶：記住規則：**  這是關於如何執行任務的記憶，也就是代理的核心指令與行為，通常包含在其 system prompt 中。代理修改自身提示以適應與改善，是常見做法。一項有效技術是「反思（Reflection）」，也就是用代理目前的指令與近期互動提示代理，然後要求它精煉自己的指令。

以下是 pseudo-code，示範代理可能如何使用反思來更新其儲存在 LangGraph BaseStore 中的程序記憶

```python
# Node that updates the agent's instructions
def update_instructions(state: State, store: BaseStore):
    namespace = ("instructions",)

    # Get the current instructions from the store
    current_instructions = store.search(namespace)[0]

    # Create a prompt to ask the LLM to reflect on the conversation
    # and generate new, improved instructions
    prompt = prompt_template.format(
        instructions=current_instructions.value["instructions"],
        conversation=state["messages"],
    )

    # Get the new instructions from the LLM
    output = llm.invoke(prompt)
    new_instructions = output["new_instructions"]

    # Save the updated instructions back to the store
    store.put(("agent_instructions",), "agent_a", {"instructions": new_instructions})


# Node that uses the instructions to generate a response
def call_model(state: State, store: BaseStore):
    namespace = ("agent_instructions",)

    # Retrieve the latest instructions from the store
    instructions = store.get(namespace, key="agent_a")[0]

    # Use the retrieved instructions to format the prompt
    prompt = prompt_template.format(
        instructions=instructions.value["instructions"]
    )
    # ... application logic continues
```

LangGraph 會將長期記憶以 JSON 文件形式儲存在 store 中。每個 memory 都會組織在自訂 namespace（像資料夾）和不同 key（像檔名）之下。這種階層式結構讓資訊易於組織與擷取。以下程式碼示範如何使用 InMemoryStore 放入、取得並搜尋 memories。

```python
from langgraph.store.memory import InMemoryStore


# A placeholder for a real embedding function
def embed(texts: list[str]) -> list[list[float]]:
    # In a real application, use a proper embedding model
    return [[1.0, 2.0] for _ in texts]


# Initialize an in-memory store. For production, use a database-backed store.
store = InMemoryStore(index={"embed": embed, "dims": 2})

# Define a namespace for a specific user and application context
user_id = "my-user"
application_context = "chitchat"
namespace = (user_id, application_context)

# 1. Put a memory into the store
store.put(
    namespace,
    "a-memory",  # The key for this memory
    {
        "rules": [
            "User likes short, direct language",
            "User only speaks English & python",
        ],
        "my-key": "my-value",
    },
)

# 2. Get the memory by its namespace and key
item = store.get(namespace, "a-memory")
print("Retrieved Item:", item)

# 3. Search for memories within the namespace, filtering by content
# and sorting by vector similarity to the query.
items = store.search(
    namespace,
    filter={"my-key": "my-value"},
    query="language preferences",
)
print("Search Results:", items)
```

## Vertex Memory Bank

Memory Bank 是 Vertex AI Agent Engine 中的一項受管理服務，為代理提供持久的長期記憶。這項服務使用 Gemini models 非同步分析對話歷史，以擷取關鍵事實與使用者偏好。

這些資訊會被持久儲存，並依照使用者 ID 等定義好的範圍組織，同時會智慧更新，以整合新資料並解決矛盾。開始新工作階段時，代理會透過完整資料回想，或使用 embeddings 進行相似度搜尋，來擷取相關記憶。這個流程讓代理能跨工作階段維持連續性，並根據回想的資訊提供個人化回應。

代理的 runner 會與 VertexAiMemoryBankService 互動，而該服務會先被初始化。這項服務會自動處理代理對話期間產生之 memories 的儲存。每個 memory 都會標記唯一的 USER\_ID 與 APP\_NAME，確保未來能準確擷取。

```python
from google.adk.memory import VertexAiMemoryBankService


agent_engine_id = agent_engine.api_resource.name.split("/")[-1]

memory_service = VertexAiMemoryBankService(
    project="PROJECT_ID",
    location="LOCATION",
    agent_engine_id=agent_engine_id,
)

session = await session_service.get_session(
    app_name=app_name,
    user_id="USER_ID",
    session_id=session.id,
)

await memory_service.add_session_to_memory(session)
```

Memory Bank 與 Google ADK 無縫整合，提供開箱即用的體驗。對於使用其他代理框架的使用者，例如 LangGraph 與 CrewAI，Memory Bank 也透過直接 API 呼叫提供支援。對有興趣的讀者而言，線上已有可示範這些整合的程式碼範例。

## 快速總覽

**是什麼**：代理式系統需要記住過往互動中的資訊，才能執行複雜任務並提供連貫體驗。如果沒有記憶機制，代理就是無狀態的，無法維持對話上下文、從經驗中學習，或為使用者個人化回應。這會從根本上限制代理只能進行簡單的一次性互動，無法處理多步驟流程或持續演變的使用者需求。核心問題是如何有效管理單一對話中即時、暫時的資訊，以及隨時間累積的大量持久知識。

**為什麼：** 標準化解法是實作一套雙元件記憶系統，區分短期與長期儲存。短期的上下文記憶會在 LLM 的上下文視窗中保存近期互動資料，以維持對話流程。對於必須持久保留的資訊，長期記憶解法會使用外部資料庫，通常是向量儲存，以進行有效率的語意擷取。像 Google ADK 這類代理式框架，提供特定元件來管理這一點，例如用於對話執行緒的 Session，以及用於其暫存資料的 State。專用的 MemoryService 則用於與長期知識庫介接，讓代理能擷取相關過往資訊並納入目前上下文。

**經驗法則：** 當代理需要做的不只是回答單一問題時，就使用此模式。對必須在整段對話中維持上下文、追蹤多步驟任務進度，或透過回想使用者偏好與歷史來個人化互動的代理而言，這是必要的。只要預期代理會根據過往成功、失敗或新取得的資訊來學習或適應，就應實作記憶管理。

**視覺摘要：**

![Memory Management Design Pattern](../assets/Memory_Management_Design_Pattern.png)

圖 1：記憶管理設計模式

## 重點整理

以下快速回顧記憶管理的重點：

* 記憶對代理追蹤事項、學習與個人化互動非常重要。  
* 對話式 AI 同時仰賴單一聊天中提供即時上下文的短期記憶，以及跨多個工作階段保留持久知識的長期記憶。  
* 短期記憶（即時資訊）是暫時性的，通常受限於 LLM 的上下文視窗，或框架傳遞上下文的方式。  
* 長期記憶（會保留下來的資訊）會使用向量資料庫等外部儲存，在不同聊天之間保存資訊，並透過搜尋存取。  
* ADK 等框架具有特定元件來管理記憶，例如 Session（聊天執行緒）、State（暫存聊天資料）與 MemoryService（可搜尋的長期知識）。  
* ADK 的 SessionService 會處理聊天工作階段的整個生命週期，包括其歷史（events）與暫存資料（state）。  
* ADK 的 session.state 是暫存聊天資料的字典。前綴（user:、app:、temp:）會指出資料所屬範圍，以及是否會保留下來。  
* 在 ADK 中，新增事件時應使用 EventActions.state\_delta 或 output\_key 來更新 state，而不是直接變更 state 字典。  
* ADK 的 MemoryService 用於將資訊放入長期儲存，並讓代理搜尋它，通常會使用工具。  
* LangChain 提供 ConversationBufferMemory 等實用工具，可自動將單一對話的歷史注入提示，讓代理能回想即時上下文。  
* LangGraph 透過使用 store 在不同使用者工作階段之間儲存並擷取語意事實、情節經驗，甚至可更新的程序規則，支援進階長期記憶。  
* Memory Bank 是一項受管理服務，會自動擷取、儲存並回想使用者特定資訊，為代理提供持久的長期記憶，讓代理能在 Google ADK、LangGraph 與 CrewAI 等框架之間支援個人化且連續的對話。

## 結論

本章深入探討代理系統中非常重要的記憶管理任務，說明短暫上下文與長期保留知識之間的差異。我們討論了這些記憶類型如何設定，以及在建構能記住事情的更智慧代理時會在哪裡使用它們。我們也詳細了解 Google ADK 如何提供 Session、State 與 MemoryService 等特定元件來處理這些需求。現在，我們已經介紹了代理如何記住短期與長期資訊，接下來可以進入它們如何學習與適應。下一個模式「學習與適應」關注的是代理如何根據新經驗或資料，改變其思考方式、行動方式或所掌握的知識。

## 參考資料

1. ADK Memory, [https://google.github.io/adk-docs/sessions/memory/](https://google.github.io/adk-docs/sessions/memory/)
2. LangGraph Memory, [https://langchain-ai.github.io/langgraph/concepts/memory/](https://langchain-ai.github.io/langgraph/concepts/memory/)
3. Vertex AI Agent Engine Memory Bank, [https://cloud.google.com/blog/products/ai-machine-learning/vertex-ai-memory-bank-in-public-preview](https://cloud.google.com/blog/products/ai-machine-learning/vertex-ai-memory-bank-in-public-preview)


