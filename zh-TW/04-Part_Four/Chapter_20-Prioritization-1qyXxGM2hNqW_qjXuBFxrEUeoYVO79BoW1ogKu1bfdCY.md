# 第 20 章：優先排序

在複雜且動態的環境中，代理（agent）經常會遇到許多可能採取的行動、彼此衝突的目標，以及有限的資源。若沒有明確流程來決定下一步行動，代理可能會效率下降、作業延誤，或無法達成關鍵目標。優先排序模式透過讓代理根據重要性、急迫性、相依關係與既定準則，評估並排序任務、目標或行動，來解決這個問題。這能確保代理把心力集中在最關鍵的任務上，進而提升效能並讓行動與目標保持一致。

## 優先排序模式概觀

代理會使用優先排序來有效管理任務、目標與子目標，並引導後續行動。當代理面對多重需求時，這個流程能促成更有依據的決策，讓重要或急迫的活動優先於較不關鍵的事項。這在真實世界情境中特別相關，因為資源通常受限、時間有限，且目標之間可能彼此衝突。

代理優先排序的基本面向通常包含幾個元素。第一，準則定義會建立任務評估所需的規則或指標。這些準則可能包含急迫性（任務的時間敏感度）、重要性（對主要目標的影響）、相依關係（該任務是否是其他任務的前置條件）、資源可用性（必要工具或資訊是否就緒）、成本／效益分析（投入心力與預期成果的比較），以及個人化代理所需的使用者偏好。第二，任務評估會根據這些已定義的準則評估每個潛在任務，方法可從簡單規則，到由 LLM 進行複雜評分或推理不等。第三，排程或選擇邏輯是指根據評估結果，選出最佳下一步行動或任務序列的演算法，可能會使用佇列或進階規劃元件。最後，動態重新優先排序允許代理在情況改變時調整優先順序，例如出現新的關鍵事件或截止期限逼近，以確保代理具備適應性與回應能力。

優先排序可以發生在不同層級：選擇整體目標（高階目標優先排序）、排列計畫中的步驟（子任務優先排序），或從可用選項中選擇下一個立即行動（行動選擇）。有效的優先排序能讓代理展現更智慧、更有效率且更穩健的行為，尤其是在複雜的多目標環境中。這與人類團隊的組織方式相呼應：管理者會考量所有成員的意見來排定任務優先順序。

## 實務應用與使用案例

在各種真實世界應用中，AI 代理會展現出成熟的優先排序能力，以做出及時且有效的決策。

* **自動化客戶支援**：代理會將系統中斷回報等緊急請求，優先於密碼重設等例行事項處理。它們也可能會優先服務高價值客戶。  
* **雲端運算**：AI 會管理並排程資源，在尖峰需求期間優先將資源分配給關鍵應用程式，同時把較不緊急的批次工作移至離峰時段，以最佳化成本。  
* **自動駕駛系統**：持續為行動排定優先順序，以確保安全與效率。例如，為避免碰撞而煞車，會優先於維持車道紀律或最佳化燃油效率。  
* **金融交易**：機器人會分析市場條件、風險承受度、利潤率與即時新聞等因素來排定交易優先順序，使高優先順序交易能迅速執行。  
* **專案管理**：AI 代理會根據截止期限、相依關係、團隊可用性與策略重要性，為專案看板上的任務排定優先順序。  
* **網路安全**：監控網路流量的代理會評估威脅嚴重性、潛在影響與資產關鍵性，為警示排定優先順序，確保能立即回應最危險的威脅。  
* **個人助理 AI**：使用優先排序來管理日常生活，依據使用者定義的重要性、即將到來的截止期限與目前上下文，整理行事曆事件、提醒與通知。

這些例子共同說明，在各種情境中，排定優先順序的能力是提升 AI 代理效能與決策能力的基礎。

## 實作程式碼範例

以下示範如何使用 LangChain 開發一個專案經理 AI 代理。此代理能協助建立任務、排定優先順序，並將任務指派給團隊成員，展現大型語言模型（large language model, LLM）結合客製工具在自動化專案管理中的應用。

