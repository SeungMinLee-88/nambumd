**### Summary

이 영상에서는 RAG(Retrieval-Augmented Generation)의 동작 과정 중 런타임 단계에 대해 설명합니다. 이전에는 문서를 로드하고, 청크 단위로 나누고, 임베딩 후 벡터 스토어에 저장하는 사전 단계가 진행됩니다. 런타임 단계에서는 사용자가 질문을 입력하면, 리트리버가 유사한 단락을 검색하여 해당 정보를 프롬프트에 넣어 LM(Language Model)에 전달하고, LM은 이를 기반으로 답변을 생성합니다. 영상에서는 이 과정에서 유사도 검색, 임베딩 변환, 리트리버의 역할 등을 설명하며, 후속 실습 코드 진행을 예고합니다.

### Highlights

- 🧑‍💻 **사전 단계와 런타임 단계 구분**: 문서 로드, 청크화, 임베딩, 벡터 스토어 저장 후, 런타임 단계에서 유저 질문에 대해 유사도 검색을 통해 관련 단락을 찾고 이를 프롬프트에 삽입하여 LM이 답변을 생성합니다.
- 🔍 **리트리버와 유사도 검색**: 질문에 대해 유사도가 높은 단락을 리트리버가 검색하며, 이는 벡터 공간에서 의미적 유사성을 바탕으로 계산됩니다.
- ⚡ **프롬프트 생성 및 문맥 추가**: 질문뿐만 아니라 검색된 문서의 단락을 문맥으로 추가하여 LM이 더 정확한 답변을 생성할 수 있도록 합니다.
- 💡 **검색기와 검색 알고리즘의 중요성**: 키워드 기반 검색과 의미 검색을 구분하며, 검색기의 성능이 결과에 중요한 영향을 미침을 강조합니다.
- 📝 **체인 생성 과정**: 질문이 리트리버에 입력되어 유사한 단락을 찾고, 이를 프롬프트에 삽입하여 최종 답변을 생성하는 복잡한 프로세스를 체인으로 연결합니다.**

https://www.youtube.com/watch?v=Fxc2AzrxOP8

---

![[rag process2.png]]
###### ChatGPT의 말:

이 이미지는 **RAG Pipeline(검색 증강 생성 파이프라인, Retrieval-Augmented Generation Pipeline)** 의 전체 과정을 시각적으로 설명하고 있습니다. 아래는 각 단계를 분석하고 설명한 내용입니다.

---

## 🧩 전체 개요

RAG 파이프라인은 **LLM(대형 언어 모델)** 이 자체 지식만으로 답을 생성하는 대신, **외부 데이터베이스나 문서에서 관련 정보를 검색하여** 답변의 정확도와 신뢰성을 높이는 구조입니다.  
즉, **검색(Retrieval)** + **생성(Generation)** 을 결합한 시스템입니다.

---

## 🔷 단계별 설명

### 1️⃣ Data Indexing (데이터 인덱싱)

- 문서, PDF, 웹페이지 등의 **외부 데이터를 수집 및 전처리**합니다.
    
- 수집된 데이터는 **청크(chunk)** 단위로 분할되고, **임베딩(embedding)** 을 통해 **벡터화** 됩니다.
    
- 이렇게 변환된 벡터들은 **벡터 데이터베이스(Vector Store)** 에 저장됩니다.
    
    - 예: FAISS, Pinecone, Milvus, Weaviate 등
        

➡️ **핵심 목적:** 나중에 관련된 문서를 빠르게 검색할 수 있도록 미리 준비하는 단계입니다.

---

### 2️⃣ Query Preprocessing & Embedding (질의 전처리 및 임베딩)

- 사용자가 입력한 질문(query)을 **전처리**하고 **임베딩 모델**을 통해 벡터로 변환합니다.
    
- 이 벡터는 **데이터베이스에 저장된 문서 벡터들과 비교**하여 유사도를 측정하는 데 사용됩니다.
    

➡️ **핵심 역할:** 사용자의 질문을 머신이 이해할 수 있는 수치 표현으로 바꾸는 과정입니다.

---

### 3️⃣ Document Retrieval (문서 검색)

- 쿼리 벡터와 가장 유사한 **상위 k개의 문서(Top-k)** 를 벡터 DB에서 검색합니다.
    
- 이때 **유사도 기반 검색(similarity search)** 이 수행됩니다.
    

➡️ **핵심 목표:** 질문과 가장 관련된 문서 후보를 찾는 단계입니다.

---

### 4️⃣ Re-ranking (재정렬)

