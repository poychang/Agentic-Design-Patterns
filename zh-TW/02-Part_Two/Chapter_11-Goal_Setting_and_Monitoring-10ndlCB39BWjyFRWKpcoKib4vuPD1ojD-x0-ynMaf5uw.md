# 第 11 章：目標設定與監控

若要讓 AI 代理（agent）真正有效且具有目的性，它們需要的不只是處理資訊或使用工具的能力；它們還需要清楚的方向感，以及知道自己是否真的成功的方法。這正是目標設定與監控模式發揮作用的地方。這個模式的重點，是為代理提供明確的目標，讓它們朝著目標前進，並配備追蹤進度、判斷目標是否已達成的手段。

## 目標設定與監控模式概觀

想像一下規劃一趟旅行。你不會憑空出現在目的地。你會先決定想去哪裡（目標狀態），弄清楚自己從哪裡出發（初始狀態），考量可用選項（交通工具、路線、預算），然後規劃一連串步驟：訂票、打包行李、前往機場或車站、搭乘交通工具、抵達、尋找住宿等等。這種循序漸進的流程，通常還會考量相依性與限制，本質上就是我們在代理式系統中所說的規劃。

在 AI 代理的脈絡中，規劃通常是指代理接收一個高階目標，然後自主或半自主地產生一系列中間步驟或子目標。這些步驟可以依序執行，也可以在更複雜的流程中執行，並可能涉及其他模式，例如工具使用、路由或多代理協作。規劃機制可能涉及精密的搜尋演算法、邏輯推理，或越來越常見地，運用大型語言模型（large language model, LLM）的能力，根據其訓練資料與對任務的理解，產生合理且有效的計畫。

良好的規劃能力讓代理能處理不是簡單單步查詢的問題。它使代理能處理多面向的請求，透過重新規劃適應情況變化，並協調複雜的工作流程。這是一個基礎模式，支撐許多進階的代理式行為，能把單純的反應式系統，轉變為可主動朝明確目標努力的系統。

## 實務應用與使用案例

目標設定與監控模式對於建構能在複雜真實情境中自主且可靠運作的代理而言至關重要。以下是一些實務應用：

* **客戶支援自動化：** 代理的目標可能是「解決客戶的帳單詢問」。它會監控對話、檢查資料庫項目，並使用工具調整帳單。成功與否會透過確認帳單變更，以及收到正面的客戶回饋來監控。如果問題未解決，它就會升級處理。  
* **個人化學習系統：** 學習代理的目標可能是「提升學生對代數的理解」。它會監控學生在練習上的進度、調整教材，並追蹤正確率與完成時間等績效指標；如果學生遇到困難，便調整其方法。  
* **專案管理助理：** 代理可以被指派「確保專案里程碑 X 在 Y 日期前完成」。它會監控任務狀態、團隊溝通與資源可用性，在目標有風險時標示延遲並建議修正行動。  
* **自動化交易機器人：** 交易代理的目標可能是「在風險承受範圍內最大化投資組合收益」。它會持續監控市場資料、目前的投資組合價值與風險指標，在條件符合目標時執行交易，並在風險門檻被突破時調整策略。  
* **機器人與自動駕駛車輛：** 自動駕駛車輛的主要目標是「安全地將乘客從 A 地載送到 B 地」。它會持續監控環境（其他車輛、行人、交通號誌）、自身狀態（速度、燃料），以及沿著規劃路線的進度，並調整駕駛行為，以安全且有效率地達成目標。  
* **內容審核：** 代理的目標可以是「識別並移除平台 X 上的有害內容」。它會監控傳入內容、套用分類模型，並追蹤誤判為陽性／陰性等指標，調整篩選條件，或將模稜兩可的案例升級給人工審核者。

這個模式是代理可靠運作、達成特定結果並適應動態條件的基礎，為智慧型自我管理提供必要的框架。

## 實作程式碼範例

為了說明目標設定與監控模式，我們提供一個使用 LangChain 和 OpenAI APIs 的範例。這個 Python 指令碼概述了一個自主 AI 代理，其設計目的是產生並改善 Python 程式碼。它的核心功能是針對指定問題產出解決方案，並確保符合使用者定義的品質基準。

它採用「目標設定與監控」模式，不只是產生一次程式碼，而是進入建立、自我評估與改進的反覆循環。代理的成功與否，是由它自身以 AI 驅動的判斷來衡量：產生的程式碼是否成功符合初始目標。最終輸出是一個經過潤飾、含有註解且可直接使用的 Python 檔案，代表這個精煉流程的成果。

 **相依套件**：

```python
pip install langchain_openai openai python-dotenv .env file with key in OPENAI_API_KEY
```

