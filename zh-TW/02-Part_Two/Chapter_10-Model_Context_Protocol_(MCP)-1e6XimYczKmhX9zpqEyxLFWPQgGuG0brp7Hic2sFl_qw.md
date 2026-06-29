# 第 10 章：Model Context Protocol

若要讓大型語言模型（large language model, LLM）有效作為代理（agent）運作，其能力必須超越多模態生成。它們需要能與外部環境互動，包括存取即時資料、使用外部軟體，以及執行特定操作任務。Model Context Protocol（MCP）透過為 LLM 提供與外部資源介接的標準化介面，回應了這項需求。此協定是促成一致且可預測整合的關鍵機制。

## MCP 模式概觀

想像有一個通用轉接器，能讓任何 LLM 插入任何外部系統、資料庫或工具，而不必為每一個項目建立客製整合。這基本上就是 Model Context Protocol（MCP）。它是一項開放標準，旨在標準化 Gemini、OpenAI 的 GPT 模型、Mixtral 與 Claude 等 LLM 與外部應用程式、資料來源和工具溝通的方式。你可以把它視為一種通用連線機制，簡化 LLM 取得上下文、執行動作，以及與各種系統互動的方式。

MCP 採用 client-server 架構。它定義 MCP server 如何公開不同元素：資料（稱為 resources）、互動式範本（本質上是 prompts），以及可執行動作的函式（稱為 tools）。接著，這些元素會由 MCP client 使用；MCP client 可以是 LLM host application，也可以是 AI 代理本身。這種標準化做法大幅降低了將 LLM 整合到各種操作環境中的複雜度。

然而，MCP 是「代理式介面」的合約，其成效高度取決於它所公開的底層 API 設計。開發者可能只是未經修改地包裝既有的 legacy APIs，這對代理而言可能並非最佳做法。例如，如果票務系統的 API 只允許逐筆擷取完整票券詳細資訊，當代理被要求摘要高優先級票券時，在大量資料下就會變得緩慢且不準確。若要真正有效，底層 API 應加入篩選與排序等確定性功能，協助非確定性的代理有效運作。這也凸顯代理不會神奇地取代確定性工作流程；它們往往需要更強的確定性支援才能成功。

此外，MCP 也可能包裝一個其輸入或輸出仍不一定能被代理理解的 API。只有在資料格式對代理友善時，API 才有用，而 MCP 本身並不強制保證這一點。例如，若為文件儲存建立 MCP server，但回傳的是 PDF 檔案，而使用該資料的代理無法剖析 PDF 內容，這樣的 server 大多沒有實際用途。更好的做法是先建立一個 API，回傳文件的文字版本，例如 Markdown，讓代理能實際讀取並處理。這說明開發者不只要考慮連線本身，也必須考慮交換資料的性質，才能確保真正相容。

## MCP 與工具函式呼叫

Model Context Protocol（MCP）與工具函式呼叫是不同的機制，都能讓 LLM 與外部能力（包括工具）互動並執行動作。雖然兩者都用來將 LLM 的能力延伸到文字生成之外，但其方法與抽象層級並不相同。

工具函式呼叫可以理解為 LLM 對特定、預先定義的工具或函式發出直接請求。請注意，在此脈絡中，我們交替使用「工具」與「函式」兩個詞。這種互動的特徵是一對一的溝通模型：LLM 依據它對使用者意圖的理解，將需要外部動作的需求格式化為請求。接著，應用程式碼執行這個請求，並將結果回傳給 LLM。這個流程通常是專有的，且會因不同 LLM 供應商而異。

相較之下，Model Context Protocol（MCP）作為標準化介面，讓 LLM 能探索、溝通並使用外部能力。它是一項開放協定，促進與各式工具和系統互動，目標是建立一個生態系，讓任何符合規範的工具都能被任何符合規範的 LLM 存取。這促進不同系統與實作之間的互通性、可組合性與可重用性。透過採用 federated model，我們能顯著提升互通性，並釋放既有資產的價值。這項策略讓我們只要以符合 MCP 的介面包裝各種分散且 legacy 的服務，就能將它們帶入現代生態系。這些服務會繼續獨立運作，但現在可以被組合到新的應用程式與工作流程中，並由 LLM 協調其協作。如此可提升敏捷性與可重用性，而不必昂貴地重寫基礎系統。

