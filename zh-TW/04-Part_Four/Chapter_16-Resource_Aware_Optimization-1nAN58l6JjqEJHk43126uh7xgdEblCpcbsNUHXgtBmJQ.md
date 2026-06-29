# 第 16 章：資源感知最佳化

資源感知最佳化（Resource-Aware Optimization）讓代理（agent）能在運作期間動態監控並管理運算、時間與財務資源。這不同於主要著重於動作排序的單純規劃。資源感知最佳化要求代理針對動作執行做出決策，以便在指定的資源預算內達成目標，或最佳化效率。這包含在較準確但成本較高的模型，以及速度較快、成本較低的模型之間做選擇；或是決定要分配額外運算資源以產生更精細的回應，還是回傳較快但細節較少的答案。

舉例來說，假設有一個代理受指派為金融分析師分析大型資料集。如果分析師立刻需要初步報告，代理可能會使用速度較快、費用較低的模型，快速摘要主要趨勢。然而，如果分析師需要針對關鍵投資決策取得高度準確的預測，且有較高預算與更多時間，代理就會分配更多資源，使用功能強大、速度較慢但更精準的預測模型。此類別中的一項關鍵策略是 fallback 機制；當偏好的模型因過載或受到節流而無法使用時，它會扮演防護措施。為了確保優雅降級，系統會自動切換到預設或成本較低的模型，維持服務連續性，而不是完全失敗。

## 實務應用與使用案例

實務使用案例包括：

* **成本最佳化的 LLM 使用：** 代理根據預算限制，決定要針對複雜任務使用大型且昂貴的 LLM，或針對較簡單的查詢使用較小、較低成本的 LLM。  
* **延遲敏感的作業：** 在即時系統中，代理會選擇速度較快、但可能較不完整的推理路徑，以確保及時回應。  
* **能源效率：** 對部署在邊緣裝置或電力有限環境中的代理，最佳化其處理方式以節省電池續航。  
* **服務可靠性的 fallback：** 當主要選擇無法使用時，代理會自動切換到備援模型，確保服務連續性與優雅降級。  
* **資料使用量管理：** 代理選擇擷取摘要資料，而不是下載完整資料集，以節省頻寬或儲存空間。  
* **自適應任務分配：** 在多代理系統中，代理會根據目前的運算負載或可用時間自行分派任務。

## 實作程式碼範例

用來回答使用者問題的智慧系統，可以評估每個問題的難度。對於簡單查詢，它會使用符合成本效益的語言模型，例如 Gemini Flash。對於複雜問題，則會考慮使用更強大但成本較高的語言模型，例如 Gemini Pro。是否使用更強大的模型，也取決於資源可用性，特別是預算與時間限制。這套系統會動態選擇合適的模型。

例如，考慮一個以階層式代理建構的旅遊規劃器。高階規劃涉及理解使用者的複雜請求、將其拆解為多步驟行程，並做出合乎邏輯的決策；這部分會由像 Gemini Pro 這類更精密且更強大的 LLM 管理。這就是需要深入理解上下文並具備推理能力的「規劃器」代理。

然而，一旦計畫建立完成，該計畫中的個別任務，例如查詢航班價格、確認飯店空房，或尋找餐廳評論，本質上都是簡單且重複的網路查詢。這些「工具函式呼叫」可以由像 Gemini Flash 這類速度較快且成本較低的模型執行。這也更容易看出，為什麼成本較低的模型可用於這些直接的網路搜尋，而複雜的規劃階段則需要更進階模型的較高智慧，以確保旅遊計畫連貫且合乎邏輯。

Google 的 ADK 透過其多代理架構支援這種做法，讓應用程式能以模組化且可擴充的方式建構。不同代理可以處理專門任務。模型彈性讓系統能直接使用各種 Gemini 模型，包括 Gemini Pro 與 Gemini Flash，或透過 LiteLLM 整合其他模型。ADK 的協調能力支援由 LLM 驅動的動態路由，以實現自適應行為。內建評估功能可系統性地評估代理效能，並可用於系統精煉（請參閱「評估與監控」章節）。

