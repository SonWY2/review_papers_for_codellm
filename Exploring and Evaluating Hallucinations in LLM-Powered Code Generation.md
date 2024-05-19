release-date: 2024-05-11
arxiv: https://arxiv.org/abs/2404.00971v2
github: 

[개요]
- 기존 연구에서 등한시 했던 코드 생성에서의 Hallucination 유형과 범위에 대한 실증적 연구(요약 및 분류) 실시
- 코드 생성에서 발생하는 Hallucination을 5개의 주요 카테고리와 19개의 세부 유형으로 분류.
- HalluCode: 코드 LLM의 성능을 평가하는 벤치마크 개발 및 제안.(공개된 곳 찾을 수 없음)
- HalluCode와 HumanEval을 사용한 환각 인식 및 완화 실험 수행.

[Taxonomy of Hallucinations in Code] - 코드에서 환각 분류
■ 분류 체계 구축
- 특정 벤치마크에서 추출한 샘플을 대상으로 솔루션을 생성하고 4명의 프로그래머의 추가 분석을 통한 코드에서의 환각 종류를 분석
- 대상 모델: CodeGen2(1B), CodeRL(770M), ChatGPT(3.5)
  - CodeGen2: 기본 temperature로 1개의 솔루션 생성
  - CodeRL: CodeT5의 강화학습 버전. Temperature 0.8을 기준으로 5개의 솔루션 생성
  - ChatGPT: 3.5-turbo 모델을 활용하여 greedy decoding으로 1개의 솔루션 / Temperature 0.8로 6개의 솔루션 생성
- 대상 데이터: HumanEval, DS-1000
  - 대상 모델을 활용하여 13,968개의 코드 수집(1,164개 문제 * 12개 솔루션)
    ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/96538c55-32e3-4403-b6fc-5ef96e6de9d3)
- 수동 분석
  - 주제 분류를 위해 3,084개중 656개 코드(전체 21%)를 샘플링하여 파일럿 분석 후 가이드(코드북) 생성.
  - 각 코드 솔루션을 독립적으로 실행하여 정답 코드와 비교하여 Hallucination 유형을 문서화 하고 전문가 논의 진행. 여러 유형이 동시에 발생 가능
  - 파일럿 분석 외 79%는 저자 중 한명과 2명의 신규 프로그래머가 검증하며 코드북이 다루지 않은 유형이 발생하는지 검증
- NLP 환각 맵핑
  - 자연어와 코드 환각 차이 이해를 위해 기존 자연어 분양의 환각 유형 분류를 따름.
  - Input-conflicting hallucination(입력 충돌), Context-conflicting hallucination(문맥 충돌), Fact-conflicting hallucination(사실 충돌)

■ 분류
![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/86426f20-2b93-4c53-9bfc-57e0f4902c4a)
- 3,086 샘플에서 2,119개 환각 발생.
- 환각 유형: 구체적으로 5가지 주요 범주와 19가지 유형의 환각으로 구성됨
  - 의도 충돌(Intent Conflicting): 사용자의 의도와 다른 내용 생성.
  - 맥락 편차(Context Deviation): 생성된 내용이 맥락과 일치하지 않음.
  - 지식 충돌(Knowledge Conflicting): API나 식별자 지식과 모순되는 내용 생성.

1) Intent conflicting; 의도 충돌(32.1%)
  - 생성된 내용이 사용자의 의도와 다름
  - 하위 분류:
    - 전반적 의미 충돌(semantic): 코드의 일반 기능이 과제 설명과 크게 다름.
      ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/4647b1b6-b73b-4224-b4ed-1c4f7dd81504)
    - 지역 의미 충돌(local semantic): 코드 내 몇몇 문장이 요구 사항과 모순됨.
      ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/bba0791b-3079-45c9-94c6-408ca1df0e34)
