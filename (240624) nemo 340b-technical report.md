Nemotron-4 340B Technical Report
release-date: 2024-06-17
arxiv: https://arxiv.org/abs/2406.11704
github: x


Alignment에 대한 파트 요약
[Reward Modeling]
- 선호도 순위 및 품질 필터링을 위한 보상 모델 학습. 해당 모델을 향후 합성 데이터 생성 필터링 및 선호도 구분에 활용.
- HelpSteer와 유사한 방법에 따라 10,000명의 인간 선호도 데이터로 구성된 데이터 세트 HelpSteer2 구성.
  - 링크: https://huggingface.co/datasets/nvidia/HelpSteer2
- 기존 쌍별 순위 모델은 응답의 길이만으로 선호도가 작성되는 등 답변에서 실제 유용성을 분리하는데 어려움이 있었음. 따라서 '다중 속성 회귀 보상 모델(multi-attribute regression reward model)' 채택
  - 다중속성 회귀 보상모델은 유사한 응답간의 미묘한 차이 포착. 보상 예측에 더 효과적
  - Nemotron-4-340B 위에 최종 softmaxt 레이어를 보상 head로 추가.
  - 이 head layer는 마지막 layer의 hidden states를 5차원의 HelpSteer 속성(유용성, 정확성, 일관성, 복잡성, 상세성)의 벡터로 매핑하는 projection
  - 보상 판단의 정도를 벤치마킹하는 RewardBench를 통해 그 성능 입증.
    Reward Bench에서 모델 정확도 : ![image](https://github.com/SonWY2/review_papers_for_codellm/assets/36894403/c84b49c0-055a-4f50-bc7e-c7a2af9ef01c)

[데이터 생성 및 구성]
- 모델의 개선됨에 따라 기존의 데이터는 모델을 정렬시키는데 점점 더 부적절해지는 경향 있음
- Synthetic Data Generation(SDG) 방법론을 해결책으로 타묵
  - 데이터 생성 파이프라인으로 전체 align과정에서 SFT와 preference 조정에 사용되는 데이터 98%를 합성해서 사용.
  - 2%는 20k의 사람 주석이 달린 데이터(SFT를 위한 10k + HelpSteer2 10k)

1) Prompt 생성
- 다양성(과제 다양성, 주제 다양성, 지시 다양성)을 함유한 프롬프트 생성.
- UltraChat 데이터 및 CAMEL의 생성에 유사한 접근 방식을 채택.
- Mixtral-8x7B-Instruct-v0.1을 generator로 활용하여 <공개 QA, 글쓰기, 비공개 QA, 수학 및 코딩 등에 작업에 대해 별도의 합성 프롬프트>를 생성함
- 이 때, seed에서 다양한 주제 및 키워드를 generator에 전달하여 프롬프트의 다양성 확보
- 또한 예상 응답 형식(예: JSON 출력)을 명시적으로 정의하는 다양한 지침 생성
- 모델의 대화능력 향상을 위해 two-turn 프롬프트 생성

