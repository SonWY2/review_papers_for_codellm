release_date: 2024-01-29
arxiv: https://arxiv.org/abs/2401.16380
github: x

[개요]
- LLM의 학습을 위한 데이터를 추가 합성하여 학습Token 수, 복잡성, 학습시간 등을 줄이고 성능을 향상시키는 방법론 제시
- WRAP: web의 문서 및 데이터를 특정 스타일(e.g., wiki style, qa style 등)로 재가공하여 생성한 고품질 합성 데이터셋 학습 방법론
- 사전학습속도 ~3x 개선되고 복잡성이 평균적으로 10%이상 개선되며 제로샷 정확도 2%이상 향상.
- 해당 방법론을 토대로 x10 데이터(3T)로 학습된 동등한 사이즈의 최신 LLM인 TinyLLaMA 보다 뛰어난 QA 모델 학습 검증
![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/c8e42ba8-a858-4716-9d5a-00ae2715e46b)
![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/8ac1f172-97e2-40e8-bf30-c01d2fcf9875)
![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/e91b4d36-9d9b-4419-ac9d-de459b9bd212)
(a) WRAP 레시피
(b) 데이터셋 구성별 특정 성능으로의 LLM 학습 수렴 속도
(c) 학습 데이터셋 구성별 벤치마크 PPL 비교(낮을수록 좋음)


[WARP; Web Rephrase Augmented Pre-training]
- 도메인에 대한 제한된 데이터를 활용하여 효율적으로 학습하기 위한 방법론
- 핵심: 문서의 style Rephrasing
-- 웹에서 수집된 기본 데이터는 언어 모델의 두드러진 사용사례인 QA 형식의 텍스트가 부족하고, 비정형화 되어 있어 학습에 비효율적임.
-- 문서를 4 가지의 다른 스타일로 재표현
--- Easy: 유아가 이해할 수 있는 콘텐츠를 생성하도록 설계된 스타일
--- Medium: Wikipedia의 표현과 유사한 고품질 문장
--- Hard: 간략하고 난해한 언어적 표현
--- Q/A: 대화 질의응답 형식
<Prompt / example>
- rephrasing을 위해 최대 300개의 Token chunking 수행. 300개 이상의 토큰을 재표현하면 종종 정보 손실 발생.
- 실제 데이터 + 합성 데이터: 실제 User의 활용에서 발생하는 오타와 같은 언어적 오류 및 기존 데이터 표현의 다양성 유지를 위해 실제 데이터와 합성 데이터를 1:1 비율로 유지
 
- 장점:
-- 생성 비용 절감: 기존 방법론들이 GPT-3.5를 활용한 것과 대비하여 Rephrasing을 위해 작은 모델 활용 가능
-- 데이터 편향 제거: 정보 유지에 대한 특성덕분에 기존 데이터의 자연스러운 다양성 활용 가능
-- 학습 효율성: 5배 적은 데이터 or 3배 적은 컴퓨팅으로 동등한 모델 훈련 가능

[실험 세팅]
- 다양한 모델 크기와 데이터 양을 사용하여 실험 수행
- 모델 아키텍처:
-- 소형 모델: 128M 파라미터
-- 중형 모델: 350M 파라미터
-- XL 모델: 1.3B 파라미터
- 훈련 프로세스:
-- 각 모델은 300k step 동안 훈련
-- max sequence length: 1024
-- learning rate: 소형 및 중형 모델은 3e-4, XL 모델은 2e-4
-- optimizer: Adam, β1 = 0.9, β2 = 0.999
- 학습 데이터:
-- C4 데이터 세트 및 C4 데이터 세트의 rephrasing 버전


[실험 결과]
■ PPL 평가
- 평가 데이터: C4 데이터와 중복되지 않는 Pile의 21개 하위 도메인에서 수행
- 각 데이터 WARP(C4 + QA-85B Token) / C4(300B Token) 에 대해 학습한 1.3B LLM에 대한 PPL비교 결과 WRAP으로 학습한 모델이 2배의 실제 데이터에서 훈련된 모델보다 성능 우수
  ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/37869aea-197c-4822-9598-2ee117b30d64)

■ Zero-shot 평가
- 다양한 NLP 능력 평가를 위해 13개의 benchmark 사용. 크게 일반적인 지식을 측정하는 기본 벤치마크와 수학, 과학과 같은 전문지식을 측정하는 벤치마크로 나눌 수 있
  <일반지식>
  ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/8a8dbdf6-2da1-47f4-b864-53bb460e8975)
  <전문지식>
  ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/9dbd5963-4f0f-4674-85ff-97042a7b53de)

  * Real Tok: 동일한 문서를 다양하게 rephrasing 가능하기 때문에 실제 원본 소스 말뭉치의 개수를 표
  * Half C4 (C4 데이터의 반)
    Full C4 (C4 전체 데이터)
    RW 160B (RefinedWeb)
    RW 320B (RefinedWeb)
    Pythia-Pile (같은 데이터로 학습된 1.4B 모델)
    TinyLLaMA (slimpajama+starcoder 1T Token으로 학습된 모델)
    Synthetic (합성)
    Synthetic + C4

