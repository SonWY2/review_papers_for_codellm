relase_date: 2024-04-16
arxiv: https://arxiv.org/abs/2404.10719
github: x

[요약]
- LLM을 인간의 선호와 일치시키는 대표적인 두 방법인 보상 기반(PPO)과 비보상 기반(DPO) 방법은 비교 분석하고 성능을 향상시킬 수 있는 요소들을 탐색.
  - RQ: DPO가 RLHF 도메인에서 PPO보다 우수한가? PPO의 추가적인 성능 향상이 가능한가?
- DPO의 한계: DPO는 모델 출력과 선호 데이터 간의 분포 이동에 민감하며, 특히 도전적인 작업(code generation)에서 성능 저하 초래 발생.
- PPO는 다양한 작업에서 일관된 성능을 보여주며 도전적인 작업(code generation)에서 dpo보다 높은 성능. 특히 advantage normalization, large batch size, exponential moving average update를 활용하여 성능 향상이 가능.

[DPO의 한계]
- 현재 많은 학술적 연구에서 DPO가 주로 사용됨.
- 그러나 DPO는 1)훈련 목표의 이론적 문제와 2)분포 밖 데이터(OOD)에 더 취약한 특징을 가지고 있음.
- ppo와 dpo는 공통적으로 reward model의 잠재적 실패를 이용해 실제 human preference를 충족하지 않고도 높은 보상 획득 가능.
- DPO는 배포되지 않은 데이터를 악용하는 솔루션을 발견할 수 있음. 즉, reference policy가 사람의 선호도에 잘 부합하는 경우에도 policy를 과도하게 벗어날 위험 있음.
  - DPO는 (win, lose)로 주어진 선호 데이터에 맞춰 정책을 최적화하기 때문에, 학습된 데이터 분포에서 벗어난 새로운 데이터에 대해 잘못된 높은 확률을 할당 할 수 있음.
  - 반면에 PPO는 reward model을 통해 OOD 샘플을 평가할 수 있기 때문에 더 합리적인 결정 가능. 또한, KL divergence term을 통해 ref model과 거리를 유지하며 새로운 데이터에 과도하게 벗어나지 않도록 regularization됨.
  - (y1, y2, y3의 샘플을 통한 예시. DPO가 분포 밖 응답에 유리한 편향된 정책을 생성할 수 있음 시사) ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/30fe302c-7828-4d02-926b-cc86a7e57c0a)

[실험/검증]
■ SafeRLHF 데이터셋에 대한 DPO/PPO 실험 검증 수행
- SafeRLHF(helpfulness와 safety에 대한 preference pair 데이터)를 통한 검증 수행
- 목표: SFT, PPO, DPO를 통해 콘텐츠 생성의 안전을 우선시하는 LLM을 훈련.
- (학습 방법에 따른 모델 성능 비교)![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/bb6a4d1f-cb55-4c25-995a-38e706c861bf)
  * + filter dual-unsafe: SafeRLHF 데이터셋에서 양쪽 응답이 모두 안전하지 않은 데이터를 필터링
  * + filter dual-safe: SafeRLHF 데이터셋에서 양쪽 응답이 모두 안전한 데이터를 필터링
  * Iterative DPO: DPO를 여러 stage로 반복 학
  * Help: helpfullness 양일수록 긍정적
  * Harm: 음수일수록 긍정적
  * S.R(Safety Rate; 안전율): 높을수록 긍정적
- 기본 모델의 영향
  - SFT(Alpaca)모델을 참조 모델로 사용할 때, DPO는 S.R이 55.4%로 낮음. 이것이 기본 모델의 출력과 선호 데이터 간 분포 이동 때문이라 추측.
  - 선호 데이터 분포의 영향도 감소를 위해 SafeRLHF 데이터셋에서 SFT(safe)를 추가 fine tuning한 후 DPO를 재학습하면 이러한 문제가 다소 해결됨.
  - 표에서 보이듯이 분포 이동 문제를 해결하면 S.R이 16.4% 증가하고 help reward가 -4.19에서 -1.52까지 증가함.
  - But, PPO를 활용한 것보다는 부족
- 선호(preference) 데이터의 민감도
  - SafeRLHF 데이테셋에서 dual-safe한 데이터를 모두 제거하면 S.R이 크게 향상되지만 Helpfulness가 감소하는 것으로 미루어 과도한 필터링은 DPO 성능에 해로울 수 있음
