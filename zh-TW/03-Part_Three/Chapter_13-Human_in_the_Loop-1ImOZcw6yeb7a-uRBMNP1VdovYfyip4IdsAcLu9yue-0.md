# 第 13 章：人類介入（Human in the Loop）

人類介入（HITL）模式代表開發與部署代理（agent）時的一項關鍵策略。它有意識地把人類認知的獨特優勢——例如判斷力、創造力與細緻理解——和 AI 的運算能力與效率交織在一起。這種策略性整合不只是可選方案，在 AI 系統日益嵌入關鍵決策流程時，往往更是必要條件。

HITL 的核心原則，是確保 AI 在倫理邊界內運作、遵循安全協定，並以最佳成效達成目標。這些考量在複雜、模糊或高風險的領域中特別迫切，因為 AI 的錯誤或誤解可能帶來重大影響。在這類情境中，讓 AI 系統完全自主、在沒有任何人類介入的情況下獨立運作，可能並不明智。HITL 承認這項現實，並強調即使 AI 技術快速進步，人類監督、策略性輸入與協作式互動仍然不可或缺。

HITL 方法本質上圍繞著人工智慧與人類智慧之間的綜效。HITL 並不把 AI 視為取代人類工作者的工具，而是將 AI 定位為擴增並強化人類能力的工具。這種擴增可以有多種形式，從自動化例行工作，到提供資料驅動的洞察以支援人類決策皆屬之。最終目標是建立一個協作生態系，讓人類與 AI 代理都能運用各自的獨特優勢，達成任何一方單獨都無法完成的成果。

在實務上，HITL 可以用多種方式實作。一種常見做法是由人類擔任驗證者或審查者，檢查 AI 輸出以確保準確性並找出潛在錯誤。另一種實作方式，則是由人類主動引導 AI 行為，即時提供回饋或進行修正。在更複雜的設定中，人類可能與 AI 以夥伴關係協作，透過互動式對話或共用介面共同解決問題或做出決策。無論具體實作方式為何，HITL 模式都強調維持人類控制與監督的重要性，確保 AI 系統持續符合人類倫理、價值、目標與社會期待。

## 人類介入模式概觀

人類介入（HITL）模式將人工智慧與人類輸入整合，以強化代理能力。這種方法承認，最佳 AI 效能通常需要自動化處理與人類洞察的結合，尤其是在高度複雜或涉及倫理考量的情境中。HITL 並非取代人類輸入，而是透過確保關鍵判斷與決策由人類理解來支撐，進而擴增人類能力。

HITL 涵蓋幾個關鍵面向：人類監督，指的是監控 AI 代理的效能與輸出（例如透過日誌審查或即時儀表板），以確保遵守指引並防止不良結果。介入與修正，發生在 AI 代理遇到錯誤或模糊情境並可能請求人類介入時；人類操作員可以修正錯誤、提供缺漏資料或引導代理，這也會回饋到後續的代理改進。用於學習的人類回饋會被收集並用來精進 AI 模型，特別是在人類回饋強化學習（reinforcement learning with human feedback）這類方法中，人類偏好會直接影響代理的學習軌跡。決策擴增是指 AI 代理向人類提供分析與建議，再由人類做出最終決策；它透過 AI 產生的洞察強化人類決策，而不是交由 AI 完全自主。人類—代理協作是一種合作式互動，人類與 AI 代理各自貢獻所長；例行資料處理可由代理負責，而創意問題解決或複雜協商則由人類處理。最後，升級政策是既定協定，用來規範代理應在何時以及如何將任務升級給人類操作員，以避免代理在超出能力範圍的情況下發生錯誤。

實作 HITL 模式，可讓代理應用於不適合或不允許完全自主的敏感領域。它也透過回饋迴路提供持續改進的機制。例如在金融領域，大型企業貸款的最終核准需要人類貸款專員評估領導者品格等質性因素。同樣地，在法律領域，正義與問責的核心原則要求人類法官保有量刑等關鍵決策的最終權限，因為這些決策涉及複雜的道德推理。

