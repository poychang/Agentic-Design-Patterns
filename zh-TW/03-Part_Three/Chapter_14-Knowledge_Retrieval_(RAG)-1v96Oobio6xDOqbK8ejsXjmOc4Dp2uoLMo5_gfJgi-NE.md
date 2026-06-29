# 第 14 章：知識檢索（RAG）

大型語言模型（large language model, LLM）在生成類人文字方面展現出相當強大的能力。然而，它們的知識庫通常侷限於訓練時所使用的資料，因此無法存取即時資訊、特定公司資料或高度專門的細節。知識檢索（RAG，亦即檢索增強生成（retrieval-augmented generation, RAG））正是為了解決這項限制。RAG 讓 LLM 能夠存取並整合外部、最新且與上下文相關的資訊，進而提升輸出內容的準確性、相關性與事實基礎。

對 AI 代理（agent）而言，這點至關重要，因為它讓代理能將行動與回應建立在即時、可驗證且超出靜態訓練資料的資料之上。這項能力使代理能準確執行複雜任務，例如存取最新公司政策以回答特定問題，或在下訂單前檢查目前庫存。透過整合外部知識，RAG 將代理從單純的對話者轉變為有效、資料驅動，且能執行有意義工作的工具。

## 知識檢索（RAG）模式概觀

知識檢索（RAG）模式會在 LLM 生成回應之前，讓它們能存取外部知識庫，因而大幅強化 LLM 的能力。RAG 不是只依賴模型內部預先訓練好的知識，而是讓 LLM 能像人類查閱書籍或搜尋網路一樣「查找」資訊。這個流程使 LLM 能提供更準確、更新且可驗證的答案。

當使用者向採用 RAG 的 AI 系統提出問題或給出提示時，查詢不會直接送進 LLM。相反地，系統會先在龐大的外部知識庫中搜尋相關資訊；這個知識庫就像一座高度組織化的文件、資料庫或網頁圖書館。這種搜尋不是簡單的關鍵字比對，而是能理解使用者意圖與字詞背後含義的「語意搜尋」。初始搜尋會取出最相關的資訊片段或「區塊」。這些擷取出的片段接著會被「增強」，也就是加入原始提示中，形成更豐富、資訊更充足的查詢。最後，這個增強後的提示會送給 LLM。藉由這些額外上下文，LLM 就能生成不只流暢自然，而且以擷取資料為事實基礎的回應。

RAG 框架提供多項重要優點。它讓 LLM 能存取最新資訊，進而克服靜態訓練資料的限制。這種方法也能透過將回應建立在可驗證資料上，降低「幻覺」風險，也就是生成錯誤資訊的風險。此外，LLM 可以運用公司內部文件或 wiki 中的專門知識。這個流程的一項關鍵優勢，是能提供「引用」，指出資訊的確切來源，進而提升 AI 回應的可信度與可驗證性。

若要充分理解 RAG 的運作方式，必須先了解幾個核心概念（見圖 1）：

### 嵌入（embedding）

在 LLM 的脈絡中，embedding 是文字的數值表示，例如單字、片語或整份文件。這些表示形式是向量，也就是一串數字。核心概念是把不同文字片段的語意與彼此關係捕捉到數學空間中。含義相近的單字或片語，在這個向量空間中的 embedding 也會彼此更接近。例如，想像一個簡單的 2D 圖表。單字 "cat" 可能以座標 (2, 3) 表示，而 "kitten" 會非常接近，位於 (2.1, 3.1)。相較之下，單字 "car" 可能有像 (8, 1) 這樣距離較遠的座標，反映出它不同的含義。實際上，這些 embedding 位於維度高得多的空間，可能有數百甚至數千個維度，因此能對語言進行非常細緻的理解。

### 文字相似度

文字相似度指的是衡量兩段文字有多相像。這可以是表層層次，也就是觀察字詞重疊程度（詞彙相似度）；也可以是更深入、以含義為基礎的層次。在 RAG 的脈絡中，文字相似度對於在知識庫中找出與使用者查詢相對應的最相關資訊至關重要。例如，請考慮這兩個句子："What is the capital of France?" 和 "Which city is the capital of France?"。雖然措辭不同，但它們問的是同一個問題。良好的文字相似度模型會辨識出這一點，即使兩個句子只有少數字詞相同，也會給它們較高的相似度分數。這通常會使用文字的 embedding 來計算。

