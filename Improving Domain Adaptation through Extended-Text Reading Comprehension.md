Microsoft
release-date: 2024-01-18
arxiv: https://arxiv.org/html/2401.07284v2
github: https://github.com/microsoft/LMOps

# 요약
- Domain Adaptation을 연구했던 이전 논문인 `Adapting Large Language Models via Reading Comprehension`의 후속 연구 수행
- raw domain 데이터의 클러스터링 + LLM 질문-쌍 전처리 + PEFT 학습방법 연구를 통해 보다 높은성능의 Domain Adaptation 연구 결과 달성
- 이전 연구의 AdaptLLM 비교 5%이상의 성능 개선

# 방법론
1) LLM 질문-쌍 처리
- 이전 연구인 AdaptLLM은 정규표현식 방식을 사용하여 질문-답변 형식으로 말뭉치를 구조화할때 올바른 도메인별 지식을 반영하는 질문을 생성하는데 어려움이 있었음
- 따라서 `질문-답변` 쌍을 형성할 때 기존과 달리 ChatGPT를 활용하여 다음과 같은 템플릿을 사용하여 질문-답변 쌍을 생성
  - {DOCUMENT} {DOMAIN}에 관한 위의 내용을 이해하는 데 도움이 될 몇 가지 질문을 하고 해당 답변을 JSON 목록으로 제공하십시오` (각 JSON에는 두 개의 키, 즉 question과 answer가 포함됩니다)
  - DOCUMENT는 raw text인 context, DOMAIN은 학습하고자 하는 도메인(예: programming)
- domain 학습 데이터는 10억개 이상의 토큰이 포함될 수 있음으로 과다한 비용 발생 위험.
- 이를 제한하기 위해 질문-답변 쌍을 생성하는 10,000개의 데이터로 LLaMA 7B짜리 모델을 ChatGPT 데이터를 통하여 finetuning 함.
  - 어떤 방식으로 fine-tuning했는지는 설명x

2) 클러스터링
- 질문 응답의 맥락을 확장하기 위해 문서 유사성 활용
- 텍스트 임베딩 모델을 활용하여 문서를 임베딩하고 LLM의 최대 컨텍스트 길이에 최대한 맞추도록 클러스터링 수행
![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/925705b9-c6c8-4a1b-90e0-265b4b32d780)

3) PEFT 학습(LoRA)
- 최근 연구에 따르면 일반적으로 PEFT(LoRA) 방법은 domain adaptation에 대해서 full fine-tuning보다 효과가 덜함
- 이 문제는 LoRA가 self-attention layers에 대해서만 한정적인 연산을 수행하여 LLM의 저장된 지식과 관련된 feed-forward layer를 무시한다는 것에 의해 발생함
- 또한  domain adaptation은 훈련 가능한 매개 변수의 양에 높은 민감도를 보임.
- 따라서 전체 미세 조정과 비교 가능하려면 rank값이 256과 같이 훨씬 크게 유지되어야 함.

# 실험
- AdaptLLM과 동일한 데이터로 도메인 학습 수행
  - 생명과학 도메인: The Pile에서 PubMed 초록 수집
  - 금융 도메인: FinGPT에서 금융 뉴스 수집
- 평가 데이터
  - 생명과학 도메인: PubMedQA, MQP McCreery, RCT, USMLE, BioMMLU, MMLU
  - 금융 도메인: ConvFinQA, FPB, NER, Headline, FiQA
- AdaptLLM과 동일한 비율로 일반 지침 데이터셋 혼합(LIMA, WizardLM, Orca)
- PEFT LoRA는 rank=8, int8 quantization 사용

# 결과
![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/26c7484d-ea51-4182-bea7-94df3b28e142)
- AdaptLLM을 능가하여 생명과학 6.8%, 금융 5.6% 성능 개선
- 이전 논문에서 주장한대로 DAPT(Domain Adaptive Pre-Training)이 base LLM에 부정적인 영향을 미치는 경우는 재현되지 않음
- 그러나 어찌되었든 그 논문에서 주장했던 AdaptLLM은 보다 효과적이었음
- 클러스터링의 효과 검증을 위해 ablation 실험을 수행한 결과 클러스터링을 도입하였을 때 모든 도메인에서 효과적이었음 확인
  ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/966bd257-f353-46d0-b1e9-3788eb308916)
- PEFT의 효과에 대해서도 hyper parameter 튜닝을 진행한 결과 훈련 매개변수의 크기가 중요했으며, full fine-tuning과 유사한 성능 발휘 했음
  ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/03a0c277-9d02-4dc4-a45a-a6fc01cbed7c)
