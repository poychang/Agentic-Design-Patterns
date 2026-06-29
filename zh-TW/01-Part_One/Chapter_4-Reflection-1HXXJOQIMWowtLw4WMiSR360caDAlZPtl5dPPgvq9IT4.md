# 第 4 章：反思

## 反思模式概觀

在前幾章中，我們探討了幾種基礎的代理式模式：用於循序執行的 Chaining、用於動態路徑選擇的 Routing，以及用於並行任務執行的 Parallelization。這些模式讓代理（agent）能更有效率、更有彈性地執行複雜任務。然而，即使有精密的工作流程，代理的初始輸出或計畫仍可能不是最佳、不夠準確，或不夠完整。這正是**反思（Reflection）**模式發揮作用的地方。

反思模式是指代理評估自己的工作、輸出或內部狀態，並利用該評估來改善效能或精修回應。這是一種自我修正或自我改進的形式，讓代理能根據回饋、內部批判，或與期望標準的比較，反覆精修輸出或調整做法。反思有時也可由另一個獨立代理協助完成，該代理的特定角色就是分析初始代理的輸出。

不同於只是把輸出直接傳給下一步的簡單循序鏈，也不同於選擇路徑的路由，反思導入了回饋迴路。代理不只是產生輸出；它接著會檢視該輸出（或產生該輸出的過程），找出潛在問題或可改善之處，並利用這些洞察產生更好的版本，或修改未來的行動。

這個流程通常包含：

1. **執行：** 代理執行任務或產生初始輸出。  
2. **評估／批判：** 代理（通常使用另一次大型語言模型（large language model, LLM）呼叫或一組規則）分析前一步的結果。這項評估可能會檢查事實準確性、連貫性、風格、完整性、是否遵循指示，或其他相關標準。  
3. **反思／精修：** 根據批判結果，代理判斷如何改善。這可能包括產生精修後的輸出、調整後續步驟的參數，甚至修改整體計畫。  
4. **迭代（選用但常見）：** 接著可以執行精修後的輸出或調整後的做法，並重複反思流程，直到達成令人滿意的結果或符合停止條件。

反思模式的一種關鍵且非常有效的實作方式，是把流程拆成兩個不同的邏輯角色：Producer 和 Critic。這通常稱為「Generator-Critic」或「Producer-Reviewer」模型。雖然單一代理也能執行自我反思，但使用兩個專門代理（或兩次具備不同系統提示的獨立 LLM 呼叫）通常能得到更穩健且較不偏頗的結果。

1. Producer Agent：此代理的主要職責是執行任務的初始作業。它完全專注於產生內容，不論是撰寫程式碼、起草部落格文章，或建立計畫。它接收初始提示，並產生第一版輸出。

2. Critic Agent：此代理的唯一目的，是評估 Producer 產生的輸出。它會收到另一組不同的指示，通常也會有不同的人設（例如：「You are a senior software engineer」、「You are a meticulous fact-checker」）。Critic 的指示會引導它依據特定標準分析 Producer 的工作，例如事實準確性、程式碼品質、風格要求或完整性。它的設計目的是找出缺陷、提出改善建議，並提供結構化回饋。

這種關注點分離很有力，因為它能避免代理審查自己工作時的「認知偏誤」。Critic 代理會以全新視角看待輸出，並且專注於找出錯誤與可改善之處。Critic 的回饋接著會傳回 Producer 代理，Producer 會把它作為指引，產生新的精修版輸出。提供的 LangChain 和 ADK 程式碼範例都實作了這種雙代理模型：LangChain 範例使用特定的 `reflector_prompt` 來建立批判者人設，而 ADK 範例則明確定義了 producer 與 reviewer 代理。

實作反思通常需要設計代理的工作流程，使其包含這些回饋迴路。這可以透過程式碼中的迭代迴圈來達成，也可以使用支援狀態管理與依據評估結果進行條件轉換的框架來完成。雖然可以在 LangChain/LangGraph、ADK 或 Crew.AI 鏈中實作單一步驟的評估與精修，但真正的迭代式反思通常涉及更複雜的協調。

反思模式對於建立能產生高品質輸出、處理細緻任務，並展現某種程度自我覺察與適應能力的代理至關重要。它讓代理不再只是單純執行指示，而是走向更精密的問題解決與內容生成形式。

