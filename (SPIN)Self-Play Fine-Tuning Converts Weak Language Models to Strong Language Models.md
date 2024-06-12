release-date: 2024-01-02
arxiv: https://arxiv.org/abs/2401.01335
github: https://github.com/uclaml/SPIN

[개요]
Can we empower a weak LLM to improve itself without acquiring additional human annotated data?
(사람의 주석이 달린 데이터를 추가로 확보하지 않고도 약한 LLM 스스로 개선할 수 있도록 지원할 수 있을까요?)

- 알파고와 같은 자가학습 방법론에서 영감을 얻은 self-play 방식의 finetuning 기법(SPIN) 소개
- 기존의 SFT 데이터를 학습데이터로 그대로 활용하여 LLM이 자신의 인스턴스를 상대로 플레이하도록 한 후 자체 생성 응답과 SFT 데이터의 y와의 응답 구분을 통해 정책을 개선하는 방식.
- 응답을 구별하는 플레이어와 응답을 생성하는 플레이어가 상호작용을 통해 주 플레이어의 정책을 최적화하고 목표 데이터 분포에 수렴하도록 학습하는 방법 (GAN 과 유사)
- DPO를 통해 훈련된 모델보다 더 뛰어난 성능

[SPIN]
- 추가적인 사람 또는 AI 피드백에 의존하지 않고 LLMs의 성능을 향상시킬 수 있는 새로운 finetuning 방법(iterative한 방식으로 LLM을 학습시킬것을 제안)
  - 일반적으로 파인튜닝된 LLM 이 주어졌을 때 SFT 방식을 추가로 적용하면 효과가 떨어지고 잠재적으로 성능이 저하될 수 있음
  - RL 적용시에는 사람의 피드백이 필요하여 데이터셋 확보가 추가적으로 필요
- main/opponent 플레이어로 구성하여 main 플레이어를 훈련시키고 opponent 플레이어를 업데이트 함.
  - main player: LLM에서 생성된 응답과 사람이 생성한 응답을 구별.
  - opponent player: 사람의 반응과 구별할 수 없는 반응을 생성. 반복 t+1 에서 이전 반복(t)의 LLM
    
