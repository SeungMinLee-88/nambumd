랜그래프는 다중 에이전트 프레임워크로, 사이클 지원과 세밀한 제어 가능성이 특징입니다. 이 시스템은 노드 간의 순환 구조를 허용하고, 사용자가 그래프의 흐름을 세밀하게 설정할 수 있습니다. 또한, 그래프의 상태 및 흐름을 정밀하게 제어하고, 휴먼 인더루프 메모리 기능을 통해 인간 개입을 가능하게 합니다. 이를 통해 사용자는 그래프의 동작을 완전히 제어할 수 있으며, 여러 머신러닝 및 파이프라인 시스템에서 활용될 수 있습니다.

### Highlights

- 🔄 **사이클 지원**: 노드 간의 순환 구조를 가능하게 하여, 에이전트가 서로 반대 방향으로 돌아갈 수 있습니다.
- 🎮 **세밀한 제어 가능**: 사용자가 그래프의 노드와 엣지를 직접 설정하여 흐름을 세밀하게 제어할 수 있습니다.
- 💾 **상태 및 흐름 정밀 제어**: 노드 간의 입력과 출력을 정밀하게 제어하여, 원하는 대로 그래프의 동작을 설정할 수 있습니다.
- 🧑‍💻 **휴먼 인더루프 메모리 기능**: 사람이 개입하여 그래프의 결과를 수정하거나 유도할 수 있는 기능을 제공합니다.
- 🖥️ **영감을 받은 프레임워크**: 구글의 프레게, 아파치 빔, 네트워크 X와 같은 시스템에서 영감을 받아 설계되었습니다.
- 🏗️ **노드, 엣지, 상태 선언**: 그래프의 동작을 구현하기 위해 노드, 엣지, 상태를 직접 선언하고 컴파일하여 실행 가능한 그래프로 변환합니다.

### keyword

- AI Agent
- Langgraph
- 프레임워크

##### 이 비디오에 대한 FAQ (베타)

#### Q: 랜 그래프의 주요 특징은 무엇인가요?

##### A: 랜 그래프의 주요 특징은 사이클을 지원하고, 세밀한 제어가 가능하다는 점입니다. 사이클 지원은 노드 간에 양방향 연결이 가능하다는 의미이며, 세밀한 제어는 사용자가 그래프의 노드와 엣지를 더 구체적으로 정의할 수 있다는 점에서 유리합니다.

#### Q: 랜 그래프에서 세밀한 제어가 가능하다는 의미는 무엇인가요?

##### A: 세밀한 제어가 가능하다는 것은 사용자가 그래프의 노드와 엣지, 그리고 그 흐름을 직접 설정하고 제어할 수 있다는 뜻입니다. 예를 들어, 각 노드의 입력과 출력 값을 사용자 정의할 수 있어 더 정밀한 작업을 할 수 있습니다.

#### Q: 랜 그래프에서의 '휴먼 인더 루프' 기능은 무엇인가요?

##### A: '휴먼 인더 루프'는 그래프의 실행 과정에서 사람이 개입할 수 있도록 하는 기능입니다. 이 기능을 통해 사용자는 그래프 실행 중에 특정 시점에서 개입하여 올바른 대답을 이끌어낼 수 있습니다.

#### Q: 랜 그래프의 코드 구성은 어떻게 되나요?

##### A: 랜 그래프의 코드 구성은 크게 스테이트(State), 노드(Node), 엣지(Edge)로 나뉩니다. 스테이트는 노드 간의 연결을 정의하고, 노드에는 LLM과 같은 에이전트를 추가하여 각 노드의 기능을 수행하게 합니다. 엣지는 노드 간의 흐름을 설정하며, 최종적으로 이들을 모두 결합하여 실행 가능한 그래프를 생성합니다.

---------------