接下來，將定義兩個設定相同、但使用不同模型與成本的代理。

```python
# Conceptual Python-like structure, not runnable code
from google.adk.agents import Agent
# from google.adk.models.lite_llm import LiteLlm  # If using models not directly supported by ADK's default Agent

# Agent using the more expensive Gemini Pro 2.5
gemini_pro_agent = Agent(
    name="GeminiProAgent",
    model="gemini-2.5-pro",  # Placeholder for actual model name if different
    description="A highly capable agent for complex queries.",
    instruction="You are an expert assistant for complex problem-solving.",
)

# Agent using the less expensive Gemini Flash 2.5
gemini_flash_agent = Agent(
    name="GeminiFlashAgent",
    model="gemini-2.5-flash",  # Placeholder for actual model name if different
    description="A fast and efficient agent for simple queries.",
    instruction="You are a quick assistant for straightforward questions.",
)
```

路由代理（Router Agent）可以根據查詢長度等簡單指標導向查詢；較短的查詢會送往成本較低的模型，較長的查詢則送往能力較強的模型。然而，更精密的路由代理可以使用 LLM 或 ML 模型來分析查詢的細微差異與複雜度。這個 LLM 路由器可以判斷哪個下游語言模型最適合。例如，要求事實回憶的查詢會路由到 Flash 模型，而需要深度分析的複雜查詢則會路由到 Pro 模型。

最佳化技術可以進一步提升 LLM 路由器的效果。提示調校涉及設計提示，引導路由器 LLM 做出更好的路由決策。以查詢及其最佳模型選擇所組成的資料集微調 LLM 路由器，可以提升其準確性與效率。這種動態路由能力能在回應品質與成本效益之間取得平衡。

```python
# Conceptual Python-like structure, not runnable code
import asyncio
from typing import AsyncGenerator

from google.adk.agents import Agent, BaseAgent
from google.adk.events import Event
from google.adk.agents.invocation_context import InvocationContext


class QueryRouterAgent(BaseAgent):
    name: str = "QueryRouter"
    description: str = "Routes user queries to the appropriate LLM agent based on complexity."

    async def _run_async_impl(self, context: InvocationContext) -> AsyncGenerator[Event, None]:
        user_query = context.current_message.text  # Assuming text input
        query_length = len(user_query.split())  # Simple metric: number of words

        if query_length < 20:  # Example threshold for simplicity vs. complexity
            print(f"Routing to Gemini Flash Agent for short query (length: {query_length})")
            # In a real ADK setup, you would 'transfer_to_agent' or directly invoke
            # For demonstration, we'll simulate a call and yield its response
            response = await gemini_flash_agent.run_async(context.current_message)
            yield Event(author=self.name, content=f"Flash Agent processed: {response}")
        else:
            print(f"Routing to Gemini Pro Agent for long query (length: {query_length})")
            response = await gemini_pro_agent.run_async(context.current_message)
            yield Event(author=self.name, content=f"Pro Agent processed: {response}")
```

評析代理（Critique Agent）會評估語言模型的回應，並提供具備多種功能的回饋。就自我修正而言，它會找出錯誤或不一致之處，促使回答代理精煉輸出以提升品質。它也會系統性評估回應以進行效能監控，追蹤準確性與相關性等指標，並將這些指標用於最佳化。

此外，它的回饋可以作為強化學習或微調的訊號；例如，若持續識別出 Flash 模型回應不足，就能精煉路由代理的邏輯。雖然評析代理不會直接管理預算，但它會透過辨識次佳的路由選擇，間接協助預算管理；例如將簡單查詢導向 Pro 模型，或將複雜查詢導向 Flash 模型，導致結果不佳。這些資訊可用於調整資源配置並節省成本。

評析代理可以設定為只審查回答代理產生的文字，也可以同時審查原始查詢與產生的文字，藉此全面評估回應是否符合最初的問題。