值得注意的是，反思與目標設定和監控（見第 11 章）之間有所交集。目標為代理的自我評估提供最終基準，而監控則追蹤其進展。在許多實務案例中，反思可能會扮演修正引擎的角色，利用監控得到的回饋來分析偏差並調整策略。這種綜效會把代理從被動執行者，轉變為能自適應地努力達成目標的有目的系統。

此外，當 LLM 保留對話記憶時（見第 8 章），反思模式的效果會顯著提升。這些對話歷史為評估階段提供關鍵上下文，讓代理不只是孤立地評估自己的輸出，而是能放在先前互動、使用者回饋與演進中的目標脈絡下進行評估。這使代理能從過去的批判中學習，避免重複犯錯。沒有記憶時，每次反思都是自成一格的事件；有了記憶，反思就成為累積式流程，每個循環都建立在前一輪之上，帶來更智慧且更具上下文感知能力的精修。

## 實務應用與使用案例

在輸出品質、準確性，或遵循複雜限制條件非常重要的情境中，反思模式很有價值：

### 1. 創意寫作與內容生成

精修生成的文字、故事、詩作或行銷文案。

* **使用案例：** 代理撰寫部落格文章。  
  * **反思：** 產生草稿，針對流暢度、語氣與清晰度進行批判，再根據批判重寫。重複此流程直到文章符合品質標準。  
  * **效益：** 產出更精緻且更有效的內容。

### 2. 程式碼生成與除錯

撰寫程式碼、識別錯誤並修正錯誤。

* **使用案例：** 代理撰寫 Python 函式。  
  * **反思：** 撰寫初始程式碼、執行測試或靜態分析、找出錯誤或低效率之處，然後根據發現修改程式碼。  
  * **效益：** 生成更穩健且可運作的程式碼。

### 3. 複雜問題解決

在多步驟推理任務中評估中間步驟或提出的解法。

* **使用案例：** 代理解邏輯謎題。  
  * **反思：** 提出一個步驟，評估它是否讓解法更接近目標或引入矛盾；必要時回溯或選擇不同步驟。  
  * **效益：** 改善代理在複雜問題空間中前進的能力。

### 4. 摘要與資訊整合

精修摘要，使其更準確、完整且簡潔。

* **使用案例：** 代理摘要一份長文件。  
  * **反思：** 產生初始摘要，將其與原始文件中的重點比較，並精修摘要以納入遺漏資訊或提升準確性。  
  * **效益：** 建立更準確且更完整的摘要。

### 5. 規劃與策略

評估提出的計畫，並找出潛在缺陷或改善之處。

* **使用案例：** 代理規劃一系列行動以達成目標。  
  * **反思：** 產生計畫，模擬其執行，或依限制條件評估其可行性，再根據評估修訂計畫。  
  * **效益：** 發展出更有效且更實際的計畫。

### 6. 對話式代理

檢視對話中的前幾輪內容，以維持上下文、修正誤解，或提升回應品質。

* **使用案例：** 客服聊天機器人。  
  * **反思：** 在使用者回應後，檢視對話歷史與上一則生成訊息，以確保連貫性，並準確處理使用者最新輸入。  
  * **效益：** 帶來更自然且更有效的對話。

反思為代理式系統加入一層後設認知，使其能從自己的輸出與流程中學習，進而產生更智慧、可靠且高品質的結果。

## 實作程式碼範例（LangChain）

實作完整的迭代式反思流程，需要狀態管理與循環執行機制。雖然這些能力可由 LangGraph 這類圖形架構框架原生處理，或透過自訂程序式程式碼完成，但使用 LCEL（LangChain Expression Language）的組合語法，也能有效示範單一反思循環的基本原理。

這個範例使用 Langchain 函式庫與 OpenAI 的 GPT-4o 模型實作反思迴圈，反覆產生並精修一個用來計算數字階乘的 Python 函式。流程從任務提示開始，先產生初始程式碼，接著根據模擬資深軟體工程師角色提出的批判，反覆對程式碼進行反思，並在每次迭代中精修程式碼，直到批判階段判定程式碼完美，或達到最大迭代次數。最後，它會印出精修後的結果程式碼。