이번 영상에서는 Langchain과 Langgraph의 차이점을 설명하고 있습니다. Langchain은 선형적인 체인 구조를 사용하여 여러 구성 요소들을 순차적으로 연결하는 방식이고, Langgraph는 비선형적이고 동적인 그래프 구조로 여러 노드와 에이전트가 협력하여 메시지의 흐름을 결정하는 방식입니다. Langchain은 일방향적인 처리 흐름을 갖고, Langgraph는 양방향으로 유연하게 흐름을 제어할 수 있는 특징이 있습니다. 두 프레임워크는 각각의 필요에 따라 활용될 수 있습니다.

### Highlights

- 🛠 **Langchain**은 선형적인 체인 구조로 구성 요소들을 순차적으로 연결해 작업을 처리합니다.
- 🧩 **Langgraph**는 비선형적이고 동적인 그래프 형태로 에이전트들이 메시지의 흐름을 결정합니다.
- 🔄 **Langchain**은 주로 문서 로딩, 텍스트 분할, 벡터 저장 등을 일방향 파이프라인 방식으로 처리합니다.
- 🤖 **Langgraph**는 에이전트가 메시지의 다음 처리 흐름을 동적으로 결정하는 양방향 시스템입니다.
- 🔄 **에이전트 시스템**은 Langchain과 Langgraph 모두에서 사용되지만, 두 시스템에서 역할과 구조가 다르게 적용됩니다.

### keyword

- Langchain
- Langgraph
- 프레임워크

##### 이 비디오에 대한 FAQ (베타)

#### Q: 랭 체인과 랭 그래프의 가장 큰 차이점은 무엇인가요?

##### A: 랭 체인은 선형적인 구조로, 메시지가 순차적으로 처리되며 각 구성 요소가 일방향으로 연결됩니다. 반면, 랭 그래프는 노드와 엣지로 이루어진 비선형적인 그래프 구조로, 메시지가 에이전트의 판단에 따라 다양한 방향으로 이동할 수 있습니다.

#### Q: 랭 체인의 주요 구성 요소는 무엇인가요?

##### A: 랭 체인의 주요 구성 요소로는 툴, 텍스트 스플리터, 아웃풋 파서, 프롬프트 모델 셋, 모델 임베딩 리트리버, 다큐먼트 로더, 벡터 스토어 등이 있으며, 이들을 선형적으로 배열하여 작업을 수행합니다.

#### Q: 랭 체인의 활용 사례는 무엇인가요?

##### A: 랭 체인은 문서 로딩, 텍스트 분할, 벡터화 등의 작업을 통해 레그 시스템을 구축할 때 주로 활용됩니다. 또한, 정보 추출, 에이전트 시스템 생성 등 다양한 용도로 사용됩니다.

#### Q: 랭 그래프는 어떤 상황에서 유용한가요?

##### A: 랭 그래프는 비선형적이고 동적인 프로세스가 필요한 시스템에 유용합니다. 에이전트가 메시지를 처리한 후, 그 흐름을 선택적으로 조정하여 다양한 경로로 처리할 수 있는 특징을 갖고 있습니다.



--------------------

LangChain에서 문서를 첨부하는 것은 주로 다양한 형식의 데이터를 가져와서 언어 모델(LLM)이 처리할 수 있는 형식으로 변환하는 과정을 의미합니다. 이는 특히 검색 증강 생성(RAG)과 같은 애플리케이션에서 핵심적인 단계입니다. 

다음은 LangChain에서 문서를 첨부하고 처리하는 주요 단계와 관련 구성 요소들입니다. 

1. 문서 로드 (Document Loaders)

문서 로더는 PDF, TXT, CSV, JSON, Word 등 다양한 형식의 파일을 LangChain의 표준 `Document` 객체로 변환합니다. 

- **`Document` 객체**: 문서의 내용을 담는 `page_content`와 문서에 대한 추가 정보를 포함하는 `metadata`로 구성됩니다.
- **다양한 로더**:
    - **PDF**: `PyPDFLoader`, `UnstructuredPDFLoader` 등을 사용하여 PDF 파일의 내용을 추출합니다.
    - **텍스트**: `TextLoader`를 사용해 `.txt` 파일을 로드합니다.
    - **디렉터리**: `DirectoryLoader`를 사용하면 특정 디렉터리에 있는 모든 파일을 로드할 수 있습니다.
    - **웹페이지**: `WebBaseLoader`와 같은 로더를 사용하여 웹페이지의 HTML 콘텐츠를 로드합니다. 