以下是 MCP 與工具函式呼叫之間根本差異的整理：

| 功能 | 工具函式呼叫 | Model Context Protocol（MCP） |
| ----- | ----- | ----- |
| **標準化** | 專有且與供應商綁定。格式與實作會因 LLM 供應商而異。 | 開放、標準化的協定，促進不同 LLM 與工具之間的互通性。 |
| **範圍** | 讓 LLM 請求執行特定、預先定義函式的直接機制。 | 更廣泛的框架，用於定義 LLM 與外部工具如何彼此探索與溝通。 |
| **架構** | LLM 與應用程式工具處理邏輯之間的一對一互動。 | client-server 架構，讓由 LLM 驅動的應用程式（clients）能連接並使用各種 MCP servers（tools）。 |
| **探索** | LLM 會在特定對話的上下文中被明確告知有哪些工具可用。 | 支援動態探索可用工具。MCP client 可查詢 server，以了解它提供哪些能力。 |
| **可重用性** | 工具整合通常與特定應用程式及所使用的 LLM 緊密耦合。 | 促進可重用、獨立的「MCP servers」開發，任何符合規範的應用程式都可存取。 |

可以把工具函式呼叫想像成為 AI 提供一組特定的客製工具，例如特定扳手與螺絲起子。對任務集合固定的工作坊而言，這很有效率。另一方面，MCP（Model Context Protocol）則像是建立一套通用、標準化的電源插座系統。它本身不提供工具，但能讓任何製造商的任何符合規範工具插入並運作，形成一個動態且持續擴展的工作坊。

簡言之，函式呼叫提供對少數特定函式的直接存取，而 MCP 則是標準化通訊框架，讓 LLM 能探索並使用大量外部資源。對簡單應用程式而言，特定工具已足夠；但對需要適應變化、彼此連結的複雜 AI 系統來說，像 MCP 這樣的通用標準至關重要。

## MCP 的其他考量

雖然 MCP 提供了強大的框架，但若要完整評估它，還必須考量幾個會影響其是否適合特定使用案例的關鍵面向。讓我們更詳細看看其中一些面向：

* **工具 vs. 資源 vs. 提示**：理解這些元件的特定角色很重要。資源是靜態資料（例如 PDF 檔案、資料庫記錄）。工具是執行動作的可執行函式（例如寄送電子郵件、查詢 API）。提示是引導 LLM 如何與資源或工具互動的範本，確保互動具備結構且有效。  
* **可探索性**：MCP 的一項關鍵優勢，是 MCP client 可以動態查詢 server，了解它提供哪些工具與資源。這種「just-in-time」探索機制對需要在不重新部署的情況下適應新能力的代理非常有力。  
* **安全性**：透過任何協定公開工具與資料，都需要強健的安全措施。MCP 實作必須包含驗證與授權，以控制哪些 clients 能存取哪些 servers，以及允許它們執行哪些特定動作。  
* **實作**：雖然 MCP 是開放標準，但其實作可能相當複雜。不過，供應商正開始簡化這個流程。例如，Anthropic 或 FastMCP 等部分模型供應商提供 SDK，抽象化大量樣板程式碼，讓開發者更容易建立並連接 MCP clients 與 servers。  
* **錯誤處理**：完整的錯誤處理策略至關重要。協定必須定義如何將錯誤（例如工具執行失敗、server 無法使用、無效請求）傳回 LLM，讓它能理解失敗原因，並可能嘗試替代做法。  
* **本機 vs. 遠端 Server**：MCP servers 可以部署在與代理相同機器的本機，也可以部署在不同 server 的遠端。若重視速度與敏感資料安全性，可能會選擇本機 server；而遠端 server 架構則能讓組織內常用工具以共享、可擴展的方式存取。  
* **隨需 vs. 批次**：MCP 可同時支援隨需互動式工作階段與較大規模的批次處理。選擇取決於應用程式，可能是需要即時工具存取的即時對話代理，也可能是批次處理記錄的資料分析管線。  
* **傳輸機制**：此協定也定義了通訊用的底層傳輸層。對本機互動而言，它使用透過 STDIO（標準輸入/輸出）的 JSON-RPC，以進行高效率的行程間通訊。對遠端連線而言，它利用 Streamable HTTP 與 Server-Sent Events（SSE）等適合 Web 的協定，實現持續且高效率的 client-server 通訊。

