# 第 12 章：例外處理與復原

若要讓 AI 代理（agent）在多樣化的真實世界環境中可靠運作，就必須能夠管理無法預見的情況、錯誤與故障。就像人類會適應意外阻礙一樣，智慧代理也需要健全的系統來偵測問題、啟動復原程序，或至少確保受控的失敗。這項必要需求構成了例外處理與復原模式的基礎。

此模式著重於開發格外耐用且具韌性的代理，使其即使面對各種困難與異常，也能維持不間斷的功能與營運完整性。它強調主動準備與被動應對策略同樣重要，以確保即使遭遇挑戰仍能持續運作。這種適應能力對代理能否在複雜且不可預測的環境中成功運作至關重要，最終也會提升其整體效能與可信度。

處理意外事件的能力能確保這些 AI 系統不僅具備智慧，也具有穩定性與可靠性，進而讓人們對其部署與效能更有信心。整合完整的監控與診斷工具，能進一步強化代理快速識別並處理問題的能力，防止潛在中斷，並確保在不斷變化的條件下運作更順暢。這些進階系統對維持 AI 作業的完整性與效率至關重要，也強化了它們管理複雜性與不可預測性的能力。

此模式有時也可以搭配反思（Reflection）使用。例如，如果初次嘗試失敗並引發例外，反思流程可以分析失敗原因，並以改良後的方法重新嘗試任務，例如使用更好的提示來解決錯誤。

## 例外處理與復原模式概觀

例外處理與復原模式處理的是 AI 代理管理營運失敗的需求。此模式包含預先設想可能發生的問題，例如工具錯誤或服務無法使用，並制定緩解這些問題的策略。這些策略可能包括錯誤記錄、重試、備援、優雅降級與通知。此外，此模式也強調復原機制，例如狀態回復、診斷、自我修正與升級處理，以便讓代理恢復到穩定運作狀態。實作此模式可以提升 AI 代理的可靠性與健壯性，使其能在不可預測的環境中運作。實際應用範例包括聊天機器人處理資料庫錯誤、交易機器人處理金融錯誤，以及智慧家庭代理處理裝置故障。此模式確保代理即使遇到複雜狀況與失敗，也能持續有效運作。

![Key Components of Exception Handling and Recovery for AI agents](../assets/Key_Components_of_Exception_Handling_and_Recovery_for_AI_agents.png)

圖 1：AI 代理例外處理與復原的關鍵元件

**錯誤偵測：** 這涉及在營運問題發生時仔細識別它們。這些問題可能表現為無效或格式錯誤的工具輸出、特定 API 錯誤（例如 404（Not Found）或 500（Internal Server Error）程式碼）、服務或 API 的回應時間異常過長，或偏離預期格式、語意不連貫且毫無意義的回應。此外，也可以導入其他代理或專門監控系統進行監控，以更主動地偵測異常，讓系統能在潛在問題擴大前先行捕捉。

**錯誤處理**：偵測到錯誤後，必須有經過周密思考的回應計畫。這包括將錯誤細節仔細記錄在日誌中，以便後續除錯與分析（logging）。重新嘗試該動作或請求，有時搭配稍微調整過的參數，可能是一種可行策略，尤其適用於暫時性錯誤（retries）。使用替代策略或方法（fallbacks）可以確保部分功能得以維持。當無法立即完全復原時，代理可以維持部分功能，至少提供一定價值（graceful degradation）。最後，在需要人類介入或協作的情況下，提醒人類操作員或其他代理可能至關重要（notification）。

**復原：** 這個階段的重點是在錯誤發生後，將代理或系統恢復到穩定且可運作的狀態。它可能包含反轉近期變更或交易，以撤銷錯誤造成的影響（state rollback）。徹底調查錯誤原因對於防止再次發生十分重要。也可能需要透過自我修正機制或重新規劃流程，調整代理的計畫、邏輯或參數，避免未來發生相同錯誤。在複雜或嚴重的情況下，將問題委派給人類操作員或更高層級的系統（escalation），可能是最佳作法。