**예시: 텍스트 파일 로드**

python

```
from langchain_community.document_loaders import TextLoader

# `example.txt` 파일을 불러옵니다.
loader = TextLoader("example.txt")
documents = loader.load()

# 문서 객체 확인
print(documents)
```

2. 문서 분할 (Text Splitters)

대부분의 언어 모델은 한 번에 처리할 수 있는 토큰(텍스트 단위)의 양에 제한이 있습니다. 따라서 길거나 복잡한 문서는 의미 있는 작은 덩어리(chunk)로 나누어 처리해야 합니다. 

- **특징**: 문맥이 끊기지 않도록 문서를 분할하는 것이 중요합니다.
- **주요 분할기**: `RecursiveCharacterTextSplitter`, `CharacterTextSplitter` 등 다양한 분할기가 있습니다. 

3. 임베딩 및 벡터 저장소 (Embeddings & Vector Stores) 

분할된 문서 덩어리(chunks)를 벡터(숫자 리스트)로 변환하여 벡터 저장소에 저장합니다. 

- **임베딩**: 텍스트의 의미를 나타내는 벡터를 생성합니다.
- **벡터 저장소**: 이렇게 생성된 벡터들을 효율적으로 검색하기 위해 저장하는 곳입니다 (예: ChromaDB, FAISS). 

4. 검색기 (Retriever) 

사용자의 질문과 관련된 문서를 벡터 저장소에서 검색하는 역할을 합니다. 

- **기능**: 사용자의 질문을 임베딩으로 변환하고, 이를 벡터 저장소의 문서 벡터들과 비교하여 관련성 높은 문서를 찾아냅니다.
- **다양한 검색기**:
    - **`VectorstoreRetriever`**: 가장 일반적인 형태로, 벡터 유사도 검색을 통해 문서를 반환합니다.
    - **`ParentDocumentRetriever`**: 작은 덩어리로 검색하고, 더 큰 상위 문서(parent document)를 반환하여 문맥을 풍부하게 만듭니다. 

5. 문서 처리 파이프라인 (RAG) 

위의 구성 요소를 결합하여 문서 기반 질의응답 시스템(RAG)을 구축할 수 있습니다.

1. **문서 로드**: `Document Loader`를 사용하여 문서를 가져옵니다.
2. **문서 분할**: `Text Splitter`를 사용하여 문서를 덩어리로 나눕니다.
3. **임베딩 생성 및 저장**: `Embeddings` 모델로 벡터를 생성하고 `Vector Store`에 저장합니다.
4. **검색**: 사용자가 질문을 하면 `Retriever`가 관련 문서를 검색합니다.
5. **답변 생성**: 검색된 문서를 언어 모델(LLM)에 함께 제공하여 최종 답변을 생성합니다.

-----


## ✅ 주요 내용 요약