2) Context Deviation; 맥락 이탈
  - 생성된 코드가 입력 또는 생성된 맥락과 일치하지 않음
  - 하위 분류:
    - 불일치(Inconsistency) (31.8%): 맥락과 일치하지 않는 코드 조각 생성. (부정확한 표현, 상수, 조건, 분기, loop 등)
      (inaccurate expression) ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/e6066eeb-cb43-42cd-ad6a-b8ed64f908d2)
      (inaccurate condition) ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/c06b1cfe-129f-4fb3-9a10-4f06e3e7ac84)
    - 반복(Repetition) (17.3%): 불필요한 반복 생성.
      (copy input) ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/a7693ca5-8a22-4d1e-a3b5-de005aeaa05f)
      (repeat within the generated code) ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/e7577fb7-1665-44a1-a7fc-bd834179b308)
    - 데드 코드(Dead Code) (3.2%): 결과적으로 사용되지 않는 코드 포함.
      (redundant statement) ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/9c5c56bd-83be-4b35-a0a1-08902a204054)
3) Knowledge Conflicting; 지식 충돌 (15.1%)
  - API나 식별자의 지식을 위반하는 코드 생성.
  - 하위 분류:
    - API 지식 충돌: API 또는 라이브러리 함수의 잘못된 적용.
    - 식별자 지식 충돌: 잘못된 변수나 식별자 사용. 변수를 오용하거나 존재하지 않거나 제대로 정의되지 않은 변수 참조 등
      (using the wrong identifier) ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/22cc12f1-c17b-4090-a85a-ba0be9519462)

[분석 결과]
■ RQ1: Distribution of hallucinations; 환각의 분포
(단일 프로그램 내 다양한 환각의 동시 발생 분포) ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/11101329-96b4-413d-8d7b-568f0837af41)
- 주요 환각 유형: 의도 충돌(Intent Conflicting)과 맥락 불일치(Context Inconsistency)가 가장 흔함. 그 뒤를 맥락 반복(Context Repetition)과 지식 충돌(Knowledge Conflicting)이 따름. 데드 코드(Dead Code)는 가장 적음.
  - 의도 충돌/맥락 불일치 > 맥락 반복 > 지식 충돌 > 데드 코드
- 동시 발생: 한 프로그램 내에서 여러 환각이 동시에 발생할 수 있음.
  (단일 프로그램 내에서 두 가지 다른 환각이 동시에 발생하는 예제) ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/034302d9-5c28-4532-9721-bcb0efbc7462)
  - 9개의 프로그램은 세 가지 유형의 환각이 동시에 나타남.
- 맥락 반복: 다른 유형의 환각과 가장 많이 동시 발생, 이는 반복 패턴이 낮은 코드 품질을 의미. 다른 유형의 환각 발생도 촉진함.
- LLM별 환각 유형 분포
  - 모델별 차이: 각 모델에서 가장 흔한 환각 유형이 다르게 나타남.
    예) CodeRL: 기능적 완전성에 중점을 두어 의도 충돌이 많이 발생.
        ChatGPT: 강력한 프롬프트 이해와 코드 생성 능력을 가졌지만 맥락 불일치와 지식 충돌이 더 많이 나타남.
  - 변이의 중요성: 서로 다른 모델 간, 심지어 동일 모델 내의 다양한 디코딩 전략(예: ChatGPT-greedy vs ChatGPT-temp-0.8)에서도 상위 3개의 환각 유형이 일치하지 않음.
    - 이는 코드 LLM의 환각 유형이 복잡하고 다양함을 나타내며, 모델 사용 시 일관되고 신뢰할 수 있는 출력을 위해 모델의 한계와 편향을 고려하는 것이 중요함을 강조.
    (다양한 LLMs에서 환각 분포) ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/0beef11c-2775-4308-988d-6e336faf1869)


■ RQ2: 환각과 기능적 정확성의 상관관계
(다양한 유형의 환각에 대한 "모든 테스트 케이스 통과 코드" (모든 테스트 케이스를 통과할 수 있는 코드), "일부 테스트 케이스 통과 코드" (적어도 하나의 테스트 케이스를 통과한 코드) 및 "모든 테스트 케이스 통과하지 못한 코드" (0개의 테스트 케이스를 통과할 수 있는 코드)의 분포) ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/d2c99fe7-2d4f-4e5f-bab9-579cb9d1bd6b)

