[개요]
- LLM의 코드 생성에서 Chain-of-Thought(CoT)와 같은 프롬프팅 기법의 정확한 메커니즘과 효과성을 탐구
- Single-turn과 Multi-turn 코드 생성 방식에서의 프롬프팅 전략 분석
- 추론(reasoning), 명령(instruction), 실행 피드백(execution feedback) prompt의 체계적인 분해와 실험 수행
- CodeContests와 TACO 벤치마크에서 다양한 규모의 LLM(Llama 3.0/3.1, 8B/70B/405B, GPT-4o)을 대상으로 광범위한 실험 수행
- 주요 발견:
  - single-turn에서 reasoning 및 instruction prompt가 가장 좋은 성능을 발휘하며 CoT는 큰 모델과 어려운 문제에서 더 효과적
  - Multi-turn도 성능을 개선하지만 CoT와 결합 시 성능이 크게 향상됨. 단, execution feedback이 항상 성능 향상으로 이어지진 않음.(다양성 감소)
  - multi-turn CoT로 LLM을 미세조정하여 추론 능력을 내재화할 수 있음
  
---
[프레임워크]
![image](https://github.com/user-attachments/assets/c42a5172-0caf-4652-bf62-fba98f933798)
** LLM Multi-turn 코드 생성 기술 평가 프레임워크 구성 **
**위: 기본 멀티 턴 설정에서, 프로그래밍 문제가 주어지면 모델은 코드 솔루션을 생성하고, 런타임 환경과 상호 작용하여 실행 피드백을 수집하며, 실패 시 재시도
**아래: 기본 설정에 더해, 추론(Reason.) prompt, 명령(Inst.) prompt, 실행 피드백 prompt를 수집. 문제 설명에 추론 prompt가 추가. 추론 prompt에 대한 답변을 생성한 후, 명령 prompt에 따라 프로그램 코드 생성

■ 문제 구성
  - 프로그래밍 문제 D = {s, u, t} 구성
    - s: 자연어로 된 문제 설명
    - u: public test 세트
    - t: private test 세트
  - 코드 샘플 c는 모든 테스트를 통과해야 correct로 판정

■ Prompt 유형
- 문제에 대한 코드 생성 시 다음 3가지 Prompt 유형으로 구분: Reasoning Prompt, Instruction Prompt, Execution Prompt

1) Reasoning prompt (NL → NL)
- 코드 생성 전 문제 이해와 해결 전략 수립을 위한 prompt(reflection, IO pair 설명 등)
- 8가지 세부 유형:
  - self-reflection: 문제를 자신의 언어로 재구성
  - explain IO pairs: 주어진 예제의 입출력 관계 설명
  - problem tag: combinatorics, dp 등 문제 유형 태그 예측
  - problem difficulty: 문제 난이도 평가와 edge case 분석
  - natural language solution: 나이브한 해법 생성 및 개선점 제시
  - multiple solutions: 여러 가능한 해결책 제시
  - write helper function docstring: 필요한 헬퍼 함수의 시그니처와 목적 기술
  - write intermediate variables: 필요한 중간 변수들의 타입과 목적 명세

2) Instruction prompt (NL → Code)
- 코드 생성 방식을 안내(helper 함수 사용, 제약사항 확인, 라이별 주석 등)
- 6가지 세부 유형:
  - no instruction: 기본 prompt
  - use helper functions: 의미있는 이름의 서브함수로 분할
  - check constraints: 제약조건과 변수 초기화 확인
  - comment for line: 각 라인별 주석 추가
  - function docstring: 각 함수의 docstring 작성
  - weak solution: 약한 해법 생성 후 개선된 알고리즘 제시

3) Execution Feedback prompt
- 실행 결과에 따른 피드백 (granularity 기준 구분)
  - Binary feedback: 단순 성공/실패 여부
  - Failed tests feedback: 실패한 테스트의 입출력값과 에러
  - Failed & passed tests feedback: 실패+성공한 테스트 정보 모두 제공
  - LDB feedback: 중간 변수값과 블록별 실행 결과 제공

>>
■ Multi-turn vs Single-turn 설정
- Single-turn: 주어진 prompt에 대해 단일 코드 샘플 c ~ π(·|p) 생성
- Multi-turn: Natural-Language-to-Code(첫 턴)와 Code-to-Code(이후 턴) 생성으로 구성
  - 최대 N번의 턴 동안 중간 코드 샘플 c1,...,cT 생성
  - 각 턴마다 이전 코드와 실행 피드백을 활용하여 다음 코드 생성
