# 附錄 E - CLI 上的 AI 代理（agent）

## 引言

開發者的命令列長久以來一直是精確、命令式指令的堡壘，如今正經歷深刻轉變。它正從單純的 shell，演進為由新一類工具驅動的智慧協作工作區：AI Agent Command-Line Interfaces（CLIs）。這些代理不再只是執行命令；它們能理解自然語言、維持對整個程式碼庫的上下文，並執行複雜的多步驟任務，自動化開發生命週期中的重要部分。

本指南深入介紹這個快速興起領域中的四個領先工具，探討它們各自的優勢、理想使用案例與不同理念，協助你判斷哪個工具最符合你的工作流程。需要注意的是，許多針對特定工具提供的範例使用案例，通常也能由其他代理完成。這些工具之間的關鍵差異，往往在於它們針對特定任務所能達成的結果品質、效率與細膩程度。已有專門設計來衡量這些能力的基準測試，後續章節將進一步討論。

## Claude CLI (Claude Code)

Anthropic 的 Claude CLI 被設計為高階 coding agent，能深入且整體性地理解專案架構。它的核心優勢在於其「代理式」特質，能為複雜的多步驟任務建立對儲存庫的心智模型。互動方式高度對話化，類似 pair programming session，會在執行前先說明計畫。這使它非常適合專業開發者處理大型專案，尤其是涉及重大重構或實作會廣泛影響架構的功能時。

**範例使用案例：**

1. **大規模重構：** 你可以指示它：「Our current user authentication relies on session cookies. Refactor the entire codebase to use stateless JWTs, updating the login/logout endpoints, middleware, and frontend token handling.」Claude 接著會讀取所有相關檔案，並執行協調一致的變更。  
2. **API 整合：** 在提供新 weather service 的 OpenAPI specification 後，你可以說：「Integrate this new weather API. Create a service module to handle the API calls, add a new component to display the weather, and update the main dashboard to include it.」  
3. **文件產生**：指向一個文件不足的複雜模組時，你可以要求：「Analyze the `./src/utils/data_processing.js` file. Generate comprehensive TSDoc comments for every function, explaining its purpose, parameters, and return value.」

Claude CLI 是專門的 coding assistant，內建核心開發任務所需的工具，包括檔案擷取、程式碼結構分析與編輯產生。它與 Git 的深度整合有助於直接管理 branch 與 commit。此代理的擴充性由 Multi-tool Control Protocol (MCP) 介接，讓使用者能定義並整合自訂工具。這使它能與私有 API 互動、查詢資料庫，以及執行專案特定腳本。這種架構讓開發者成為代理功能範圍的裁定者，實質上也將 Claude 定位為由使用者定義工具強化的推理引擎。

## Gemini CLI

Google 的 Gemini CLI 是多用途、開源的 AI 代理，兼具能力與易用性。它以先進的 Gemini 2.5 Pro model、巨大的上下文視窗，以及多模態能力（處理影像與文字）脫穎而出。它的開源特性、慷慨的免費方案，以及「Reason and Act」迴圈，使它成為透明、可控且優秀的全方位工具，適用於從愛好者到企業開發者的廣泛受眾，尤其適合 Google Cloud 生態系中的使用者。

**範例使用案例：**

1. **多模態開發：** 你提供設計檔中某個網頁元件的截圖（gemini describe component.png），並指示它：「Write the HTML and CSS code to build a React component that looks exactly like this. Make sure it's responsive.」  
2. **雲端資源管理：** 使用其內建 Google Cloud 整合時，你可以下達命令：「Find all GKE clusters in the production project that are running versions older than 1.28 and generate a gcloud command to upgrade them one by one.」  
3. **企業工具整合（透過 MCP）：** 開發者提供 Gemini 一個名為 get-employee-details 的自訂工具，可連接公司的內部 HR API。提示是：「Draft a welcome document for our new hire. First, use the get-employee-details --id=E90210 tool to fetch their name and team, and then populate the welcome_template.md with that information.」  
4. **大規模重構**：開發者需要重構大型 Java codebase，將已棄用的 logging library 替換為新的 structured logging framework。他們可以搭配 Gemini 使用這樣的提示：Read all *.java files in the 'src/main/java' directory. For each file, replace all instances of the 'org.apache.log4j' import and its 'Logger' class with 'org.slf4j.Logger' and 'LoggerFactory'. Rewrite the logger instantiation and all .info(), .debug(), and .error() calls to use the new structured format with key-value pairs.

Gemini CLI 配備一組內建工具，使它能與環境互動。這些工具包括檔案系統操作工具（例如讀取與寫入）、執行命令的 shell tool，以及透過 web fetching 與 searching 存取網際網路的工具。為了取得更廣泛的上下文，它使用專門工具一次讀取多個檔案，並使用 memory tool 保存資訊供後續 session 使用。這些功能建立在安全基礎之上：sandboxing 會隔離模型的動作以避免風險，而 MCP servers 則作為橋接，讓 Gemini 能安全連接到你的本機環境或其他 API。