> **注意事項：** 儘管 HITL 模式有其優點，也有重要限制，其中最主要的是缺乏可擴展性。人類監督雖然能提供高準確度，但操作員無法管理數百萬個任務，因而形成根本取捨，通常需要採用混合式方法：以自動化處理規模，以 HITL 確保準確性。此外，這種模式的成效高度仰賴人類操作員的專業能力；例如，AI 可以產生軟體程式碼，但只有熟練的開發者能準確找出細微錯誤，並提供正確指引來修正。當使用 HITL 產生訓練資料時，這種專業需求同樣存在，因為人類標註者可能需要接受特殊訓練，學會如何以能產生高品質資料的方式修正 AI。最後，實作 HITL 也會帶來重大的隱私疑慮，因為敏感資訊在揭露給人類操作員之前，通常必須經過嚴格匿名化，這又為流程增加了一層複雜度。

## 實際應用與使用案例

人類介入模式對許多產業與應用都至關重要，尤其是在準確性、安全性、倫理或細緻理解極為重要的場景中。

* **內容審核：** AI 代理可以快速篩選大量線上內容是否違規（例如仇恨言論、垃圾訊息）。然而，模糊案例或臨界內容會升級給人類審核員審查並做出最終決定，以確保細緻判斷並遵循複雜政策。  
* **自動駕駛：** 自駕車雖然能自主處理多數駕駛任務，但在 AI 無法有把握應對的複雜、不可預測或危險情況下（例如極端天氣、異常路況），其設計會將控制權交還給人類駕駛。  
* **金融詐欺偵測：** AI 系統可以根據模式標記可疑交易。不過，高風險或模糊警示通常會送交人類分析師進一步調查、聯絡客戶，並最終判定交易是否涉及詐欺。  
* **法律文件審閱：** AI 可以快速掃描並分類數千份法律文件，以找出相關條款或證據。接著，人類法律專業人員會審查 AI 發現的準確性、上下文與法律影響，尤其是在關鍵案件中。  
* **客戶支援（複雜查詢）：** 聊天機器人可以處理例行客戶詢問。如果使用者的問題過於複雜、情緒強烈，或需要 AI 無法提供的同理心，對話就會無縫轉交給人類支援代理。  
* **資料標記與標註：** AI 模型通常需要大量已標記資料集進行訓練。人類會被納入流程，準確標記影像、文字或音訊，提供 AI 學習所需的 ground truth。隨著模型演進，這是一個持續進行的流程。  
* **生成式 AI 精進：** 當大型語言模型（large language model, LLM）產生創意內容（例如行銷文案、設計構想）時，人類編輯或設計師會審查並精進輸出，確保其符合品牌準則、能與目標受眾產生共鳴，並維持品質。  
* **自主網路：** AI 系統能運用關鍵績效指標（KPIs）與已識別模式來分析警示，並預測網路問題與流量異常。然而，關鍵決策——例如處理高風險警示——經常會升級給人類分析師。這些分析師會進一步調查，並對是否核准網路變更做出最終判定。

這個模式展現了一種實用的 AI 實作方法。它運用 AI 提高可擴展性與效率，同時維持人類監督，以確保品質、安全與倫理合規。

「Human-on-the-loop」是這個模式的一種變體，由人類專家定義整體政策，再由 AI 處理即時動作以確保符合規範。以下來看兩個例子：