首先，請確認已安裝必要的函式庫：

```bash
pip install langchain langchain-community langchain-openai
```

你也需要在環境中設定所選語言模型的 API 金鑰（例如 OpenAI、Google Gemini、Anthropic）。

```python
import os
from dotenv import load_dotenv
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.messages import SystemMessage, HumanMessage


# --- Configuration ---
# Load environment variables from .env file (for OPENAI_API_KEY)
load_dotenv()

# Check if the API key is set
if not os.getenv("OPENAI_API_KEY"):
    raise ValueError("OPENAI_API_KEY not found in .env file. Please add it.")

# Initialize the Chat LLM. We use gpt-4o for better reasoning.
# A lower temperature is used for more deterministic outputs.
llm = ChatOpenAI(model="gpt-4o", temperature=0.1)


def run_reflection_loop():
    """
    Demonstrates a multi-step AI reflection loop to progressively improve a Python function.
    """
    # --- The Core Task ---
    task_prompt = """
    Your task is to create a Python function named `calculate_factorial`.
    This function should do the following:
    1.  Accept a single integer `n` as input.
    2.  Calculate its factorial (n!).
    3.  Include a clear docstring explaining what the function does.
    4.  Handle edge cases: The factorial of 0 is 1.
    5.  Handle invalid input: Raise a ValueError if the input is a negative number.
    """

    # --- The Reflection Loop ---
    max_iterations = 3
    current_code = ""

    # We will build a conversation history to provide context in each step.
    message_history = [HumanMessage(content=task_prompt)]

    for i in range(max_iterations):
        print("\n" + "=" * 25 + f" REFLECTION LOOP: ITERATION {i + 1} " + "=" * 25)

        # --- 1. GENERATE / REFINE STAGE ---
        # In the first iteration, it generates. In subsequent iterations, it refines.
        if i == 0:
            print("\n>>> STAGE 1: GENERATING initial code...")
            # The first message is just the task prompt.
            response = llm.invoke(message_history)
            current_code = response.content
        else:
            print("\n>>> STAGE 1: REFINING code based on previous critique...")
            # The message history now contains the task,
            # the last code, and the last critique.
            # We instruct the model to apply the critiques.
            message_history.append(HumanMessage(content="Please refine the code using the critiques provided."))
            response = llm.invoke(message_history)
            current_code = response.content

        print("\n--- Generated Code (v" + str(i + 1) + ") ---\n" + current_code)
        message_history.append(response)  # Add the generated code to history

        # --- 2. REFLECT STAGE ---
        print("\n>>> STAGE 2: REFLECTING on the generated code...")
        # Create a specific prompt for the reflector agent.
        # This asks the model to act as a senior code reviewer.
        reflector_prompt = [
            SystemMessage(content="""
                You are a senior software engineer and an expert
                in Python.
                Your role is to perform a meticulous code review.
                Critically evaluate the provided Python code based
                on the original task requirements.
                Look for bugs, style issues, missing edge cases,
                and areas for improvement.
                If the code is perfect and meets all requirements,
                respond with the single phrase 'CODE_IS_PERFECT'.
                Otherwise, provide a bulleted list of your critiques.
            """),
            HumanMessage(content=f"Original Task:\n{task_prompt}\n\nCode to Review:\n{current_code}"),
        ]

        critique_response = llm.invoke(reflector_prompt)
        critique = critique_response.content

        # --- 3. STOPPING CONDITION ---
        if "CODE_IS_PERFECT" in critique:
            print("\n--- Critique ---\nNo further critiques found. The code is satisfactory.")
            break

        print("\n--- Critique ---\n" + critique)
        # Add the critique to the history for the next refinement loop.
        message_history.append(HumanMessage(content=f"Critique of the previous code:\n{critique}"))

    print("\n" + "=" * 30 + " FINAL RESULT " + "=" * 30)
    print("\nFinal refined code after the reflection process:\n")
    print(current_code)


if __name__ == "__main__":
    run_reflection_loop()
```

