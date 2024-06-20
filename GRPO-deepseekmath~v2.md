deepseek-math: https://arxiv.org/html/2402.03300v3
deepseek-v2: https://arxiv.org/html/2405.04434v4

[Group Relative Policy Optimization]

![image](https://github.com/SonWY2/review_papers_for_codellm/assets/36894403/2a8ac347-1c48-466d-b4ee-18865a0ee236)
ppo 수식: ![image](https://github.com/SonWY2/review_papers_for_codellm/assets/36894403/a7e3a936-8b50-4678-ab4a-8515d74897ad)
dpo 수식: ![image](https://github.com/SonWY2/review_papers_for_codellm/assets/36894403/f5c852b5-0ba3-4961-98aa-73261231cb1c)
알고리즘: ![image](https://github.com/SonWY2/review_papers_for_codellm/assets/36894403/55be0491-5ca6-4243-9ba6-1d0807e2017e)


- LLM은 마지막 토큰에서만 보상 점수가 부여되기 때문에 각 토큰에서 정확한 가치 함수를 학습하는 것은 복잡
- 따러서 GRPO에서는 추가적인 가치 함수 근사 필요 없고, 대신 동일한 query에 대해 여러 샘플링된 출력 결과의 평균 보상을 기준으로 사용
- 가치 함수 근사가 필요 없어서 메모리와 계산적 부담 감소
- GRPO는 보상에서 KL 페널티를 추가하는 대신 학습된 정책과 기준 정책 간의 KL 차이를 손실에 직접 추가하여 정규화
  -  KL term은 수식(4)
- outcome supervision :
  - 보상들은 reward model을 통해서 계산되고 이 reward는 그룹 평균을 빼고 표준 편차로 나누는 정규화된 값으로 변환
- process supervision:
  - 주어진 질문 q및 샘플링 된 출력 G에 대해 프로세스 보상 모델이 각 출력 단계의 점수를 매겨 해당 보상 제공.
- 즉 모든 토큰의 이점 A^_i,t를 정규화된 보상으로 설정한 후 grpo 목적 함수를 최대화하여 정책 최적화 수행