- 선호 데이터 분포 영향
  - 추가 데이터를 수집하는 것이 성능에 이점이 있는지 추가 분석 수행
  - 기존 선호 데이터 대신 SFT(Safe)로 새로운 응답을 생성 후, 학습된 reward model을 통해 rabeling 하고 이 데이터로 DPO를 반복적으로 학습
  - 결론적으로 **Iterative DPO 방법을 통해 추가적인 개선 가능.** PPO의 helpfulness에는 미치지 못하나 S.R은 더 높은 수치.
- 실용적 고찰
  - DPO 성능은 모델과 선호 데이터셋 간의 분포 이동을 완화함으로써 향상될 수 있음 확인.
  - 분포 이동과 데이터의 noise 문제 완화를 위해 iterative한 DPO 학습 방법 권장.
  - 단, 완벽한 데이터가 반복적으로 추가되더라도 코드 생성과 같은 어려운 작업에서는 여전히 만족스럽지 않을 수 있음(추가실에서 확인)
  
[추가 실험]
■ 타 RLHF 방법론의 정렬 방법 추가 비교
- OpenAssistant Reward 모델로 평가 점수 비교(높을수록 좋음)
- PRO(Preference ranking optimization), RRHF(pangu-coder2의 RRTF 방법론)을 추가 비교 분석
- RRHF와 PRO는 DPO와 PPO보다 못했으며 PPO가 제일 선호되는 답변 생성
  (HH-RLHF test set에 대한 결과) ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/2e220d62-3b5c-4410-9bce-c41564afdb2e)

■ base 모델 변경에 따른 비교 분석
(LLaMA 1/2 7B 모델의 SafeRLHF 벤치마크 결과) ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/0497bd0c-4da6-4df9-8948-5af1eb0a16b3)
* Beaver: 공식으로 릴리즈된 모델.
- LLaMA 모델 학습에서도 PPO가 대체로 우수하며 안정적임.

■ Code generation에 대한 DPO/PPO 실험 검증
- APPS 데이터
  - 학습 데이터 구성
    - PPO: 테스트 케이스를 보상 신호로 사용(모두 통과하면 10, 아니면 0)
    - DPO: pair 데이터가 없으므로 DPO-iter방식 채용. 베이스 모델을 활용하여 5개의 코드를 샘플링하고 생성된 코드를 테스트 케이스로 정확성 확인.
  - 다양한 사이즈의 모델(CodeLLaMA 7/13/34B)와 타 모델들도 테스트
    - PPO는 모델 사이즈가 커짐에 따라 얻을 수 있는 효과도 긍정적이었음. (테스크 에키스를 보상 신호로 사용하는 것이 유의미한 효과 있음 방증)
    - DPO는 DPO-iter를 사용했음에도 SFT보다 개선된 성능X
  (Apps pass@ 결과) ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/a4dea38c-a509-4441-ad21-2742aaaeb47d)
  * AlphaCode를 제외하고 모두 pass@5 수치
  * AlphaCode는 문제 당 1000개 솔루션을 샘플링하고 모든 테스트케이스를 통과한 5개를 다시 추려 hidden 테스크 케이스로 재측정한 값
  * Intro, Inter, Comp는 문제 유형
- Code Contest
  - PPO는 APPS와 유사한 방식으로 학습. DPO는 데이터 셋에서 제공하는 올바른 코드와 잘못된 코드를 사용하여 선호도 데이터 세트 구성
  - 결론적으로 APPS 데이터 측정과 유사한 결론 도달. DPO를 반복해도 오히려 성능이 저하됨.
  (Code Contest pass@ 결과) ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/4ef06ef9-392b-454c-ac02-2b9fe0a2c28f)
  * 10@1k: 문제 설명에 있는 공개 테스트에서 1000개의 샘플이 평가되고, 그 중 10개만 숨겨진 테스트에 제출되는 것 의미.
 
[결론]
- DPO는 기본 모델 출력과 선호 데이터간의 분포 이동에 지나치게 민감함
- 반복적인 Iterative-DPO 방법이 정적 데이터에서 훈련하는 기본 DPO 보다 나은 방식임을 제안
- 그러나 이러한 방법도 Code 생성과 같은 어려운 task에서는 모델의 성능을 향상시키지 못하거나 오히려 저하시킨다는 점을 발견
- PPO는 어려운 task에서도 최대 성능 발휘.
- 단, Reward 방식에 대한 추가적인 연구가 부족하여 Reward 방식 변화에 따라 결과가 변경될지는 미지수.

