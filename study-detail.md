
# 문서 형식에 따른 split 차이

문서는 docx 를 활용하는게 좋다. pdf 문서는 한글 줄바꿈된 문자를 한 단어로 인식하지 못하는 문제가 있다.
( 영문 pdf는 단어가 줄바꿈되면 단어 자체가 다음줄로 내려감. )
docx는 단락으로 split을 하기 때문에 유리하다.

# 기본적인 RAG 흐름과 Langchain 사용시 RAG 편리함

### 기본 흐름

1. 문서 내용 읽기
    - langchain-community Docx2txtLoader 사용
2. 문서를 쪼개기
    - 토큰 수 초과 방지하고, input이 길면 답변 생성 자체가 오래걸린다.
    - langchaing-text-splitters RecursiveCharacterTextSplitter
    - chunk size 1500 토큰수
    - chunk_overlat 200 의도적으로 겹치게 읽게 하여 유사도검색시 우리가 원하는 문서를 가져올 확률을 끌어올린다. 문맥을 어느정도 같이 전달하려는 목적
3. 임베딩 , 벡터 데이터베이스에 저장.
    - 임베팅 모델을 따로 지정해야함.
    - 쪼개놓은 document list를 임베딩 모델과 함께 벡터 DB로 저장함.
    - 임베딩 모델에 따라서 차원(Dimensions)을 표현 가능한 array size가 다르기 때문에 모델별로 체크가 필요함.
4. 질문이 올 때 벡터 데이터베이스 유사도 검색.
    - 존댓말로 질문하면 퀄리티가 더 높아질 수 있음.
    - db.similarity_search(query, k=2)
    - Chroma의 경우 k 인자로 결과를 몇개 얻을 지 지정할 수 있다.
5. 유사도 검색으로 가져온 문서를 LLM에 질문과 같이 전달한다.

### langchain-hub, RetrievalQA 사용하기
- langchain-hub
    - 프롬프트를 많이 모아둠
    - 챗 프롬프트 템플릿을 가지고 있다.
    - 현재는 langsmith 으로 옮겨졌다.
    - 기존 langchainhub 패키지는 deprecated 상태이다.

