# 第 9 章：學習與適應

學習與適應對於提升人工智慧代理的能力至關重要。這些過程讓代理能超越預先定義的參數，透過經驗與環境互動自主改進。藉由學習與適應，代理可以有效處理新情境，並在不需要持續人工介入的情況下最佳化自身效能。本章將詳細探討支撐代理學習與適應的原則與機制。

## 全貌

代理會根據新的經驗與資料，改變其思考方式、行動或知識，藉此學習與適應。這讓代理能從單純遵循指令，逐步演進為會隨時間變得更聰明的系統。

* **強化學習：** 代理嘗試不同動作，並因正向結果獲得獎勵、因負向結果受到懲罰，進而在變動情境中學會最佳行為。這對控制機器人或玩遊戲的代理很有用。  
* **監督式學習：** 代理從已標記的範例中學習，建立輸入與期望輸出之間的關聯，進而支援決策與模式辨識等任務。這很適合用於分類電子郵件或預測趨勢的代理。  
* **非監督式學習：** 代理在未標記資料中發現隱藏的連結與模式，有助於洞察、組織資料，並建立對環境的心理地圖。這對在沒有特定指引下探索資料的代理很有用。  
* **LLM 型代理的少樣本／零樣本學習：** 運用 LLM 的代理能透過少量範例或清楚指令，快速適應新任務，進而迅速回應新的命令或情境。  
* **線上學習：** 代理會持續用新資料更新知識，這對動態環境中的即時反應與持續適應非常重要。對處理連續資料串流的代理而言尤其關鍵。  
* **基於記憶的學習：** 代理會回想過去經驗，以便在類似情境中調整目前行動，提升上下文感知與決策品質。這對具備記憶回想能力的代理很有效。

代理會根據學習成果改變策略、理解或目標，藉此完成適應。對處於不可預測、持續變動或全新環境中的代理而言，這一點非常重要。

**Proximal Policy Optimization (PPO)** 是一種強化學習演算法，用於在具有連續動作範圍的環境中訓練代理，例如控制機器人的關節，或控制遊戲中的角色。它的主要目標，是可靠且穩定地改善代理的決策策略，也就是其 policy。

PPO 的核心概念，是對代理的 policy 進行小幅、謹慎的更新。它會避免可能導致效能崩潰的劇烈變更。其運作方式如下：

1. 收集資料：代理使用目前的 policy 與環境互動（例如玩遊戲），並收集一批經驗（狀態、動作、獎勵）。  
2. 評估「替代」目標：PPO 會計算潛在的 policy 更新會如何改變預期獎勵。不過，它不只是最大化這個獎勵，而是使用一個特殊的「裁剪」目標函數。  
3. 「裁剪」機制：這是 PPO 穩定性的關鍵。它會在目前 policy 周圍建立一個「信任區域」或安全區。演算法會被防止進行與目前策略差異過大的更新。這種裁剪就像安全煞車，確保代理不會採取巨大且高風險的一步，導致先前學到的成果被抵消。

簡而言之，PPO 會在提升效能與維持接近已知可行策略之間取得平衡，避免訓練期間發生災難性失敗，並帶來更穩定的學習。

**Direct Preference Optimization (DPO)** 是一種較新的方法，專門設計用來讓大型語言模型（large language model, LLM）與人類偏好對齊。相較於使用 PPO 來完成這項任務，DPO 提供了更簡單且更直接的替代方案。

若要理解 DPO，先理解傳統以 PPO 為基礎的對齊方法會很有幫助：

* PPO 方法（兩步驟流程）：  
  1. 訓練獎勵模型：首先，你會收集人類回饋資料，讓人們評分或比較不同的 LLM 回應（例如：「回應 A 比回應 B 更好」）。這些資料會用來訓練另一個 AI 模型，稱為獎勵模型；它的工作是預測人類會給任何新回應多少分。  
  2. 使用 PPO 進行微調：接著，使用 PPO 對 LLM 進行微調。LLM 的目標是產生能從獎勵模型取得最高分的回應。獎勵模型在訓練遊戲中扮演「裁判」。

這個兩步驟流程可能很複雜且不穩定。例如，LLM 可能找到漏洞，學會「操弄」獎勵模型，讓不佳的回應也能取得高分。