你可以把這個指令碼想像成一位被指派到專案的自主 AI 程式設計師，這樣最能理解它（見圖 1）。當你把一份詳細的專案簡報交給 AI 時，流程就開始了；這份簡報就是它需要解決的特定程式設計問題。

```python
# MIT License
# Copyright (c) 2025 Mahtab Syed
# https://www.linkedin.com/in/mahtabsyed/

"""
Hands-On Code Example - Iteration 2
-  To illustrate the Goal Setting and Monitoring pattern, we have an example using LangChain and OpenAI APIs:

Objective: Build an AI Agent which can write code for a specified use case based on specified goals:
-  Accepts a coding problem (use case) in code or can be as input.
-  Accepts a list of goals (e.g., "simple", "tested", "handles edge cases")  in code or can be input.
-  Uses an LLM (like GPT-4o) to generate and refine Python code until the goals are met. (I am using max 5 iterations, this could be based on a set goal as well)
-  To check if we have met our goals I am asking the LLM to judge this and answer just True or False which makes it easier to stop the iterations.
-  Saves the final code in a .py file with a clean filename and a header comment.
"""

import os
import random
import re
from pathlib import Path

from langchain_openai import ChatOpenAI
from dotenv import load_dotenv, find_dotenv


# 🔐 Load environment variables
_ = load_dotenv(find_dotenv())
OPENAI_API_KEY = os.getenv("OPENAI_API_KEY")
if not OPENAI_API_KEY:
    raise EnvironmentError("❌ Please set the OPENAI_API_KEY environment variable.")

# ✅ Initialize OpenAI model
print("📡 Initializing OpenAI LLM (gpt-4o)...")
llm = ChatOpenAI(
    model="gpt-4o",  # If you dont have access to got-4o use other OpenAI LLMs
    temperature=0.3,
    openai_api_key=OPENAI_API_KEY,
)


# --- Utility Functions ---
def generate_prompt(
    use_case: str, goals: list[str], previous_code: str = "", feedback: str = ""
) -> str:
    print("📝 Constructing prompt for code generation...")
    base_prompt = f"""
You are an AI coding agent. Your job is to write Python code based on the following use case:

Use Case: {use_case}

Your goals are:
{chr(10).join(f"- {g.strip()}" for g in goals)}
"""
    if previous_code:
        print("🔄 Adding previous code to the prompt for refinement.")
        base_prompt += f"\nPreviously generated code:\n{previous_code}"
    if feedback:
        print("📋 Including feedback for revision.")
        base_prompt += f"\nFeedback on previous version:\n{feedback}\n"

    base_prompt += "\nPlease return only the revised Python code. Do not include comments or explanations outside the code."
    return base_prompt


def get_code_feedback(code: str, goals: list[str]) -> str:
    print("🔍 Evaluating code against the goals...")
    feedback_prompt = f"""
You are a Python code reviewer. A code snippet is shown below. Based on the following goals:

{chr(10).join(f"- {g.strip()}" for g in goals)}

Please critique this code and identify if the goals are met. Mention if improvements are needed for clarity, simplicity, correctness, edge case handling, or test coverage.

Code:
{code}
"""
    return llm.invoke(feedback_prompt)


def goals_met(feedback_text: str, goals: list[str]) -> bool:
    """
    Uses the LLM to evaluate whether the goals have been met based on the feedback text.
    Returns True or False (parsed from LLM output).
    """
    review_prompt = f"""
You are an AI reviewer.

Here are the goals:
{chr(10).join(f"- {g.strip()}" for g in goals)}

Here is the feedback on the code:
\"\"\"
{feedback_text}
\"\"\"

Based on the feedback above, have the goals been met?

Respond with only one word: True or False.
"""
    response = llm.invoke(review_prompt).content.strip().lower()
    return response == "true"


def clean_code_block(code: str) -> str:
    lines = code.strip().splitlines()
    if lines and lines[0].strip().startswith("```"):
        lines = lines[1:]
    if lines and lines[-1].strip() == "```":
        lines = lines[:-1]
    return "\n".join(lines).strip()


def add_comment_header(code: str, use_case: str) -> str:
    comment = f"# This Python program implements the following use case:\n# {use_case.strip()}\n"
    return comment + "\n" + code


def to_snake_case(text: str) -> str:
    text = re.sub(r"[^a-zA-Z0-9 ]", "", text)
    return re.sub(r"\s+", "_", text.strip().lower())


