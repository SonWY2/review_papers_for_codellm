deepseek-math: https://arxiv.org/html/2402.03300v3
deepseek-v2: https://arxiv.org/html/2405.04434v4

GRPO 
[Group Relative Policy Optimization]

![image](https://github.com/SonWY2/review_papers_for_codellm/assets/36894403/2a8ac347-1c48-466d-b4ee-18865a0ee236)
ppo 수식: ![image](https://github.com/SonWY2/review_papers_for_codellm/assets/36894403/a7e3a936-8b50-4678-ab4a-8515d74897ad)
dpo 수식: ![image](https://github.com/SonWY2/review_papers_for_codellm/assets/36894403/f5c852b5-0ba3-4961-98aa-73261231cb1c)

- 그룹 보상 기반 : GRPO는 여러 샘플링된 출력의 평균 reward를 기준선으로 사용. PPO에서 가치 함수 추정(critic model) 사용 X
  - 메모리와 계산 자원 절약
- PPO와 달리 GRPO는 reward 계산에서 KL 패널티를 추가하지 않고, 학습된 정책과 기준 정책 사이의 loss에 직접 추가
  - PPO는 reward 값에서 직접적으로 KL 패널티를 빼서 정책 업데이트
    ppo kl penalty : ![image](https://github.com/SonWY2/review_papers_for_codellm/assets/36894403/ae1de05b-2740-454e-b87e-62d9b3119244)
  - GRPO는 loss function에 직접 추가됨.
    grpo kl penalty : ![image](https://github.com/SonWY2/review_papers_for_codellm/assets/36894403/9134490e-bc62-4ab2-9b91-7661bfd95367)
  - 계산이 단순화되고 안정적인 학습 유도
- 따라서 GRPO는 PPO와 A^_i,t(advantage)를 계산하는데 차이가 있음.
  - PPO는 GAE를 사용하여 이점을 계산하는데 반해 GRPO는 그룹 내 출력들의 상대적인 보상을 기반으로 계산
  - 즉 각 질문 query에 대해 old정책모델에서 여러 개의 출력을 샘플링하고 이 출력들에 대해 보상 모델을 사용하여 보상 r_i를 계산
  - 추가적으로 이 샘플링된 출력 간의 보상을 평균과 표준 편차를 사용하여 정규화 한 값이 advantage로 활용됨.
- 과정
알고리즘: ![image](https://github.com/SonWY2/review_papers_for_codellm/assets/36894403/55be0491-5ca6-4243-9ba6-1d0807e2017e)
  - 강화학습 단계에 따라 보상 모델이 충분하지 않을 수 있다는 점을 감안하여 GRPO를 반복적으로 수행
  - t+1 iteration에서는 정책모델에서 샘플링 결과를 기반으로 보상 모델을 위한 새로운 train dataset 생성하여 10%의 과거 데이터를 포함하는 재생 매커니즘을 사용하여 보상 모델 계속 훈련.
  - 이 후 참조 모델을 정책 모델로 설정하고 새로운 보상 모델로 정책 모델을 계속 훈
- 각 출력의 끝에서만 보상을 제공하는 것은 수학적 작업에서 정책을 감독하는 데 충분히 효율적이지 않을 수 있음
- 이에 각 출력의 단계에서 점수를 매기고 보상을 얻는 프로세스 감독 방식도 테스트 수행
  outcome supervision RL 방식: ![image](https://github.com/SonWY2/review_papers_for_codellm/assets/36894403/f2327b67-5cbc-4442-8e55-b4c0b6fecc7c)
  process supervision RL 방식: ![image](https://github.com/SonWY2/review_papers_for_codellm/assets/36894403/5e039b70-3f60-4992-bb35-a2dfbb2dc3e9)
  * outcome 방식과 다르게 각 토큰 단계에서 reward를 계산

[Insight of Reinforcement Learning]
- 
- RL 훈련데이터는 144K개의 질문으로 구성된 GSM8K와 MATH와 관련한 CoT 형식의 질문.

[강화학습 통찰력]
■ 각 학습 방법론별 비교
- SFT, RFT, DPO, Online RFT, PPO/GRPO 비교
  (표)- SFT : 사람이 선택한 SFT 데이터에서 사전 훈련된 모델을 미세조정
  - RFT: SFT 모델을 SFT 질문을 기반으로 한 SFT 모델에서 샘플링된 필터링된 출력물을 기반으로 더 세밀하게 조정.
  - DPO:  SFT 모델을 SFT 모델에서 샘플링된 증가된 출력물을 사용하여 쌍별 DPO 손실을 이용해 미세 조정
  - Online RFT: SFT 모델을 사용하여 정책 모델을 시작하고 실시간 정책 모델에서 샘플링된 증강된 출력으로 미세 조정
  - PPO/GRPO: SFT 모델을 사용하여 정책 모델을 초기화하고 실시간 정책 모델에서 샘플링된 출력을 강화
  각 method 비교: ![image](https://github.com/SonWY2/review_papers_for_codellm/assets/36894403/6efa89de-82bd-425d-abf1-b44a367f3a19)

■ 데이터 소스에 관한 관찰
- 데이터를 온라인/오프라인 샘플링 두 가지 범주로 분류
  - 온라인: 학습 데이터가 실시간 훈련 정책 모델의 탐색 결과에서 발생 (online RFT, GRPO)
  - 오프라인: 학습 데이터가 초기 SFT 모델의 샘플링 결과에서 발생 (RFT, DPO)
- 아래 그림과 같이 온라인 학습이 더 큰 이점 제공. 특히 RFT vs Online RFT에서 online RFT가 뒤로 갈수록 큰 우위 차지.
- 초기 단계에서 actor와 SFT모델이 매우 유사하며 샘필링된 데이터에서 약간의 차이만 드러나나 이후 단계에서는 큰 차이가 발생하며 샘플링에 더 큰 이점을 제공하기 때문
각 method별 DeepSeekMath-Inst 1.3B모델 학습 결과: ![image](https://github.com/SonWY2/review_papers_for_codellm/assets/36894403/326cbe62-ff72-4e9c-87ae-4d890460b95e)

■ Gradient Coefficient에 관한 관찰
- 크게 보상함수를 Rule과 Model로 구분.
  - Rule: 답변의 정확성을 기반으로 응답의 품질을 판단
  - Model: 각 응답에 점수를 매기기 위해 보상 모델을 훈련시키고 활용. Reward Model의 학습데이터는 규칙 판단에 기반
- GRPO는 보상 모델이 제공하는 보상 값에 따라 기울기 계수를 고유하게 조정하고 다양한 응답에 따라 차등적으로 강화 및 불이익 가능하여 다른 방식보다 뛰어남
RFT reward function : ![image](https://github.com/SonWY2/review_papers_for_codellm/assets/36894403/7c2b3789-f5c7-468d-b755-2e8cdfae6e81)
GRPO reward function : ![image](https://github.com/SonWY2/review_papers_for_codellm/assets/36894403/395ab9c1-13d8-4044-87d6-af17727936e6)
- 또한 반복적인 형태의 강화학습을 적용하면 성능이 크게 향상됨(특히 첫번째 에서)
DeepSeekMath-7B의 반복 강화학습에 따른 정확도 변화 : ![image](https://github.com/SonWY2/review_papers_for_codellm/assets/36894403/9bd6664e-8f89-4baa-a0be-a997f3b0db94)

[Why RL Works?]
- RL 모델을 Pass@K 및 Maj@K의 두 가지 metric으로 평가 수행
  K변화에 따른 SFT/RL 모델에서 Pass@K/Maj@K 측정 결과 : ![image](https://github.com/SonWY2/review_papers_for_codellm/assets/36894403/61f2eee4-5cc5-4ba5-8773-4d60d8d4982e)
- RL은 Maj@K의 성능을 향상시키지만 Pass@K는 아님.
- RL이 출력 분포를 더 견고하게 만들어 전체적인 성능을 향상시키는 것으로 보임
  - => 즉, 근본적인 능력의 향상보다는 TopK에서 올바른 응답을 증가시킴으로써 이루어짐

[더 효과적인 강화 학습을 하기 위한 방법?]
- 데이터 소스, 알고리즘, 보상 함수 3가지의 구성요소를 컨트롤
1) 데이터 소스
- GRPO에서는 무분별한 핵심 샘플링을 통해 출력 샘플링 했음.
- 트리 탐색 방법 기반 등 고급 샘플링 전략을 파이프라인에 통합하면 더 성능 향상이 가능할 것으로 예측
  
2) 알고리즘
- 데이터와 보상 신호를 처리하여 기울기 계수에 보상 신호를 업데이트하고 모델 배개변수를 업데이트. 특정 토큰의 조건부 확률을 증가시키거나 감소시키는 형태
- 그러나 보상 신호가 항상 신뢰할 수 없으며 공개된 데이터셋은 잘못된 정보가 많이 포함되어 있음.
- 따라서 noise가 있는 보상 신호에 강건한 알고리즘이 필요할 것
  
3) 보상 함수
- 보상 모델의 일반화 능력을 향상하는 방법이 중요. 보상 모델의 분포 밖의 질문과 decoding 출력을 처리하기 위해 효과적으로 일반화 되어야 함.
- 세밀한 훈련 신호를 제공할 수 있는 고품질 프로세스 보상 모델을 효율적으로 구축하는 방법 필요

---
DeepSeek v2
- DeepSeek-Math에서 사용한 GRPO그대로 활용
- 단, Outcome supervision RL 방식을 사용하여 최종 출력 reward를 통해 가치 함수 계산함. Process supervision이 복잡한 태스크에 대해 더 좋은 성능을 발휘한다 했는데 왜인지 언급이 없음.
DeepSeek-V2 논문의 가치함수 수식 :  ![image](https://github.com/SonWY2/review_papers_for_codellm/assets/36894403/e85c65bd-d95c-4eb6-b8a7-179815429034)

■ 훈련 전략
- two stage 학습 전략 채택
- 1 stage에서는 reasoning을 위한 추론 정렬 수행하고
- 2 stage에서 일반적인 답변 개선을 위한 다중 보상 프레임워크 채택
  stage framework : ![image](https://github.com/SonWY2/review_papers_for_codellm/assets/36894403/c2c64495-acd6-4a3e-8f08-69de33f3018a)
- 1 stage 보상 모델 학습을 위해 compiler 기반 피드백으로 코드 선호 데이터를 선별하고, ground-truth 레이블을 바탕으로 수학적 선호 데이터 선별
- 보상 훈련을 위해 DeepSeek-V2 Chat로 보상 모델을 초기화하고 point-wise나 pair-wise loss로 훈련했음.
- 훈련 추론에 대한 다른 병렬 전략 / 학습 추론에 vLLM 활용 / 모델을 CPU-GPU 오프로드 스케쥴링 전략 등 효율 최적화를 위한 몇가지 테크닉 적용

■ 결과
![image](https://github.com/SonWY2/review_papers_for_codellm/assets/36894403/87a9094a-c2e9-4d4a-b477-d5132ffa1fb3)
![image](https://github.com/SonWY2/review_papers_for_codellm/assets/36894403/4853b78f-ef50-43c4-9575-8041f367a6ae)
- 의도했던 대로 성능 향상 달성
- 단, 강화학습의 "정렬 세금"으로 특정 벤치마크에서 일반적인 성능이 저하되는 현상이 발생했음
- 자신들은 최종적으로 절충안에 달성했다고 하지만 미래에 추가적인 연구가 필요하다고 언급
- 그리고 DeepSeek-Math에서 더 효과적인 강화학습을 위한 개선포인트들이 v2버전에서는 아직 적용되지 않았음.