```python
import os
import asyncio
from typing import List, Optional, Dict, Type

from dotenv import load_dotenv
from pydantic import BaseModel, Field
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.tools import Tool
from langchain_openai import ChatOpenAI
from langchain.agents import AgentExecutor, create_react_agent
from langchain.memory import ConversationBufferMemory


# --- 0. Configuration and Setup ---
# Loads the OPENAI_API_KEY from the .env file.
load_dotenv()

# The ChatOpenAI client automatically picks up the API key from the environment.
llm = ChatOpenAI(temperature=0.5, model="gpt-4o-mini")


# --- 1. Task Management System ---
class Task(BaseModel):
    """Represents a single task in the system."""
    id: str
    description: str
    priority: Optional[str] = None  # P0, P1, P2
    assigned_to: Optional[str] = None  # Name of the worker


class SuperSimpleTaskManager:
    """An efficient and robust in-memory task manager."""

    def __init__(self):
        # Use a dictionary for O(1) lookups, updates, and deletions.
        self.tasks: Dict[str, Task] = {}
        self.next_task_id = 1

    def create_task(self, description: str) -> Task:
        """Creates and stores a new task."""
        task_id = f"TASK-{self.next_task_id:03d}"
        new_task = Task(id=task_id, description=description)
        self.tasks[task_id] = new_task
        self.next_task_id += 1
        print(f"DEBUG: Task created - {task_id}: {description}")
        return new_task

    def update_task(self, task_id: str, **kwargs) -> Optional[Task]:
        """Safely updates a task using Pydantic's model_copy."""
        task = self.tasks.get(task_id)
        if task:
            # Use model_copy for type-safe updates.
            update_data = {k: v for k, v in kwargs.items() if v is not None}
            updated_task = task.model_copy(update=update_data)
            self.tasks[task_id] = updated_task
            print(f"DEBUG: Task {task_id} updated with {update_data}")
            return updated_task

        print(f"DEBUG: Task {task_id} not found for update.")
        return None

    def list_all_tasks(self) -> str:
        """Lists all tasks currently in the system."""
        if not self.tasks:
            return "No tasks in the system."

        task_strings = []
        for task in self.tasks.values():
            task_strings.append(
                f"ID: {task.id}, Desc: '{task.description}', "
                f"Priority: {task.priority or 'N/A'}, "
                f"Assigned To: {task.assigned_to or 'N/A'}"
            )
        return "Current Tasks:\n" + "\n".join(task_strings)


task_manager = SuperSimpleTaskManager()


# --- 2. Tools for the Project Manager Agent ---
# Use Pydantic models for tool arguments for better validation and clarity.
class CreateTaskArgs(BaseModel):
    description: str = Field(description="A detailed description of the task.")


class PriorityArgs(BaseModel):
    task_id: str = Field(description="The ID of the task to update, e.g., 'TASK-001'.")
    priority: str = Field(description="The priority to set. Must be one of: 'P0', 'P1', 'P2'.")


class AssignWorkerArgs(BaseModel):
    task_id: str = Field(description="The ID of the task to update, e.g., 'TASK-001'.")
    worker_name: str = Field(description="The name of the worker to assign the task to.")


def create_new_task_tool(description: str) -> str:
    """Creates a new project task with the given description."""
    task = task_manager.create_task(description)
    return f"Created task {task.id}: '{task.description}'."


def assign_priority_to_task_tool(task_id: str, priority: str) -> str:
    """Assigns a priority (P0, P1, P2) to a given task ID."""
    if priority not in ["P0", "P1", "P2"]:
        return "Invalid priority. Must be P0, P1, or P2."
    task = task_manager.update_task(task_id, priority=priority)
    return f"Assigned priority {priority} to task {task.id}." if task else f"Task {task_id} not found."


def assign_task_to_worker_tool(task_id: str, worker_name: str) -> str:
    """Assigns a task to a specific worker."""
    task = task_manager.update_task(task_id, assigned_to=worker_name)
    return f"Assigned task {task.id} to {worker_name}." if task else f"Task {task_id} not found."


# All tools the PM agent can use
pm_tools = [
    Tool(
        name="create_new_task",
        func=create_new_task_tool,
        description="Use this first to create a new task and get its ID.",
        args_schema=CreateTaskArgs
    ),
    Tool(
        name="assign_priority_to_task",
        func=assign_priority_to_task_tool,
        description="Use this to assign a priority to a task after it has been created.",
        args_schema=PriorityArgs
    ),
    Tool(
        name="assign_task_to_worker",
        func=assign_task_to_worker_tool,
        description="Use this to assign a task to a specific worker after it has been created.",
        args_schema=AssignWorkerArgs
    ),
    Tool(
        name="list_all_tasks",
        func=task_manager.list_all_tasks,
        description="Use this to list all current tasks and their status."
    ),
]


# --- 3. Project Manager Agent Definition ---
pm_prompt_template = ChatPromptTemplate.from_messages([
    ("system", """You are a focused Project Manager LLM agent. Your goal is to manage project tasks efficiently.
      When you receive a new task request, follow these steps:
    1.  First, create the task with the given description using the `create_new_task` tool. You must do this first to get a `task_id`.
    2.  Next, analyze the user's request to see if a priority or an assignee is mentioned.
        - If a priority is mentioned (e.g., "urgent", "ASAP", "critical"), map it to P0. Use `assign_priority_to_task`.
        - If a worker is mentioned, use `assign_task_to_worker`.
    3.  If any information (priority, assignee) is missing, you must make a reasonable default assignment (e.g., assign P1 priority and assign to 'Worker A').
    4.  Once the task is fully processed, use `list_all_tasks` to show the final state.

    Available workers: 'Worker A', 'Worker B', 'Review Team'
    Priority levels: P0 (highest), P1 (medium), P2 (lowest)
    """),
    ("placeholder", "{chat_history}"),
    ("human", "{input}"),
    ("placeholder", "{agent_scratchpad}")
])

# Create the agent executor
pm_agent = create_react_agent(llm, pm_tools, pm_prompt_template)
pm_agent_executor = AgentExecutor(
    agent=pm_agent,
    tools=pm_tools,
    verbose=True,
    handle_parsing_errors=True,
    memory=ConversationBufferMemory(memory_key="chat_history", return_messages=True)
)


# --- 4. Simple Interaction Flow ---
async def run_simulation():
    print("--- Project Manager Simulation ---")

    # Scenario 1: Handle a new, urgent feature request
    print("\n[User Request] I need a new login system implemented ASAP. It should be assigned to Worker B.")
    await pm_agent_executor.ainvoke({"input": "Create a task to implement a new login system. It's urgent and should be assigned to Worker B."})

    print("\n" + "-" * 60 + "\n")

    # Scenario 2: Handle a less urgent content update with fewer details
    print("[User Request] We need to review the marketing website content.")
    await pm_agent_executor.ainvoke({"input": "Manage a new task: Review marketing website content."})

    print("\n--- Simulation Complete ---")


# Run the simulation
if __name__ == "__main__":
    asyncio.run(run_simulation())
```

