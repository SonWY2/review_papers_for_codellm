[요약]
- Instruction-Tuning은 Instruction의 다양성 강화를 통해 최적화 가능. 현재 현존하는 데이터셋들의 커버리지는 불충분함.
- Explore-Instruct: 도메인별 Instruction 튜닝에 사용되는 데이터 범위를 향상시키는 방법론 제안
  - 재귀적 너비/깊이 탐색(DFS) 기반 계층적 태스크 분할
- Explore-Instruct 방법론을 통해 생성한 데이터로 학습한 모델이 기존 human-curated나 self-instruct 방식으로 생성한 데이터를 학습한 모델 대비 우수한 성능 발휘

[EXPLORE-INSTRUCT 구현]
- 핵심 개념
  - 너비(Breadth): instruction의이 도메인 내 다양한 카테고리의 작업을 포함하는지. 한 도메인 내의 다양한 작업 범주에 대한 이해를 촉진하는 능력 위함.
  - 깊이(Depth): 세분화된 과제 분해와 관련. 철저한 이해와 정확한 문제 해결을 촉진.
    
- 탐색 방식
  - Lookahead Exploration(전방 탐색)
    - 작업을 더 깊이 탐색. 세분화된 하위 작업 발견 목적.
    - 주어진 과제 $$V_i$$를 M개의 하위 작업으로 분해. 관련은 있으나 기존 작업과는 다르도록 생성 유도.
    - 예)
      대상작업 $$V_i$$: 코드 생성
      생성된 하위 작업:
         1) 함수 작성: 주어진 사야에 따라 특정 기능 수행하는 함수 작성
         2) 오류 수정: 주어진 코드에서 오류 찾아 수정
         3) 코드 리팩토링: 기존 코드를 효율적이거나 쉽게 재구성
  - Backtracking Exploration(후방 탐색)
    - 다양한 분기 탐색으로 작업 범위 확대, 다양성 확보
    - $$V_j$$ 작업이 주어졌을 때, 이의 부모 작업인 $$V_i$$를 탐색
    - 이후 부모 작업 $$V_i$$ 에서 M개의 새로운 하위 작업 탐색
    - 예)
      대상 작업 $$V_j$$: 함수 작성
      탐색된 부모 작업 $$V_i$$: 코드 생성
      $$V_i$$에서 생성된 M개의 하위 작업:
         1) 단위 테스트 작성: 함수나 모듈을 위한 단위 테스트 코드 작성
         2) 문서화 : 문서화 주석 추가
         3) 성능 최적화: 코드 실행 속도, 메모리 개선

- 생성 프로세스
  - DFS(깊이 우선 탐색) 방식으로 Lookahead, backtracking 방식을 활용하여 도메인 내 작업 탐색. 
  - 중지 기준: 최대 탐색 깊이(K)와 너비(B)에 도달할 때까지 탐색을 계속합니다.
    - K (최대 깊이): 탐색이 도달해야 하는 최대 깊이.
    - B (최대 너비): 각 깊이에서 탐색할 최대 작업 수.
  - 앞선 프로세스에서 생성된 도메인 트리의 각 sub task에 대해 LLM(여기서는 gpt-3.5)을 사용하여 N개의 명령어와 해당 응답 생성.

[EXPLORE-INSTRUCT 데이터 비교 분석]
- 비교 대상 데이터: Human-Curated, Domain-Aware Self-Instruct
  - Human-Curated: 오픈 소스 데이터셋(supernatural instruction)에서 선별 후 10,000개의 훈련 데이터 무작위 샘플링
  - Domain-Aware Self-Instruct: 최대 탐색 깊이 K=0. 기존 self-instruct와 유사한 방식으로 explore-instruct 보다는 단순화된 형태로 데이터 구성. 초기 시드 컬렉션에서 부터 무작위로 선택/통합해가며 10,000개 샘플링.