### 語意相似度與距離

語意相似度是更進階的文字相似度形式，純粹聚焦於文字的含義與上下文，而不只是使用了哪些字詞。它旨在理解兩段文字是否傳達相同概念或想法。語意距離則與此相反；高語意相似度代表低語意距離，反之亦然。在 RAG 中，語意搜尋依賴尋找與使用者查詢語意距離最小的文件。例如，片語 "a furry feline companion" 與 "a domestic cat" 除了 "a" 之外沒有共同字詞。然而，理解語意相似度的模型會辨識出它們指的是同一件事，並判定兩者高度相似。這是因為它們的 embedding 在向量空間中會非常接近，表示語意距離很小。這就是讓 RAG 即使在使用者措辭與知識庫文字不完全相同時，仍能找到相關資訊的「智慧搜尋」。

![RAG Core Concept: Chunking, Embeddings, and Vector Database](../assets/RAG_Core_Concepts_Chunking_Embeddings_and_Vector_Database.png)

圖 1：RAG 核心概念：分塊、embedding 與向量資料庫

### 文件分塊

分塊是將大型文件拆解成較小、較易管理的片段，也就是「區塊」的流程。為了讓 RAG 系統有效運作，它不能把整份大型文件都餵給 LLM，而是處理這些較小的區塊。文件如何分塊，對於保留資訊的上下文與含義非常重要。例如，與其把 50 頁的使用手冊視為單一文字區塊，分塊策略可能會將它拆成章節、段落，甚至句子。例如，"Troubleshooting" 章節會是與 "Installation Guide" 不同的獨立區塊。當使用者詢問特定問題時，RAG 系統就可以擷取最相關的疑難排解區塊，而不是整份手冊。這讓擷取流程更快，也讓提供給 LLM 的資訊更聚焦、更符合使用者當下需求。文件分塊後，RAG 系統必須採用擷取技術，為特定查詢找出最相關的片段。主要方法是向量搜尋，它使用 embedding 與語意距離，找出與使用者問題在概念上相似的區塊。BM25 則是較舊但仍有價值的技術；它是一種關鍵字演算法，會根據詞頻對區塊排序，而不理解語意。為了兼具兩者優點，通常會採用混合搜尋方法，將 BM25 的關鍵字精準度與語意搜尋的上下文理解結合起來。這種融合能帶來更穩健且準確的擷取，同時捕捉字面相符與概念相關性。

### 向量資料庫

向量資料庫是一種專門設計用來有效儲存與查詢 embedding 的資料庫。文件分塊並轉換成 embedding 後，這些高維度向量會被儲存在向量資料庫中。傳統擷取技術，例如以關鍵字為基礎的搜尋，很擅長找出包含查詢中精確字詞的文件，但缺乏對語言的深層理解。它們不會辨識出 "furry feline companion" 的意思是 "cat"。這正是向量資料庫擅長之處。它們是專為語意搜尋而建置的。透過將文字儲存為數值向量，它們能根據概念含義找出結果，而不只是比對關鍵字重疊。當使用者查詢也被轉換成向量時，資料庫會使用高度最佳化的演算法（例如 HNSW \- Hierarchical Navigable Small World）快速搜尋數百萬個向量，並找出含義最「接近」的向量。這種方法對 RAG 遠比傳統方式更優越，因為即使使用者措辭與來源文件完全不同，它也能找出相關上下文。本質上，其他技術搜尋的是字詞，而向量資料庫搜尋的是含義。這項技術有多種實作形式，從 Pinecone 與 Weaviate 等託管資料庫，到 Chroma DB、Milvus 與 Qdrant 等開源解決方案皆有。即使是既有資料庫，也可以加入向量搜尋能力，例如 Redis、Elasticsearch 與 Postgres（使用 pgvector 擴充功能）。核心擷取機制通常由 Meta AI 的 FAISS 或 Google Research 的 ScaNN 等函式庫驅動，這些都是此類系統效率的基礎。