* **自動化金融交易系統**：在這個情境中，人類金融專家會設定整體投資策略與規則。例如，人類可能將政策定義為：「維持 70% 科技股與 30% 債券的投資組合；不得在任何單一公司投入超過 5%；任何股票若跌到低於買入價 10%，即自動賣出。」接著，AI 會即時監控股市，當這些預先定義的條件成立時立即執行交易。AI 根據人類操作員所設定較慢速、較具策略性的政策，處理即時且高速的動作。  
* **現代客服中心**：在這種設定中，人類管理者會為客戶互動建立高階政策。例如，管理者可能設定規則，如「任何提到『服務中斷』的來電，都應立即轉接給技術支援專員」，或「如果客戶的語調顯示高度挫折，系統應主動提供直接連接人類代理的選項」。AI 系統接著會處理初始客戶互動，即時聆聽並解讀客戶需求。它會自主執行管理者的政策，立即轉接來電或提供升級選項，而不需要每一個個案都有人類介入。這讓 AI 能依照人類操作員提供的較慢速策略性指引，管理大量即時動作。

## 實作程式碼範例

為了示範人類介入模式，ADK 代理可以識別需要人類審查的情境，並啟動升級流程。這讓代理在自主決策能力有限，或需要複雜判斷的情況下，可以引入人類介入。這並不是孤立功能；其他熱門框架也採用了類似能力。舉例來說，LangChain 也提供工具來實作這類互動。

```python
from typing import Optional

from google.adk.agents import Agent
from google.adk.tools.tool_context import ToolContext
from google.adk.callbacks import CallbackContext
from google.adk.models.llm import LlmRequest
from google.genai import types


# Placeholder for tools (replace with actual implementations if needed)
def troubleshoot_issue(issue: str) -> dict:
    return {"status": "success", "report": f"Troubleshooting steps for {issue}."}


def create_ticket(issue_type: str, details: str) -> dict:
    return {"status": "success", "ticket_id": "TICKET123"}


def escalate_to_human(issue_type: str) -> dict:
    # This would typically transfer to a human queue in a real system
    return {"status": "success", "message": f"Escalated {issue_type} to a human specialist."}


technical_support_agent = Agent(
    name="technical_support_specialist",
    model="gemini-2.0-flash-exp",
    instruction="""
    You are a technical support specialist for our electronics company.
    FIRST, check if the user has a support history in state["customer_info"]["support_history"].
    If they do, reference this history in your responses.

    For technical issues:
    1. Use the troubleshoot_issue tool to analyze the problem.
    2. Guide the user through basic troubleshooting steps.
    3. If the issue persists, use create_ticket to log the issue.

    For complex issues beyond basic troubleshooting:
    1. Use escalate_to_human to transfer to a human specialist.

    Maintain a professional but empathetic tone. Acknowledge the frustration technical issues can cause,
    while providing clear steps toward resolution.
    """,
    tools=[troubleshoot_issue, create_ticket, escalate_to_human],
)


def personalization_callback(
    callback_context: CallbackContext, llm_request: LlmRequest
) -> Optional[LlmRequest]:
    """Adds personalization information to the LLM request."""
    # Get customer info from state
    customer_info = callback_context.state.get("customer_info")
    if customer_info:
        customer_name = customer_info.get("name", "valued customer")
        customer_tier = customer_info.get("tier", "standard")
        recent_purchases = customer_info.get("recent_purchases", [])

        personalization_note = (
            f"\nIMPORTANT PERSONALIZATION:\n"
            f"Customer Name: {customer_name}\n"
            f"Customer Tier: {customer_tier}\n"
        )
        if recent_purchases:
            personalization_note += f"Recent Purchases: {', '.join(recent_purchases)}\n"

        if llm_request.contents:
            # Add as a system message before the first content
            system_content = types.Content(
                role="system",
                parts=[types.Part(text=personalization_note)],
            )
            llm_request.contents.insert(0, system_content)

    return None  # Return None to continue with the modified request
```

這段程式碼提供了一個藍圖，用於使用 Google 的 ADK 建立技術支援代理，並以 HITL 框架為設計核心。代理會作為智慧型第一線支援，配置特定指示，並配備 `troubleshoot_issue`、`create_ticket`、`escalate_to_human` 等工具，以管理完整的支援工作流程。升級工具是 HITL 設計的核心部分，確保複雜或敏感案例會交由人類專家處理。