- 비교 결과:
  - ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/7330e092-9144-43d9-a8eb-a54e5812fc33)
    - Explore-Instruct 방식으로 생성된 instruction이 고유한 동사-명사 쌍 수가 다른 방식에 비해 많음.
    - 또한 다른 방식 대비 Instruction에서 사용되는 동사 쌍이 더 균일하게 분포되어 있음
  - ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/7da64062-bf00-4aed-b13a-d278128b4eb8)
    - seed data와 생성된 데이터의 ROUGE-L 중첩도 분석 결과 Explore-Instruct 방법론의 Rouge-L 점수가 상대적으로 작은 값에 집중됨.
  - 즉, Explore-Instruct가 도메인 공간에 대한 적극적인 탐색을 통해 다양성과 도메인 커버리지를 향상시킬 수 있음을 시사

[실험]
- 3가지 도메인에 대해 실험 수행; rewriting, brainstorming, math
- 측정방식 설정:
  - rewriting, brainstorming의 경우 자동 평가와 사람 평가를 사용하여 비교 성능 측정
  - math의 경우 수학 문제 풀이 정답에 대한 acc 메트릭을 사용하여 성능 측정
- 비교군: 
  - Explore-LM: Explore-instruct 데이터로 학습된 모델
  - Explore-LM-ext: Explore-instruct 데이터 수를 늘려 학습된 모델
  - DomainCurated-LM: Human-curated 데이터로 학습된 모델
  - Domain-Instruct-LM: Self-instruct 데이터로 학습된 모델
- Explore-Instruct 데이터 생성 조건
  - K=2. 각 깊이에서 탐색 폭 B=8, B=6으로 설정. 필터 threshold는 0.7
  - 생성 후 10,000개 무작위 샘플링
- 하이퍼파라미터 설정
  - 모델: LLaMA 7B
  - lr 2e-5, batch_size 128, AdamW, Context length 512, deepspeed zero-3
 
[실험 결과]
- Explore-LM은 자동/인간 평가 모두에서 다른 모델보다 우수한 성능. 최대 탐색 깊이(K), 데이터 수, 다양성 필터 순으로 성능에 큰 영향을 끼침
- 자동 평가 결과:
  - ChatGPT로 자동 평가 수행: 유용성, 관련성, 정확성, 세부수준을 기준으로 이유와 근거를 포함하여 스코어링 시킴.
  - ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/56a3db1d-2b02-41fa-a50e-974d62dc8e6d)
    ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/948cb5da-95e8-4527-84a5-ec7073002eaf)
  - 모든 영역에서 다른 기준 모델 대비 상당한 성능 우위.
- 사람 평가 결과:
  - 세명의 평가자에게 승/패/동점 을 기준으로 평가시킴.
  - ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/a6182fd0-a6bc-49f6-bd6e-4edb4a7c9b14)
  - 자동 평가와 대체로 유사하게 기준 모델 대비 우위.
- Explore-Instruct 설정에 따른 결과 비교
  - 최대 탐색 깊이(K) 
    - ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/5c9ae502-8c88-47cf-bc5c-20170c5b0f22)
    - 최대 탐색 깊이(K)를 0~2 범위내에서 변경하는 실험 결과, 최대 탐색 깊이를 늘리면 모델 성능 향상 확인
  - 데이터 양
    - ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/57997302-bfb6-4d58-8a7b-6d42742b8193)
    - 2,000 ~ 32,000개까지 데이터 수를 변화시켜 본 결과 2,000개의 Explore-Instruct가 10,000개로 학습된 Domain-Instruct-LM보다 뛰어남은 확인하였으나, 도메인 별로 차이가 발생한다는 것을 확인
    - 브레인 스토밍 영역에서는 데이터를 32,000개까지 늘려도 성능은 1.42%만 향상되었으나, 재작성 및 수학에서는 35.32%, 21.74%나 향상됨.
    - (이는 From Quantity to Quality 논문에서 코드와 같은 도메인에서는 데이터를 많이 가져가야 한다는 결과와 유사한 모습을 보여준 것으로 생각됨)
  - 데이터 품질
    - ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/e2b87271-59d0-44d9-bb64-9a14438c7073)
    - 다양성 필터 사용함에 따라 약간의 성능 향상 관찰.
    - 이 부분도 도메인에 따라 결과가 다랐으며, 재작성 및 수학 영역은 3~8%정도 성능 향상