### RAG 的挑戰

儘管 RAG 模式功能強大，仍然有其挑戰。主要問題之一，是回答查詢所需的資訊可能不限於單一區塊，而是分散在文件的多個部分，甚至多份文件之中。在這種情況下，擷取器可能無法收集所有必要上下文，導致答案不完整或不準確。系統成效也高度取決於分塊與擷取流程的品質；如果擷取到不相關的區塊，就可能引入雜訊並混淆 LLM。此外，有效整合可能彼此矛盾的來源資訊，對這些系統而言仍是一項重大障礙。除此之外，另一項挑戰是 RAG 需要先預處理整個知識庫，並將其儲存在向量資料庫或圖形資料庫等專用資料庫中，這是一項相當可觀的工作。因此，這些知識需要定期對帳與更新，才能保持最新；在處理公司 wiki 等持續演進的來源時，這尤其重要。整個流程也可能對效能造成明顯影響，包括增加延遲、營運成本，以及最終提示中使用的 token 數量。

總結來說，檢索增強生成（RAG）模式代表 AI 邁向更有知識且更可靠的一大進展。透過將外部知識擷取步驟無縫整合到生成流程中，RAG 解決了獨立 LLM 的部分核心限制。embedding 與語意相似度等基礎概念，再加上關鍵字搜尋與混合搜尋等擷取技術，使系統能智慧地找到相關資訊；而策略性的分塊則讓這些資訊更容易管理。整個擷取流程由專門設計的向量資料庫支援，能大規模儲存並有效查詢數百萬個 embedding。雖然擷取分散或矛盾資訊的挑戰仍然存在，RAG 仍能讓 LLM 產生不僅符合上下文，也以可驗證事實為依據的答案，進而提升人們對 AI 的信任與實用價值。  

### Graph RAG

GraphRAG 是檢索增強生成的進階形式，使用知識圖譜而不是單純的向量資料庫來進行資訊擷取。它透過在這個結構化知識庫中，沿著資料實體（節點）之間的明確關係（邊）進行導覽，來回答複雜查詢。它的一項關鍵優勢，是能夠整合分散在多份文件中的資訊來生成答案；這正是傳統 RAG 常見的不足之處。透過理解這些連結，GraphRAG 能提供更符合上下文且更細緻的回應。

使用案例包括複雜的財務分析、將公司與市場事件連結起來，以及在科學研究中發現基因與疾病之間的關係。不過，它的主要缺點是建立與維護高品質知識圖譜所需的複雜度、成本與專業知識都相當高。相較於較簡單的向量搜尋系統，這種設定也較不彈性，並可能帶來更高延遲。系統成效完全取決於底層圖形結構的品質與完整性。因此，GraphRAG 能為複雜問題提供更優越的上下文推理能力，但實作與維護成本也高得多。總結來說，當深入且相互連結的洞察比標準 RAG 的速度與簡潔性更重要時，它最能發揮優勢。

### 代理式 RAG

這個模式的演進形式稱為 **代理式 RAG**（見圖 2），它引入推理與決策層，大幅提升資訊擷取的可靠性。它不只是擷取與增強資訊，而是由一個「代理」——專門的 AI 元件——擔任知識的關鍵守門員與精煉者。如下列情境所示，這個代理不會被動接受初次擷取到的資料，而會主動檢視其品質、相關性與完整性。

首先，代理擅長反思與來源驗證。如果使用者問："What is our company's policy on remote work?"，標準 RAG 可能會同時取出一篇 2020 年的部落格文章與官方 2025 年政策文件。然而，代理會分析文件的中繼資料，辨識 2025 年政策是最新且最具權威性的來源，並在把正確上下文送給 LLM 以產生精確答案之前，先排除過時的部落格文章。

![Agentic RAG Introduces Reasoning Agent](../assets/Agentic_RAG_Introduces_Reasoning_Agent.png)

圖 2：代理式 RAG 引入推理代理，主動評估、調和並精煉擷取到的資訊，以確保最終回應更準確且更值得信賴。