實作這種健全的例外處理與復原模式，可以將 AI 代理從脆弱且不可靠的系統，轉變為穩健、可靠的元件，能在充滿挑戰且高度不可預測的環境中有效且具韌性地運作。這能確保代理維持功能、將停機時間降到最低，並在面對意外問題時仍提供順暢且可靠的體驗。

## 實際應用與使用案例

例外處理與復原對於任何部署在真實世界情境中的代理都至關重要，因為真實環境無法保證條件永遠完美。

* **客服聊天機器人：** 如果聊天機器人嘗試存取客戶資料庫，而資料庫暫時停擺，它不應該當機。相反地，它應該偵測 API 錯誤，告知使用者目前有暫時性問題，或許建議稍後再試，或將查詢升級交由人類代理處理。  
* **自動化金融交易：** 嘗試執行交易的交易機器人可能會遇到「資金不足」錯誤或「市場已關閉」錯誤。它需要透過記錄錯誤來處理這些例外，不要一再嘗試同一筆無效交易，並可能通知使用者或調整其策略。  
* **智慧家庭自動化：** 控制智慧燈具的代理可能因網路問題或裝置故障而無法開燈。它應該偵測此失敗，或許進行重試；如果仍然失敗，則通知使用者燈無法開啟，並建議手動介入。  
* **資料處理代理：** 負責處理一批文件的代理可能會遇到損毀檔案。它應該跳過損毀檔案、記錄錯誤、繼續處理其他檔案，並在最後報告被跳過的檔案，而不是停止整個流程。  
* **網頁擷取代理：** 當網頁擷取代理遇到 CAPTCHA、網站結構變更或伺服器錯誤（例如 404 Not Found、503 Service Unavailable）時，需要優雅地處理這些情況。這可能包含暫停、使用 proxy，或回報失敗的特定 URL。  
* **機器人與製造：** 執行組裝任務的機械手臂可能因未對準而無法拿取元件。它需要偵測此失敗（例如透過感測器回饋）、嘗試重新調整、重試拿取；如果問題持續存在，則提醒人類操作員或切換到不同元件。

簡而言之，面對真實世界的複雜性時，此模式是建構不僅具備智慧，而且可靠、具韌性並對使用者友善的代理之基礎。

## 實作程式碼範例（ADK）

例外處理與復原對系統的健壯性與可靠性至關重要。以代理對失敗工具呼叫的回應為例。這類失敗可能源自錯誤的工具輸入，或工具所依賴的外部服務發生問題。

```python
from google.adk.agents import Agent, SequentialAgent


# Agent 1: Tries the primary tool. Its focus is narrow and clear.
primary_handler = Agent(
    name="primary_handler",
    model="gemini-2.0-flash-exp",
    instruction="""
    Your job is to get precise location information. Use the get_precise_location_info
    tool with the user's provided address.
    """,
    tools=[get_precise_location_info],
)

# Agent 2: Acts as the fallback handler, checking state to decide its action.
fallback_handler = Agent(
    name="fallback_handler",
    model="gemini-2.0-flash-exp",
    instruction="""
    Check if the primary location lookup failed by looking at state["primary_location_failed"].
    - If it is True, extract the city from the user's original query and use the get_general_area_info tool.
    - If it is False, do nothing.
    """,
    tools=[get_general_area_info],
)

# Agent 3: Presents the final result from the state.
response_agent = Agent(
    name="response_agent",
    model="gemini-2.0-flash-exp",
    instruction="""
    Review the location information stored in state["location_result"]. Present this information
    clearly and concisely to the user. If state["location_result"] does not exist or is empty,
    apologize that you could not retrieve the location.
    """,
    tools=[],  # This agent only reasons over the final state.
)

# The SequentialAgent ensures the handlers run in a guaranteed order.
robust_location_agent = SequentialAgent(
    name="robust_location_agent",
    sub_agents=[primary_handler, fallback_handler, response_agent],
)
```