Model Context Protocol 使用 client-server model 來標準化資訊流。理解元件互動，是掌握 MCP 進階代理式行為的關鍵：

1. **大型語言模型（large language model, LLM）**：核心智慧。它處理使用者請求、制定計畫，並決定何時需要存取外部資訊或執行動作。  
2. **MCP Client**：這是圍繞 LLM 的應用程式或包裝器。它扮演中介角色，將 LLM 的意圖轉換為符合 MCP 標準的正式請求。它負責探索、連接並與 MCP Servers 溝通。  
3. **MCP Server**：這是通往外部世界的閘道。它向任何經授權的 MCP Client 公開一組工具、資源與提示。每個 server 通常負責特定領域，例如連接公司內部資料庫、電子郵件服務或 public API。  
4. ​​**選用第三方（3P）服務：** 這代表 MCP Server 管理並公開的實際外部工具、應用程式或資料來源。它是執行所請求動作的最終端點，例如查詢專有資料庫、與 SaaS platform 互動，或呼叫 public weather API。

互動流程如下：

1. **探索**：MCP Client 代表 LLM 查詢 MCP Server，詢問它提供哪些能力。server 會回應一份 manifest，列出可用工具（例如 send_email）、資源（例如 customer_database）與提示。  
2. **請求制定**：LLM 判斷它需要使用已探索到的其中一個工具。例如，它決定要寄送電子郵件。它會制定請求，指定要使用的工具（send_email）與必要參數（recipient、subject、body）。  
3. **Client 通訊**：MCP Client 取得 LLM 制定好的請求，並將其作為標準化呼叫傳送給適當的 MCP Server。  
4. **Server 執行**：MCP Server 接收請求。它會驗證 client、驗證請求，然後透過與底層軟體介接來執行指定動作（例如呼叫電子郵件 API 的 send() function）。  
5. **回應與上下文更新**：執行後，MCP Server 將標準化回應傳回 MCP Client。此回應會指出動作是否成功，並包含任何相關輸出（例如已寄出電子郵件的確認 ID）。client 接著將此結果傳回 LLM，更新其上下文，並讓它能繼續任務的下一步。

## 實務應用與使用案例

MCP 顯著擴展了 AI/LLM 的能力，使其更加多用途且強大。以下是九個關鍵使用案例：

