![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/519c2313-2efb-436e-9cdf-6a4b6dd3a4d8)![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/86532e80-47cb-4fbf-a33a-3925f8b63767)Microsoft
release-date: 2023-09-18
arxiv: https://arxiv.org/pdf/2309.09530v1.pdf
github: https://github.com/microsoft/LMOps


[요약]
- LLM에 대한 Continued Pre-training에 대해 전반적인 실험과 조사 수행
- 특히 도메인 특화 데이터에 지속적으로 사전 훈련시킴으로써 얻을 수 있는 도메인 지식과 프롬프트 응답 능력에 미치는 영향 탐구
- continous pre-training에서 발생하는 단점인 `프롬프트 성능의 저하를 방지하기 위하여` 원본 코퍼스를 `독해 문제 텍스트(reading comprehension으로 명명함)`로 변환하는 방법을 제안
- 법률 도메인에서 7B 언어 모델은 BloombergGPT-50B와 같은 특정 도메인에 특화된 훨씬 더 큰 규모의 모델들과 경쟁할 수 있는 성능을 달성하여 검증



[Reading comprehension 방법론]
- 도메인 corpus에 대한 continous pre-training은 모델에 도메인 지식을 부여하지만, 프롬프트에 대한 응답 능력을 저하시키는 문제를 해결하기 위해 제시됨
- 인간 학습 방식에서 영감을 받아 질문에 답하는 능력을 향상시키는 방법을 시도하게 되었음
- 학습과정을 `독해-이해` 2stage로 나눔.
- `독해`는 raw 텍스트를 학습 하는 과정이며, `이해`는 질문-답변 형식을 따르는 task에 대한 훈련임
  - 즉, `독해`는 교과서의 내용, `이해`는 연습문제와 같은 형식이라 볼 수 있음
- `이해`의 질문과 답변은 raw 텍스트로부터 정규식 추출을 통해 생성함
  - Summarization
  - Word-to-text task: 도메인 코퍼스의 특정 단어를 포함하는 문장을 생성하도록 유도
  - Natural Langauge Task: 두 문장을 추출하여 두 문장의 관계를 유추하도록 구성 (entailment, neutral 등). 
  - Commonsense Reasoning: 문장 내의 원인-결과 논리 식별
  - Paraphrase Detection: 두 문장의 의미적으로 동일한지 여부를 판단하도록 요청
  - Text Completion
- `Reading comprehension` 예시
![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/4435d184-9a12-479b-980d-06d8220567f7)


[Continued Pre-training 실험]
- 생물의학/법률/금융 도메인에 대한 데이터로 실험
- 도메인 데이터로의 Continued Pre-training이 모델 성능에 미치는 영향을 평가하기 위하여 prompting, fine-tuning evaluation, domain knowledge probing 진행
- 도메인 데이터 + reading comprehension 의 비율은 1:1~2 수준
![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/180d8528-d8ec-443c-99e1-18ddeb494ec8)
![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/9d470385-6dca-4330-a91c-b1d611a22012)



![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/60a9e235-a53e-4f67-adc5-f4480e67574d)
- Prompting vs Fine-Tuning
  - 도메인 코퍼스에 대한 Continued Pre-training 이후 세 도메인에서 성능 향상이 나타나지만, prompting 성능에서는 뚜렷한 하락이 관찰됨
- Domain Knowledge Probing
  - LAMA와 유사한 방법으로 Domain Knowledge Probing 수행.
  - 각 도메인에서 supervised-fine tuning 학습 데이터를 기반으로 도메인별 knowledge probing 벤치마크 데이터셋을 생성함
  - Continued Pre-training 이후 점수가 향상되어 실제로 도메인별 지식을 습득한다는 것 확인 가능
- 분석 결과 도메인별 prompting 성능의 하락은 결국 prompting 능력 자체의 감소에서 기인한 것이라 볼 수 있음
  -  특정 도메인 내에서의 data corpus의 다양성이 제한되어 있기 때문에 raw 텍스트에서 유도된 입력-출력 패턴이 제한되었을 것.
-  즉, prompting 능력을 향상시키는 것은 습득한 도메인 지식을 효과적으로 활용하는데 매우 중요함

[실험 결과]
![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/bfad0d21-eba1-4d13-8cde-abb08fdc13bc)
- Reading comprehension을 활용한 AdaptLLM이 전반적으로 좋은 성능

![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/043a7206-af53-4959-bd01-78635769a665)
* raw text: 사전 훈련 데이터
* Read. Compre: raw 텍스트롤 Read comprehension 형태의 질문-응답 형태로 변환
* Gen. Ins : 일반 instruction 데이터(wizard, orca 등)
* Read. + Gen. Ins : Read Comprehension에 일반 instruction 데이터 추가
- raw 데이터를 read comprehension 형태를 가공하여 추가하고, 일반 instruction 데이터를 추가하여 학습하면 가장 좋은 결과 