- 코드 오류: 모든 오류가 환각으로 인한 것은 아니며, 각 환각 유형이 오류 발생에 미치는 영향은 다를 수 있음.
  - 테스트 통과 코드 비율: 각 환각 유형에 해당하는 코드가 모든 테스트 케이스를 통과하는 비율은 10% 이하, 첫 두 유형의 경우 2% 미만.
  - 부분 통과 코드 비율: 첫 두 유형의 경우 10% 이상의 솔루션이 일부 테스트 케이스를 통과.
- 오류의 원인: 환각이 포함된 오류 코드 중 약 3%는 환각으로 직접 유발되지 않음. 이러한 코드 중 31%의 실제 오류 원인은 환각과 관련됨.
  예) 반복된 오류 코드 스니펫 생성, 반복 환각이 직접적인 오류 원인은 아니지만 다른 문제를 나타낼 수 있음.
- 환각 없는 오류 코드 비율: 오류가 포함된 코드 중 약 18.27%는 환각과 무관한 오류임.
  - 이는 단순한 문법 오류, 프롬프트의 모호성, 특정 제한이나 단계를 무시한 것, 경계 상황 처리 부족, 논리적 부주의 등의 이유로 발생.


[벤치마크 - HalluCode]
■ 구축
- 코드 LLM 구축을 위해 환각을 인식하고 완화하는 데 도움을 줄 수 있는 평가 벤치마크, HalluCode 제시. 5,663개의 Python 코드 생성 작업, 참조 솔루션, 및 대응하는 환각된 코드로 구성되어 있음.
- Code Alpaca를 기반으로 20,000개의 샘플을 자동 생성한 후 cherry picking 함.
- 데이터 전처리: Python 프로그램에 초점을 맞춰 환각 주입의 품질을 보장하기 위함.
  - Python 코드가 아닌 데이터를 필터링: 휴리스틱 규칙을 사용하여 14,311개의 데이터를 제외.
  - 입력 또는 출력과 일치하지 않는 몇 가지 지침을 식별하고 제거(48개)하여 최종적으로 5,663개의 유효한 데이터를 확보.
- 데이터 유효성 검사
  - 5,663개의 필터링된 데이터 중 360개를 무작위로 샘플링하여 환각 여부를 확인.
  - 16개의 환각 사례를 확인하여 약 4.44%의 환각률 도출. 이 정도의 환각률은 허용 가능하다 결론
- 환각 유형 할당
  - fully-connected network를 훈련하여 각 환각 유형에 대한 점수를 생성하고, o-1 프로그래밍 알고리즘을 사용하여 점수에 따라 환각 유형 할당.
  - 환각 코드 생성: 각 환각 카테고리에 대한 하위 카테고리 별로 환각 코드를 생성하기 위한 지침 기반 및/또는 휴리스틱 규칙 기반 접근법 설계.
