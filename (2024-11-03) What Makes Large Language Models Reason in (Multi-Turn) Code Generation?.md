release-date: 2024-10-12
arxiv: https://arxiv.org/abs/2410.08105
github: x

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

■ Single-turn vs Multi-turn generation: Problem Setting
- Single-turn 
  - 프로그래밍 문제 D = {s, u, t} 구성
    - s: 자연어로 된 문제 설명
    - u: public test 세트 (코드의 기본적인 정확성 검증용. multi-turn 피드백 생성에 활용)
    - t: private test 세트. 최종 코드의 완전환 정확성 검증용(예 :edge case)
  - 코드 샘플 c는 모든 테스트를 통과해야 correct로 판정
- Multi-turn
  - NL → Code (첫 턴)
    - 자연어 문제 설명을 코드로 변환하는 초기 생성. 기본적인 문제 해결 전략 구현 및 가장 기본적인 해결 방안 제시.
  - Code → Code (이후 턴)
    - 이전 코드와 피드백을 기반으로 개선하는 코드 생성. 각 턴마다 다른 전략 적용 가능하며 이전 시도의 문제점 해결 시도.
  - 코드 시퀀스 생성: c1,...,cT
    - ci ~ π(·|p1, c1, p2, ..., ci-1, pi)
      - p1: 초기 사용자 프롬프트(문제설명)
      - pi: i번째 턴의 피드백 프롬프트
      - ci: i번째 생성 코드 
    - T ≤ N (최대 턴 수)
    - public 테스트 통과하거나 N턴 도달할 때까지 반복


■ Prompt 유형
- 문제에 대한 코드 생성 시 다음 3가지 Prompt 유형으로 구분: Reasoning Prompt, Instruction Prompt, Execution Prompt
![image](https://github.com/user-attachments/assets/16981413-78bd-4c13-a098-e25e0e6e93f6)
** 실험에 사용된 prompt 유형 개요 **

1) Reasoning prompt (NL → NL)
- 코드 생성 전 문제 이해와 해결 전략 수립을 위한 prompt(reflection, IO pair 설명 등)
- 세부 유형:
  - self-reflection: 문제를 자신의 언어로 재구성, 세부사항/뉘앙스/예제 분석
  - explain IO pairs: 주어진 예제의 입출력 관계 설명, 일관성 검증
  - problem tag: combinatorics, dp 등 문제 유형 유형 예측
  - problem difficulty: 문제 난이도 평가, edge case 분석
  - natural language solution: 나이브한 해법 생성 및 개선점 제시
  - multiple solutions: 여러 가능한 해결책 제시
  - write helper function docstring: 필요한 헬퍼 함수의 시그니처와 목적 기술
  - write intermediate variables: 필요한 중간 변수들의 타입과 목적 명세

2) Instruction prompt (NL → Code)
- 코드 생성 방식을 안내(helper 함수 사용, 제약사항 확인, 라이별 주석 등)
- 6가지 세부 유형:
  - use helper functions: 의미있는 이름의 서브함수로 분할
  - double check the import, variable, constraints: 제약조건과 변수 초기화 확인
  - comment before each line: 각 라인별 주석 추가
  - docstring before each function: 각 함수의 docstring 작성. 용도 및 입력값과 출력값을 예상하고 설명. 
  - generate weak solution and a second better one: 약한 해법 생성 후 해당 약점을 파악한 다음 문제해결을 위한 개선된 알고리즘 제시
  - step by step: 단계적 알고리즘 제안

3) Execution Feedback prompt
- 실행 결과에 따른 피드백 (granularity 기준 구분)
  - Binary feedback: 단순 성공/실패 여부
  - Failed tests feedback: Runtime 에러가 발생한 경우 traceback과 함께 실패한 단위테스트의 예상/실제 값
  - Failed & passed tests feedback: 실패+성공한 테스트 정보 모두 제공
  - LDB feedback: 중간 변수값과 블록별 실행 결과 제공