這段程式碼使用 Python 與 LangChain 實作一個簡單的任務管理系統，用來模擬由大型語言模型驅動的專案經理代理。

此系統使用 SuperSimpleTaskManager 類別在記憶體中有效率地管理任務，並透過字典結構快速擷取資料。每個任務都由 Task Pydantic 模型表示，包含唯一識別碼、描述文字、選填的優先層級（P0、P1、P2），以及選填的指派對象。記憶體使用量會依任務類型、工作者數量與其他影響因素而有所不同。任務管理器提供建立任務、修改任務，以及擷取所有任務的方法。

代理透過一組已定義的 Tools 與任務管理器互動。這些工具可用於建立新任務、為任務指派優先順序、將任務分配給人員，以及列出所有任務。每個工具都經過封裝，以便與 SuperSimpleTaskManager 的執行個體互動。系統使用 Pydantic 模型描述工具所需的引數，藉此確保資料驗證。

AgentExecutor 會搭配語言模型、工具集與對話記憶元件進行設定，以維持上下文連續性。程式也定義了特定的 ChatPromptTemplate，用來引導代理在專案管理角色中的行為。該提示指示代理先建立任務，再依指定內容指派優先順序與人員，最後輸出完整任務清單。若資訊缺漏，提示中也規定了預設指派方式，例如 P1 優先順序與 'Worker A'。

