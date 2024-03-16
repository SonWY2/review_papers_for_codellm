release-date: 2023-12-25
arxiv: https://arxiv.org/pdf/2312.15685.pdf
github: https://github.com/hkust-nlp/deita

[요약]
- Instruction Tuning을 통한 모델 정렬을 위한 자동 데이터 선택 전략인 DEITA를 제안
- Complexity, Quality, Diversity 3가지 측면에서 데이터를 측정하기 위한 연구 진행
  - 각 지표에 대한 다양한 측정 방식을 통제된 실험으로 비교 제시.
- DEITA 방법론을 통해 선별된 6K 데이터의 SFT 훈련으로 SOTA 모델과 우수하거나 동등한 성능 발휘

- 과거 연구에서 LLM의 거의 모든 지식이 사전 훈련 중에 습득되며, Instruction Tuning은 기존 모델 능력을 원하는 방향으로 정렬하는 것이라고 주장(LIMA)
- Instruction Tuning 정렬에 사용하는 데이터가 증가함에 따라 고품질의 데이터를 활용하더라도 성능 증가폭이 둔화되는 현상이 발생.

[WHAT MAKES GOOD DATA FOR ALIGNMENT?]
- 메트릭 기반으로 데이터 하위 집합 선택 → 선택된 데이터로 Instruction 튜닝 → 평가
- 데이터 풀:
  ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/43738a64-5c3a-4dae-846e-e8298fbf7390)
  - $$X_{sota}$$: 최근 LLM정렬에 사용된 최신 학습 데이터셋의 모음으로, 복잡하고 다양한 고품질 데이터
    - WizardLM, ShareGPT, UltraChat 등을 결합한 300K 규모
  - $$X_{base}$$: 품질이 전반적으로 낮고, 중복되는 형태의 데이터.
    - Alpaca, Doly, OAssit, FLAN 2002 등을 결합한 100K 규모
- 훈련 및 평가: LLaMA-1 13B 모델을 통해 학습. MT-Bench 벤치마크로 평가

1) Complexity 복잡성 기반 데이터 선별 전략 π:
- 기존에 존재하는 복잡도 평가 metric과 자체적으로 제시하는 Evol Complexity 방법론을 활용하여 복잡도 평가 metric을 비교 실험 수행
 | **종류**                | **정의**                                                          |
|-----------------------|-----------------------------------------------------------------|
| **랜덤 샘플링**            | seed data에서 무작위 선택                                              |
| **Instruction 길이**    | 문자열 길이가 길수록 복잡하다고 정의                                            |
| **Direct Scroing**    | ChatGPT를 통해 난이도와 복잡도 스코어링                                       |
| **Instruction Node**  | ChatGPT사용하여 Instruction을 semantic tree로 변환, 트리의 노드 수를 복잡도로 측정   |
| **Instag Complexity** | ChatGPT사용하여 샘플에 태그를 지정하고, LLaMA를 Tagger로 학습시킨 후, 태그의 수를 복잡도로 활용 |
| **IFD**               | response loss를 기반으로 측정하는 지표                                     |
| **Evol Complexity**   |  그림1                                                               |
 - Evol Complexity
   - Evol 방식으로 5번의 변형을 통해 Seed data포함 총 6개의 Instruction set 구성
   - 6개의 Instruction을 한번에 ChatGPT에 입력하여 복잡성 점수 c 획득. Instruction 간의 미묘한 차이를 포착하기 위해 6개를 한번에 입력함.
     ```
      Ranking the following questions according to the difficulty and complexity. Score 1-5.
      You can give a score of 6 if the question is too complex for you to answer it. You should
      respond with the format:\n [1] Score: 1\n [2] Score: 2\n
      [1] <Instruction 1>
      [2] <Instruction 2>
      [3] <Instruction 3>
      [4] <Instruction 4>
      [5] <Instruction 5>
     ```
   - 작은 데이터로 라벨링 후 LLaMA-1 7B를 훈련시켜 Scorer로 활용.
   - multi-turn 대화의 경우 각 턴마다 개별적인 점수를 매겨 총합을 최종 점수로 활용
 - 결과:
   - ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/249af12a-6af6-4a63-9d68-f10accd11059)
   - Evol Complexity가 $$X_{sota}$$와 $$X_{base}$$ 둘 모두에서 우수한 성능을 달성
   - PPL은 랜덤 샘플링 기법보다 더 나쁜 결과가 나왔으며, 추가 조사를 통해 PPL이 큰 샘플은 일반적으로 매우 짧은 응답이었음을 확인함.

2) Quality
- Evol Compleixty와 유사하게 품질에 기반한 Evol quality 방법을 제안하고 여러 metric과 비교 실험 수행
 | **종류**                | **정의**                                                          |
|-----------------------|-----------------------------------------------------------------|
| **랜덤 샘플링**            | seed data에서 무작위 선택                                              |
| **Instruction 길이**    | 문자열 길이가 길수록 품질이 높다고 정의                                            |
| **Direct Scroing**    | ChatGPT를 통해 응답(response)의 정확성을 평가                                    |
| **Evol Quality**   |  그림1                                                            |
- DS 프롬프트
```
다이렉트 스코어링 prompt
We would like to request your feedback on the performance of AI assistant in response
to the given question displayed following.
##Tips:Please rate according to the accuracy of the response to the instruction and
the input. Each assistant receives a score on a scale of 0 to 5, where a higher score
indicates higher level of the accuracy. You must just give a score without any other
reasons.
##Question:
<Instruction>
##Response:
<Response>
##Score:
```
- Evol Qualtiy
  - 주어진 데이터 샘플 $$ (I^ {(0)}_k , R^{(0)}_k )$$에 대해 Evol 방식으로 품질을 높이도록 ChatGPT에 요청
  - 유용성 향상, 관련성 강화, 심층성 강화, 창의성 촉진, 추가 세부 정보 제공 등의 방식.
  - TODO: 프롬프트 삽입
  - M=5, 5회 반복 후 Evol된 새로운 Response 0~M까지 획득
  - Evol Complexity와 마찬가지로 6개의 응답을 한번에 ChatGPT에 제시하여 응답의 순위 및 점수를 매겨 품질 점수 q 획득.