這段程式碼使用 ADK 的 SequentialAgent 與三個子代理，定義了一個穩健的位置資訊擷取系統。`primary_handler` 是第一個代理，會嘗試使用 `get_precise_location_info` 工具取得精確的位置資訊。`fallback_handler` 作為備援，會透過檢查狀態變數來確認主要查詢是否失敗。如果主要查詢失敗，備援代理會從使用者的查詢中擷取城市，並使用 `get_general_area_info` 工具。`response_agent` 是此序列中的最後一個代理。它會檢視儲存在狀態中的位置資訊。此代理設計用來向使用者呈現最終結果。如果找不到位置資訊，它會致歉。SequentialAgent 確保這三個代理依照預先定義的順序執行。這種結構讓位置資訊擷取能採用分層方法。

## 重點速覽

**是什麼：** 在真實世界環境中運作的 AI 代理，必然會遇到無法預見的情況、錯誤與系統故障。這些中斷可能包含工具失敗、網路問題與無效資料，並威脅代理完成任務的能力。如果沒有結構化方式管理這些問題，代理在面對意外障礙時可能會變得脆弱、不可靠，並容易完全失敗。這種不可靠性會讓它們難以部署在需要穩定效能的關鍵或複雜應用中。

**為什麼**：例外處理與復原模式提供了一種標準化解決方案，用於建構穩健且具韌性的 AI 代理。它賦予代理預測、管理並從營運失敗中復原的代理式能力。此模式包含主動錯誤偵測，例如監控工具輸出與 API 回應，以及被動處理策略，例如用於診斷的 logging、重試暫時性失敗，或使用備援機制。對於更嚴重的問題，它定義了復原協定，包括回復到穩定狀態、透過調整計畫進行自我修正，或將問題升級給人類操作員。這種系統化方法能確保代理維持營運完整性、從失敗中學習，並在不可預測的環境中可靠運作。

**經驗法則：** 對於任何部署在動態真實世界環境中的 AI 代理，只要可能發生系統失敗、工具錯誤、網路問題或不可預測的輸入，且營運可靠性是關鍵需求，就應使用此模式。

**視覺摘要：**

![Exception Handling Pattern](../assets/Exception_Handling_Pattern.png)

圖 2：例外處理模式

## 重要重點

需要記住的要點：

* 例外處理與復原對於建構穩健且可靠的代理至關重要。  
* 此模式包含偵測錯誤、優雅地處理錯誤，以及實作復原策略。  
* 錯誤偵測可以包含驗證工具輸出、檢查 API 錯誤碼，以及使用逾時。  
* 處理策略包括 logging、重試、備援、優雅降級與通知。  
* 復原著重於透過診斷、自我修正或升級處理來恢復穩定運作。  
* 此模式確保代理即使在不可預測的真實世界環境中，也能有效運作。

## 結論

本章探討例外處理與復原模式，這是開發穩健且可靠 AI 代理的關鍵。此模式處理的是 AI 代理如何識別並管理意外問題、實作適當回應，並復原到穩定營運狀態。本章討論此模式的各個面向，包括錯誤偵測、透過 logging、重試與備援等機制處理這些錯誤，以及用來讓代理或系統恢復正常功能的策略。例外處理與復原模式的實際應用橫跨多個領域，用以展示它在處理真實世界複雜性與潛在失敗時的相關性。這些應用顯示，為 AI 代理配備例外處理能力，有助於提升它們在動態環境中的可靠性與適應能力。

## 參考資料

1. McConnell, S. (2004). *Code Complete (2nd ed.)*. Microsoft Press.
2. Shi, Y., Pei, H., Feng, L., Zhang, Y., & Yao, D. (2024). *Towards Fault Tolerance in Multi-Agent Reinforcement Learning*. arXiv preprint arXiv:2412.00534.
3. O'Neill, V. (2022). *Improving Fault Tolerance and Reliability of Heterogeneous Multi-Agent IoT Systems Using Intelligence Transfer*. Electronics, 11(17), 2724.