程式碼包含一個非同步性質的模擬函式（`run_simulation`），用來示範代理的操作能力。此模擬會執行兩個不同情境：管理一項已指定人員的緊急任務，以及在輸入資訊極少的情況下管理一項較不緊急的任務。由於 AgentExecutor 中啟用了 verbose=True，代理的動作與邏輯流程會輸出到主控台。

# 快速概覽

**內容：** 在複雜環境中運作的 AI 代理會面對大量可能行動、彼此衝突的目標與有限資源。若沒有清楚的方法來決定下一步，這些代理可能變得效率低落且成效不佳，導致重大作業延誤，甚至完全無法達成主要目標。核心挑戰在於管理這些龐大的選項，確保代理能有目的且合乎邏輯地行動。

**原因：** 優先排序模式讓代理能夠排序任務與目標，為這個問題提供標準化解法。它會建立急迫性、重要性、相依關係與資源成本等明確準則，再讓代理根據這些準則評估每個可能行動，以判斷最關鍵且最及時的行動方案。這種代理式能力使系統能動態適應變化中的情況，並有效管理受限資源。透過聚焦在最高優先順序的項目上，代理的行為會變得更智慧、更穩健，也更符合其策略目標。

**經驗法則：** 當代理式系統必須在資源受限的情況下，自主管理多個且經常彼此衝突的任務或目標，才能在動態環境中有效運作時，就使用優先排序模式。

**視覺摘要：**

**![優先排序設計模式](../assets/Prioritization_Design_Pattern.png )

圖 1：優先排序設計模式

# 重點整理

* 優先排序讓 AI 代理能在複雜且多面向的環境中有效運作。  
* 代理會使用急迫性、重要性與相依關係等既定準則來評估並排序任務。  
* 動態重新優先排序讓代理能根據即時變化調整其作業焦點。
* 優先排序會發生在不同層級，涵蓋整體策略目標與立即戰術決策。
* 有效的優先排序能提升 AI 代理的效率與作業穩健性。

# 結論

總而言之，優先排序模式是有效代理式 AI 的基石，使系統能以目的性與智慧應對動態環境的複雜性。它讓代理能自主評估大量彼此衝突的任務與目標，並針對有限資源應該投入何處做出有理據的決策。這種代理式能力超越了單純的任務執行，使系統能扮演主動且具策略性的決策者。透過權衡急迫性、重要性與相依關係等準則，代理展現出成熟且近似人類的推理流程。

這種代理式行為的一項關鍵特徵是動態重新優先排序，它賦予代理在條件改變時即時調整焦點的自主性。如程式碼範例所示，代理會解讀模糊請求、自主選擇並使用適當工具，並以合乎邏輯的順序安排其行動來完成目標。這種自我管理工作流程的能力，正是區分真正代理式系統與簡單自動化腳本的要素。最終，掌握優先排序，是建立穩健且智慧的代理之基礎，讓它們能在任何複雜的真實世界情境中有效且可靠地運作。

# 參考資料

1. Examining the Security of Artificial Intelligence in Project Management: A Case Study of AI-driven Project Scheduling and Resource Allocation in Information Systems Projects ; [https://www.irejournals.com/paper-details/1706160](https://www.irejournals.com/paper-details/1706160)
2. AI-Driven Decision Support Systems in Agile Software Project Management: Enhancing Risk Mitigation and Resource Allocation; [https://www.mdpi.com/2079-8954/13/3/208](https://www.mdpi.com/2079-8954/13/3/208)  