1. **LangGraph 소개**
    
    - LangGraph는 LangChain 위에 구축된 새 프레임워크로, 단순한 일방향 워크플로우(DAG: Directed Acyclic Graph)가 아닌 **순환(cyclic) 워크플로우**를 지원합니다. [Medium](https://becomingahacker.org/a-quick-introduction-to-langgraph-enhancing-llm-applications-with-cyclic-workflows-145f61f38747)
        
    - 즉, 여러 체인(chains)이나 액터(actors)를 여러 단계에 걸쳐 반복적으로 상호작용시키면서 상태(state)를 갱신하고, 이를 기반으로 다시 작업을 이어가는 구조에 초점이 맞춰져 있습니다. [Medium](https://becomingahacker.org/a-quick-introduction-to-langgraph-enhancing-llm-applications-with-cyclic-workflows-145f61f38747)
        
2. **“순환 워크플로우(Cycles)”가 왜 중요한가**
    
    - 일반적인 워크플로우에서는 한 번의 흐름이 끝나면 끝이지만, 순환 워크플로우에서는 이전 단계의 출력이 다음 단계의 입력이 되어 **반복적이고 상태 기반(stateful)인 흐름**을 만듭니다. [Medium](https://becomingahacker.org/a-quick-introduction-to-langgraph-enhancing-llm-applications-with-cyclic-workflows-145f61f38747)
        
    - 이런 구조는 챗봇, 에이전트 기반 시스템, 학습 시스템 등에서 더 자연스럽고 인간에 가까운 상호작용을 구현하는 데 유리합니다. [Medium](https://becomingahacker.org/a-quick-introduction-to-langgraph-enhancing-llm-applications-with-cyclic-workflows-145f61f38747)
        
3. **상태 관리(State management)**
    
    - LangGraph에서 각 노드(node)가 그래프 실행 중에 상태(state)를 전달하고, 이 상태는 단순한 데이터 집합이 아니라 각 노드의 출력에 의해 갱신됩니다. [Medium](https://becomingahacker.org/a-quick-introduction-to-langgraph-enhancing-llm-applications-with-cyclic-workflows-145f61f38747)
        
    - 예를 들어, `MessageGraph`라는 클래스는 채팅 모델(chat model)을 이용한 애플리케이션에 적합한 상태 구성(state)으로 설계되어 있으며, 채팅 메시지의 리스트 등을 상태로 사용합니다. [Medium](https://becomingahacker.org/a-quick-introduction-to-langgraph-enhancing-llm-applications-with-cyclic-workflows-145f61f38747)
        
4. **적용 가능성 및 활용 사례**
    
    - 단순한 질의응답 챗봇을 넘어 **적응형 학습 시스템(adaptive learning)**, **복잡한 의사결정 도구(decision-making tools)**, **동적 시뮬레이션 환경(simulation environments)** 등에 유용하다고 글은 설명합니다. [Medium](https://becomingahacker.org/a-quick-introduction-to-langgraph-enhancing-llm-applications-with-cyclic-workflows-145f61f38747)
        
    - 예컨대, 테이블(table) 데이터에 대해 반복적으로 가공하고 추론하는 “Chain-of-Table” 접근 방식이 언급되어 있으며, LangGraph를 통해 이러한 반복적 데이터 처리가 가능해졌다는 언급이 있습니다. [Medium](https://becomingahacker.org/a-quick-introduction-to-langgraph-enhancing-llm-applications-with-cyclic-workflows-145f61f38747)
        
5. **부가기능 및 특징**
    
    - 내장된 지속성(persistence), 사람 참여(human-in-the-loop), 그래프 시각화(visualization), ‘시간여행(time-travel)’ 같은 기능을 지원한다고 소개됩니다. [Medium](https://becomingahacker.org/a-quick-introduction-to-langgraph-enhancing-llm-applications-with-cyclic-workflows-145f61f38747)
        
    - 개발자들이 예제 코드나 Jupyter 노트북을 통해 LangGraph를 실험해볼 수 있도록 자료가 제공되어 있다는 언급이 있습니다. [Medium](https://becomingahacker.org/a-quick-introduction-to-langgraph-enhancing-llm-applications-with-cyclic-workflows-145f61f38747)
        

---

## 📝 왜 이 글이 의미 있는가

- LLM(대형 언어 모델)을 단순히 한 번 호출해서 끝나는 방식이 아니라, **연속적인 상호작용** 및 **상태 갱신 기반의 흐름**으로 설계할 수 있다는 점이 이전보다 더 강조됩니다.
    
- LangChain 기반으로 발전해 온 생태계에서 한 단계 더 나아가서 **에이전트-기반 구조, 상태ful 워크플로우** 등이 실험 가능한 수준으로 소개된다는 점에서 기술적 전환점으로 볼 수 있습니다.
    
- 실제로 개발자들에게 “이런 구조로 만들면 더 자연스럽고 강력한 애플리케이션이 가능하다”는 인사이트를 준다는 점에서 실무적 가치도 높습니다.