- 검색된 문서들을 단순 유사도 순서가 아닌, **의미적 관련성** 기준으로 다시 정렬합니다.
    
- 이 과정에는 **cross-encoder reranker** 모델을 사용하는 경우가 많습니다.  
    (예: Cohere Reranker, OpenAI re-ranking API)
    

➡️ **핵심 목적:** 더 “진짜로 관련 있는” 문서를 상위에 올려 LLM이 더 좋은 맥락을 가지게 합니다.

---

### 5️⃣ Prompt Augmentation (프롬프트 확장)

- 선택된 문서들을 **프롬프트(prompt)** 에 함께 삽입합니다.
    
- 예:
    
    `Context: [문서1 내용...] [문서2 내용...] Question: 사용자가 입력한 질문`
    
- 이렇게 함으로써 LLM이 **추가 지식을 활용하여 답변을 생성**할 수 있습니다.
    

➡️ **핵심 포인트:** 문서 내용을 LLM의 “컨텍스트”로 주입합니다.

---

### 6️⃣ Response Generation & Postprocessing (응답 생성 및 후처리)

- LLM이 문서 기반으로 **최종 답변을 생성**합니다.
    
- 필요 시 **출처 링크, 하이라이트, 포맷 정리** 등의 후처리 과정을 거칩니다.
    
- 경우에 따라 **사후 검증(Post-processing)**, **출처 신뢰도 평가**, **정확성 필터링**도 포함됩니다.
    

➡️ **결과물:** 정확하고 맥락이 풍부한 응답 + 출처가 명확한 결과

---

## ⚙️ 요약 정리

|단계|이름|주요 역할|
|---|---|---|
|1|Data Indexing|데이터 청크화 및 벡터 인덱싱|
|2|Query Embedding|질의 임베딩 변환|
|3|Document Retrieval|유사 문서 검색|
|4|Re-ranking|검색 결과 재정렬|
|5|Prompt Augmentation|문서 내용 포함 프롬프트 구성|
|6|Response Generation|LLM 응답 생성 및 후처리|

---

## 🧠 한 줄 요약

> **RAG Pipeline은 “검색 + 생성”의 결합으로, LLM이 최신 외부 정보를 활용해 신뢰도 높은 답변을 생성하도록 하는 구조적 프로세스입니다.**

---

원하신다면 이 이미지를 기반으로 **Agentic RAG 버전(자율적 도구 선택형 RAG)** 으로 확장된 구조도 함께 설명드릴 수 있습니다. 원하시나요?

---
## Agentic RAG란?

**Agentic RAG**는 단순히 검색 후 답변을 생성하는 기본 RAG에서 발전해,  
**“에이전트(Agent)”가 자율적으로 의사결정하고, 도구(tool)와 외부 API를 활용해 최적의 정보를 찾아내는 RAG 시스템**입니다.

즉, RAG + Agentic Decision Layer가 결합된 형태로,  
“어떤 데이터베이스를 검색할지, 어떤 쿼리를 수정할지, 어떤 도구(API)를 호출할지”를 스스로 판단합니다.

---

## 🧩 Agentic RAG Pipeline 단계별 구성

|단계|이름|주요 역할|
|---|---|---|
|1|**Data Indexing**|(기본 RAG와 동일) 문서 청크화, 임베딩, 벡터 인덱싱|
|2|**Query Analysis & Decomposition**|에이전트가 사용자의 요청을 분석해 하위 작업(Subtasks)으로 분해|
|3|**Tool Selection & Planning**|검색, API, 계산기, DB 질의 등 사용할 도구를 선택 및 실행 계획 세움|
|4|**Iterative Retrieval (Dynamic Search)**|쿼리를 스스로 개선하면서 반복 검색 및 재랭킹 수행|
|5|**Context Integration (Dynamic Prompting)**|선택된 문맥을 프롬프트에 삽입하며, 필요 시 reasoning trace를 포함|
|6|**Response Generation & Self-Verification**|응답 생성 후, 사실 검증 및 품질 평가 수행 (자기 점검 루프)|

---

## 🔍 각 단계 세부 설명

### 1️⃣ Data Indexing

- **벡터 스토어** 구축은 기존 RAG와 동일하지만, Agentic RAG에서는 **여러 소스(DB, API, 문서)** 를 동시에 관리할 수 있습니다.
    
- 예: 회사 내부 문서 + 웹 검색 + API 응답 등
    

---

### 2️⃣ Query Analysis & Decomposition

- LLM 에이전트가 질문을 분석하고, 필요한 **하위 태스크(sub-query)** 로 분리합니다.
    