2) 합성 single-turn Prompt
open Q&A : ![image](https://github.com/SonWY2/review_papers_for_codellm/assets/36894403/2d125974-4b36-4eff-889a-83f34b8b2ea7)
writing : ![image](https://github.com/SonWY2/review_papers_for_codellm/assets/36894403/dfc04344-dd3d-461f-9a29-8cd0276f30be)
closed Q&A : ![image](https://github.com/SonWY2/review_papers_for_codellm/assets/36894403/08d2b469-a8ae-4ed4-8250-a59ee4bdeb5b)
Math & Programming :![image](https://github.com/SonWY2/review_papers_for_codellm/assets/36894403/d5ed4487-6b39-43a3-8daf-a02b83d4ca7b)

3) Synthetic two-turn prompts.
- SFT 대화 데이터세트는 원래 일반적으로 멀티턴이고 선호도 데이터는 싱글턴이지만 여기에선 선호도 데이터도 멀티턴(two-trun)으로 구축.
  (prompt는 User:XXX, Assistant:XXX, User:XXX 의 형태)
- sharegpt에서 첫번째 user/assistant 대화내역을 가져오고 두번째 user는 모델로 생성

4) 합성 Instruction-following Prompt
- single-turn 의 경우 prompt(예: 머신러닝에 대한 에세이를 작성하라) + instruction(예: 3단락 안으로 작성하라) 의 결합으로 특정 지침이 결합된 query 생성
- multi-turn의 경우 single-turn의 reponse를 merge해서 새로운 추가 instruction을 결합

5) 합성 Dialogue 생성
- 각 대화가 three-turn으로 구성되도록 설계.
- 반복적인 역할극을 통해 모델이 assistant와 user 역할을 시뮬레이션
  - user 턴에서 원하는 행동을 유도하기 위해 모델에 대화 기록과 함께 사용자 개성을 정의하는 명시적인 prompt 제공
  - user 질문 모방을 위해 공손한 표현(예: 감사합니다 등)은 제외하도록 후처리
- 생성된 데이터에 대해 Nemotron-4-340B-Reward 모델을 활용하여 일정 수준 이상의 대화 품질의 데이터만 유지.

6) Real-world LMSYS prompts.
- 실제 사용자 요청을 더 잘 반영하기 위해 LMSYS 데이터 추가(chatbot arena 데이터)
- 데이터를 분리해서 SFT용과 선호도 용으로 분리하여 활용함
- SFT데이터에서는 안전하지 않은 flag 데이터는 제거하지만 선호도 데이터에서는 부정적인 feedback을 위해 그대로 유지.
합성 데이터 vs LMSYS 데이터의 helpfulness 측정결과: ![image](https://github.com/SonWY2/review_papers_for_codellm/assets/36894403/46b9f75d-5d5b-4b35-872c-83f3de9ddeda)
* 단, 간단한 프롬프트가 더 도움이 된다는 평가를 받기 쉬움 유의. LMSYS가 더 길고 복잡함.

7) 합성 Preference Data Generation
- HelpSteer2 선호도 데이터로 Nemotron-4-340B-Reward 훈련
- (prompt, Y_chosen, Y_reject) 의 형태로 합성 선호도 데이터 생성을 위해 노력함
■ 응답 생성
- 여러 모델을 활용하여 기본 설정 데이터(합성 단일턴, instruction following, two-turn, sharegpt, LMSYS, GSM9K, Math 등) 의 응답 생성하여 다양한 응답 확보.
- Mt-Bench에서 가장 성능이 좋은 모델은 여러번 샘플링하여 더 까다로운 합성 선호도 샘플도 구축
■ 데이터 선호도 평가 및 정제
- Ground-truth 판단
  - 수학의 경우 ground-truth, 파이썬의 경우 compile feedback을 사용하여 평가하여 올바른 응답과 잘못된 응답을 구분
- LLM-as-judge, Reward-Model-as-Judge
  - LLM으로 응답을 비교하기 위해 평가 실시
    - 위치 편향을 피하기 위해 순서를 바꿔서 두 번 요청.
    - 두 번 모두 일관된 선호가 결정되면 유효하다고 판단하여 트리플렛(prompt, Y_chosen, Y_reject)으로 구성
  - Reward Model Judge
    - Nemotron-4-340B-Reward를 통해 (prompt, y) 쌍으로 보상 예측 추가 탐색.
  - 실험 결과 LLM-as-judge보다 Reward-Model-As-Judge가 더 정확도가 높았음.
  - 특히 Chat-hard에서 평균 정확도가 크게 차이남.

8) 추가 데이터 소스
■ topic following
- 주제 일관성을 위한 추가 데이터셋 CantTalkAboutThis 통합.
- 의도적으로 산만한 대화가 산재되어 있는 데이터셋을 통합하여 task 중심의 상호작용 동안 모델이 주제에 집중할 수 있는 능력 향상 도모
■ Incapable tasks
- 모델이 스스로 불가능한 작업(인터넷 접속, 실시간 지식 등)은 hallucination을 발생시킬 수 있음
- 이를 완화시키기 위해 직접 작성한 예시를 사용하여 LLM에 다양한 질문을 생성하도록 유도하는 few-shot 방식으로 질문을 만들고 LLM에 거부 응답을 명시적으로 요청하여 거부 응답 데이터셋 제작
- 처리할 수 없는 작업에 대한 핸들링 가능해짐
■ STEM datasets
- 논리 지식 향상용 Open-Platypus, PRM800K, SciBench, ARB, openbookQA 포함
■ Document-based reasoning and QA.
- 수치 추론 능력을 향상시키기 위해 FinQA
- 문맥화된 QA의 정확도를 높이기 위해 Chatqa: Surpassing gpt-4 on conversational qa and rag의 인간 주석 데이터 사용
- 반정형 데이터에 대한 모델의 이해를 강화하기 위해 wikitablequestions 데이터 세트 활용
■ Function calling
- 함수 호출 기능 향상을 위해 glaive-function-calling-v2(https://huggingface.co/datasets/glaiveai/glaive-function-calling-v2) 포함

[Alignment algorithm - SFT]
- 모든 데이터를 동시에 학습하면 행동 충돌이 발생하여 최적의 정렬 달성 X
- 특히 코딩 작업에서 이러한 현상이 강하게 관찰됨. 데이터 샘플링 가중치로도 해결하지 못함.
- 이러한 문제의 해결을 위해 2단계 SFT 전략 사용
1) 첫번째 단계 - code SFT
  - code 능력 향상을 위해 상당한 양의 데이터가 필요하다는 사실을 발견함
  - self-instruct와 evol-instruct를 활용하여 고품질 seed에서 삽성 샘플 생성
  - 생성한 데이터를 적합성 판단을 통해 필터링 후 지속적인 evol을 통해 일정 규모의 데이터 수집
  - 총 800K의 학습 데이터로 확장
  - 3e-7의 constant learning-rate와 128 global batch로 1 epoch만 학습했음