這個架構的一項關鍵特色，是能透過專用 callback function 達成深度個人化。在聯絡 LLM 之前，這個函式會從代理的狀態中動態擷取客戶特定資料，例如姓名、等級與購買紀錄。接著，這些上下文會以系統訊息的形式注入提示中，讓代理能提供高度客製且資訊充分的回應，並引用使用者的歷史紀錄。透過結合結構化工作流程、必要的人類監督與動態個人化，這段程式碼成為一個實用範例，展示 ADK 如何促成精密且強健的 AI 支援解決方案開發。

## 概覽

**What:** AI 系統，包括進階 LLM，常常難以處理需要細緻判斷、倫理推理，或深入理解複雜且模糊上下文的任務。在高風險環境中部署完全自主的 AI 具有重大風險，因為錯誤可能導致嚴重的安全、財務或倫理後果。這些系統缺乏人類所具備的內在創造力與常識推理。因此，在關鍵決策流程中只依賴自動化通常並不明智，也可能削弱系統整體的有效性與可信度。

**Why:** 人類介入（HITL）模式透過將人類監督策略性地整合進 AI 工作流程，提供一套標準化解決方案。這種代理式方法建立了一種共生夥伴關係：AI 負責繁重的運算與資料處理，人類則提供關鍵驗證、回饋與介入。透過這種方式，HITL 確保 AI 行動符合人類價值與安全協定。這個協作框架不僅能降低完全自動化的風險，也能透過持續從人類輸入中學習來提升系統能力。最終，這會帶來更強健、準確且合乎倫理的成果，而這些成果是人類或 AI 單獨都無法達成的。

**Rule of Thumb:** 當你在錯誤會造成重大安全、倫理或財務後果的領域部署 AI 時，應使用此模式，例如醫療照護、金融或自主系統。對於 LLM 無法可靠處理、涉及模糊性與細微差異的任務，例如內容審核或複雜客戶支援升級，這個模式也不可或缺。當目標是用高品質、由人類標記的資料持續改進 AI 模型，或精進生成式 AI 輸出以符合特定品質標準時，也應採用 HITL。

**Visual Summary:**

![Human in the Loop Design Pattern](../assets/Human_in_the_Loop_Design_Pattern.png)

圖 1：人類介入設計模式

## 重點摘要

重點包括：

* 人類介入（HITL）將人類智慧與判斷整合進 AI 工作流程。  
* 在複雜或高風險情境中，它對安全、倫理與有效性至關重要。  
* 關鍵面向包括人類監督、介入、用於學習的回饋，以及決策擴增。  
* 升級政策對代理而言很重要，能讓代理知道何時應轉交給人類。  
* HITL 讓負責任的 AI 部署與持續改進成為可能。  
* 人類介入的主要缺點，在於其本質上缺乏可擴展性，會在準確度與處理量之間形成取捨，並且仰賴高度熟練的領域專家才能有效介入。
* 它的實作也帶來營運挑戰，包括需要訓練人類操作員以產生資料，以及透過匿名化敏感資訊來處理隱私疑慮。

## 結論

本章探討了重要的人類介入（HITL）模式，強調它在建立強健、安全且合乎倫理的 AI 系統中所扮演的角色。我們討論了將人類監督、介入與回饋整合進代理工作流程，如何能大幅提升其效能與可信度，尤其是在複雜且敏感的領域中。實際應用展示了 HITL 的廣泛用途，從內容審核與醫療診斷，到自動駕駛與客戶支援皆然。概念性程式碼範例則讓我們初步看到 ADK 如何透過升級機制促成人類—代理互動。隨著 AI 能力持續進步，HITL 仍然是負責任 AI 開發的基石，確保人類價值與專業知識始終位於智慧系統設計的核心。

## 參考資料

1. A Survey of Human-in-the-loop for Machine Learning, Xingjiao Wu, Luwei Xiao, Yixuan Sun, Junhang Zhang, Tianlong Ma, Liang He, [https://arxiv.org/abs/2108.00941](https://arxiv.org/abs/2108.00941)