```
according to the quality of their response. Score each response from 1 to 5, with 6
reserved for responses that are already very well written and cannot be improved further.
Your evaluation should consider factors such as helpfulness, relevance, accuracy, depth,
creativity, and level of detail of the response.
Use the following format:
[Response 1] Score:
[Response 2] Score:
#Question#: <Instruction>
#Response List#:
[Response 1] <Response 1>
[Response 2] <Response 2>
[Response 3] <Response 3>
[Response 4] <Response 4>
[Response 5] <Response 5>
```
  - LLaMA-1 7B 모델에 fine tuning하여 품질 점수 예측할 수 있도록 수행.
- 결과:
  - ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/42b62d76-e907-441c-844b-3c2c5e564630)
  - Evol Quality의 접근 방식이 일관되게 우수
  - 품질 분산이 높은 X_base가 더 큰 영향을 받음
  - 응답 길이가 최종 alignment 성능에 긍정적인 영향을 끼치지만, 이미 품질이 높은 데이터 세트에서는 효과가 크지 않았음.

3) Diversity
- 다양성이 정렬에 미치는 영향과 다양성과 간결성 유지를 위한 효과적인 전략
 | **종류**                | **정의**                                                          |
|-----------------------|-----------------------------------------------------------------|
| **랜덤 샘플링**            | seed data에서 무작위 선택                                              |
| **Instag Diversity**    | ChatGPT로 Instruction에 대한 태깅 후, 태그 집합에 없는 새로운 태그가 등장할 때마다 해당 데이터셋 추가 |
| **Repr Filter**    | LLaMA-1 13B로 문장을 임베딩하고 cosine similarity에서 특정 threshold 이상의 거리를 가진 데이터만 수집 |
- Repr Filter
  - LLaMA-1 13B 모델을 활용하여 데이터 샘플을 각각 임베딩하고 Cosine similarity로 다양성 측정
  - 데이터 집합 대비 특정 threshold 이상인 데이터 샘플을 데이터 집합에 재귀적으로 추가해가며 필터링
  - 해당 논문에선 유사도 0.9로 설정 (TODO 부록 c.1)
- 결과:
  - ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/12810814-b441-49fc-a4cd-e4da15e07020)
  - Repr filter로 선별한 데이터로 학습한 경우가 X_sota, X_base 모두에서 가장 높은 점수 획득

[DEITA]
- 단순하고 직관적으로 evol compelxity 점수 c에 quality 점수 q를 곱하여 s := c*q 로 복잡성과 품질을 결합한 통합 점수 evol score 'S' 획득.
- 다중턴의 경우 각 턴마다 점수를 계산하고 합산하여 최종 점수 S 산출
- 시드 데이터풀의 데이터 중 evol score가 가장 높은 샘플부터 시작하여 Repre filter 전략에 따라 중복된 샘플을 하나씩 버림
- TODO 알고리즘 및 그림 1

[DEITA 학습 실험 설정]
- 베이스 모델:
  - LLaMA-1 13B, LLaMA-2 13B, Mistral-7B에 학습
  - Vicuna, WizardLM, Mistral-Instruct, Zephyr 등과 비교
- 학습 파라미터: TODO
- 데이터:
  - X_sota에서 각각 6k, 10k로 선별하여 학습.
  - Alphagasus, LIMA, TAGLM과 같은 다른 데이터 선택 접근 방식과 비교
- MT-Bench, AlpacaEval, ARC, hellaSwag, MMLU, TruthfulQA, 사람 평가로 evaluation.

[결과]
![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/6127f8e6-53a6-4da1-8764-4b1ca4f532e3)
![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/d66f9b84-d428-41b9-9d50-0ff79c1cc095)
- DEITA 모델이 다른 SFT 모델보다 우수한 성능 발휘
- AlpacaEval에서 DEITA의 이득이 MT-Bench 이득과 명확하게 일치하지 않는다는 점 발견.
  - 하위 작업에 대한 세부 수행능력 평가 결과 DEITA mistral 모델은 코딩, 수학, reasoning과 같은 고급 능력에 대한 성능이 향상되어 MT-Bench의 점수가 향상되었으나, AlpacaEval에서 두드러지지 않아 이러한 현상이 나타났다 함.
- 또한, 데이터의 선별과정이 어느정도의 선호도 반영과정과 유사하기 때문에 DPO를 추가로 적용하면 더욱 효과가 좋음 확인.
![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/b412872a-b2c8-4142-b9d0-2d97ce175520)
- 데이터 확장 효과를 조사한 결과 DEITA는 3k의 데이터로 300k의 데이터를 학습한 것과 동일한 성능을 제공했음
- 단, 데이터 선택 방식에 따라 선택량이 증가하면 특정 시점에서 결국 성능이 감소하는 현상이 발생했음.
- 이는 정렬에 적합한 데이터의 비율은 생각보다 제한적이라는 것을 시사.
- 더 많은 컴퓨팅 파워를 사용하더라도 정렬 성능이 반드시 향상되지 않는다는 것을 확인함.