* DPO 方法（直接流程）：DPO 完全跳過獎勵模型。它不是先把人類偏好轉換成獎勵分數，再針對該分數最佳化，而是直接使用偏好資料來更新 LLM 的 policy。  
* 它透過一種數學關係運作，直接把偏好資料連結到最佳 policy。本質上，它是在教模型：「提高產生類似*偏好*回應的機率，並降低產生類似*不受偏好*回應的機率。」

本質上，DPO 透過直接在人類偏好資料上最佳化語言模型，簡化了對齊流程。這避免了訓練與使用獨立獎勵模型的複雜度與潛在不穩定性，使對齊流程更有效率且更穩健。

## 實務應用與使用案例

適應型代理會透過經驗資料驅動的反覆更新，在多變環境中展現更好的效能。

* **個人化助理代理** 會透過長期分析個別使用者行為，精進互動協議，確保產生高度最佳化的回應。  
* **交易機器人代理** 會根據高解析度的即時市場資料動態調整模型參數，最佳化決策演算法，進而最大化財務報酬並降低風險因素。  
* **應用程式代理** 會根據觀察到的使用者行為進行動態修改，最佳化使用者介面與功能，進而提高使用者參與度與系統直覺性。  
* **機器人與自動駕駛車輛代理** 會整合感測器資料與歷史行動分析，強化導航與反應能力，使其能在多樣化環境條件下安全且有效率地運作。  
* **詐欺偵測代理** 會使用新識別出的詐欺模式精進預測模型，改善異常偵測能力，強化系統安全並將財務損失降至最低。  
* **推薦代理** 會運用使用者偏好學習演算法，提高內容選擇精準度，提供高度個人化且符合上下文的推薦。  
* **遊戲 AI 代理** 會動態調整策略演算法，提升玩家參與度，並增加遊戲複雜度與挑戰性。  
* **知識庫學習代理**：代理可以運用檢索增強生成（retrieval-augmented generation, RAG）來維護問題描述與已驗證解法的動態知識庫（請參閱第 14 章）。透過儲存成功策略與曾遇到的挑戰，代理可以在決策期間參照這些資料，套用過去成功的模式或避開已知陷阱，進而更有效地適應新情境。

## 案例研究：自我改進的編碼代理（SICA）

由 Maxime Robeyns、Laurence Aitchison 與 Martin Szummer 開發的 Self-Improving Coding Agent (SICA)，代表了代理式學習的一項進展，展現代理修改自身原始碼的能力。這與一個代理訓練另一個代理的傳統做法不同；SICA 同時扮演修改者與被修改的實體，反覆精進其程式碼庫，以提升在各種編碼挑戰中的表現。

SICA 的自我改進透過反覆循環運作（見圖 1）。一開始，SICA 會檢視其過去版本的封存，以及這些版本在基準測試上的表現。它會根據一個同時考量成功率、時間與運算成本的加權公式，選出效能分數最高的版本。接著，這個被選中的版本會進入下一輪自我修改。它會分析封存內容以找出可能的改進，然後直接修改自己的程式碼庫。修改後的代理會再接受基準測試，結果則記錄在封存中。這個流程會不斷重複，促成其直接從過去表現中學習。這種自我改進機制讓 SICA 能在不依賴傳統訓練典範的情況下演進自身能力。