>>

■ 코드 생성 설정 (multi-turn vs single-turn)
1) Single-turn 설정
- prompt p에 대해 단일 코드 샘플 c ~ π(·|p) 생성
- Reasoning과 Instruction prompt 그리드 서치:
  - 8개 reasoning × 6개 instruction = 48가지 조합 실험
- Nucleus sampling (top-p=0.95)와 temperature=1.0 사용

2) Multi-turn 설정
- 최대 3턴으로 제한 (compute-optimal 기준)
- 각 턴마다 다음 수식으로 코드 샘플 생성:
  ci ~ π(·|p1, c1, p2, ..., ci-1, pi)
  - p1: 초기 사용자 prompt
  - pi (i>1): 실행 피드백 prompt

3) CoT-retry 전략
- 문제 난이도에 따라 적응적 추론 적용
- 1턴: CoT 없이 시도
- 2턴: 실패 시 "write code solution with guidelines" prompt 추가
- 3턴: 실패 시 "generate naive solution" prompt 추가

■ Rejection Sampling Fine-tuning (RFT)
1. 데이터 생성
- CoT-retry로 각 문제당 200개 궤적 생성 (최대 3턴)
- 마지막 턴에서 모든 테스트 통과한 궤적만 60% 유지
- Single-turn과 Multi-turn 궤적으로 분리
- 문제당 최대 50개 해법으로 LSH 기반 중복제거

2. 데이터 처리
- CoT prompt 제거하여 후처리
- TACO 테스트셋과의 contamination 체크 및 제거
  - 문장 임베딩 유사도 0.8 이상 필터링
  - 솔루션 크로스 테스트로 중복 검증

3. 학습 설정
- Multi-turn 궤적만 사용하여 학습
- 하이퍼파라미터:
  - Learning rate: 2e-6
  - Steps: 545
  - Sequence length: 8192
  - Global batch size: 524288 tokens
  - Cosine scheduling (warmup 10 steps)
  - Peak learning rate의 10%까지 annealing

■ 평가 방법
1. pass@k 메트릭
- k개 샘플 중 하나라도 모든 테스트 통과할 확률
- 계산 비용 고려하지 않는 한계

2. pass n@k 메트릭
- k개 생성 중 최대 n개 제출에서 통과 확률
- Public 테스트 성능 기반으로 n개 선택
- Multi-turn 설정과의 공정한 비교 가능



---
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
  - 
---
[파인튜닝 방법 - RFT(Rejection Sampling Fine-tuning)]

1. 데이터 수집:
- Llama 3.1 70B 모델에 CoT-retry 전략 적용해 CodeContests 학습셋에서 데이터 생성
- 각 문제당 200개의 코드 궤적 생성 (최대 3번 시도)
- 마지막 턴에서 모든 테스트를 통과한 궤적만 60% 유지
- Single-turn과 Multi-turn 궤적으로 구분하여 LSH 기반 중복제거 수행

2. 파인튜닝:
- CoT prompt를 제거한 상태로 Multi-turn 궤적에 대해 표준 cross-entropy loss로 학습
- 학습 파라미터:
  - learning rate: 2e-6
  - 545 steps
  - sequence length: 8192
  - global batch size: 524288 tokens
  - cosine scheduling (warmup 10 steps)
  - peak learning rate의 10%까지 annealing

[실험 결과]
■ Single-turn 설정에서의 발견
1. Reasoning과 Instruction prompt 결합이 최상의 성능
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
- 명시적인 CoT prompt 없이도 추론 능력 내재화 성공
- CodeContests와 TACO 모두에서 일반화된 성능 개선
- Multi-turn 설정에서 더 다양한 코드 생성과 자가 수정 능력 향상
- 특히 긴 궤적에서의 성능이 크게 향상됨

---
[결론]
- CoT는 모델 크기가 크고 문제가 어려울수록 더 효과적
- Multi-turn은 CoT와 결합했을 때 가장 효과적
- 실행 피드백은 단순할수록 코드 다양성 유지에 도움
- RFT를 통해 모델이 추론 과정을 효과적으로 내재화할 수 있음을 입증