2) 두번째 단계 - General SFT
  - 앞서 설명한 code 제외 데이터 200K가 혼합된 데이터 세트로 추가 학습 수행
  - 망각 세금을 완화하기 위하여 Code SFT의 2% 데이터를 포함하여 학습
  - global batch 128 사이즈로 3epoch 학습. [1e-7, 5e-7]

[Alignment algorithm - Iterative Weak-to-Strong Alignment] 
- 데이터를 점진적으로 최적화를 향해 개선하여 반복적으로 학습하는 접근 방식 채택
- 이 접근 방식은 alignment와 데이터 합성의 강점을 결합하여 서로 상호 향상시키고 지속적인 개선 유도
- 모델의 품질은 여러 평가 지표의 조합으로 정의됨
- 초기 모델은 대화 및 선호도 데이터 모두에 대한 generator로 사용
- 이 생성된 데이터는 SFT및 선호도 조정에 사용하여 더 나은 모델을 정렬하는데 사용됨.
- 특이하게 teacher 모델은 student 모델의 크기에 구애받지 않고 기본 모델과 정렬 데이터가 개선됨에 따라 새로 정렬된 모델은 초기 모델을 상당한 차이로 능가 가능했음.
  - 초기 데이터 생성 모델은 Mixtral-8x7B-Instruct-v0.1
  - 생성된 데이터로 학습한 모델은 Nemotron-4-340B-Base (teacher가 student보다 작음)
  - 두 번째 반복에서는 결과물인 340B-Interm-1-Instruct가 데이터 generator로 활용
  - 두 번째 생성된 데이터는 340B-Interm-2-Base를 340B-Interm-2-Chat으로 훈련하는데 사용
Weak-to-Strong workflow : ![image](https://github.com/SonWY2/review_papers_for_codellm/assets/36894403/f6d3388e-2561-488e-ad25-a9690534d566)


[Alignment algorithm - Preference align(강화학습)]; DPO, RPO
- DPO와 본 논문에서 추가로 제시하는 RPO(Reward-aware Preference optimiation) 방법 모두 사용하여 여러 번 반복 개선작업 수행
1) DPO; Direct Preference Optimization
- 너무 오래 훈련하면 과적합이 발생하며 한 지표(예: Mt-bench)의 개선은 일반적으로 다른 지표(예: MMLU)의 저하를 동반했음.
- 위 문제를 완하하기 위해 기존 바닐라 DPO loss에 대해서 선택한 응답에 가중치를 부여한 SFT loss를 추가했음
- 저품질 데이터를 필터링하고 잘못된 선호도 학습을 막기 위해 ground-truth 등으로 실측되지 않은 데이터는 Nemotron-4-340B-Reward를 통해 고품질 데이터를 선택함.
- 총 160K의 다양한 태스크를 포함한 데이터 세트 구성
- global batch size 256, 1 epoch, [3e-8, 3e-7], KL 정규화 계수 [3e-4, 3e-3] 에서, SFT 가중치는 [1e-5, 1e-3] 에서 조정됨
2) RPO; Reward-aware Preference Optimization 
- DPO는 이진 순서만 사용하지만, 실제 보상 간의 차이는 더 유의미하다는 사실에서 착안. 
- RPO는 보상 gap을 근사화하는 방법으로 학습하여 과적합 문제를 방지할 수 있음
RPO 수식 :  ![image](https://github.com/SonWY2/review_papers_for_codellm/assets/36894403/e0d420b7-d936-459a-b72f-2d48584fee62)
D 메트릭 : ![image](https://github.com/SonWY2/review_papers_for_codellm/assets/36894403/bf1d688b-23db-4a59-8a0e-e0b763a9281e)
- DPO에서 훈련된 체크포인트를 초기화 및 참조 정책으로 사용하여 RPO 학습
- 품질 필터링이 덜 가혹한 300K의 예제로 구성된 데이터 사용
- DPO보다 더 작은 SFT 가중치 계수 1e-5 사용.
- 𝜂=1, 𝑙𝑟=3⁢𝑒⁢-⁢7 로 설정하고 KL 계수 β=[1e-3, 1.]에서 조정.
- 1 epoch 만으로 모든 작업에서 모델이 균일하게 개선됨 확인. 그러나 각 반복에서 이전 반복의 체크포인트를 초기화/참조 정책으로 사용하여 총 3번의 RPO 반복 실행.(해당 모델이 Nemotron-4-340B-Instruct)