■ 평가
- HalluCode를 통해 연구자들은 LLM이 생성한 코드의 환각을 심층적으로 연구할 수 있으며 LLM이 생성한 코드에서 환각을 인식하고 완화할 수 있는 능력을 평가할 수 있음
- 본 실험에서의 평가 대상 모델: ChatGPT 3.5, CodeLLaMA-7B, DeepSeek-Coder-7B
- 프롬프트 구: Objective - Hallucination categories description - Task(3단계)
  - Objective: 역할(코드 환각 인식기)과 LLM의 작업 설명
    (Your role is to act as a code hallucination recognizer. Your task is as follows: I will present you with a passage consisting of a question, an input, and an answer. The question poses a programming problem, and the input is some preconditions for the question this programming problem, such as variable definitions, constant definitions, etc., and the answer is a code snippet that may potentially contain a hallucination attempting to solve this problem.
)
  - Hallucination categories description: 다섯 가지 주요 코드 환곽의 설명과 예제 제공. LLM에게 가이드 역할
    (hall_type0. Intent Conflicting: Assess if the answer is less relevant to the question. This includes conflicts in organizing logic, such as an overall function that significantly deviates from the question without fulfilling its explicit function, or local logic conflicts with certain statements or logic less relevant to the question. Example:“question: Create a while loop to print all elements of an array.input:arr = [1, 2, 3, 4, 5] answer: arr = [1, 2, 3, 4, 5]\n i = 0\n while i <  len(arr):\n if arr[i] % 2 != 0:\n print(arr[i])\n i += 1 ”)
  - Task: 환각 인식 및 완화 작업과 출력 형식을 설명. 
    - CodeLlama와 DeepSeek-Coder는 환각 인식만 수행.(모델들이 본질적으로 환각을 완화할 수 없음)
      (Your task is to answer me with “Yes” or “No” whether the “answer” contains a code hallucination based on the previously mentioned hall types. If the “answer” contains a code hallucination based on the previously mentioned hall types, respond me only with “Yes” and the number of its hall type. If not, you just respond “No”. This is the format of your answer: “Yes, 0 (1,2,3,4)” or “No”.Now evaluate the following passage and give me your answer:)
    - ChatGPT는 환각 인식과 함께 환각을 제거하는 추가 작업 수행.
      (... If the “answer” contains a code hallucination based on the previously mentioned hall types, respond me with “Yes” and the number of its hall type. Then, respond the hallucination-free code after you modify the “answer” to remove its hallucination. Your response format is: “Yes, 1 (the number of its hall type). Right answer: (the hallucination-free code after you modify the “answer” to remove its hallucination).” If not, ...)
- 지표
  - Valid Rate (VR): 유효 출력의 비율 평가.
  - Accuracy of Hallucination Existence Recognition (Accrec): 환각 존재 인식의 정확도.
  - Accuracy of Hallucination Type Recognition (Acctype(i)): 환각 유형 인식의 정확도.
    - type0: 의도 충돌(Intent Conflicting)
    - type1: 맥락 편차-불일치(Context Deviation-Inconsistency)
    - type2: 맥락 편차-반복(Context Deviation-Repetition)
    - type3: 지식 충돌(Knowledge Conflicting)
    - type4: 맥락 편차-데드 코드(Context Deviation-Dead Code)
  - Accuracy of Hallucination Mitigation (Accmit): 환각 완화 능력 평가.(chatgpt만 해당)
- 결과
  ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/ab46cec9-2728-4f33-9540-cab9a2647c9d)
  - 전반적으로 ChatGPT는 두 벤치마크 모두에서 98% VR을 달성하여 다른 오픈 소스를 크게 앞섬. 오픈소스 모델은 과제를 이해하고 답변 생성에 어려움 겪음
  - 환각 인식 능력:
    - ChatGPT가 HalluCode와 HumanEval 양쪽에서 모두 최상의 결과 기록.
    - 전반적으로 환각 유형을 인식하는 것은 어려운 작업으로 강력한 LLM 조차도 주목할 성과는 달성하지 못한듯 보임.
    - 특히, 유형4 Dead Code를 가장 식별하기 어려워 함
  - 환각 완화 능력:
    - 오픈소스 모델들은 완화에 성공하지 못했으며, ChatGPT도 16%만 성공 했음. LLM의 경우 환각을 완화하는 것이 인식하는 것보다 더 어려움.

[논의점 및 결론론]
- 코드 LLM도 환각에 영향을 받는 것이 관찰됨
- 코드 LLM에서의 환각은 잘못된 코드 생성으로 이어질 수 있었으며 또한 LLM이 프롬프트를 통해 환각을 감지하고 수정하는 것이 어려운 것을 발견
- 코드 생성 맥락에서 효과적으로 환각을 완하하는 저문 기술 개발이 중요함
- 벤치마크 데이터 HalluCode는 환각 인식기를 훈련하는데 사용될 수 있으며 보상 모델로 활용 가능함.
 

  
