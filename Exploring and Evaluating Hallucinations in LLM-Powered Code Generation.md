release-date: 2024-05-11
arxiv: https://arxiv.org/abs/2404.00971v2
github: 

[개요]
- 기존 연구에서 등한시 했던 코드 생성에서의 Hallucination 유형과 범위에 대한 실증적 연구(요약 및 분류) 실시
- 코드 생성에서 발생하는 Hallucination을 5개의 주요 카테고리와 19개의 세부 유형으로 분류.
- HalluCode: 코드 LLM의 성능을 평가하는 벤치마크 개발 및 제안.
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
■ 수동 분석
- 주제 분류를 위해 3,084개중 656개 코드(전체 21%)를 샘플링하여 파일럿 분석 후 가이드(코드북) 생성.
- 각 코드 솔루션을 독립적으로 실행하여 정답 코드와 비교하여 Hallucination 유형을 문서화 하고 전문가 논의 진행. 여러 유형이 동시에 발생 가능
- 파일럿 분석 외 79%는 저자 중 한명과 2명의 신규 프로그래머가 검증하며 코드북이 다루지 않은 유형이 발생하는지 검증
■ NLP 환각 맵핑
- 자연어와 코드 환각 차이 이해를 위해 기존 자연어 분양의 환각 유형 분류를 따름.
- Input-conflicting hallucination(입력 충돌), Context-conflicting hallucination(문맥 충돌), Fact-conflicting hallucination(사실 충돌)

[분류/분석 결과]
![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/86426f20-2b93-4c53-9bfc-57e0f4902c4a)
- 3,086 샘플에서 2,119개 환각 발생.
- 환각 유형: 구체적으로 5가지 주요 범주와 19가지 유형의 환각으로 구성됨
  - 의도 충돌(Intent Conflicting): 사용자의 의도와 다른 내용 생성.
  - 맥락 편차(Context Deviation): 생성된 내용이 맥락과 일치하지 않음.
  - 지식 충돌(Knowledge Conflicting): API나 식별자 지식과 모순되는 내용 생성.
■ 주요 환각 유형
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
















3. 환각 유형
의도 충돌(Intent Conflicting): 사용자의 요구사항과 일치하지 않는 코드.
맥락 불일치(Context Inconsistency): 코드의 다른 부분과 일관성이 없는 코드.
맥락 반복(Context Repetition): 중복된 코드.
데드 코드(Dead Code): 실행되지 않는 코드.
지식 충돌(Knowledge Conflicting): 사실과 맞지 않는 코드.
4. 연구 결과
코드 LLM은 다양한 유형의 환각에 영향을 받음.
여러 환각이 동시에 발생할 수 있으며, 대부분은 기능적 오류를 초래하거나 이를 나타냄.
LLM의 코드 생성 능력을 향상시키기 위해 환각 탐지 및 완화 기술 개발 필요.
5. 기여
LLM이 코드 생성에서 발생할 수 있는 환각 유형을 분석하고 분류한 첫 종합 연구 수행.
환각 분포와 코드 정확성 간의 상관관계를 체계적으로 분석.
환각 평가를 위한 벤치마크 HalluCode 개발 및 공개.
HalluCode와 HumanEval을 사용하여 최신 코드 LLM의 환각 인식 실험 수행.