def save_code_to_file(code: str, use_case: str) -> str:
    print("💾 Saving final code to file...")

    summary_prompt = (
        f"Summarize the following use case into a single lowercase word or phrase, "
        f"no more than 10 characters, suitable for a Python filename:\n\n{use_case}"
    )
    raw_summary = llm.invoke(summary_prompt).content.strip()
    short_name = re.sub(r"[^a-zA-Z0-9_]", "", raw_summary.replace(" ", "_").lower())[:10]

    random_suffix = str(random.randint(1000, 9999))
    filename = f"{short_name}_{random_suffix}.py"
    filepath = Path.cwd() / filename

    with open(filepath, "w") as f:
        f.write(code)

    print(f"✅ Code saved to: {filepath}")
    return str(filepath)


# --- Main Agent Function ---
def run_code_agent(use_case: str, goals_input: str, max_iterations: int = 5) -> str:
    goals = [g.strip() for g in goals_input.split(",")]

    print(f"\n🎯 Use Case: {use_case}")
    print("🎯 Goals:")
    for g in goals:
        print(f"  - {g}")

    previous_code = ""
    feedback = ""

    for i in range(max_iterations):
        print(f"\n=== 🔁 Iteration {i + 1} of {max_iterations} ===")
        prompt = generate_prompt(
            use_case,
            goals,
            previous_code,
            feedback if isinstance(feedback, str) else feedback.content,
        )

        print("🚧 Generating code...")
        code_response = llm.invoke(prompt)
        raw_code = code_response.content.strip()
        code = clean_code_block(raw_code)
        print("\n🧾 Generated Code:\n" + "-" * 50 + f"\n{code}\n" + "-" * 50)

        print("\n📤 Submitting code for feedback review...")
        feedback = get_code_feedback(code, goals)
        feedback_text = feedback.content.strip()
        print("\n📥 Feedback Received:\n" + "-" * 50 + f"\n{feedback_text}\n" + "-" * 50)

        if goals_met(feedback_text, goals):
            print("✅ LLM confirms goals are met. Stopping iteration.")
            break

        print("🛠️ Goals not fully met. Preparing for next iteration...")
        previous_code = code

    final_code = add_comment_header(code, use_case)
    return save_code_to_file(final_code, use_case)


# --- CLI Test Run ---
if __name__ == "__main__":
    print("\n🧠 Welcome to the AI Code Generation Agent")

    # Example 1
    use_case_input = "Write code to find BinaryGap of a given positive integer"
    goals_input = "Code simple to understand, Functionally correct, Handles comprehensive edge cases, Takes positive integer input only, prints the results with few examples"
    run_code_agent(use_case_input, goals_input)

    # Example 2
    # use_case_input = "Write code to count the number of files in current directory and all its nested sub directories, and print the total count"
    # goals_input = (
    #     "Code simple to understand, Functionally correct, Handles comprehensive edge cases, Ignore recommendations for performance, Ignore recommendations for test suite use like unittest or pytest"
    # )
    # run_code_agent(use_case_input, goals_input)

    # Example 3
    # use_case_input = "Write code which takes a command line input of a word doc or docx file and opens it and counts the number of words, and characters in it and prints all"
    # goals_input = "Code simple to understand, Functionally correct, Handles edge cases"
    # run_code_agent(use_case_input, goals_input)