第二，代理擅長調和知識衝突。想像一位財務分析師詢問："What was Project Alpha's Q1 budget?" 系統擷取到兩份文件：一份初始提案寫著預算為 €50,000，另一份最終財務報告則列為 €65,000。代理式 RAG 會辨識這項矛盾，將財務報告優先視為較可靠來源，並提供經驗證的數字給 LLM，確保最終答案以最準確的資料為基礎。

第三，代理可以執行多步驟推理，以整合出複雜答案。如果使用者問："How do our product's features and pricing compare to Competitor X's?"，代理會將問題拆解成不同子查詢。它會分別搜尋自家產品功能、自家定價、Competitor X 的功能，以及 Competitor X 的定價。收集這些個別資訊後，代理會將它們整合成結構化的比較上下文，再餵給 LLM，使其能產生單純擷取無法生成的完整回應。

第四，代理可以辨識知識缺口並使用外部工具。假設使用者問："What was the market's immediate reaction to our new product launched yesterday?" 代理搜尋每週更新一次的內部知識庫，卻找不到相關資訊。辨識出這個缺口後，它可以啟用工具，例如即時網路搜尋 API，來尋找最近的新聞文章與社群媒體情緒。接著，代理會使用這些剛收集到的外部資訊，提供最新答案，克服靜態內部資料庫的限制。

### 代理式 RAG 的挑戰

雖然功能強大，代理式層也帶來一組新的挑戰。主要缺點是複雜度與成本大幅增加。設計、實作與維護代理的決策邏輯和工具整合，需要大量工程投入，也會增加運算費用。這種複雜度也可能導致延遲增加，因為代理的反思、工具使用與多步驟推理循環，比標準的直接擷取流程需要更多時間。此外，代理本身也可能成為新的錯誤來源；有缺陷的推理流程可能讓它陷入無用迴圈、誤解任務，或不當丟棄相關資訊，最終降低回應品質。

### 總結

代理式 RAG 是標準擷取模式的精密演進，將其從被動資料管線轉變為主動解決問題的框架。透過嵌入能評估來源、調和衝突、拆解複雜問題並使用外部工具的推理層，代理能大幅提升生成答案的可靠性與深度。這項進展讓 AI 更值得信賴且更有能力，但也伴隨系統複雜度、延遲與成本方面的重要取捨，必須審慎管理。

## 實務應用與使用案例

知識檢索（RAG）正在改變大型語言模型（LLM）在各產業中的運用方式，強化它們提供更準確且更符合上下文回應的能力。

應用包括：

* **企業搜尋與 Q\&A：** 組織可以開發內部聊天機器人，使用 HR 政策、技術手冊與產品規格等內部文件回答員工詢問。RAG 系統會從這些文件中擷取相關段落，為 LLM 的回應提供資訊。  
* **客戶支援與服務台：** 以 RAG 為基礎的系統可以存取產品手冊、常見問題（FAQ）與支援票證中的資訊，為客戶查詢提供精確且一致的回應。這能減少例行問題對人工直接介入的需求。  
* **個人化內容推薦：** RAG 不只進行基本關鍵字比對，而是能辨識並擷取與使用者偏好或先前互動在語意上相關的內容（文章、產品），進而提供更相關的推薦。  
* **新聞與時事摘要：** LLM 可以與即時新聞來源整合。當使用者詢問時事時，RAG 系統會擷取近期文章，讓 LLM 產生最新摘要。

透過納入外部知識，RAG 將 LLM 的能力從簡單溝通延伸到能作為知識處理系統運作。

## 實作程式碼範例（ADK）

為了說明知識檢索（RAG）模式，讓我們來看三個範例。

第一個範例說明如何使用 Google Search 執行 RAG，並讓 LLM 以搜尋結果為依據。由於 RAG 涉及存取外部資訊，Google Search 工具是內建擷取機制的直接範例，能用來增強 LLM 的知識。

```python
from google.adk.tools import google_search
from google.adk.agents import Agent


search_agent = Agent(
    name="research_assistant",
    model="gemini-2.0-flash-exp",
    instruction="You help users research topics. When asked, use the Google Search tool",
    tools=[google_search],
)
```