langchain.chains RetrievalQA
    - qa_chain = RetrievalQA.from_chain_type(llm, database.as_retriever(), chain_type_kwargs={"prompt: prompt}) 
        - database.as_retriever() 는 벡터 DB라면 제공하는 기능이기 때문에 추후 의존성 변화가 있더라도 이 코드는 유지할 수 있다.
    - qa_chain({"query" : query})
    - 엄청 간단한 절차로 query할 수 있다.

# Langchain 사용 안할 경우
1. 문서를 읽어서 paragraph 단위로 나눈다. 
    - paragraph(문단) 단위로 쪼갬. 맥락을 보존하기 위함
    - 쪼개기전에 full_text도 구해야함.
2. 문서를 직접 쪼갠다. tiktoken 라이브러리로 토큰 계산한다.
    - full_text 넘겨서 encode하면 총 토큰수가 나옴.
    - decode하면 토큰수만큼의 텍스트로 변환됨.
    - 원하는 토큰 수 (chunk_size) 만큼 문서를 쪼개고 , decode해서 텍스트로 변환시킨다.
    - 모델의 context window를 넘어서지 않게 토큰을 줄여야 함.
3. 벡터 데이터베이에 저장.
    - 직접 db index 값 지정, 잘라낸 것 저장 등 손수 처리해야하는 코드가 많아짐.
    - 검색할때도 손수 처리해야 하는 코드가 많아짐.
4. llm에 질의할 떄에도 비슷하다.

# 답변 퀄리티, 정확도를 높이기 위한 전처리
1. 사내 문서에 이미지가 포함된 경우 파싱을 하지 못해서 올바르지 못한 데이터를 전달할 수 있다.
2. 이미지를 파싱할 수 있는 형태로 변환하여 임베딩을 해야 한다.
3. 테이블의 구조를 md 문법으로 바꾸는것이 model이 이해하기 편하다.

# Retriever 효율 개선
- 벡터 DB에 임베딩 해놓고 검색을 하는 것이기 때문에, 올바른 질문을 만드는 것이 중요하다.
- 예를들면 거주자의 소득세는 5천만원이고 그에 따른 세율은 ~~% 이다 라는 문서 내용이 있을 경우
- "직장인의 소득세는 얼마인가 ?" 라는 query를 했을 경우 우리가 원하는 답변은 우선순위가 낮아진다.
- 거주자와 직장인은 벡터 DB 상에서 유사도가 낮아 낮은 우선순위로 답변이 올 가능성이 높기 때문이다.
- 이를 해결하기 위해서는 Langchain 의 특성인 chain을 통해 , 직장인 -> 거주자 변환 뒤에 query할 수 있도록 최적화 할 수 있다.

1. 쿼리 수정 기법 사용

```python

dictionary = ["사람을 나타내는 표현 -> 거주자"]

prompt = 사용자의 질문을 보고, 우리의 사전을 참고해서 사용자의 질문을 변경해주세요. 만약 변경할 필요가 없다고 판단되면, 사용자의 질문을 변경하지 않아도 됩니다.
그런 경우에는 질문만 리턴해주세요. 
사전 : {dictionary}
질문 : {{question}}

// 질문 변경을 위한 chain
dictionary_chain = prompt || llm || StrOutputParser()

// 질문 변경
new_question = dictionary_chain.invoke({"question" : query})

tax_chain = {"query" : dictionary_chain} | qa_chain

// dictionary에 사용될 parameter전달
ai_res = tax_chain.invoke({"question" : query})

```

# Chat history
- 히스토리를 알려주므로써 사용자가 직전에 질문했던 내용을 기억하여, 같은 맥락 안에서 대답하게 할 수 있다.
- 매 질문마다 llm은 대화내용을 기억하지 못하니 history를 넣어주므로써 대화내용을 기억하는 것 처럼 답변하게 할 수 있다.
- 아래와 같은 형식으로 히스토리 내용을 전달하여 맥락을 갖을 수 있도록 한다.
    - system : text
    - human : text

# Few shot prompting
- 답변의 예제 format을 반복적으로 여러 케이스로 주므로써 llm 답변 퀄리티를 높일 수 있다.
- zero-shot일 경우 정확도는 10 이하로 매우 떨어지고, one-shot 최소 1번 이상 format을 지정해주면 정확도가 매우 상승한다.
- 채팅 히스토리를 관리하는 프롬프트와 같은 형태이다.
    - system : text
    - human : text
- 무작정 예제를 많이 넣지 말고 1개씩 늘려보면서 깎는게 중요함.

# LLM Evaluation
- LangSmith로 질문 목록과 답변 상세 데이터를 보며 지속적으로 유지보수 할 수 있도록 기능을 제공한다.
- 서비스를 안정적으로 운영하기 위해서 사용한다.
- 사용자가 정확한 정보를 받을 수 있는지 검사한다.
- LLM이 잘못된 정보, 할루시네이션을 방지하기 위해 사용한다.
- cost, answer socre, 할루시네이션 score 등을 확인할 수 있다.
- 예를들어 답변 평가자, 답변 도움 평가자(helpulness_evaluator) 등을 만들어 평가를 시킬 수 있다.

### Dataset
- 도메인 전문가가 작성한 정답지이다.

# LangSmith 란 ?
- 요청 하나가 내부에서 어떻게 처리됐는지 단계별로 추적하고 품질을 평가하는 플랫폼이다.
- RAG 챗봇에서 틀린 답이 나왔을 때 일반 로그만으로는 원인을 알기 어렵다. 검색이 엉뚱한 청크를 가져왔는지,  
  청크는 맞는데 프롬프트가 문제였는지, 모델이 무시했는지를 구분해야 하는데, 이 중간 과정을 보여 주는 도구가 LangSmith이다.