這段程式碼一開始會設定環境、載入 API 金鑰，並以較低的 temperature 初始化 GPT-4o 這類強大的語言模型，以產生更聚焦的輸出。核心任務由一段提示定義，要求建立一個用來計算數字階乘的 Python 函式，並包含 docstring、邊界案例（0 的階乘），以及負數輸入的錯誤處理等具體要求。`run_reflection_loop` 函式會協調迭代式精修流程。在迴圈中，第一次迭代時，語言模型會根據任務提示產生初始程式碼。在後續迭代中，它會根據前一步的批判精修程式碼。另一個獨立的「reflector」角色同樣由語言模型扮演，但使用不同的系統提示，並以資深軟體工程師身分根據原始任務要求批判生成的程式碼。這項批判會以條列問題清單呈現；若未發現問題，則回傳 `CODE_IS_PERFECT`。迴圈會持續進行，直到批判指出程式碼已經完美，或達到最大迭代次數。每一步都會維持對話歷史並傳給語言模型，為生成／精修與反思階段提供上下文。最後，腳本會在迴圈結束後印出最後生成的程式碼版本。

## 實作程式碼範例（ADK）

現在來看一個使用 Google ADK 實作的概念性程式碼範例。具體而言，這段程式碼透過 Generator-Critic 結構展現反思：一個元件（Generator）產生初始結果或計畫，另一個元件（Critic）提供關鍵回饋或批判，引導 Generator 產生更精修或更準確的最終輸出。

```python
from google.adk.agents import SequentialAgent, LlmAgent


# The first agent generates the initial draft.
generator = LlmAgent(
    name="DraftWriter",
    description="Generates initial draft content on a given subject.",
    instruction="Write a short, informative paragraph about the user's subject.",
    output_key="draft_text",  # The output is saved to this state key.
)

# The second agent critiques the draft from the first agent.
reviewer = LlmAgent(
    name="FactChecker",
    description="Reviews a given text for factual accuracy and provides a structured critique.",
    instruction="""
    You are a meticulous fact-checker.
    1. Read the text provided in the state key 'draft_text'.
    2. Carefully verify the factual accuracy of all claims.
    3. Your final output must be a dictionary containing two keys:
       - "status": A string, either "ACCURATE" or "INACCURATE".
       - "reasoning": A string providing a clear explanation for your status, citing specific issues if any are found.
    """,
    output_key="review_output",  # The structured dictionary is saved here.
)

# The SequentialAgent ensures the generator runs before the reviewer.
review_pipeline = SequentialAgent(
    name="WriteAndReview_Pipeline",
    sub_agents=[generator, reviewer],
)

# Execution Flow:
# 1. generator runs -> saves its paragraph to state['draft_text'].
# 2. reviewer runs -> reads state['draft_text'] and saves its dictionary output to state['review_output'].
```

這段程式碼示範如何在 Google ADK 中使用循序代理管線來生成與審查文字。它定義了兩個 LlmAgent 實例：generator 與 reviewer。generator 代理的設計目的，是針對指定主題建立初始草稿段落。它被指示撰寫一段簡短且具資訊量的內容，並將輸出儲存到狀態鍵 `draft_text`。reviewer 代理則扮演 generator 所產生文字的事實查核者。它被指示從 `draft_text` 讀取文字，並驗證其事實準確性。reviewer 的輸出是一個含有兩個鍵的結構化 dictionary：status 與 reasoning。status 表示文字是「ACCURATE」或「INACCURATE」，而 reasoning 則提供該狀態的說明。這個 dictionary 會儲存到狀態鍵 `review_output`。名為 `review_pipeline` 的 SequentialAgent 會被建立用來管理兩個代理的執行順序。它確保 generator 先執行，接著才執行 reviewer。整體執行流程是：generator 產生文字，然後將其儲存到狀態中。隨後，reviewer 從狀態讀取這段文字，進行事實查核，並把查核結果（status 與 reasoning）儲存回狀態。這條管線讓內容建立與審查能透過不同代理，以結構化流程進行。

**注意：** 對於有興趣的讀者，也可以使用 ADK 的 LoopAgent 作為替代實作。

在結束前，值得注意的是，雖然反思模式能顯著提升輸出品質，但也伴隨重要取捨。迭代流程雖然強大，卻可能帶來更高成本與延遲，因為每個精修迴圈都可能需要新的 LLM 呼叫，因此對時間敏感的應用未必理想。此外，這個模式也相當消耗記憶體；每次迭代都會讓對話歷史擴張，包含初始輸出、批判與後續精修內容。