第二個範例說明如何在 Google ADK 中運用 Vertex AI RAG 功能。提供的程式碼示範如何從 ADK 初始化 VertexAiRagMemoryService。這能用來建立與 Google Cloud Vertex AI RAG Corpus 的連線。此服務透過指定 corpus 資源名稱，以及 `SIMILARITY_TOP_K` 與 `VECTOR_DISTANCE_THRESHOLD` 等選用參數進行設定。這些參數會影響擷取流程。`SIMILARITY_TOP_K` 定義要擷取的最相似結果數量。`VECTOR_DISTANCE_THRESHOLD` 則為擷取結果設定語意距離上限。這項設定讓代理能從指定的 RAG Corpus 執行可擴充且持久的語意知識擷取。此流程有效地將 Google Cloud 的 RAG 功能整合進 ADK 代理，進而支援以事實資料為基礎的回應開發。

```python
# Import the necessary VertexAiRagMemoryService class from the google.adk.memory module.
from google.adk.memory import VertexAiRagMemoryService


RAG_CORPUS_RESOURCE_NAME = "projects/your-gcp-project-id/locations/us-central1/ragCorpora/your-corpus-id"

# Define an optional parameter for the number of top similar results to retrieve.
# This controls how many relevant document chunks the RAG service will return.
SIMILARITY_TOP_K = 5

# Define an optional parameter for the vector distance threshold.
# This threshold determines the maximum semantic distance allowed for retrieved results;
# results with a distance greater than this value might be filtered out.
VECTOR_DISTANCE_THRESHOLD = 0.7

# Initialize an instance of VertexAiRagMemoryService.
# This sets up the connection to your Vertex AI RAG Corpus.
# - rag_corpus: Specifies the unique identifier for your RAG Corpus.
# - similarity_top_k: Sets the maximum number of similar results to fetch.
# - vector_distance_threshold: Defines the similarity threshold for filtering results.
memory_service = VertexAiRagMemoryService(
    rag_corpus=RAG_CORPUS_RESOURCE_NAME,
    similarity_top_k=SIMILARITY_TOP_K,
    vector_distance_threshold=VECTOR_DISTANCE_THRESHOLD,
)
```

## 實作程式碼範例（LangChain）

第三個範例，讓我們使用 LangChain 逐步走過一個完整範例。

