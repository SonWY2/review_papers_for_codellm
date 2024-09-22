release-date: 2024-04-03
arxiv: https://arxiv.org/abs/2404.02823
github: https://github.com/ConiferLM/Conifer

[개요]
- 대형 언어 모델(LLM)의 복잡한 제약조건을 포함한 지시사항 따르기 능력 향상에 초점을 맞춘 연구
- 기존 LLM들이 복잡한 제약조건이 있는 지시사항을 따르는 데 어려움을 겪는 문제 해결 시도
- Conifer(Complex Constrained Instruction Following)라는 새로운 instruction tuning 데이터셋 소개
- GPT-4를 활용한 데이터 생성 및 큐레이션 방법론 제시
- 단계적 학습 방식(progressive learning scheme) 제안
- 7B 규모의 모델로 최신 오픈소스 모델들을 능가하는 성능 달성

[Conifer 데이터셋 구축 방법] (Figure 2 참조)
![image](https://github.com/user-attachments/assets/318c9043-c991-440a-94c6-f5878c22f144)
(Conifer 데이터셋 구축 workflow)

1. 지시사항 수집
a) 쿼리 재구성(Query Reframing)
- ShareGPT에서 6,000개의 사용자 쿼리를 seed로 사용
- GPT-4를 사용해 각 쿼리를 3가지 이상의 다른 형태로 재구성
- 핵심 내용은 유지하면서 다양한 관점 제공

b) 제약조건 생성(Constraint Generation)
- 재구성된 지시사항의 주제와 객체를 식별
- 잠재적 제약조건 목록 생성
- 제약조건을 broader categories와 specific items로 분류
  - 예: 'Cultural Background'(카테고리), 'Chinese', 'Continental'(아이템)

c) 재조합(Recombination)
- 생성된 제약조건 목록에서 선택하여 지시사항에 통합
- 난이도에 따라 1-2개의 카테고리와 2-3개의 아이템 통합
- 최대 5단계의 난이도로 지시사항 생성
- 각 난이도는 이전 난이도보다 더 복잡한 도전 제시

d) 형식/수치 제약조건 추가
- 1,000개의 지시사항에 추가 형식/수치 제약조건 부여
- 32개의 고유한 형식/수치 제약조건 생성 및 적용
- GPT-4를 활용해 3개의 seed 제약조건으로부터 다양한 제약조건 합성

e) 2단계 필터링
- 1차 필터: 쿼리 재구성 후 적용, 맥락이 부족한 지시사항 제거
- 2차 필터: 재조합 후 적용, 제약조건 간 충돌 해결
- GPT-4를 활용하여 품질 관리 수행

2. 단계적 학습 방식(Progressive Learning Scheme)
a) Easy-to-Hard 진행: 
- 다양한 난이도의 지시사항을 멀티턴 대화 형식으로 구성
- 쉬운 과제부터 시작해 점진적으로 어려운 과제로 진행
- 각 seed 지시사항에 대해 최대 5단계의 난이도 생성
- curriculum learning에서 영감을 받은 방식

b) 프로세스 피드백을 통한 학습:
- 내부 피드백: GPT-4가 형식/수치 제약조건 준수 방법을 명시적으로 설명
- 외부 피드백: GPT-4가 모델의 응답을 평가하고 판단 제공
- {어려운 지시사항, 모델 응답, GPT-4 판단, GPT-4 응답} 형식의 대화 구성
- process supervision 연구에서 영감을 받은 방식

[Conifer 데이터셋 통계] (Table 1 참조)
- 총 13,606개의 대화 샘플
- 대화당 평균 3.02 턴으로 구성
- Easy-to-Hard 부분: 10,302 샘플
  • 난이도 1: 9,167 지시사항
  • 난이도 2: 8,146 지시사항
  • 난이도 3: 7,157 지시사항
  • 난이도 4: 6,144 지시사항
  • 난이도 5: 5,017 지시사항
- 프로세스 피드백 부분: 3,304 샘플
  • 내부 피드백: 977 샘플
  • 외부 피드백: 2,327 샘플

[실험 설정]
- 베이스 모델: 
  • Mistral-7B
  • LLaMA-2-13B
