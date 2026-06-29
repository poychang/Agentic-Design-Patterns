# 附錄 D - 使用 AgentSpace 建置代理

## 概覽

AgentSpace 是一個平台，設計目標是透過將人工智慧整合到日常工作流程中，促成「代理驅動的企業」。它的核心是提供橫跨組織整個數位足跡的統一搜尋能力，包括文件、電子郵件與資料庫。此系統運用進階 AI 模型，例如 Google 的 Gemini，來理解並合成來自這些不同來源的資訊。

此平台能建立並部署專門的 AI「代理（agent）」，這些代理可以執行複雜任務並自動化流程。這些代理不只是聊天機器人；它們可以自主推理、規劃並執行多步驟動作。例如，代理可以研究某個主題、編寫附有引用來源的報告，甚至產生音訊摘要。

為了達成這一點，AgentSpace 會建構企業知識圖譜，對人員、文件與資料之間的關係進行映射。這讓 AI 能理解上下文，並提供更相關且個人化的結果。此平台也包含名為 Agent Designer 的 no-code 介面，可在不需要深厚技術專業的情況下建立自訂代理。

此外，AgentSpace 支援多代理系統，不同 AI 代理可以透過名為 Agent2Agent (A2A) Protocol 的開放協定進行通訊與協作。這種互通性允許更複雜且經過協調的工作流程。安全性是基礎組成要素，具備角色型存取控制（role-based access controls）與資料加密等功能，以保護敏感的企業資訊。最終，AgentSpace 的目標是將智慧型自主系統直接嵌入組織的營運結構中，以提升生產力與決策品質。

## 如何使用 AgentSpace UI 建置代理

Fig. 1 說明如何從 Google Cloud Console 選取 AI Applications 來存取 AgentSpace。

![GCP: Access AgentSpace](../assets/GCP_Access_AgentSpace.png)

Fig. 1:  如何使用 Google Cloud Console 存取 AgentSpace

你的代理可以連接到各種服務，包括 Calendar、Google Mail、Workaday、Jira、Outlook 與 Service Now（見 Fig. 2）。

![GCP: Integrate with diverse services](../assets/GCP_Integrate_with_diverse_services.png)

Fig. 2: 與各種服務整合，包括 Google 與第三方平台。

接著，代理可以使用自己的提示，從 Google 提供的預製提示圖庫中選擇，如 Fig. 3 所示。

![GCP: Googles Gallery of Pre-Assembled Prompts](../assets/GCP_Googles_Gallery_of_Pre_Assembled_Prompts.png)

Fig.3: Google 的預組裝提示圖庫

或者，你也可以建立自己的提示，如 Fig.4 所示，之後會由你的代理使用。  

![GCP: Customizing the Agent's Prompt](../assets/GCP_Customizing_the_Agents_Prompt.png)

Fig.4: 自訂代理的提示

AgentSpace 提供多項進階功能，例如與資料存放區（datastores）整合以儲存你自己的資料、與 Google Knowledge Graph 或你的私有 Knowledge Graph 整合、用於將代理公開到 Web 的 Web 介面，以及用於監控使用情況的 Analytics 等（見 Fig. 5）。

![GCP: AgentSpace Advanced Capabilities](../assets/GCP_AgentSpace_Advanced_Capabilities.png)

Fig. 5: AgentSpace 進階能力

完成後，即可存取 AgentSpace 聊天介面（Fig. 6）。

![GCP: AgentSpace User Interface for initiating a chat with your Agent](../assets/GCP_AgentSpace_User_Interface_for_initiating_a_chat_with_your_Agent.png)

Fig. 6: 用於與你的代理啟動聊天的 AgentSpace 使用者介面。

## 結論

總結來說，AgentSpace 提供一個功能性框架，用於在組織既有數位基礎設施中開發與部署 AI 代理。此系統的架構會將自主推理與企業知識圖譜映射等複雜後端流程，連結到用於建構代理的圖形化使用者介面。透過此介面，使用者可以整合各種資料服務，並透過提示定義代理的操作參數，進而設定代理，產生自訂且具上下文感知能力的自動化系統。

這種方法抽象化了底層技術複雜度，使使用者能在不需要深厚程式設計專業的情況下，建構專門的多代理系統。主要目標是將自動化分析與操作能力直接嵌入工作流程，藉此提升流程效率並強化資料驅動分析。若需要實務教學，可以使用實作學習模組（hands-on learning modules），例如 Google Cloud Skills Boost 上的「Build a Gen AI Agent with Agentspace」lab，該 lab 提供結構化環境以取得技能。

## 參考資料

1. Create a no-code agent with Agent Designer, [https://cloud.google.com/agentspace/agentspace-enterprise/docs/agent-designer](https://cloud.google.com/agentspace/agentspace-enterprise/docs/agent-designer)
2. Google Cloud Skills Boost, [https://www.cloudskillsboost.google/](https://www.cloudskillsboost.google/)