![SICA's self-improvement, learning and adapting based on its past versions](../assets/SICAs_self_improvement_learning_and_adapting_based_on_its_past_versions.png)

圖 1：SICA 的自我改進，根據其過去版本進行學習與適應

SICA 經歷了顯著的自我改進，帶來程式碼編輯與導覽方面的進展。起初，SICA 使用基本的檔案覆寫方式來變更程式碼。之後，它發展出能進行更智慧、具上下文感知編輯的「Smart Editor」。這進一步演進為「Diff-Enhanced Smart Editor」，納入 diff 以支援目標式修改與基於模式的編輯，並加入「Quick Overwrite Tool」以降低處理需求。

SICA 進一步實作了「Minimal Diff Output Optimization」與「Context-Sensitive Diff Minimization」，使用抽象語法樹（Abstract Syntax Tree, AST）解析來提高效率。此外，它也加入了「SmartEditor Input Normalizer」。在導覽方面，SICA 獨立建立了「AST Symbol Locator」，使用程式碼的結構地圖（AST）來識別程式碼庫中的定義。後來，它又開發了「Hybrid Symbol Locator」，結合快速搜尋與 AST 檢查。此功能又透過「Optimized AST Parsing in Hybrid Symbol Locator」進一步最佳化，聚焦於相關程式碼區段，提升搜尋速度。（見圖 2）

![Performance across Iterations](../assets/Performance_across_Iterations.png)

圖 2：跨反覆迭代的效能。關鍵改進已標註其對應的工具或代理修改。（由 Maxime Robeyns、Martin Szummer、Laurence Aitchison 提供）

SICA 的架構包含一組基礎工具包，用於基本檔案操作、命令執行與算術計算。它也包含結果提交機制，以及呼叫專門子代理（編碼、問題解決與推理）的機制。這些子代理會分解複雜任務並管理 LLM 的上下文長度，特別是在延伸的改進循環期間。

一個非同步監督者，也就是另一個 LLM，會監控 SICA 的行為，識別迴圈或停滯等潛在問題。它會與 SICA 溝通，並可在必要時介入以停止執行。監督者會收到 SICA 行動的詳細報告，包括 callgraph，以及訊息與工具動作的記錄，以識別模式與低效率之處。

SICA 的 LLM 會以結構化方式，在其上下文視窗，也就是短期記憶中組織資訊，這對其運作至關重要。這個結構包含定義代理目標的 System Prompt、工具與子代理文件，以及系統指令。Core Prompt 則包含問題陳述或指令、開啟檔案的內容，以及目錄地圖。Assistant Messages 會記錄代理的逐步推理、工具與子代理呼叫紀錄和結果，以及監督者通訊。這種組織方式促進高效率的資訊流，強化 LLM 運作並降低處理時間與成本。起初，檔案變更會以 diff 形式記錄，只顯示修改內容並定期整併。

**SICA：程式碼一覽：** 深入 SICA 的實作可以看出支撐其能力的幾項關鍵設計選擇。如前所述，該系統以模組化架構建構，納入多個子代理，例如編碼代理、問題解決代理與推理代理。這些子代理由主代理呼叫，方式很像工具呼叫，用來分解複雜任務並有效管理上下文長度，特別是在那些延伸的後設改進反覆迭代期間。

此專案正在積極開發，目標是為有興趣在工具使用與其他代理式任務上對 LLM 進行訓練後處理的人，提供一個穩健框架。完整程式碼可在 [https://github.com/MaximeRobeyns/self_improving_coding_agent/](https://github.com/MaximeRobeyns/self_improving_coding_agent/) GitHub repository 進一步探索與貢獻。

為了安全性，該專案高度強調 Docker 容器化，這表示代理會在專用 Docker container 中執行。這是一項關鍵措施，因為它提供與主機的隔離；考量到代理能執行 shell commands，這能降低意外操作檔案系統等風險。

為了確保透明度與控制，系統透過互動式網頁提供強大的可觀測性，將 event bus 上的事件與代理的 callgraph 視覺化。這能讓使用者全面洞察代理的行動，檢視個別事件、閱讀監督者訊息，並摺疊子代理追蹤，以便更清楚理解。

就其核心智慧而言，代理框架支援整合來自不同提供者的 LLM，讓使用者能試驗不同模型，找出最適合特定任務的選擇。最後，一個關鍵元件是非同步監督者，也就是與主代理並行執行的 LLM。這個監督者會定期評估代理行為是否出現病態偏差或停滯，並可在必要時透過傳送通知，甚至取消代理執行來介入。它會收到系統狀態的詳細文字表示，包括 callgraph，以及由 LLM 訊息、工具呼叫與回應組成的事件串流，使其能偵測低效率模式或重複工作。

SICA 初始實作中的一項顯著挑戰，是提示 LLM 型代理在每次後設改進反覆迭代中，獨立提出新穎、創新、可行且有吸引力的修改。這項限制，特別是在促進 LLM 代理的開放式學習與真實創造力方面，仍是當前研究的關鍵探討領域。

## AlphaEvolve 與 OpenEvolve

**AlphaEvolve** 是由 Google 開發的 AI 代理，設計用於發現與最佳化演算法。它結合 LLM，特別是 Gemini models（Flash 與 Pro）、自動化評估系統，以及演化演算法框架。此系統旨在推進理論數學與實務運算應用。

AlphaEvolve 使用 Gemini models 的集成。Flash 用於產生廣泛的初始演算法提案，而 Pro 則提供更深入的分析與精煉。接著，系統會根據預先定義的標準自動評估並評分所提出的演算法。這項評估會提供回饋，用於反覆改善解法，進而產生最佳化且新穎的演算法。

在實務運算方面，AlphaEvolve 已部署於 Google 的基礎架構中。它已展現資料中心排程方面的改善，使全球運算資源使用量降低 0.7%。它也透過為即將推出的 Tensor Processing Units (TPUs) 中的 Verilog code 建議最佳化，對硬體設計做出貢獻。此外，AlphaEvolve 也加速了 AI 效能，包括讓 Gemini 架構中的核心 kernel 速度提升 23%，並對 FlashAttention 的低階 GPU instructions 達成最高 32.5% 的最佳化。

在基礎研究領域，AlphaEvolve 促成了矩陣乘法新演算法的發現，包括一種針對 4x4 複數值矩陣的方法，使用 48 次純量乘法，超越先前已知解法。在更廣泛的數學研究中，它在 75% 的案例中，重新發現了超過 50 個開放問題的既有最先進解法，並在 20% 的案例中改進了既有解法；其中的例子包括 kissing number problem 的進展。

**OpenEvolve** 是一種演化式編碼代理，運用 LLM（見圖 3）反覆最佳化程式碼。它協調由 LLM 驅動的程式碼生成、評估與選擇管線，持續強化適用於各種任務的程式。OpenEvolve 的一項關鍵特性，是它能演化整個程式碼檔案，而不僅限於單一函式。此代理設計具備多用途性，支援多種程式語言，並相容於任何 LLM 的 OpenAI-compatible APIs。此外，它納入多目標最佳化，允許彈性的 prompt engineering，並具備分散式評估能力，以有效率地處理複雜的編碼挑戰。

![OpenEvolve Architecture](../assets/OpenEvolve_Architecture.png)

圖 3：OpenEvolve 的內部架構由 controller 管理。此 controller 協調數個關鍵元件：program sampler、Program Database、Evaluator Pool 與 LLM Ensembles。其主要功能是促進這些元件的學習與適應流程，以提升程式碼品質。

這段程式碼片段使用 OpenEvolve library 對程式執行演化式最佳化。它使用初始程式、評估檔案與設定檔的路徑來初始化 OpenEvolve system。`evolve.run(iterations=1000)` 這一行會啟動演化流程，執行 1000 次反覆迭代，以找出改進後的程式版本。最後，它會印出在演化期間找到的最佳程式指標，並格式化到小數點後四位。

```python
from openevolve import OpenEvolve


# Initialize the system
evolve = OpenEvolve(
    initial_program_path="path/to/initial_program.py",
    evaluation_file="path/to/evaluator.py",
    config_path="path/to/config.yaml",
)

# Run the evolution
best_program = await evolve.run(iterations=1000)

print("Best program metrics:")
for name, value in best_program.metrics.items():
    print(f"  {name}: {value:.4f}")
```

## 快速總覽

**是什麼：** AI 代理經常在動態且不可預測的環境中運作，而預先編寫的邏輯並不足夠。當面對初始設計時未預期的新情境時，其效能可能下降。如果無法從經驗中學習，代理就無法隨時間最佳化策略或個人化互動。這種僵固性限制了它們的有效性，也使它們無法在複雜的真實世界情境中達成真正的自主性。

**為什麼：** 標準化解法是整合學習與適應機制，將靜態代理轉變為動態且不斷演進的系統。這讓代理能根據新資料與互動，自主精進其知識與行為。代理式系統可以使用各種方法，從強化學習到更進階的自我修改技術，如 Self-Improving Coding Agent (SICA) 所示。Google 的 AlphaEvolve 等進階系統則運用 LLM 與演化演算法，為複雜問題發現全新且更有效率的解法。透過持續學習，代理可以掌握新任務、提升效能，並在不需要持續人工重新編程的情況下適應變動條件。

**經驗法則：** 當你建構的代理必須在動態、不確定或持續演進的環境中運作時，請使用這個模式。對於需要個人化、持續效能改進，以及能自主處理新情境的應用程式而言，這個模式不可或缺。

**視覺摘要：**

![Learning and Adapting Pattern](../assets/Learning_and_Adapting_Pattern.png)

圖 4：學習與適應模式

## 重點整理

* 學習與適應，是指代理運用自身經驗，把事情做得更好，並處理新情境。  
* 「適應」是來自學習的可見變化，會反映在代理的行為或知識上。  
* SICA，也就是 Self-Improving Coding Agent，會根據過去表現修改自己的程式碼來進行自我改進。這催生了 Smart Editor 與 AST Symbol Locator 等工具。  
* 具備專門「子代理」與「監督者」，有助於這些自我改進系統管理大型任務並維持在正確方向上。  
* LLM 的「上下文視窗」設定方式（包含 system prompts、core prompts 與 assistant messages），對代理運作效率非常重要。  
* 這個模式對於需要在持續變動、不確定，或需要個人化處理的環境中運作的代理至關重要。  
* 建構會學習的代理，通常意味著要將它們連接到機器學習工具，並管理資料流動方式。  
* 配備基本編碼工具的代理系統，可以自主編輯自身，並因此提升在基準任務上的效能。  
* AlphaEvolve 是 Google 的 AI 代理，運用 LLM 與演化框架自主發現並最佳化演算法，顯著推進基礎研究與實務運算應用。

## 結論

本章探討了學習與適應在人工智慧中的關鍵角色。AI 代理會透過持續取得資料與累積經驗來提升效能。Self-Improving Coding Agent (SICA) 便是其中的例子，它能透過修改程式碼自主提升自身能力。

我們已回顧代理式 AI 的基本組成，包括架構、應用、規劃、多代理協作、記憶管理，以及學習與適應。學習原則對於多代理系統中的協同改進尤其重要。若要做到這一點，調校資料必須準確反映完整互動軌跡，捕捉每個參與代理各自的輸入與輸出。

這些元素促成了重大進展，例如 Google 的 AlphaEvolve。這套 AI 系統透過 LLM、自動化評估與演化方法，獨立發現並精煉演算法，推動科學研究與運算技術進步。這類模式可以組合起來，建構精密的 AI 系統。AlphaEvolve 這類發展顯示，由 AI 代理自主進行演算法發現與最佳化是可以達成的。

## 參考資料

1. Sutton, R. S., & Barto, A. G. (2018). *Reinforcement Learning: An Introduction*. MIT Press.
2. Goodfellow, I., Bengio, Y., & Courville, A. (2016). *Deep Learning*. MIT Press.
3. Mitchell, T. M. (1997). *Machine Learning*. McGraw-Hill.
4. **Proximal Policy Optimization Algorithms** by John Schulman, Filip Wolski, Prafulla Dhariwal, Alec Radford, and Oleg Klimov. 你可以在 arXiv 找到它：[https://arxiv.org/abs/1707.06347](https://arxiv.org/abs/1707.06347)
5. Robeyns, M., Aitchison, L., & Szummer, M. (2025). *A Self-Improving Coding Agent*. arXiv:2504.15228v2. [https://arxiv.org/pdf/2504.15228](https://arxiv.org/pdf/2504.15228)  [https://github.com/MaximeRobeyns/self_improving_coding_agent](https://github.com/MaximeRobeyns/self_improving_coding_agent)
6. AlphaEvolve blog, [https://deepmind.google/discover/blog/alphaevolve-a-gemini-powered-coding-agent-for-designing-advanced-algorithms/](https://deepmind.google/discover/blog/alphaevolve-a-gemini-powered-coding-agent-for-designing-advanced-algorithms/)
7. OpenEvolve, [https://github.com/codelion/openevolve](https://github.com/codelion/openevolve)