* **資料庫整合：** MCP 讓 LLM 與代理能順暢地存取並互動於資料庫中的結構化資料。例如，使用 MCP Toolbox for Databases，代理可以查詢 Google BigQuery datasets，以擷取即時資訊、產生報告或更新記錄，全都由自然語言命令驅動。  
* **生成式媒體協調：** MCP 讓代理能與進階生成式媒體服務整合。透過 MCP Tools for Genmedia Services，代理可以協調涉及 Google 的 Imagen（影像生成）、Google 的 Veo（影片建立）、Google 的 Chirp 3 HD（逼真語音）或 Google 的 Lyria（音樂作曲）的工作流程，讓 AI 應用程式能動態建立內容。  
* **外部 API 互動：** MCP 提供標準化方式，讓 LLM 呼叫任何外部 API 並接收回應。這表示代理可以擷取即時天氣資料、取得股票價格、寄送電子郵件，或與 CRM systems 互動，將其能力大幅延伸到核心語言模型之外。  
* **基於推理的資訊擷取：** MCP 利用 LLM 強大的推理能力，促成有效且依查詢而定的資訊擷取，超越傳統搜尋與檢索系統。傳統搜尋工具可能回傳整份文件，而代理則可以分析文字，並擷取能直接回答使用者複雜問題的精確條款、數字或陳述。  
* **自訂工具開發：** 開發者可以建置自訂工具，並透過 MCP server（例如使用 FastMCP）公開。這讓專門的內部函式或專有系統能以標準化、易於使用的格式提供給 LLM 與其他代理，而不需要直接修改 LLM。  
* **標準化 LLM 到應用程式通訊：** MCP 確保 LLM 與其互動應用程式之間具備一致的通訊層。這降低整合負擔，促進不同 LLM 供應商與 host applications 之間的互通性，並簡化複雜代理式系統的開發。  
* **複雜工作流程協調：** 透過結合各種由 MCP 公開的工具與資料來源，代理可以協調高度複雜的多步驟工作流程。例如，代理可以從資料庫擷取客戶資料、產生個人化行銷圖片、草擬客製電子郵件，然後寄出，全程與不同 MCP services 互動。  
* **IoT 裝置控制：** MCP 可促進 LLM 與物聯網（Internet of Things, IoT）裝置互動。代理可以使用 MCP 將命令傳送給智慧家電、工業感測器或機器人，使自然語言控制與實體系統自動化成為可能。  
* **金融服務自動化：** 在金融服務中，MCP 可讓 LLM 與各種金融資料來源、交易平台或合規系統互動。代理可以分析市場資料、執行交易、產生個人化財務建議，或自動化法規報告，同時維持安全且標準化的通訊。

簡言之，Model Context Protocol（MCP）讓代理能從資料庫、API 與 Web 資源存取即時資訊。它也讓代理能透過整合並處理各種來源的資料，執行寄送電子郵件、更新記錄、控制裝置，以及執行複雜任務等動作。此外，MCP 支援 AI 應用程式的媒體生成工具。

## 使用 ADK 的實作程式碼範例

本節概述如何連接到提供檔案系統操作的本機 MCP server，讓 ADK 代理能與本機檔案系統互動。

### 使用 MCPToolset 設定代理

若要設定代理與檔案系統互動，必須建立 `agent.py` 檔案（例如位於 `./adk_agent_samples/mcp_agent/agent.py`）。`MCPToolset` 會在 `LlmAgent` 物件的 `tools` list 中具現化。請務必將 `args` list 中的 `"/path/to/your/folder"` 替換為本機系統上一個 MCP server 可存取目錄的絕對路徑。此目錄會成為代理執行檔案系統操作的 root。

```python
import os

from google.adk.agents import LlmAgent
from google.adk.tools.mcp_tool.mcp_toolset import MCPToolset, StdioServerParameters


# Create a reliable absolute path to a folder named 'mcp_managed_files'
# within the same directory as this agent script.
# This ensures the agent works out-of-the-box for demonstration.
# For production, you would point this to a more persistent and secure location.
TARGET_FOLDER_PATH = os.path.join(
    os.path.dirname(os.path.abspath(__file__)),
    "mcp_managed_files",
)

# Ensure the target directory exists before the agent needs it.
os.makedirs(TARGET_FOLDER_PATH, exist_ok=True)

root_agent = LlmAgent(
    model="gemini-2.0-flash",
    name="filesystem_assistant_agent",
    instruction=(
        "Help the user manage their files. You can list files, read files, and write files. "
        f"You are operating in the following directory: {TARGET_FOLDER_PATH}"
    ),
    tools=[
        MCPToolset(
            connection_params=StdioServerParameters(
                command="npx",
                args=[
                    "-y",  # Argument for npx to auto-confirm install
                    "@modelcontextprotocol/server-filesystem",
                    # This MUST be an absolute path to a folder.
                    TARGET_FOLDER_PATH,
                ],
            ),
            # Optional: You can filter which tools from the MCP server are exposed.
            # For example, to only allow reading:
            # tool_filter=['list_directory', 'read_file']
        )
    ],
)
```