- 예:  
    “삼성 Gauss2의 주요 특징과 경쟁사 모델 비교”  
    →  
    (1) 삼성 Gauss2 정보 검색  
    (2) 경쟁사 AI 모델 검색  
    (3) 비교 요약 생성
    

➡️ LangChain이나 LangGraph에서는 이 과정을 **“graph node decomposition”** 으로 구현합니다.

---

### 3️⃣ Tool Selection & Planning

- Agent는 현재 태스크에 맞는 **도구를 자동 선택**합니다.  
    (예: VectorDB 검색, SQL 질의, WebSearch, API call 등)
    
- 계획은 일반적으로 **ReAct(Reason + Act)** 혹은 **LangGraph의 state machine** 로직을 기반으로 수행됩니다.
    

➡️ 예:

`Thought: 최신 정보가 필요하므로 웹 검색이 필요함. Action: Search("Samsung Gauss2 launch 2025")`

---

### 4️⃣ Iterative Retrieval (Dynamic Search)

- 단 한 번의 검색이 아니라, **검색 → 평가 → 재검색** 루프를 수행합니다.
    
- 이때 **reranker** 와 **query reformulator** 가 함께 작동하여 검색 품질을 향상시킵니다.
    
- 필요 시 **feedback loop** (예: “충분한 정보가 없다면 다시 검색”)도 포함됩니다.
    

➡️ 예:

- Round 1: “Gauss2 특징” 검색
    
- Round 2: “Gauss2 경쟁 모델 비교” 검색
    
- Round 3: “Gauss2 inference optimization” 검색
    

---

### 5️⃣ Context Integration (Dynamic Prompting)

- Agent가 관련 문맥만 선별적으로 통합하여 **Adaptive Prompt**를 구성합니다.
    
- 프롬프트 내에는 reasoning trace, 검색 근거, 인용 출처 등이 포함될 수 있습니다.
    

예시:

`You are an expert AI analyst. Context: [문서1: Gauss2 기능 개요] [문서2: OpenAI GPT-4o 비교표] Question: 두 모델의 차이점을 설명하세요.`

---

### 6️⃣ Response Generation & Self-Verification

- LLM이 답변을 생성한 뒤, **Self-Consistency Check** 또는 **Fact-Verification**을 수행합니다.
    
- OpenAI의 “**Critic Agent**” 또는 Anthropic의 “**Constitutional AI Layer**”처럼,  
    별도의 **검증 에이전트(Verifier)** 가 잘못된 정보를 교정할 수도 있습니다.
    

➡️ 최종 출력은:

- 정확성(Precision)
    
- 인용 출처 포함
    
- 일관된 서술 구조
    

---

## 🧠 핵심 요약

|비교 항목|기본 RAG|Agentic RAG|
|---|---|---|
|검색 방식|정적(top-k 검색)|동적, 반복적 검색|
|쿼리 처리|단일 쿼리|다중 쿼리 분해 및 계획|
|도구 활용|벡터 DB 중심|다중 툴(Web/API/DB 등) 활용|
|프롬프트 구성|단순 문맥 삽입|동적 컨텍스트 + reasoning trace|
|품질 보정|없음|Self-Verification 포함|
|대표 구현|LangChain, LlamaIndex|LangGraph, CrewAI, AutoRAG, DSPy 등|

---

## 🚀 한 줄 요약

> **Agentic RAG = RAG + 자율적 의사결정(AI Agent Layer)**  
> LLM이 스스로 정보를 탐색·선택·검증하는 지능형 검색-생성 파이프라인입니다.


앞으로의 진화는 단순히 검색을 보강하는 차원을 넘어설 거예요. 멀티모달 RAG는 텍스트를 넘어 이미지·음성까지 아우르는 증강을 가능하게 하고, Agentic RAG 아키텍처는 검색과 추론을 능동적으로 조율하며 스스로 지식 활용 경로를 결정하는 자율성을 부여해요. 여기에 MCP(Model Context Protocol) 같은 표준이 결합되면, 다양한 외부 도구와 데이터 소스가 유연하게 연결되면서 RAG의 확장성은 한층 더 커질 거예요. ✨

출처: [https://tech.ktcloud.com/entry/2025-08-ktcloud-ai-rag-시스템구조-이해](https://tech.ktcloud.com/entry/2025-08-ktcloud-ai-rag-%EC%8B%9C%EC%8A%A4%ED%85%9C%EA%B5%AC%EC%A1%B0-%EC%9D%B4%ED%95%B4) [kt cloud Tech Blog:티스토리]