---
[실험 세팅 #1 - 프롬프팅]

■ 모델
- Llama Instruct 시리즈 (3.0, 3.1)
  - 8B, 70B 모델
- Llama 3.1 405B, GPT-4o (작은 샘플링 영역에서만 사용)

■ 벤치마크
- CodeContests
  - 학습셋 13k 문제, 검증셋 117문제, 테스트셋 165문제
  - 각 문제마다 public/private/generated 테스트 포함
- TACO
  - CodeContests, APPS 등에서 수집된 문제들
  - 테스트셋: easy, medium, medium-hard, hard, very-hard 각 200문제
  - 첫 번째 테스트 케이스를 public 테스트로 사용

■ 평가 지표
- pass@k: k개 샘플 중 하나라도 모든 테스트를 통과할 확률
- pass n@k: k개 생성 중 최대 n개 제출에서 하나라도 통과할 확률
  - n개 제출은 public 테스트 성능 기반으로 선택
  - n=k인 경우 두 메트릭이 동일
- 무조건 k를 늘리는 것은 계산적으로 최적이 아님.

![image](https://github.com/user-attachments/assets/a6016347-f04e-4af5-be15-9c6251989b1c)



** Llama 3.1 70B 의 code context 턴 수에 따른 pass rate **


■ 실험 설정 상세
- Single-turn 실험
  - Reasoning과 Instruction 프롬프트의 그리드 서치
    - 8개 reasoning × 6개 instruction = 48가지 조합
    - 모든 모델에 대해 동일한 조합 평가
    - 각 조합의 pass@1, pass@10, pass@100 성능 측정
  - 샘플링 설정
    - Nucleus sampling (top-p=0.95)
    - Temperature=1.0으로 출력 다양성 확보
    - 모든 LLM에 대해 동일한 샘플링 파라미터 적용

- Multi-turn 실험
  - 턴 수 최적화
    - 최대 3턴으로 제한
    - pass 10@100 메트릭으로 compute-optimal 분석
    - 3턴 이상에서는 compute 효율성 감소 확인
  - 각 피드백 유형별 pass n@k 성능 비교
  - CoT-retry 전략
    - Turn 1: 기본 프롬프트로 시도
    - Turn 2: 실패 시 "write code solution with guidelines" 추가
    - Turn 3: 실패 시 "generate naive solution" 추가

■ 세부 실험 설정
- Single-turn 샘플링
  - pass@k 평가: k = {1, 10, 100}
  - 모델 크기별 성능 변화 분석
  - Temperature 1.0과 0.2 비교 실험

- Multi-turn 샘플링
  - 문제당 200개 trajectory 생성
  - pass n@k 평가: (n, k) = {(1,3), (10,30), (33,100), (100,300)}
  - bootstrapping으로 10000회 재샘플링

- 모델별 특화 실험
  - GPT-4o: 제한된 pass 1@1, pass 1@3 평가
  - Llama 3.1 405B: 작은 샘플링 영역만 평가
  - Llama 3.0/3.1 8B, 70B: 모든 실험 수행

[실험 결과 #1 - 프롬프팅]
![image](https://github.com/user-attachments/assets/62e4a071-21b5-4001-a497-a72ecf3281c7)
** multi-turn CoT를 통해 10%이상 성능 향상 **

![image](https://github.com/user-attachments/assets/77d62426-9fed-4810-a092-6bb0e6a1b0c0)
** CoT를 통한 모델별 벤치마킹 점수 변화 **

■ Single-turn 설정에서의 발견
1) Reasoning과 Instruction prompt 결합이 최상의 성능
![image](https://github.com/user-attachments/assets/2f85e6df-2629-4871-bc2e-06fff1fcddc1)
** reasoning 과 instruction을 결합한 경우가 최상의 성능 **
- 더 큰 모델이나 어려운 문제에서 더 효과적
- 일부 CoT는 성능을 저하시킬 수 있음
- 모델별로 최적 프롬프트 조합이 다름
- Solution-based instruction이 Llama 3.1 시리즈에서 가장 효과적
  ![image](https://github.com/user-attachments/assets/3c11fd04-86bc-4a85-ba57-da6ea3fae1c0)
  ** Llama 3.1 모델에서의 instruction prompt 유형별 비교 **

2) 모델 크기에 따른 CoT 효과
- 작은 모델(8B)에서는 상대적으로 작은 이득
  - Llama 3.0 8B: pass@100에서 +5.0%
  - Llama 3.1 8B: pass@100에서 +3.3%
- 큰 모델(70B 이상)에서 더 큰 성능 향상
  - Llama 3.0 70B: pass@100에서 +9.3%
  - Llama 3.1 70B: pass@100에서 +5.2%

3) 문제 난이도에 따른 CoT 효과 
- 어려운 문제일수록 CoT의 효과가 더 크게 나타남
  ![image](https://github.com/user-attachments/assets/2475aae3-6b06-4d63-a38c-c9d667be3f26)
  ** 문제 난이도별 pass rate변화 **
- TACO의 very-hard 테스트에서 Llama 3.0 8B의 pass@100이 2배 가까이 향상(2.1% → 3.9%)

■ Multi-turn 설정에서의 발견
1) 기본 Multi-turn의 한계
- 단독으로는 modest한 성능 향상(대부분 +2% 미만)
- 일부 경우 single-turn보다 성능이 떨어짐
  * 특히 Llama 3.0/3.1 8B에서 발생
- CoT와 결합 시 모든 모델에서 유의미한 성능 향상

2) 실행 피드백의 영향
- 단순한 피드백이 더 효과적
- 상세한 실행 피드백이 항상 성능 향상으로 이어지지는 않음
- 생성된 프로그램의 다양성 감소로 큰 샘플링에서 성능 저하 발생
  ![image](https://github.com/user-attachments/assets/db5f75e3-6ea6-4d1e-b619-8a9eb37e6111)
  ** feedback 유형별 대화 내의 연속된 코드의유사성 점수 분포 ** 
- Binary/Failed tests feedback이 최적

3) reasoning 프롬프트의 누적 효과
- 코드 생성 전에더 많은 추론 정보를 포함하는 것이 정확한 솔루션 코드로 안내할 것이라 생각될 수 있음
- 그러나 경험적으로 모든 턴 안에서 reasoning 단계 수는 '하나'가 적합. 많아질수록 성능 저하됨. 

4) CoT-retry 전략의 효과
- 첫 시도 실패 시에만 CoT 적용
- 특히 Llama 3.1 모델에서 모든 샘플링 크기와 벤치마크에서 가장 우수



---
[실험 세팅 #2 - RFT(Rejection Sampling Fine-tuning)]
- CoT Rejection Sampling Fine-tuning 기법은 모델의 추론을 내면화할 수 있는 방법임.

■ 데이터 수집 및 필터링
1) 초기 데이터 생성
  - Llama 3.1 70B 모델에 CoT-retry 전략 적용
  - 각 문제당 200개의 코드 trajectory 생성 (최대 3턴)
  - CodeContests 학습셋의 7238개 문제에 대해 생성

2) 데이터 필터링
  - 마지막 턴에서 모든 테스트를 통과한 궤적만 60% 유지
  - Single-turn과 Multi-turn 궤적으로 분리
  - LSH 기반 중복제거: 문제당 최대 50개 해법으로 제한

3) Contamination 제거
  - TACO 테스트셋과 중복되는 288개 문제 제거
  - 최종 6950개 문제의 데이터 사용
  - Single-turn에서 6422개, Multi-turn에서 7463개 엔트리 제거