`npx`（Node Package Execute）隨 npm（Node Package Manager）5.2.0 及後續版本一併提供，是一項工具，可直接從 npm registry 執行 Node.js packages。這免除了全域安裝的需求。本質上，`npx` 是 npm package runner，常用於執行許多以 Node.js packages 發布的 community MCP servers。

必須建立 `__init__.py` 檔案，以確保 agent.py 檔案被辨識為 Agent Development Kit（ADK）可探索 Python package 的一部分。此檔案應位於與 [agent.py](http://agent.py) 相同的目錄。

```python
# ./adk_agent_samples/mcp_agent/__init__.py 
from . import agent
```

當然，也可以使用其他支援的命令。例如，可透過以下方式連接到 python3：

```python
connection_params = StdioConnectionParams(
    server_params={
        "command": "python3",
        "args": ["./agent/mcp_server.py"],
        "env": {
            "SERVICE_ACCOUNT_PATH": SERVICE_ACCOUNT_PATH,
            "DRIVE_FOLDER_ID": DRIVE_FOLDER_ID,
        },
    }
)
```

在 Python 脈絡中，UVX 是指一種命令列工具，會利用 uv 在暫時、隔離的 Python 環境中執行命令。基本上，它讓你可以執行 Python tools 與 packages，而不必在全域或專案環境中安裝它們。你可以透過 MCP server 執行它。

```python
connection_params = StdioConnectionParams(
    server_params={
        "command": "uvx",
        "args": ["mcp-google-sheets@latest"],
        "env": {
            "SERVICE_ACCOUNT_PATH": SERVICE_ACCOUNT_PATH,
            "DRIVE_FOLDER_ID": DRIVE_FOLDER_ID,
        },
    }
)
```

建立 MCP Server 後，下一步是連接到它。

## 使用 ADK Web 連接 MCP Server

首先，執行 'adk web'。在終端機中前往 mcp_agent 的父目錄（例如 adk_agent_samples），並執行：

```python
cd ./adk_agent_samples # Or your equivalent parent directory 
adk web
```

ADK Web UI 在瀏覽器中載入後，從代理選單中選取 `filesystem_assistant_agent`。接著，嘗試下列提示：

* 「顯示這個資料夾的內容。」  
* 「讀取 `sample.txt` 檔案。」（這裡假設 `sample.txt` 位於 `TARGET_FOLDER_PATH`。）  
* 「`another_file.md` 裡有什麼？」

## 使用 FastMCP 建立 MCP Server

FastMCP 是高階 Python framework，旨在簡化 MCP servers 的開發。它提供抽象層來簡化協定複雜度，讓開發者能專注於核心邏輯。

此 library 可透過簡單的 Python decorators 快速定義工具、資源與提示。其一大優勢是自動 schema 生成，能智慧解讀 Python function signatures、type hints 與 documentation strings，以建構必要的 AI model interface specifications。這種自動化可將手動設定降至最低，並減少人為錯誤。

除了基本工具建立之外，FastMCP 也支援 server composition 與 proxying 等進階架構模式。這使複雜、多元件系統的模組化開發，以及將既有服務無縫整合進 AI 可存取框架成為可能。此外，FastMCP 也包含針對高效率、分散式與可擴展 AI-driven applications 的最佳化。

## 使用 FastMCP 設定 Server

## 為了說明，請考慮 server 提供的一個基本「greet」工具。啟用後，ADK 代理與其他 MCP clients 可以使用 HTTP 與此工具互動

```python
# fastmcp_server.py
# This script demonstrates how to create a simple MCP server using FastMCP.
# It exposes a single tool that generates a greeting.
# 1. Make sure you have FastMCP installed:
# pip install fastmcp

from fastmcp import FastMCP, Client


# Initialize the FastMCP server.
mcp_server = FastMCP()


# Define a simple tool function.
# The `@mcp_server.tool` decorator registers this Python function as an MCP tool.
# The docstring becomes the tool's description for the LLM.
@mcp_server.tool
def greet(name: str) -> str:
    """
    Generates a personalized greeting.

    Args:
        name: The name of the person to greet.

    Returns:
        A greeting string.
    """
    return f"Hello, {name}! Nice to meet you."


# Or if you want to run it from the script:
if __name__ == "__main__":
    mcp_server.run(
        transport="http",
        host="127.0.0.1",
        port=8000,
    )
```

這個 Python script 定義了一個名為 greet 的單一函式，該函式接受一個人的姓名並回傳個人化問候語。此函式上方的 @tool() decorator 會自動將它註冊為 AI 或其他程式可使用的工具。FastMCP 會使用該函式的 documentation string 與 type hints，告訴代理此工具如何運作、需要哪些輸入，以及會回傳什麼。

執行此 script 時，它會啟動 FastMCP server，並在 localhost:8000 監聽請求。這會讓 greet 函式成為可用的 network service。之後即可設定代理連接到這個 server，並使用 greet 工具作為較大型任務的一部分來產生問候語。server 會持續執行，直到手動停止為止。

## 透過 ADK 代理使用 FastMCP Server

ADK 代理可以設定為 MCP client，以使用正在執行的 FastMCP server。這需要使用 FastMCP server 的網路位址來設定 HttpServerParameters，通常是 <http://localhost:8000>。

可以包含 `tool_filter` 參數，將代理的工具使用限制在 server 提供的特定工具，例如 'greet'。當收到像「Greet John Doe」這樣的請求時，代理內嵌的 LLM 會辨識出透過 MCP 可用的 'greet' 工具，以引數 "John Doe" 呼叫它，並回傳 server 的回應。此流程示範了將透過 MCP 公開的使用者定義工具與 ADK 代理整合。

若要建立此設定，需要一個代理檔案（例如位於 ./adk_agent_samples/fastmcp_client_agent/ 的 agent.py）。此檔案會具現化 ADK 代理，並使用 HttpServerParameters 與運作中的 FastMCP server 建立連線。

```python
# ./adk_agent_samples/fastmcp_client_agent/agent.py
import os

from google.adk.agents import LlmAgent
from google.adk.tools.mcp_tool.mcp_toolset import MCPToolset, HttpServerParameters


# Define the FastMCP server's address.
# Make sure your fastmcp_server.py (defined previously) is running on this port.
FASTMCP_SERVER_URL = "http://localhost:8000"

root_agent = LlmAgent(
    model="gemini-2.0-flash",  # Or your preferred model
    name="fastmcp_greeter_agent",
    instruction='You are a friendly assistant that can greet people by their name. Use the "greet" tool.',
    tools=[
        MCPToolset(
            connection_params=HttpServerParameters(
                url=FASTMCP_SERVER_URL,
            ),
            # Optional: Filter which tools from the MCP server are exposed
            # For this example, we're expecting only 'greet'
            tool_filter=["greet"],
        )
    ],
)
```

此 script 定義了一個名為 `fastmcp_greeter_agent` 的代理，使用 Gemini language model。它被賦予特定 instruction，要扮演一個能依姓名問候他人的友善助理。關鍵在於，程式碼為此代理配備了一個工具來執行任務。它設定 MCPToolset 連接到在 localhost:8000 上執行的獨立 server，預期該 server 是前一個範例中的 FastMCP server。此代理被明確授予存取該 server 上 greet 工具的權限。本質上，這段程式碼設定了系統的 client 端，建立一個理解自身目標是問候他人，且確切知道該使用哪個外部工具來完成目標的智慧代理。

必須在 `fastmcp_client_agent` 目錄中建立 `__init__.py` 檔案。這可確保該代理被 ADK 辨識為可探索的 Python package。

首先，開啟新的終端機並執行 `python fastmcp_server.py`，以啟動 FastMCP server。接著，在終端機中前往 `fastmcp_client_agent` 的父目錄（例如 `adk_agent_samples`），並執行 `adk web`。ADK Web UI 在瀏覽器中載入後，從代理選單選取 `fastmcp_greeter_agent`。接著你可以輸入像「Greet John Doe」這樣的提示來測試它。代理會使用 FastMCP server 上的 `greet` 工具來建立回應。

## 重點概覽

**What：** 若要作為有效的代理運作，LLM 必須超越單純文字生成。它們需要能與外部環境互動，以存取即時資料並使用外部軟體。若沒有標準化的通訊方法，LLM 與外部工具或資料來源之間的每一項整合都會變成客製、複雜且無法重用的工作。這種臨時做法會阻礙擴展性，並使建置複雜、彼此連結的 AI 系統變得困難且低效。

**Why：** Model Context Protocol（MCP）透過作為 LLM 與外部系統之間的通用介面，提供標準化解決方案。它建立一項開放、標準化的協定，定義外部能力如何被探索與使用。MCP 採用 client-server model，允許 servers 向任何符合規範的 client 公開工具、資料資源與互動式提示。由 LLM 驅動的應用程式會扮演這些 clients，以可預測的方式動態探索並與可用資源互動。這種標準化做法促成由可互通、可重用元件組成的生態系，大幅簡化複雜代理式工作流程的開發。

**經驗法則：** 當你建置複雜、可擴展或企業級代理式系統，且系統需要與多樣且不斷演進的外部工具、資料來源與 API 互動時，請使用 Model Context Protocol（MCP）。當不同 LLM 與工具之間的互通性是優先事項，且代理需要能在不重新部署的情況下動態探索新能力時，它非常適合。對於只有固定且有限數量預先定義函式的較簡單應用程式，直接工具函式呼叫可能就已足夠。

**視覺摘要：**

![Model Context Protocol](../assets/Model_Context_Protocol.png)

圖 1：Model Context Protocol

## 關鍵重點

以下是關鍵重點：

* Model Context Protocol（MCP）是一項開放標準，促進 LLM 與外部應用程式、資料來源和工具之間的標準化通訊。  
* 它採用 client-server 架構，定義公開與使用資源、提示和工具的方法。  
* Agent Development Kit（ADK）同時支援使用既有 MCP servers，以及透過 MCP server 公開 ADK tools。  
* FastMCP 簡化 MCP servers 的開發與管理，特別適合公開以 Python 實作的工具。  
* MCP Tools for Genmedia Services 讓代理能與 Google Cloud 的生成式媒體能力（Imagen、Veo、Chirp 3 HD、Lyria）整合。  
* MCP 讓 LLM 與代理能與真實世界系統互動、存取動態資訊，並執行文字生成之外的動作。

## 結論

Model Context Protocol（MCP）是一項開放標準，促進大型語言模型（large language models, LLMs）與外部系統之間的通訊。它採用 client-server 架構，讓 LLM 能透過標準化工具存取資源、使用提示並執行動作。MCP 讓 LLM 能與資料庫互動、管理生成式媒體工作流程、控制 IoT 裝置，以及自動化金融服務。實務範例示範如何設定代理與 MCP servers 溝通，包括 filesystem servers 與使用 FastMCP 建置的 servers，說明它與 Agent Development Kit（ADK）的整合。MCP 是開發互動式 AI 代理的關鍵元件，能將能力延伸到基本語言能力之外。

## 參考資料

1. Model Context Protocol (MCP) Documentation. (Latest). *Model Context Protocol (MCP)*. [https://google.github.io/adk-docs/mcp/](https://google.github.io/adk-docs/mcp/)  
2. FastMCP Documentation. FastMCP. [https://github.com/jlowin/fastmcp](https://github.com/jlowin/fastmcp)  
3. MCP Tools for Genmedia Services. *MCP Tools for Genmedia Services*. [https://google.github.io/adk-docs/mcp/\#mcp-servers-for-google-cloud-genmedia](https://google.github.io/adk-docs/mcp/#mcp-servers-for-google-cloud-genmedia)  
4. MCP Toolbox for Databases Documentation. (Latest). *MCP Toolbox for Databases*. [https://google.github.io/adk-docs/mcp/databases/](https://google.github.io/adk-docs/mcp/databases/)