## 一覽

**是什麼：** 代理的初始輸出通常不是最佳，可能存在不準確、不完整，或未能滿足複雜需求的問題。基本的代理式工作流程缺乏內建流程，讓代理辨識並修正自己的錯誤。解法是讓代理評估自己的工作；或以更穩健的方式，引入另一個邏輯代理作為批判者，避免初始回應在不論品質如何的情況下直接成為最終結果。

**為什麼：** 反思模式透過引入自我修正與精修機制來提供解法。它建立一個回饋迴路，由「producer」代理產生輸出，接著由「critic」代理（或 producer 本身）根據預先定義的標準進行評估。接著使用這項批判產生改進版本。這種生成、評估與精修的迭代流程，會逐步提升最終結果的品質，帶來更準確、連貫且可靠的成果。

**經驗法則：** 當最終輸出的品質、準確性與細節比速度與成本更重要時，就使用反思模式。它特別適合生成精緻長篇內容、撰寫與除錯程式碼，以及建立詳細計畫等任務。當任務需要高度客觀性，或需要通才型 producer 代理可能忽略的專業評估時，請採用獨立的 critic 代理。

**視覺摘要：**

![Reflection Design Pattern, Self-Reflection](../assets/Reflection_Design_Pattern_Self_Reflection.png)

圖 1：反思設計模式，自我反思

![Reflection Design Pattern, Producer and Critique Agent](../assets/Reflection_Design_Pattern_Producer_and_Critique_Agent.png)

圖 2：反思設計模式，producer 與 critique 代理

## 重點整理

* 反思模式的主要優勢，是能反覆自我修正並精修輸出，進而大幅提升品質、準確性，以及對複雜指示的遵循程度。  
* 它包含執行、評估／批判與精修所構成的回饋迴路。對於需要高品質、準確或細緻輸出的任務而言，反思不可或缺。  
* 一種強大的實作方式是 Producer-Critic 模型，其中由獨立代理（或透過提示塑造的角色）評估初始輸出。這種關注點分離能提升客觀性，並允許更專門、結構化的回饋。  
* 然而，這些優點的代價是更高的延遲與運算成本，同時也更容易超過模型的上下文視窗，或受到 API 服務節流限制。  
* 雖然完整的迭代式反思通常需要具狀態的工作流程（例如 LangGraph），但可以在 LangChain 中使用 LCEL 實作單一步驟的反思，將輸出傳遞給批判流程並進行後續精修。  
* Google ADK 可透過循序工作流程促成反思，讓一個代理的輸出由另一個代理批判，並允許後續精修步驟。  
* 此模式讓代理能執行自我修正，並隨著時間提升其效能。

## 結論

反思模式在代理的工作流程中提供了關鍵的自我修正機制，使其能超越單次執行並進行迭代式改善。這是透過建立一個迴圈來達成：系統產生輸出、根據特定標準評估輸出，然後使用該評估產生精修後的結果。這項評估可以由代理本身執行（自我反思），也可以由獨立的 critic 代理執行；後者通常更有效，也是此模式中的關鍵架構選擇。

雖然完全自主的多步驟反思流程需要穩健的狀態管理架構，但其核心原則可以透過單一「生成—批判—精修」循環有效示範。作為一種控制結構，反思可與其他基礎模式整合，用來建構更穩健且功能更複雜的代理式系統。

## 參考資料

以下是進一步閱讀反思模式與相關概念的一些資源：

1. Training Language Models to Self-Correct via Reinforcement Learning, [https://arxiv.org/abs/2409.12917](https://arxiv.org/abs/2409.12917)
2. LangChain Expression Language (LCEL) Documentation: [https://python.langchain.com/docs/introduction/](https://python.langchain.com/docs/introduction/)
3. LangGraph Documentation:[https://www.langchain.com/langgraph](https://www.langchain.com/langgraph)
4. Google Agent Developer Kit (ADK) Documentation (Multi-Agent Systems): [https://google.github.io/adk-docs/agents/multi-agents/](https://google.github.io/adk-docs/agents/multi-agents/)