■ 데이터 처리
- CoT 프롬프트 제거
  - 수집된 trajectory에서 CoT 관련 프롬프트 제거
  - 모델이 자체적으로 추론 능력을 학습하도록 유도

- 최종 데이터셋 구성
  - 177,475개 Single-turn 궤적 (143M 토큰)
  - 160,600개 Multi-turn 궤적 (285M 토큰)
  - Multi-turn 궤적만 사용하여 학습 진행

■ 학습 파라미터
- learning rate: 2e-6
- Gradient update steps: 545
- sequence length: 8192
- global batch size: 524288 tokens
- cosine scheduling (warmup 10 steps)
- peak learning rate의 10%까지 annealing

---
[실험 결과 #2 - RFT(Rejection Sampling Fine-tuning)]
- 명시적인 CoT prompt 없이도 추론 능력 내재화 성공
- CodeContests test set에서 pass 10@100: 50.9% → 57.2%
  ![image](https://github.com/user-attachments/assets/00480e2c-effa-4f85-b435-93b563bd0382)
  ** RFT 모델(Llama 3.1 70B)의 
- TACO 모든 난이도에서 일관된 성능 향상
  ![image](https://github.com/user-attachments/assets/25edca0b-1f8b-4fe9-a973-e150ff2962f7)
  ** multi-turn CoT와 RFT 모델의 TACO 측정 결과 **
- Multi-turn 설정에서 더 높은 성능. 특히 긴 궤적에서의 성능이 크게 향상됨
- 더 많은 텍스트 출력 생성 (non-code fraction 0.37 → 0.50)
  ![image](https://github.com/user-attachments/assets/64a43667-610d-467c-8beb-cb760fbe45ee)
  ** RFT 모델의 Non-Code Fraction 값 변화 **
- 더 다양한 코드 생성 패턴, 자가 수정 능력 향상
  ![image](https://github.com/user-attachments/assets/b0239675-c7f6-4428-bfe2-20eea24956ad)
  ** RFT 여부에 따른 코드 유사도 변화 **


---

[결론]
- CoT는 모델 크기가 크고 문제가 어려울수록 더 효과적이지만, 부적절한 CoT는 성능을 저하시킬 수 있음
- 최적의 CoT 전략은 모델과 샘플 크기에 따라 다르며, 하나의 프롬프트가 모든 상황에서 최선은 아님
- Reasoning과 Instruction 프롬프트를 결합했을 때 가장 좋은 성능을 보임
- Multi-turn은 단독으로는 제한적이나 CoT와 결합 시 모든 모델에서 유의미한 성능 향상을 보임
- 실행 피드백은 상세할수록 생성된 프로그램의 다양성이 감소하여 단순한 피드백이 더 효과적
- CoT-retry 전략(첫 시도 실패 시에만 CoT 적용)이 계산 효율성 측면에서 가장 효과적
- RFT를 통해 모델이 CoT의 추론 과정을 내재화할 수 있으며, 명시적 프롬프트 없이도 지속적인 성능 향상이 가능함