■ Main player의 학습
- objective function: main 플레이어 f_t+1 이 목표 데이터 분포 p_data와 opponent 플레이어의 분포 pθ_t 사이의 예상값 차이를 최대화
  ![image](https://github.com/SonWY2/review_papers_for_codellm/assets/36894403/f69f9c4b-213d-4953-b77a-6ecd962a872b)
  * F_t는 함수클래스의 집합으로 pθ_t에 종속되어 있음.
  * f_t+1과 x prompt에 대한 응답 시퀀스 y가 주어졌을 때, f_t+1(x,y)의 값은 y가 pθ이 아닌 p_data에서 유래했다는 main player의 믿음의 정도를 반영함
  * main 플레이어 f_t+1은 y가 p_data에서 샘플링 되었을 때 높은 값, opponent 플레이어의 응답 시퀀스에서 샘플링 되었을 때 낮은 값을 산출해야 함

- 일반적인 최적화 문제로써 아래 수식을 풀 수 있음
  ![image](https://github.com/SonWY2/review_papers_for_codellm/assets/36894403/12d2517f-655a-4bca-ac65-bd528d79e463)
  * loss 함수 l을 도입하여 argmin 최소화 문제로 변환
  * loss 함수는 값이 커도 천천히 증가하고 음수로 출력되지 않는 logistic loss function 사용

■ opponent player 업데이트
- main 플레이어가 실제 p_data와 생성된 데이터를 구별할 수 없는 응답을 생성하도록 하는것은 예상값을 최대화함으로써 달성 가능
- 단, t의 분포 pθ_t와 pθ_t+1의 과도한 편차를 방지하고 selfplay 안정화를 위해 KL 정규화 항 통합
  (opponent player 최적화 문제) ![image](https://github.com/SonWY2/review_papers_for_codellm/assets/36894403/9c8a6a0f-4789-42fe-a815-0461e4c049ed)
  (최적화 문제의 closed-form solution) ![image](https://github.com/SonWY2/review_papers_for_codellm/assets/36894403/841ea534-05e9-47b3-8c56-ad81dfb43867)
- opponent player는 main player 의 피드백을 통해 응답의 품질 개선

■ 통합된 end-to-end 학습 objective
- 위에서 언급한 main/opponent player를 아래식을 통해 한번에 업데이트.
- lambda 부분은 현재모델과 이전모델의 출력 확률의 로그 비율
- θt+1 를 업데이트하기 위해, 우리는 현재 모델 θ가 이전 모델 θt 와 얼마나 다른지 측정. 비교를 통해 업데이트.
  (end-to-end training objective) ![image](https://github.com/SonWY2/review_papers_for_codellm/assets/36894403/73aa7d50-c8a6-4663-8a6b-cb308b041cce)
  (SPIN 업데이트 algorithm) ![image](https://github.com/SonWY2/review_papers_for_codellm/assets/36894403/d87f0193-a5f7-4c55-9960-0828d51eb95e)

[실험 설정]
- 기본 모델
  - Zephyr-7b-sft-full (mistral-7B에서 Ultrachat200k로 파인튜닝된 모델)
- 학습과정
  - Iter0에서 50k, Iter1~3에서 100K 샘플링하여 사용
  - epoch 2
- 평가
  - Huggingface Open LLM Leaderboard
    (평가 데이터 상세 내용) ![image](https://github.com/SonWY2/review_papers_for_codellm/assets/36894403/079aa8e0-6773-4ce6-9299-a68a5712ace6)

[실험 결과]
- iter 횟수가 증감함에 따라 성능이 지속적으로 증가함.
- 단, t+1의 반복은 t 반복에서의 개선폭보다 작음. 이를 모델의 한계점이라 추측
  (SPIN학습에 따른 benchmark 측정 결과) ![image](https://github.com/SonWY2/review_papers_for_codellm/assets/36894403/662b2899-17b4-4f8d-abac-4aff1d27bba7)
  ![image](https://github.com/SonWY2/review_papers_for_codellm/assets/36894403/9f909fb9-e4c3-408b-b9d6-4d45747f554f)
- DPO랑 비교
  - DPO는 학습을 위해 새로운 추가 데이터를 활용하였지만 성능 측면에서 같은 데이터를 반복적으로 활용한 SPIN보다 낮음.
  (SPIN vs DPO) ![image](https://github.com/SonWY2/review_papers_for_codellm/assets/36894403/76e3dd83-eb71-4f12-8c59-bbf946c80ca6)

[Ablation study]
■ training size
- 다양한 training data 크기가 SPIN 성능에 미치는 영향 조사
- SPIN은 데이터 크기가 커지면서 성능도 지속적으로 향상되는 반면 SFT는 오히려 성능이 저하되는 현상 발생
  (SPIN vs SFT 데이터 사이즈에 따른 Open LLM 리더보드의 평균점수 비교) ![image](https://github.com/SonWY2/review_papers_for_codellm/assets/36894403/a5be221e-1840-4db3-9dfd-cba9c05044d6)
    * SPIN은 iteration 마다 학습 크기를 14k, 26k, 50k로 점진적 확대
    * SFT는 mistral-7B를 3 epoch로 완전히 미세조정한 결과 비교

■ Iterative Training vs Training for More Epoch
- epoch의 수를 늘리는 것과 반복적인 학습 방법의 점수 차이 비교 실험
- epoch는 처음 2번째까지 가장 큰 개선이 이루어지는 반면 그 이후부터는 소폭 상승하거나 감소하는 현상도 발생
- 단일 반복으로는 달성할 수 있는 성능에 내재적인 한계가 있음을 시사.
  (epoch 반복에 대한 성능 저하) ![image](https://github.com/SonWY2/review_papers_for_codellm/assets/36894403/bbe3e430-f519-4813-9311-18f2f2f754b5)

■ 다른 벤치마크 추가 조사
- MT-bench, Big-bench, OpenBookQA 등 다양한 과제에 대해서도 추가 조사.
- SPIN은 대부분의 다양한 작업에서 반복을 진행하며 성능이 향상됨.
  (MT-bench에서의 모델 성능 비교) ![image](https://github.com/SonWY2/review_papers_for_codellm/assets/36894403/3b9a5194-9d81-4cba-98a8-8c12d8f52f69)
  (여러 벤치마크에서의 모델 성능 비교) ![image](https://github.com/SonWY2/review_papers_for_codellm/assets/36894403/cab368aa-1984-4bfb-ba9a-78549a30c6b5)




