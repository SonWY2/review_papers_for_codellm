relase date: 2024-02-22
arxiv: https://arxiv.org/abs/2402.14658
github: https://github.com/OpenCodeInterpreter/OpenCodeInterpreter?tab=readme-ov-file


# 요약
- 코드 생성, 실행 및 반복적인 코드 개선을 위한 시스템(프로세스)를 구축하고 해당 시스템으로 OpenCodeInterpreter 방법론 및 모델 제안
  - User와 Model 및 Compiler 간의 상호작용을 68,000개의 멀티턴 형식으로 구축한 Code-Feedback 데이터셋으로 학습
- 실행(execution)과 human-feedback을 통합하고, compiler의 결과를 통해 오류를 수정하여 코드를 개선하고자 함
- OpenCodeInterpreter-33B는 GPT-4와 유사한 성능 달성

# Code-Feedback
코드 생성을 위한 3가지의 데이터 수립기준을 세우고 해당 기준을 다성하는 5가지 데이터셋 구성 방법 제시.
- 데이터셋 수립 기준
  - 1)실제 세계의 쿼리: 다양성 + 복잡성을 갖춘 실제 사용자의 쿼리와 가깝도록 구축
  - 2)multi-turn 대화 구조: 대화 구조로 구성되어 있으며 multi-turn 구조는 다시 실행 피드백, 인간 피드백 2가지 형태로 나뉨.
    - 실행 피드백(Execution Feedback): compiler의 출력 및 진단 관련 내용
    - 인간 피드백(Human Feedback): 사용자로부터의 추가 지침이나 명령(을 흉내)
  - 3)User는 텍스트, assistant는 코드로 응답하는 구조
- 데이터셋 구성 방법(5가지)
  - similar query packing for challenging query pools
  - human feedback simulation for challenging query pools
  - code correction for challenging query pools
  - similar problem packing for LeetCode problems
  - follow-up Q&A for LeetCode problems
- seed데이터(=쿼리 pool)
  - 필터링 후 156K, 원본은 287k 분량의 오픈소스 데이터셋(Magiccoder-OSS-instruct, code-ShareGPT-python, Magiccoder-Evol-Instruct, Evol-Instruct-Code)
    - Qwen-72B-chat 을 사용하여 선택적 필터링 수행.
    - seed 데이터를 complexity 기준으로 1~5 사이의 점수로 scoring 후, ≤4 의 데이터만 필터링
    - 선택의 견고성을 위해 두 가지의 다른 프롬프트로 반복 수행
    - #TODO 접기(프롬프트)
  - LeetCode의 코딩 과제 


1) 오픈소스 데이터셋 수집
2) 복잡성 기준으로 선택적 필터링을 통한 정제
3) E.F(execution feedback) 및 H.F(human feedback)과 결합하여 멀티턴 대화 데이터로 변환
3-1) single-turn seed
3-1-1) simillar query packing
   - Bert 임베딩을 활용하여 query 기준 가까운 벡터화 표현으로 변환
   - seed에서 선택된 쿼리에 대해 K-nearest neighbor 알고리즘을 활용하여 가장 가까운 네 개의 대응 query 식별
   - 이 때, 고유성 유지를 위해 한번 선택된 query는 후보군에서 제외(중복 방지)
   - 105K개 single-turn 인스턴스에서 16.6K의 multi-turn 인스턴스 생성됨
3-1-2) human feedback simulation
   - 사용자 상호작용과 피드백 중신의 솔루션 개선 과정을 학습시키기 위함.
   - GPT-3.5와 GPT-4를 사용하여 simulator 개발
   - seed에서 선택된 쿼리에 대해 GPT-3.5로 초기 응답을 생성하고, 여기에서 코드를 추출하여 실행.
   - 이 실행의 결과와 compiler 의 결과를 GPT-4에 입력하여 후속 응답 유도.
   - 이 과정은 올바른 해결책이 생성될때까지 최대 3번 반복됨.
   - syntax/formatting, effeciency, functionality, clarity, bugs, security, compatibility, resource use, scalability, best practices와 같은 10가지 범주를 사전에 정의한 후 GPT-4에서 가장 관련성 있는 범주를 선택하여 해당 범주 내에서 적절한 응답을 생성하도록 유도.
   - 앞선 단계에서 생성된 피드백을 multi-turn 데이터로 통합.
   - 51K개의 예제 생성
   - #TODO 3.5 프롬프트 + 4.0프롬프트예시 통합필요
3-1-3) code correction
   - 개발자가 지속적으로 코드를 디버깅하고 개선하는 실제 코딩 주기 모방하고자 함
   - GPT-4를 활용하여 의도적으로 잘못된 코드 조각 생성 유도 #TODO 프롬프트
   - 생성된 오류 코드의 실행 결과(execution feedback)를 다시 multi-turn 대화로 통합하는 방식으로 디버깅 과정을 시뮬레이션 함.
   - 500개의 구체적인 오류 수정 데이터 생성
3-2) leet-code seed
3-2-1) similar problem packing for LeetCode problems
   - 동일한 질문에 대해서 시간/공간 복잡성이 다르거나 다른 프로그래밍 언어로 구현된 솔루션들을 분리
   - 대안적 문제 해결방법에 대한 지식을 학습할 수 있는 200개 multi-round 인스턴스를 생성.
3-2-2) follow-up Q&A for LeetCode problems
   - LeetCode의 솔루션은 자연어 설명이 부족하여 GPT-4를 활용하여 텍스트 설명과 코드 조각을 통합
   - 응답의 명확성과 교육적 가치를 보장하기 위함.
   - #TODO: 프롬프트 추가

# 실험 셋업
- 베이스 모델: CodeLLaMA와 DeekSeekCoder 7/13/34/70B
- epoch 3, learning-rate 2e-5, warmup-ratio 0.05, cosine lr scheduler, token len 4096
  * 왜 모델 사이즈에 따라 하이퍼파라미터 최적화 안했지?
- 평가 셋: Humaneval, Humaneval+, MBPP, MBPP+로 실험 수행
- 평가 방법: single-turn(기존 방식), multi-turn 
  - single-turn 평가: 기존 평가 방식과 동일. 단 입력을 instruction 형태로 포맷팅하여 single-turn 생성 테스트(magiccoder와 유사)
  - multi-turn 평가: 반복적인 feedback(E.F, H.F, H.F oracle)과 통합하여 코드 생성 개선 측정
    - 최대 두 라운드까지 feedback을 반복함.
    - 평가는 통과하지 못한 결과에 대해 Execption, Not expected result, Timeout 의 세가지 시나리오로 제시
   
# 실험 결과
- Single-turn 코드 생성 #TODO 표1
  - OpenCodeInterpreter가 SOTA에 근접
- Multi-turn 코드 생성
  - 두 라운드 제한에서 E.F, H.F를 통합한 결과
  - GPT-4에 근접
  - H.F oracle 의 경우 예산 제한으로 6.7B와 33B모델에 집중한 결과 GPT-4를 상회.
 
# 데이터 관련 실험
- WizardCoder 110k 데이터셋을 활용하여 code-feedback 매커니즘의 영향을 분석하여 최적의 데이터 혼합 식별 검증
- 고품질 single-turn 데이터와 code-feedback 데이터를 2:1로 혼합할 때 가장 좋은 성능 발휘
  - 고품질 single-turn 데이터가 중심을 잡아주는게 중요
- code-feedback 데이터 구축 방식의 개별적인 실험과 E.F와의 통합 실험을 통해 해당 구축 방식이 전반적으로 복잡한 코딩 과제의 해결에 효과적이었음을 주장








  
