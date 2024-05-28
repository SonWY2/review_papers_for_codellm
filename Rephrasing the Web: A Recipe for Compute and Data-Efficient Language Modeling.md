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
- 평가 데이터:
-- C4 데이터와 중복되지 않는 Pile의 21개 하위 도메인에서 수행

[실험 결과]
■ PPL 평가
- 