- 합성+원본(synteic+C4) 데이터로 학습한 모델이 평균 49.4%으로 더 높은 평균 성능 발휘. → NLP 모델의 일반적인 이해 능력도 향상시킬 수 있음.
- 10배의 데이터로 학습한 TinyLLaMA와 유사하거나 더 뛰어남 → 실제 데이터의 필터링이나 추가로 인한 이점은 매우 비효율
- 일반 지식과 달리 전문지식 벤치마크에서는 TinyLLaMA보다 점수가 떨어짐.
-- 이는 합성 데이터가 새로운 지식을 전달할 수는 없다고 여겨짐. 더 큰 데이터 셋에서 얻을 수 있는 추가적인 지식까지 커버는 불가능한 것으로 보임
-- 더 큰 데이터의 장점에도 불구하고 성능 차이는 매우 미미함. 데이터 추가의 개선은 포화 상태.
- 일반적으로 기반 데이터와 합성 데이터를 결합하였을 때 성능 향상이 발생하여 잘 통합되지만, TruthfulQA의 경우 4% 정도 감소하여 실제 데이터가 합성 데이터를 방해하는 경우도 관찰되긴 함.

[추가 분석]
■ RQ1: 실제 C4 데이터를 확보하는 것이 얼마나 중요?
- 특정 벤치마크의 경우 합성 데이터만 존재해도 성능이 올라갈 수 있으나 평균적으로 PPL 나빠짐.
-- 데이터가 특수 문자가 없고 고도로 구조화 되어 너무 깨끗하여 제한이 발생한 것으로 추측
<합성 / 합성+실제 데이터의 PPL 변화>
![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/1103f4af-6c4d-4ab6-8fda-69107e0b0a5f)

■ RQ2: 여러 합성 데이터 세트를 조합하면 성능 향상?
- 여러 스타일을 결합하여 모델의 PPL이 개선되는 것을 확인했음
<여러 스타일의 결합에 따른 PPL 변화>
![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/1998ecbd-cb97-49d6-8302-d04ad6919fd7)
* C4+QA: 기본 C4+QA 스타일 합성 데이터
  C4+Med: 기본 C4+Medium 스타일 합성 데이터
  Combined-1:1: 기본 C4 + (QA+Med; 1:1비율) 합성
  Combined-1:2: 기본 C4 + (QA+Med; 1:2비) 합성

■ RQ3 : 고품질 re-phraser를 갖는 것의 중요성?
- T4, Qwen-1.8B-chat, Mistral-7B-chat, Vicuna-13B-chat-v1.3 4가지 모델을 데이터 합성 모델로 사용하여 비교.
- 모든 모델로 합성한 데이터로 합성하면 PPL이 개선되나 Vicuna-13B와 같이 성능좋음 모델을 생성모델로 사용한 경우 PPL이 더 개선됨 관찰.
<re-phraser 모델 설정에 따른 PPL 변화>
![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/fc78d55c-f76b-4532-b74e-40b5e5041c24)

■ RQ4: 합성 데이터 vs 증강 데이터
- NL-Augmentor(동의어 대체 및 임의 삭제로 데이터 증강하는 라이브러리)를 통해 증강한 데이터와 와 WARP 데이터를 비교한 결과 증강보다 합성 데이터가 성능이 훨씬 더 좋음.
<합성 데이터 및 증강 데이터로 학습한 PPL 변화>
![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/c3428f7e-48ad-4bd6-9187-0a6f7bb7edbb)
* Med+C4: Medium 스타일 합성 데이터 + 기본 C4
  Del+C4: 임의 삭제 증강 + 기본 C4
  Sub-C4: 동의어 대체 + 기본 C4

■ RQ5: 합성 데이터 스타일과 특수 도메인과의 관계
- 도메인과 합성 데이터 스타일이 유사한 경우 성능 향상에 직접적 영향
- 그러나 단일 스타일이 모든 도메인에서 일괄적으로 좋은 성능 발휘 X. 일반화를 위해 LLM을 다양한 스타일의 데이터로 학습 필요

■ RQ6 : rephrase 모델에서의 편향 발생 여부
- 코사인 유사도로 판별결과 합성 데이터 생성시 re-phraser가 새로운 정보를 추가하지 않고 실제 구문과 유사한 의미를 유지함.

[결론]
- 고품질 데이터가 부족한 시나리오 에서 WARP 방식은 기존 데이터의 단순 반복 학습보다 더 많은 가치 제공 가능
- 합성 데이터는 다양한 텍스트 도메인에 대한 일반화와 사전 학습 데이터 세트에서 잘 표현되지 않는 스타일의 텍스트를 생성하는 데 도움이 될 수 있음. 