```

除了這份簡報之外，你還會提供一份嚴格的品質檢查清單，代表最終程式碼必須符合的目標——例如「解決方案必須簡單」、「必須在功能上正確」，或「需要處理非預期的邊界案例」。

![Goal Setting and Monitor example](../assets/Goal_Setting_and_Monitoring.png)

圖 1：目標設定與監控範例

手上有這項任務後，AI 程式設計師就開始工作，產出第一版程式碼。然而，它不會立即提交這個初版，而是停下來執行一個關鍵步驟：嚴謹的自我審查。它會仔細把自己的成果與你提供的品質檢查清單逐項比對，扮演自己的品質保證檢查員。完成檢查後，它會對自己的進度給出一個簡單且不偏不倚的判定：如果成果符合所有標準，就是 "True"；如果未達標，就是 "False"。

如果判定是 "False"，AI 不會放棄。它會進入審慎的修訂階段，運用自我批判得到的洞察，找出弱點並智慧地重寫程式碼。這種起草、自我審查與精煉的循環會持續進行，每次迭代都試圖更接近目標。這個流程會重複，直到 AI 終於透過滿足每一項要求而達到 "True" 狀態，或直到達到預先定義的嘗試次數上限，就像開發人員面對截止期限工作一樣。一旦程式碼通過最終檢查，指令碼就會打包這個潤飾過的解決方案，加入有用的註解，並儲存到一個乾淨的新 Python 檔案中，準備好可供使用。

**注意事項與考量：** 必須注意，這是一個示範性說明，而不是可用於正式環境的程式碼。對於真實世界的應用，必須考量多個因素。LLM 可能無法完全掌握目標的預期含義，並可能錯誤地評估自己的表現為成功。即使目標已被充分理解，模型仍可能產生幻覺。當同一個 LLM 同時負責撰寫程式碼與判斷其品質時，它可能更難發現自己正朝錯誤方向前進。

歸根究柢，LLM 不會像施魔法一樣產生完美無瑕的程式碼；你仍然需要執行並測試產出的程式碼。此外，這個簡單範例中的「監控」相當基礎，會造成流程可能永遠執行下去的潛在風險。

```text
Act as an expert code reviewer with a deep commitment to producing clean, correct, and simple code. Your core mission is to eliminate code "hallucinations" by ensuring every suggestion is grounded in reality and best practices. When I provide you with a code snippet, I want you to: -- Identify and Correct Errors: Point out any logical flaws, bugs, or potential runtime errors. -- Simplify and Refactor: Suggest changes that make the code more readable, efficient, and maintainable without sacrificing correctness. -- Provide Clear Explanations: For every suggested change, explain why it is an improvement, referencing principles of clean code, performance, or security. -- Offer Corrected Code: Show the "before" and "after" of your suggested changes so the improvement is clear. Your feedback should be direct, constructive, and always aimed at improving the quality of the code.
```

更穩健的方法，是將這些關切點分離，為一組代理團隊賦予特定角色。例如，我曾使用 Gemini 建立一組個人 AI 代理團隊，其中每個代理都有特定角色：

* Peer Programmer：協助撰寫程式碼與腦力激盪。  
* Code Reviewer：抓出錯誤並提出改進建議。  
* Documenter：產生清楚且精簡的文件。  
* Test Writer：建立完整的單元測試。  
* Prompt Refiner：最佳化與 AI 的互動。

在這個多代理系統中，Code Reviewer 作為與程式設計師代理分離的實體，其提示類似於範例中的評審，能大幅改善客觀評估。這種結構自然會帶來更好的實務做法，因為 Test Writer 代理可以滿足為 Peer Programmer 產出的程式碼撰寫單元測試的需求。

我把加入這些更精密控制，並讓程式碼更接近可用於正式環境的任務，留給有興趣的讀者。

## 快速概覽

**是什麼**：AI 代理通常缺乏清楚方向，使它們無法在簡單的反應式任務之外，有目的地行動。沒有明確目標時，它們無法獨立處理複雜的多步驟問題，也無法協調精密的工作流程。此外，它們本身沒有內建機制可判斷自身行動是否正導向成功結果。這限制了它們的自主性，也使它們無法在僅執行任務尚不足夠的動態真實情境中真正有效。

**為什麼**：目標設定與監控模式透過在代理式系統中嵌入目的感與自我評估，提供標準化解決方案。它涉及明確定義代理要達成的清楚、可衡量目標。同時，它也建立監控機制，持續根據這些目標追蹤代理的進度與其環境狀態。這形成一個關鍵的回饋迴路，使代理能評估自身表現、修正路線，並在偏離成功路徑時調整計畫。透過實作這個模式，開發人員可以把簡單的反應式代理，轉變為能自主且可靠運作的主動式、目標導向系統。

**經驗法則**：當 AI 代理必須在沒有持續人類介入的情況下，自主執行多步驟任務、適應動態條件，並可靠達成特定高階目標時，就使用這個模式。

**視覺摘要**：

![Goal Design Pattern](../assets/Goal_Design_Pattern.png)

圖 2：目標設計模式

## 重點整理

重點包括：

* 目標設定與監控為代理配備目的，以及追蹤進度的機制。  
* 目標應該具體、可衡量、可達成、相關且有時限（SMART）。  
* 清楚定義指標與成功標準，是有效監控的必要條件。  
* 監控涉及觀察代理行動、環境狀態與工具輸出。  
* 來自監控的回饋迴路，讓代理能調整、修訂計畫或升級問題。  
* 在 Google 的 ADK 中，目標通常透過代理指示傳達，而監控則透過狀態管理與工具互動完成。

## 結論

本章聚焦於目標設定與監控這個關鍵典範。我強調這個概念如何把 AI 代理從單純反應式系統，轉變為主動且目標驅動的實體。本文強調定義清楚、可衡量目標的重要性，並強調建立嚴謹監控程序以追蹤進度。實務應用展示了這個典範如何支援多個領域中可靠的自主運作，包括客戶服務與機器人。一個概念性的程式碼範例說明了如何在結構化框架中實作這些原則，使用代理指令與狀態管理來引導並評估代理達成指定目標的程度。最終，讓代理具備制定並監督目標的能力，是建構真正智慧且可問責 AI 系統的基本步驟。

## 參考資料

1. SMART 目標框架. [https://en.wikipedia.org/wiki/SMART\_criteria](https://en.wikipedia.org/wiki/SMART_criteria)