```python
CRITIC_SYSTEM_PROMPT = """
You are the **Critic Agent**, serving as the quality assurance arm of our collaborative research assistant system. Your primary function is to **meticulously review and challenge** information from the Researcher Agent, guaranteeing **accuracy, completeness, and unbiased presentation**. Your duties encompass: * **Assessing research findings** for factual correctness, thoroughness, and potential leanings. * **Identifying any missing data** or inconsistencies in reasoning. * **Raising critical questions** that could refine or expand the current understanding. * **Offering constructive suggestions** for enhancement or exploring different angles. * **Validating that the final output is comprehensive** and balanced. All criticism must be constructive. Your goal is to fortify the research, not invalidate it. Structure your feedback clearly, drawing attention to specific points for revision. Your overarching aim is to ensure the final research product meets the highest possible quality standards. 
"""
```

Critic Agent 會根據預先定義的系統提示運作，該提示概述其角色、職責與回饋方式。為此代理設計良好的提示時，必須清楚確立其作為評估者的功能。它應指定需要批判性聚焦的範圍，並強調提供建設性回饋，而不是單純否定。提示也應鼓勵辨識優點與缺點，並引導代理如何組織與呈現回饋。

## OpenAI 實作程式碼

此系統使用資源感知最佳化策略，有效率地處理使用者查詢。它會先將每個查詢分類為三種類別之一，以判斷最合適且最具成本效益的處理路徑。這種方法能避免將運算資源浪費在簡單請求上，同時確保複雜查詢獲得必要的處理。三種類別如下：

* simple：適用於可直接回答、無需複雜推理或外部資料的直觀問題。  
* reasoning：適用於需要邏輯演繹或多步驟思考流程的查詢，這類查詢會路由到更強大的模型。  
* `internetsearch`：適用於需要最新資訊的問題，會自動觸發 Google Search 以提供最新答案。