```python
import os
import requests
from typing import List, Dict, Any, TypedDict

from langchain_community.document_loaders import TextLoader
from langchain_core.documents import Document
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_community.embeddings import OpenAIEmbeddings
from langchain_community.vectorstores import Weaviate
from langchain_openai import ChatOpenAI
from langchain.text_splitter import CharacterTextSplitter
from langchain.schema.runnable import RunnablePassthrough
from langgraph.graph import StateGraph, END

import weaviate
from weaviate.embedded import EmbeddedOptions
import dotenv


# Load environment variables (e.g., OPENAI_API_KEY)
dotenv.load_dotenv()

# Set your OpenAI API key (ensure it's loaded from .env or set here)
# os.environ["OPENAI_API_KEY"] = "YOUR_OPENAI_API_KEY"


# --- 1. Data Preparation (Preprocessing) ---

# Load data
url = "https://github.com/langchain-ai/langchain/blob/master/docs/docs/how_to/state_of_the_union.txt"
res = requests.get(url)
with open("state_of_the_union.txt", "w") as f:
    f.write(res.text)

loader = TextLoader("./state_of_the_union.txt")
documents = loader.load()

# Chunk documents
text_splitter = CharacterTextSplitter(chunk_size=500, chunk_overlap=50)
chunks = text_splitter.split_documents(documents)

# Embed and store chunks in Weaviate
client = weaviate.Client(embedded_options=EmbeddedOptions())

vectorstore = Weaviate.from_documents(
    client=client,
    documents=chunks,
    embedding=OpenAIEmbeddings(),
    by_text=False,
)

# Define the retriever
retriever = vectorstore.as_retriever()

# Initialize LLM
llm = ChatOpenAI(model_name="gpt-3.5-turbo", temperature=0)


# --- 2. Define the State for LangGraph ---
class RAGGraphState(TypedDict):
    question: str
    documents: List[Document]
    generation: str


# --- 3. Define the Nodes (Functions) ---
def retrieve_documents_node(state: RAGGraphState) -> RAGGraphState:
    """Retrieves documents based on the user's question."""
    question = state["question"]
    documents = retriever.invoke(question)
    return {"documents": documents, "question": question, "generation": ""}


def generate_response_node(state: RAGGraphState) -> RAGGraphState:
    """Generates a response using the LLM based on retrieved documents."""
    question = state["question"]
    documents = state["documents"]

    # Prompt template from the PDF
    template = """You are an assistant for question-answering tasks. Use the following pieces of retrieved context to answer the question. If you don't know the answer, just say that you don't know. Use three sentences maximum and keep the answer concise.
Question: {question}
Context: {context}
Answer: """
    prompt = ChatPromptTemplate.from_template(template)

    # Format the context from the documents
    context = "\n\n".join([doc.page_content for doc in documents])

    # Create the RAG chain
    rag_chain = prompt | llm | StrOutputParser()

    # Invoke the chain
    generation = rag_chain.invoke({"context": context, "question": question})

    return {"question": question, "documents": documents, "generation": generation}


# --- 4. Build the LangGraph Graph ---
workflow = StateGraph(RAGGraphState)

# Add nodes
workflow.add_node("retrieve", retrieve_documents_node)
workflow.add_node("generate", generate_response_node)

# Set the entry point
workflow.set_entry_point("retrieve")

# Add edges (transitions)
workflow.add_edge("retrieve", "generate")
workflow.add_edge("generate", END)

# Compile the graph
app = workflow.compile()


# --- 5. Run the RAG Application ---
if __name__ == "__main__":
    print("\n--- Running RAG Query ---")
    query = "What did the president say about Justice Breyer"
    inputs = {"question": query}
    for s in app.stream(inputs):
        print(s)

    print("\n--- Running another RAG Query ---")
    query_2 = "What did the president say about the economy?"
    inputs_2 = {"question": query_2}
    for s in app.stream(inputs_2):
        print(s)
```

這段 Python 程式碼展示了一條使用 LangChain 與 LangGraph 實作的檢索增強生成（RAG）管線。流程一開始會從文字文件建立知識庫，將文件切分成區塊並轉換成 embedding。接著，這些 embedding 會儲存在 Weaviate 向量儲存區中，以便有效擷取資訊。LangGraph 中的 StateGraph 用來管理兩個關鍵函式之間的工作流程：`retrieve_documents_node` 與 `generate_response_node`。`retrieve_documents_node` 函式會根據使用者輸入查詢向量儲存區，以識別相關文件區塊。隨後，`generate_response_node` 函式會使用擷取到的資訊與預先定義的提示範本，透過 OpenAI 大型語言模型（LLM）產生回應。`app.stream` 方法允許查詢透過 RAG 管線執行，展示系統生成符合上下文輸出的能力。

## 快速概覽

**是什麼：** LLM 具備令人印象深刻的文字生成能力，但根本上受限於其訓練資料。這些知識是靜態的，表示不包含即時資訊或私有、特定領域資料。因此，它們的回應可能過時、不準確，或缺乏專門任務所需的特定上下文。這項落差限制了它們在需要最新且以事實為基礎答案的應用中的可靠性。

**為什麼：** 檢索增強生成（RAG）模式透過將 LLM 連接到外部知識來源，提供標準化解決方案。收到查詢時，系統會先從指定知識庫擷取相關資訊片段。接著，這些片段會附加到原始提示中，以即時且具體的上下文豐富提示。然後，這個增強後的提示會送給 LLM，使它能產生準確、可驗證，且以外部資料為基礎的回應。這個流程有效地將 LLM 從閉卷推理者轉變為開卷推理者，大幅提升其實用性與可信度。

**經驗法則：** 當你需要 LLM 根據特定、最新或專有資訊回答問題或生成內容，而這些資訊並非其原始訓練資料的一部分時，就使用這個模式。它非常適合用來建置內部文件上的 Q\&A 系統、客戶支援機器人，以及需要具引用、可驗證且以事實為基礎回應的應用程式。

