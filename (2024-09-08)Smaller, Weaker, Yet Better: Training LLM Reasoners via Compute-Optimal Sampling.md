![image](https://github.com/user-attachments/assets/7aaef6db-6c0d-44c3-80fe-f683a37505e5)release-date: 2024-08-30
arxiv: https://www.arxiv.org/abs/2408.16737
github: x

google deepmind

[개요]
![image](https://github.com/user-attachments/assets/bcbfa6fa-8a52-43e3-a91f-6e8e7e437648)

- 강력한 언어 모델(LM)에서 생성된 고품질 합성 데이터로 LM의 추론 성능을 향상시키는 것이 일반적인 전략.
- But, 이 논문에서는 고정된 추론 예산(FLOPs) 하에서 이 전략이 계산 최적인지 재검토함
- 더 강력하지만 비용이 높은(SE; Strong-Expensive) 모델과 더 약하지만 저렴한(WC: Weak-Cheaper) 모델에서 합성 데이터를 생성하는 것의 트레이드오프를 조사
- 주요 발견:
  - WC 모델에서 생성된 데이터가 더 높은 coverage와 diversity를 가질 수 있으나, 더 높은 false positive rate도 보임
  - 다양한 벤치마크와 finetuning 설정에서 같은 예산하에 생성된 WC 생성 데이터로 학습한 모델이 SE 생성 데이터로 학습한 모델을 일관되게 능가함
  - 이는 SE 모델에 의존하는 기존 관행에 도전하며, WC가 고급 LLM를 훈련하는 데 계산 최적의 접근 방식일 수 있음을 시사함
    - *추측: 원인은 점점 더 강력해지는 sLLM의 성능 때문인 듯

[방법론]
■ 고정된 예산에서 샘플링
- 고정된 샘플링 예산(FLOPs)에서 SE 모델보다 WC 모델에서 더 많은 샘플을 생성할 수 있음
- WC 모델(P_WC 매개변수)과 SE 모델(P_SE 매개변수)의 샘플링 비율:
  S_WC = (P_SE / P_WC) * S_SE
  (S_WC, S_SE는 각 모델에서 생성하는 샘플 수)
  - 예: SE 모델이 27B 매개변수, WC 모델이 9B 매개변수인 경우
     S_WC = (27/9) * S_SE = 3 * S_SE
- 이 비율은 모델 매개변수 비율에 따라 선형적으로 증가함


■ Finetuning 설정
- Student-LM finetuning: WC와 SE 데이터로 별도의 student LM을 finetuning (knowledge distillation)
- WC-LM finetuning: WC 모델을 WC 데이터(self-improvement)와 SE 데이터(distillation)로 finetuning
- SE-LM finetuning: SE 모델을 WC 데이터(weak-to-strong improvement)와 SE 데이터(self-improvement)로 finetuning

![image](https://github.com/user-attachments/assets/5d407ecd-3d43-4326-a35f-e3e7b1fc5910)
< SFT 설정 >


[실험 설정]
■ seed 데이터셋: 
- MATH (Hendrycks et al., 2021): 다양한 난이도의 수학 문제 (7500 훈련, 500 테스트)
- GSM-8K (Cobbe et al., 2021): 초등학교 수준의 수학 문제 (7500 훈련, 1319 테스트)
- Functional MATH (Srivastava et al., 2024): 전이 학습 연구용
■ 학습 모델: 
- WC: Gemma2-9B
- SE: Gemma2-27B
- Student: Gemma-7B
■ 하이퍼 파라미터:
- 배치 크기:
  - Gemma2-9B, Gemma2-27B : 32
  - Gemma-7B : 8
- steps:
  - 낮은 예산: Gemma2 모델 600 steps, Gemma-7B 2400 steps
  - 높은 예산: Gemma2 모델 6000 steps, Gemma-7B 24000 steps
  - 체크포인트: 10개의 균등 간격 체크포인트 저장, 최고 검증 정확도 모델 선택
- learning-rate:
  - {1e-7, 5e-7, 1e-6} 중 검증 성능 기반 선택
■ 합성 데이터 생성:
- MATH: 4-shot prompt
- GSM-8K: 8-shot prompt
- 샘플링 예산: 
  - 낮은 예산: SE 1개, WC 3개 솔루션/문제
  - 높은 예산: SE 10개, WC 30개 솔루션/문제
- 필터링:
  - 중간 추론 단계가 잘못된 데이터는 제거하지 않고 오직 솔루션(ground-truth)의 일치 여부만으로 데이터 필터링 수행
  - fpr의 데이터도 학습에 긍정적인 영향을 줄 수 있을 것이라 추측측
■ 평가 지표: 
- pass@1 accuracy
- maj@k (k = 1, 4, 8, 16) : temperature 0.7로 k개 솔루션 생성 후 다수결로 최종 답변 선택
■ 합성 데이터 평가: 
- coverage@k: k개의 솔루션을 샘플링했을 때, 최소 하나의 정확한 솔루션을 가진 고유한 문제의 비율
- diversity@k: k개의 솔루션을 샘플링했을 때, 문제당 평균 고유 정확 솔루션의 수
- false positive rate (FPR): 인간 평가 및 Gemini-Pro-1.5 자동 평가
  - 인간 평가: 각 모델에서 무작위로 50개 솔루션 선택
  - 자동 평가: Gemini-Pro-1.5를 사용하여 500개 솔루션 평가

[실험 결과]
■ 합성 데이터 분석
- Coverage:
  - MATH: WC가 SE보다 11%(낮은 예산), 6%(높은 예산) 더 높음
  - GSM-8K: WC가 SE보다 8%(낮은 예산), 1%(높은 예산) 더 높음
- Diversity:
  - MATH: WC가 SE보다 86%(낮은 예산), 125%(높은 예산) 더 높음
  - GSM-8K: WC가 SE보다 134%(낮은 예산), 158%(높은 예산) 더 높음
- FPR:
  - MATH: WC가 SE보다 7% 더 높음
  - GSM-8K: WC가 SE보다 2% 더 높음
![image](https://github.com/user-attachments/assets/8d9b1955-95ea-4e73-be25-056716eb2279)
< MATH 데이터에 대한 합성 데이터 분석 >

■ Finetuning 결과
- Student-LM (Gemma-7B):
  - MATH: WC 데이터가 SE 데이터보다 6%(낮은 예산), 5.8%(높은 예산) 더 높은 성능
  - GSM-8K: WC 데이터가 SE 데이터보다 4.2%(낮은 예산), 1.3%(높은 예산) 더 높은 성능
   
- WC-LM (Gemma2-9B):
  - MATH: WC 데이터(self-improvement)가 SE 데이터(distillation)보다 3.8%(낮은 예산), 2%(높은 예산) 더 높은 성능
  - GSM-8K: WC 데이터가 1.5%(낮은 예산) 더 높은 성능, 높은 예산에서는 비슷한 성능

- SE-LM (Gemma2-27B):
  - MATH: WC 데이터가 SE 데이터보다 5.8%(낮은 예산), 4.3%(높은 예산) 더 높은 성능
  - GSM-8K: WC 데이터가 SE 데이터보다 1.2%(낮은 예산), 1.5%(높은 예산) 더 높은 성능

  ![image](https://github.com/user-attachments/assets/2276dc2c-72b0-447c-96af-5c5f841208d0)
  < SFT 결과 - MATH >
  
  ![image](https://github.com/user-attachments/assets/4515f305-6999-45b5-b122-b2566fc0968d)
  < SFT 결과 - GSM-8K >


■ 일반화 능력 (Functional MATH)
- Gemma-7B: WC 데이터로 학습한 모델이 5.8% - 6.5% 더 높은 성능
- Gemma2-9B: WC 데이터(self-improvement)가 2.5% - 4.5% 더 높은 성능
- Gemma2-27B: WC 데이터와 SE 데이터가 비슷한 성능, k=8에서 WC가 2% 더 높음
![image](https://github.com/user-attachments/assets/dc6c5577-b449-431d-8c12-6dd2289ee52f)

< 일반화 능력 결과 >

[Ablation Studies]
■ 데이터셋 크기의 영향: 
- 500 문제로 축소한 MATH 데이터셋에서도 SE 데이터를 사용하는 것보다 WC 데이터를 사용하는 것이 이점 有
- Student-LM: 12.93%, WC-LM: 11.4%, SE-LM: 5.1%의 상대적 이득

![image](https://github.com/user-attachments/assets/b3cba448-b6ef-4684-b49a-0c881836bfeb)
< 데이터셋 크기에 따른 영향 >

■ 기본 vs 계산 최적 샘플링: 
- 계산 비용이 일치하는 WC 샘플링(3개/문제)이 단순히 동일한 수의 샘플을 생성하는 것(1개/문제)보다 우수
- 계산 최적 샘플링이 SE 데이터보다 더 나은 성능 제공

![image](https://github.com/user-attachments/assets/bbc1843f-7632-4ebf-80eb-da0abdadd5e7)
< 합성 데이터의 동일한 개수 샘플링 vs 계산량 일치 샘플링의 비교 >


3. Coverage와 Diversity의 역할: 
- 높은 coverage와 diversity가 모두 강력한 모델 훈련에 중요함
- (높은 coverage, 높은 diversity) > (높은 coverage, 낮은 diversity) > (낮은 coverage, 낮은 diversity)

![image](https://github.com/user-attachments/assets/7220320b-5923-4f6c-82f8-caded2da9949)
< 커버리지와 다양성의 역할 이해 > * (a) 낮은 다양성, 낮은 커버리지: 문제당 1개 솔루션 수집 (b) 높은 다양성, 높은 커버리지: 문제당 30개 솔루션 수집 (c) 문제당 30개의 솔루션 수집 후 1개의 올바른 솔루션만 유지 (높은 커버리지, 낮은 다양성)


[SoTA 언어 모델로 확장]
- Gemini-1.5-Pro (SE)와 Gemini-1.5-Flash (WC) 모델 사용
- 가격 기반 계산 최적 샘플링: Pro 1개 vs Flash 35개 솔루션/문제
- 결과:
  - Gemma-7B: Flash 데이터가 Pro 데이터보다 31.6% 더 높은 성능
  - Gemma2-9B: Flash 데이터가 14.4% 더 높은 성능
  - Gemma2-27B: Flash 데이터가 10.9% 더 높은 성능
- 비용 효율적인 데이터 샘플링:
  - Flash에서 5개 솔루션/문제 (Pro의 0.15배 비용)로도 Pro보다 우수한 성능 달성
  - Gemma-7B: 19.1%, Gemma2-9B: 9.8%, Gemma2-27B: 5.7%의 상대적 이득

![image](https://github.com/user-attachments/assets/71f61754-acc4-4d20-a130-83e8dbb2aad1)
< Finetuning Gemma with Gemini-1.5 data >

[결론 및 향후 전망]
- small LLM의 추론 성능이 큰 LLM보다 빠르게 향상되고 있어, 이 연구 결과의 중요성이 앞으로 더욱 커질 것으로 예상됨
- WC 모델에서의 계산 최적 샘플링이 SE 모델보다 더 효과적인 LLM 훈련 방법이 될 수 있음
- 향후 LLM 훈련의 기초가 될 수 있는 결과를 제시함