程式碼採用 MIT license，並可在 Github 取得：([https://github.com/mahtabsyed/21-Agentic-Patterns/blob/main/16ResourceAwareOptLLMReflectionv2.ipynb](https://github.com/mahtabsyed/21-Agentic-Patterns/blob/main/16_Resource_Aware_Opt_LLM_Reflection_v2.ipynb))

```python
# MIT License
# Copyright (c) 2025 Mahtab Syed
# https://www.linkedin.com/in/mahtabsyed/

import os
import json
import requests
from dotenv import load_dotenv
from openai import OpenAI


# Load environment variables
load_dotenv()

OPENAI_API_KEY = os.getenv("OPENAI_API_KEY")
GOOGLE_CUSTOM_SEARCH_API_KEY = os.getenv("GOOGLE_CUSTOM_SEARCH_API_KEY")
GOOGLE_CSE_ID = os.getenv("GOOGLE_CSE_ID")

if not OPENAI_API_KEY or not GOOGLE_CUSTOM_SEARCH_API_KEY or not GOOGLE_CSE_ID:
    raise ValueError(
        "Please set OPENAI_API_KEY, GOOGLE_CUSTOM_SEARCH_API_KEY, and GOOGLE_CSE_ID in your .env file."
    )

client = OpenAI(api_key=OPENAI_API_KEY)


# --- Step 1: Classify the Prompt ---
def classify_prompt(prompt: str) -> dict:
    system_message = {
        "role": "system",
        "content": (
            "You are a classifier that analyzes user prompts and returns one of three categories ONLY:\n\n"
            "- simple\n"
            "- reasoning\n"
            "- internet_search\n\n"
            "Rules:\n"
            "- Use 'simple' for direct factual questions that need no reasoning or current events.\n"
            "- Use 'reasoning' for logic, math, or multi-step inference questions.\n"
            "- Use 'internet_search' if the prompt refers to current events, recent data, or things not in your training data.\n\n"
            "Respond ONLY with JSON like:\n"
            '{ "classification": "simple" }'
        ),
    }
    user_message = {"role": "user", "content": prompt}

    response = client.chat.completions.create(
        model="gpt-4o",
        messages=[system_message, user_message],
        temperature=1,
    )
    reply = response.choices[0].message.content
    return json.loads(reply)


# --- Step 2: Google Search ---
def google_search(query: str, num_results: int = 1) -> list:
    url = "https://www.googleapis.com/customsearch/v1"
    params = {
        "key": GOOGLE_CUSTOM_SEARCH_API_KEY,
        "cx": GOOGLE_CSE_ID,
        "q": query,
        "num": num_results,
    }
    try:
        response = requests.get(url, params=params)
        response.raise_for_status()
        results = response.json()
        if "items" in results and results["items"]:
            return [
                {
                    "title": item.get("title"),
                    "snippet": item.get("snippet"),
                    "link": item.get("link"),
                }
                for item in results["items"]
            ]
        else:
            return []
    except requests.exceptions.RequestException as e:
        return {"error": str(e)}


# --- Step 3: Generate Response ---
def generate_response(prompt: str, classification: str, search_results=None) -> tuple[str, str]:
    if classification == "simple":
        model = "gpt-4o-mini"
        full_prompt = prompt

    elif classification == "reasoning":
        model = "o4-mini"
        full_prompt = prompt

    elif classification == "internet_search":
        model = "gpt-4o"
        # Convert each search result dict to a readable string
        if search_results:
            search_context = "\n".join(
                [
                    f"Title: {item.get('title')}\nSnippet: {item.get('snippet')}\nLink: {item.get('link')}"
                    for item in search_results
                ]
            )
        else:
            search_context = "No search results found."
        full_prompt = (
            "Use the following web results to answer the user query: "
            f"{search_context}\nQuery: {prompt}"
        )
    else:
        # Fallback
        model = "gpt-4o"
        full_prompt = prompt

    response = client.chat.completions.create(
        model=model,
        messages=[{"role": "user", "content": full_prompt}],
        temperature=1,
    )
    return response.choices[0].message.content, model


# --- Step 4: Combined Router ---
def handle_prompt(prompt: str) -> dict:
    classification_result = classify_prompt(prompt)
    classification = classification_result["classification"]

    search_results = None
    if classification == "internet_search":
        search_results = google_search(prompt)

    answer, model = generate_response(prompt, classification, search_results)
    return {"classification": classification, "response": answer, "model": model}


if __name__ == "__main__":
    test_prompt = "What is the capital of Australia?"
    # test_prompt = "Explain the impact of quantum computing on cryptography."
    # test_prompt = "When does the Australian Open 2026 start, give me full date?"

    result = handle_prompt(test_prompt)

    print("🔍 Classification:", result["classification"])
    print("🧠 Model Used:", result["model"])
    print("🧠 Response:\n", result["response"])
```

這段 Python 程式碼實作了一個提示路由系統，用於回答使用者問題。它會先從 .env 檔案載入 OpenAI 與 Google Custom Search 所需的 API key。核心功能在於將使用者提示分類為三種類別：simple、reasoning 或 internet search。專用函式會使用 OpenAI 模型執行此分類步驟。如果提示需要最新資訊，系統會使用 Google Custom Search API 執行 Google 搜尋。另一個函式接著會根據分類選擇合適的 OpenAI 模型，產生最終回應。對於 internet search 查詢，搜尋結果會作為上下文提供給模型。主要的 `handleprompt` 函式會協調此工作流程，在產生回應前呼叫分類函式與搜尋函式（如有需要）。它會回傳分類、使用的模型，以及產生的答案。這套系統能有效率地將不同類型的查詢導向最佳化的方法，以取得更好的回應。

# 實作程式碼範例（OpenRouter）

OpenRouter 透過單一 API endpoint，提供連接數百種 AI 模型的統一介面。它提供自動容錯移轉與成本最佳化，並能透過你偏好的 SDK 或框架輕鬆整合。

```python
import json
import requests

response = requests.post(
    url="https://openrouter.ai/api/v1/chat/completions",
    headers={
        "Authorization": "Bearer <OPENROUTER_API_KEY>",
        "HTTP-Referer": "<YOUR_SITE_URL>",  # Optional. Site URL for rankings on openrouter.ai.
        "X-Title": "<YOUR_SITE_NAME>",      # Optional. Site title for rankings on openrouter.ai.
    },
    data=json.dumps({
        "model": "openai/gpt-4o",  # Optional
        "messages": [
            {
                "role": "user",
                "content": "What is the meaning of life?"
            }
        ]
    }),
)
```

這段程式碼片段使用 requests library 與 OpenRouter API 互動。它會將使用者訊息以 POST request 傳送到 chat completion endpoint。request 包含帶有 API key 的授權標頭，以及選填的網站資訊。目標是從指定的語言模型取得回應，在此案例中為 "openai/gpt-4o"。

Openrouter 提供兩種不同方法，用於路由並決定處理指定請求時使用的運算模型。

* **自動模型選擇：** 此功能會將請求路由到從精選可用模型集合中選出的最佳化模型。選擇依據是使用者提示的具體內容。最終處理該請求的模型識別碼會在回應的 metadata 中回傳。

```json
{  
    "model": "openrouter/auto",  
    ... // Other params 
}
```

* **循序模型 fallback：** 此機制透過允許使用者指定階層式模型清單，提供作業冗餘。系統會先嘗試使用序列中指定的主要模型處理請求。如果此主要模型因任何錯誤狀況而無法回應，例如服務不可用、速率限制或內容過濾，系統會自動將請求重新路由到序列中的下一個指定模型。此流程會持續進行，直到清單中的某個模型成功執行請求，或清單耗盡為止。作業的最終成本，以及回應中回傳的模型識別碼，都會對應到成功完成運算的模型。

```json
{  
    "models": ["anthropic/claude-3.5-sonnet", "gryphe/mythomax-l2-13b"],  
    ... // Other params }
```

OpenRouter 提供詳細的排行榜（[https://openrouter.ai/rankings](https://openrouter.ai/rankings)），會根據可用 AI 模型的累積 token 產出量進行排名。它也提供來自不同供應商的最新模型（ChatGPT、Gemini、Claude）（見圖 1）。

![OpenRouter Web site](../assets/OpenRouter_Web_Site.png) 

圖 1：OpenRouter Web site（[https://openrouter.ai/](https://openrouter.ai/)）

## 超越動態模型切換：代理資源最佳化的完整光譜

資源感知最佳化對於開發代理系統至關重要，因為這些系統必須在真實世界限制下有效率且有效地運作。以下介紹幾項額外技術：

**動態模型切換**是一項關鍵技術，涉及根據手邊任務的複雜性與可用運算資源，策略性地選擇大型語言模型。面對簡單查詢時，可以部署輕量且具成本效益的 LLM；而複雜、多面向的問題，則需要使用更精密且資源密集的模型。

**自適應工具使用與選擇**確保代理能從一組工具中智慧選擇，針對每個特定子任務挑選最合適且最有效率的工具，同時審慎考量 API 使用成本、延遲與執行時間等因素。這種動態工具選擇會透過最佳化外部 API 與服務的使用，提升整體系統效率。

**上下文修剪與摘要**在管理代理處理的資訊量方面扮演重要角色；它會透過智慧摘要並選擇性保留互動歷史中最相關的資訊，策略性地降低提示 token 數量並減少推論成本，避免不必要的運算開銷。

**主動式資源預測**涉及透過預測未來工作負載與系統需求來預先判斷資源需求，讓系統能主動配置與管理資源，確保系統回應能力並防止瓶頸。

**成本敏感探索**在多代理系統中，會將最佳化考量延伸到傳統運算成本之外，納入通訊成本；這會影響代理協作與分享資訊所採用的策略，目標是將整體資源支出降到最低。

**節能部署**專為資源限制嚴格的環境量身設計，目標是將代理系統的能源足跡降到最低，延長運作時間並降低整體執行成本。

**平行化與分散式運算感知**會運用分散式資源提升代理的處理能力與吞吐量，將運算工作負載分散到多台機器或處理器上，以達成更高效率與更快的任務完成速度。

**學得的資源配置政策**導入學習機制，讓代理能根據回饋與效能指標，隨時間調整並最佳化其資源配置策略，透過持續精煉提升效率。

**優雅降級與 fallback 機制**確保代理系統即使在資源限制嚴重時，仍能繼續運作，雖然可能是以較低容量運作；系統會優雅地降低效能，並退回替代策略以維持運作並提供必要功能。

## 快速概覽

**What：** 資源感知最佳化處理的是在智慧系統中管理運算、時間與財務資源消耗的挑戰。以 LLM 為基礎的應用程式可能成本高且速度慢，而為每項任務選擇最佳模型或工具通常效率不佳。這形成了一項根本取捨：系統輸出品質與產生該輸出所需資源之間的取捨。若沒有動態管理策略，系統就無法適應不同的任務複雜度，也無法在預算與效能限制內運作。

**Why：** 標準化解法是建構一個代理式系統，能根據手邊任務智慧監控並配置資源。此模式通常會先使用「路由代理」分類傳入請求的複雜度。接著，請求會被轉送到最合適的 LLM 或工具：簡單查詢使用快速且低成本的模型，複雜推理則使用更強大的模型。「評析代理」可以進一步透過評估回應品質來精煉流程，提供回饋以隨時間改善路由邏輯。這種動態的多代理方法能確保系統有效率地運作，在回應品質與成本效益之間取得平衡。

**Rule of Thumb：** 當你在嚴格的 API 呼叫或運算能力財務預算下運作、建構快速回應時間至關重要的延遲敏感應用程式、將代理部署到電池續航有限等資源受限的邊緣裝置、以程式化方式平衡回應品質與營運成本之間的取捨，以及管理不同任務具有不同資源需求的複雜多步驟工作流程時，請使用此模式。

**視覺摘要：**

![Resource-Aware Optimization Design Pattern](../assets/Resource_Aware_Optimization_Design_Pattern.png)

圖 2：資源感知最佳化設計模式

## 重點整理

* 資源感知最佳化不可或缺：代理可以動態管理運算、時間與財務資源。關於模型使用與執行路徑的決策，會根據即時限制與目標做出。  
* 可擴充性的多代理架構：Google 的 ADK 提供多代理框架，支援模組化設計。不同代理（回答、路由、評析）負責特定任務。  
* 由 LLM 驅動的動態路由：路由代理會根據查詢複雜度與預算，將查詢導向語言模型（簡單查詢使用 Gemini Flash，複雜查詢使用 Gemini Pro）。這會最佳化成本與效能。  
* 評析代理功能：專用的評析代理會提供用於自我修正、效能監控與精煉路由邏輯的回饋，提升系統有效性。  
* 透過回饋與彈性進行最佳化：用於評析的評估能力，以及模型整合彈性，都有助於自適應且能自我改進的系統行為。  
* 其他資源感知最佳化：其他方法包括自適應工具使用與選擇、上下文修剪與摘要、主動式資源預測、多代理系統中的成本敏感探索、節能部署、平行化與分散式運算感知、學得的資源配置政策、優雅降級與 fallback 機制，以及關鍵任務優先排序。

## 結論

資源感知最佳化是開發代理不可或缺的一環，能讓代理在真實世界限制下有效率地運作。透過管理運算、時間與財務資源，代理可以達成最佳效能與成本效益。動態模型切換、自適應工具使用，以及上下文修剪等技術，對達成這些效率至關重要。學得的資源配置政策與優雅降級等進階策略，能在不同條件下提升代理的適應能力與韌性。將這些最佳化原則整合到代理設計中，是建構可擴充、穩健且永續 AI 系統的基礎。

## 參考資料

1. Google's Agent Development Kit (ADK): [https://google.github.io/adk-docs/](https://google.github.io/adk-docs/)
2. Gemini Flash 2.5 & Gemini 2.5 Pro:  [https://aistudio.google.com/](https://aistudio.google.com/)
3. OpenRouter: [https://openrouter.ai/docs/quickstart](https://openrouter.ai/docs/quickstart)
