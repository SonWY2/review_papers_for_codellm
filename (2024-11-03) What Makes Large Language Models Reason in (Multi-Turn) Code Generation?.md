[개요]
- LLM의 코드 생성에서 Chain-of-Thought(CoT)와 같은 프롬프팅 기법의 정확한 메커니즘과 효과성을 탐구한 연구
- Single-turn과 Multi-turn 코드 생성 방식에서의 프롬프팅 전략 분석
- 추론(reasoning), 명령(instruction), 실행 피드백(execution feedback) 프롬프트의 체계적인 분해와 실험 진행
- CodeContests와 TACO 벤치마크에서 다양한 규모의 LLM(Llama 3.0/3.1, 8B/70B/405B, GPT-4o)을 대상으로 광범위한 실험 수행

[연구 방법론]
■ Framework 구성
- 문제에 대한 코드 생성 시 다음 3가지 프롬프트 유형으로 구분:
  1. Reasoning 프롬프트: 코드 생성 전 추론 유도 (NL → NL)
     - 문제 자체 반영, IO pair 설명, 문제 태그 예측 등
  2. Instruction 프롬프트: 코드 생성 방식 안내 (NL → Code)
     - 헬퍼 함수 사용, 제약사항 확인, 라인별 주석 등
  3. Execution Feedback 프롬프트: 실행 결과에 따른 피드백
     - Binary feedback
     - Failed tests feedback  
     - Failed & passed tests feedback
     - LDB feedback

■ Multi-turn vs Single-turn 설정
- Single-turn: 주어진 프롬프트에 대해 단일 코드 샘플 c ~ π(·|p) 생성
- Multi-turn: Natural-Language-to-Code(첫 턴)와 Code-to-Code(이후 턴) 생성으로 구성
  - 최대 N번의 턴 동안 중간 코드 샘플 c1,...,cT 생성
  - 각 턴마다 이전 코드와 실행 피드백을 활용하여 다음 코드 생성

[실험 설정]
■ 모델
- Llama Instruct 시리즈 (3.0, 3.1)
  - 8B, 70B 모델
- Llama 3.1 405B, GPT-4o (작은 샘플링 영역에서만 사용)

■ 벤치마크
1. CodeContests
- 학습셋 13k 문제, 검증셋 117문제, 테스트셋 165문제
- 각 문제마다 public/private/generated 테스트 포함

2. TACO
- CodeContests, APPS 등에서 수집된 문제들
- 테스트셋: easy, medium, medium-hard, hard, very-hard 각 200문제
- 첫 번째 테스트 케이스를 public 테스트로 사용

■ 평가 지표
- pass@k: k개 샘플 중 하나라도 모든 테스트를 통과할 확률
- pass n@k: k개 생성 중 최대 n개 제출에서 하나라도 통과할 확률
  - n개 제출은 public 테스트 성능 기반으로 선택
  - n=k인 경우 두 메트릭이 동일

[파인튜닝 방법 - RFT(Rejection Sampling Fine-tuning)]
1. 데이터 수집:
- Llama 3.1 70B 모델에 CoT-retry 전략 적용해 CodeContests 학습셋에서 데이터 생성
- 각 문제당 200개의 코드 궤적 생성 (최대 3번 시도)
- 마지막 턴에서 모든 테스트를 통과한 궤적만 60% 유지
- Single-turn과 Multi-turn 궤적으로 구분하여 LSH 기반 중복제거 수행

2. 파인튜닝:
- CoT 프롬프트를 제거한 상태로 Multi-turn 궤적에 대해 표준 cross-entropy loss로 학습
- 학습 파라미터:
  - learning rate: 2e-6
  - 545 steps
  - sequence length: 8192
  - global batch size: 524288 tokens
  - cosine scheduling (warmup 10 steps)
  - peak learning rate의 10%까지 annealing

[실험 결과]
■ Single-turn 설정에서의 발견
1. Reasoning과 Instruction 프롬프트 결합이 최상의 성능
- 더 큰 모델이나 어려운 문제에서 더 효과적
- 일부 CoT는 성능을 저하시킬 수 있음

2. 모델 크기에 따른 CoT 효과
- 작은 모델(8B)에서는 상대적으로 작은 이득
- 큰 모델(70B 이상)에서 더 큰 성능 향상

3. 문제 난이도에 따른 CoT 효과 
- 어려운 문제일수록 CoT의 효과가 더 크게 나타남
- TACO의 very-hard 테스트에서 Llama 3.0 8B의 pass@100이 2배 가까이 향상(2.1% → 3.9%)

■ Multi-turn 설정에서의 발견
1. 기본 Multi-turn의 한계
- 단독으로는 modest한 성능 향상
- 일부 경우 single-turn보다 성능이 떨어짐
- CoT와 결합 시 모든 모델에서 유의미한 성능 향상

2. 실행 피드백의 영향
- 상세한 실행 피드백이 항상 성능 향상으로 이어지지는 않음
- 생성된 프로그램의 다양성 감소로 큰 샘플링에서 성능 저하 발생

3. CoT-retry 전략의 효과
- 첫 시도 실패 시에만 CoT 적용
- 특히 Llama 3.1 모델에서 모든 샘플링 크기와 벤치마크에서 가장 우수

■ RFT 파인튜닝 결과
- 명시적인 CoT 프롬프트 없이도 추론 능력 내재화 성공
- CodeContests와 TACO 모두에서 일반화된 성능 개선
- Multi-turn 설정에서 더 다양한 코드 생성과 자가 수정 능력 향상
- 특히 긴 궤적에서의 성능이 크게 향상됨

[결론]
- CoT는 모델 크기가 크고 문제가 어려울수록 더 효과적
- Multi-turn은 CoT와 결합했을 때 가장 효과적
- 실행 피드백은 단순할수록 코드 다양성 유지에 도움
- RFT를 통해 모델이 추론 과정을 효과적으로 내재화할 수 있음을 입증