**視覺摘要：**

![Knowledge Retrieval Pattern Database](../assets/Knowledge_Retrieval_Pattern_Database.png)

知識檢索模式：AI 代理查詢並擷取結構化資料庫中的資訊

![Knowledge Retrieval Pattern Search](../assets/Knowledge_Retrieval_Pattern_Search.png)

圖 3：知識檢索模式：AI 代理根據使用者查詢，從公開網際網路尋找並整合資訊。

## 重點整理

* 知識檢索（RAG）讓 LLM 能存取外部、最新且特定的資訊，進而強化 LLM。  
* 此流程包含擷取（在知識庫中搜尋相關片段）與增強（將這些片段加入 LLM 的提示）。  
* RAG 協助 LLM 克服訓練資料過時等限制、降低「幻覺」，並能整合特定領域知識。  
* RAG 讓答案可以歸因，因為 LLM 的回應是以擷取來源為基礎。  
* GraphRAG 運用知識圖譜理解不同資訊片段之間的關係，使其能回答需要整合多個來源資料的複雜問題。  
* 代理式 RAG 不只進行簡單資訊擷取，而是使用智慧代理主動推理、驗證並精煉外部知識，確保答案更準確且可靠。  
* 實務應用涵蓋企業搜尋、客戶支援、法律研究與個人化推薦。

## 結論

總結來說，檢索增強生成（RAG）透過將大型語言模型連接到外部且最新的資料來源，解決大型語言模型靜態知識的核心限制。這個流程會先擷取相關資訊片段，再增強使用者提示，使 LLM 能生成更準確且更具上下文意識的回應。這是由 embedding、語意搜尋與向量資料庫等基礎技術所促成，這些技術會根據含義而不只是關鍵字來尋找資訊。透過將輸出建立在可驗證資料上，RAG 能大幅降低事實錯誤，並允許使用專有資訊，再透過引用提升信任。

進階演進形式代理式 RAG 引入推理層，主動驗證、調和並整合擷取到的知識，以提升可靠性。同樣地，GraphRAG 等專門方法運用知識圖譜導覽明確的資料關係，使系統能為高度複雜且相互連結的查詢整合答案。這個代理可以解決互相衝突的資訊、執行多步驟查詢，並使用外部工具尋找缺失資料。雖然這些進階方法增加了複雜度與延遲，卻大幅提升最終回應的深度與可信度。這些模式的實務應用已經在改變各個產業，從企業搜尋與客戶支援，到個人化內容傳遞皆然。儘管仍有挑戰，RAG 仍是讓 AI 更有知識、更可靠且更實用的關鍵模式。最終，它將 LLM 從閉卷對話者轉變為強大的開卷推理工具。

## 參考資料

1. Lewis, P., et al. (2020). *Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks*. [https://arxiv.org/abs/2005.11401](https://arxiv.org/abs/2005.11401)
2. Google AI for Developers Documentation.  *Retrieval Augmented Generation - [https://cloud.google.com/vertex-ai/generative-ai/docs/rag-engine/rag-overview](https://cloud.google.com/vertex-ai/generative-ai/docs/rag-engine/rag-overview)*
3. Retrieval-Augmented Generation with Graphs (GraphRAG), [https://arxiv.org/abs/2501.00309](https://arxiv.org/abs/2501.00309)
4. LangChain and LangGraph: Leonie Monigatti, "Retrieval-Augmented Generation (RAG): From Theory to LangChain Implementation,"  [*https://medium.com/data-science/retrieval-augmented-generation-rag-from-theory-to-langchain-implementation-4e9bd5f6a4f2*](https://medium.com/data-science/retrieval-augmented-generation-rag-from-theory-to-langchain-implementation-4e9bd5f6a4f2)
5. Google Cloud Vertex AI RAG Corpus [*https://cloud.google.com/vertex-ai/generative-ai/docs/rag-engine/manage-your-rag-corpus#corpus-management*](https://cloud.google.com/vertex-ai/generative-ai/docs/rag-engine/manage-your-rag-corpus#corpus-management)