## Aider

Aider 是開源 AI coding assistant，會直接處理你的檔案並將變更 commit 到 Git，像真正的 pair programmer 一樣工作。它的標誌性特點是直接；它會套用編輯、執行測試來驗證，並自動 commit 每一個成功的變更。由於不綁定特定模型，它讓使用者完整掌控成本與能力。它以 git 為中心的工作流程，非常適合重視效率、控制力，以及所有程式碼修改都有透明且可稽核軌跡的開發者。

**範例使用案例：**

1. **測試驅動開發（TDD）：** 開發者可以說：「Create a failing test for a function that calculates the factorial of a number.」Aider 寫出測試且測試失敗後，下一個提示是：「Now, write the code to make the test pass.」Aider 會實作該函式，並再次執行測試確認。  
2. **精準修復 bug：** 給定 bug report 後，你可以指示 Aider：「The `calculate_total` function in billing.py fails on leap years. Add the file to the context, fix the bug, and verify your fix against the existing test suite.」  
3. **相依性更新：** 你可以指示它：「Our project uses an outdated version of the 'requests' library. Please go through all Python files, update the import statements and any deprecated function calls to be compatible with the latest version, and then update requirements.txt.」

## GitHub Copilot CLI

GitHub Copilot CLI 將廣受歡迎的 AI pair programmer 延伸到終端機，其主要優勢在於與 GitHub 生態系的原生深度整合。它能理解 *GitHub 內* 專案的上下文。它的代理能力讓它可以被指派 GitHub issue、處理修正，並提交 pull request 供人類 review。

**範例使用案例：**

1. **自動化 issue 解決：** 管理者將 bug ticket（例如「Issue #123: Fix off-by-one error in pagination」）指派給 Copilot 代理。該代理接著會 checkout 新 branch、撰寫程式碼，並提交引用該 issue 的 pull request，全程不需要開發者手動介入。  
2. **具儲存庫感知的問答：** 團隊中的新開發者可以問：「Where in this repository is the database connection logic defined, and what environment variables does it require?」Copilot CLI 會利用它對整個 repo 的理解，提供包含檔案路徑的精確答案。  
3. **Shell 命令助手：** 當不確定複雜 shell command 時，使用者可以問：gh? find all files larger than 50MB, compress them, and place them in an archive folder. Copilot 會產生執行該任務所需的精確 shell command。

## Terminal-Bench：Command-Line Interfaces 中 AI 代理的基準測試

Terminal-Bench 是一個新穎的評估框架，旨在評估 AI 代理在 command-line interface 中執行複雜任務的熟練程度。由於終端機具備文字為基礎且可沙箱化的特性，因此被視為 AI 代理運作的理想環境。初始版本 Terminal-Bench-Core-v0 包含 80 個人工策劃的任務，涵蓋 scientific workflows 與 data analysis 等領域。為確保公平比較，研究者開發了極簡代理 Terminus，作為各種語言模型的標準化 testbed。此框架被設計為可擴充，可透過 containerization 或直接連線整合不同代理。未來發展包括啟用大規模平行評估，以及納入既有基準測試。此專案鼓勵開源貢獻，以擴充任務並協作強化框架。

## 結論

這些強大的 AI command-line agents 的出現，標誌著軟體開發的根本轉變，將終端機轉化為動態且協作的環境。如我們所見，沒有單一「最佳」工具；相反地，一個充滿活力的生態系正在形成，每個代理都提供專門優勢。理想選擇完全取決於開發者需求：Claude 適合複雜架構任務，Gemini 適合多用途與多模態問題解決，Aider 適合以 git 為中心且直接的程式碼編輯，而 GitHub Copilot 則適合無縫整合 GitHub workflow。隨著這些工具持續演進，熟練運用它們將成為必要技能，並從根本上改變開發者建置、偵錯與管理軟體的方式。

## 參考資料

1. Anthropic. *Claude*. [https://docs.anthropic.com/en/docs/claude-code/cli-reference](https://docs.anthropic.com/en/docs/claude-code/cli-reference)
2. Google Gemini Cli [https://github.com/google-gemini/gemini-cli](https://github.com/google-gemini/gemini-cli)
3. Aider. [https://aider.chat/](https://aider.chat/)  
4. GitHub *Copilot CLI* [https://docs.github.com/en/copilot/github-copilot-enterprise/copilot-cli](https://docs.github.com/en/copilot/github-copilot-enterprise/copilot-cli)  
5. Terminal Bench: [https://www.tbench.ai/](https://www.tbench.ai/)