- 데이터셋: 13k Conifer + 53k ShareGPT (총 66k)
- 학습 방법: 
  • Supervised Fine-Tuning (SFT)
  • Direct Preference Optimization (DPO)
    - UltraFeedback 데이터셋 사용
- 평가 벤치마크:
  • 복잡한 지시사항 따르기: IFEval, FollowBench, InFoBench
  • 일반적인 인간 선호도 일치: AlpacaEval 2.0, MT-Bench
- 비교 모델: 
  • GPT-4, GPT-3.5 Turbo
  • Qwen-72B-Chat, LLaMA-2-70B-Chat
  • Vicuna-13B-v1.5, Deita-7B, Zephyr-7B-beta
  • WizardLM (Evol-Instruct), Muffin

[주요 실험 결과] (Table 2, Table 3 참조)
1. 복잡한 지시사항 따르기 벤치마크 (Table 2)
- IFEval: 
  • Conifer-7B-DPO 모델이 52.3% 달성, 오픈소스 모델 중 최고 성능
- FollowBench:
  • Conifer-7B-DPO가 평균 50.0% 달성
  • Level 4-5 난이도에서 각각 47.1%, 41.0% 달성, 70B 모델들과 유사한 성능
  • Qwen-72B-Chat(55.3%)에 이어 두 번째로 높은 성능
- InFoBench:
  • Conifer-7B-DPO가 82.3 점수 달성, 7B 모델 중 최고 성능
  • LLaMA-2-70B-Chat(84.4)와 근소한 차이

2. 일반적인 인간 선호도 일치 벤치마크 (Table 3)
- AlpacaEval 2.0:
  • Conifer-7B-DPO가 17.1% LC Win Rate 달성
  • Zephyr-7B-beta(13.2%), Deita-7B-v1.0(16.1%) 등 오픈소스 7B 모델들 상회
- MT-Bench:
  • Conifer-7B-DPO가 7.25 점수 기록
  • Deita-7B-v1.0(7.55)에 근접한 성능

[Ablation Study 주요 결과] (Table 4 참조)
- Easy-to-Hard 진행 방식의 효과성 입증:
  • 랜덤 셔플이나 Hard-to-Easy 순서로 변경 시 성능 저하
  • 단일 난이도만 사용 시 특정 난이도 구간에서 성능 저하
- 프로세스 피드백의 중요성:
  • 내부 피드백: IFEval 성능 향상에 효과적
  • 외부 피드백: FollowBench의 고난도 문제 해결에 도움
  • 내부+외부 피드백 조합이 가장 우수한 성능
- 형식 및 수치 제약조건의 중요성:
  • 해당 제약조건 포함 시 IFEval 성능 향상

[데이터 오염 검증] (Figure 3, Table 5 참조)
- 코사인 유사도 분석 (Figure 3): 
  • Conifer와 테스트 셋 간 낮은 유사도 확인
  • ShareGPT 대비 더 낮은 유사도 보임
- GPT-4를 활용한 유사 샘플 탐지 (Table 5): 
  • Conifer: IFEval 0.55%, FollowBench 0.53%, InFoBench 2.20%
  • ShareGPT: IFEval 5.36%, FollowBench 6.25%, InFoBench 7.20%
  • Conifer가 ShareGPT 대비 훨씬 낮은 유사 샘플 비율 보임

[데이터셋 분석] (Figure 4, Figure 5 참조)
- 턴 수, 지시사항 길이, 응답 길이 분포 (Figure 4)
- Deita complexity/quality scorer를 이용한 복잡도 및 품질 점수 분포 (Figure 5)
  • Conifer가 ShareGPT 대비 더 높은 복잡도와 품질 점수 분포를 보임

[결론]
- Conifer 데이터셋과 학습 방식을 통해 LLM의 복잡한 제약조건 포함 지시사항 따르기 능력 크게 향상
- 7B 규모 모델로 70B 규모 모델들과 경쟁력 있는 성능 달성
- 향후 더 큰 모델 및 다양한 도메인에 적용 가능성 제시
- 오픈소스 커뮤니티에 데이터셋 및 코드 공개로 후속 연구 촉진 기대
