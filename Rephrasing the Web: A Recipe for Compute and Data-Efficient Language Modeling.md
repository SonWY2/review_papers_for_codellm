release_date: 2024-01-29
arxiv: https://arxiv.org/abs/2401.16380
github: x

[개요]
- LLM의 학습을 위한 데이터를 추가 합성하여 학습Token 수, 복잡성, 학습시간 등을 줄이고 성능을 향상시키는 방법론 제시
- WRAP: web의 문서 및 데이터를 특정 스타일(e.g., wiki style, qa style 등)로 재가공하여 생성한 고품질 합성 데이터셋 학습 방법론
- 사전학습속도 ~3x 개선되고 복잡성이 평균적으로 10%이상 개선되며 제로샷 정확도 2%이상 향상
![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/c8e42ba8-a858-4716-9d5a-00ae2715e46b)
![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/8ac1f172-97e2-40e8-bf30-c01d2fcf9875)
![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/e91b4d36-9d9b-4419-ac9d-de459b9bd212)
(a) WRAP 레시피
(b) 데이터셋 구성별 특정 성능으로의 LLM 학습 수렴 속도
(c) 학습 데이터셋 구성별 벤치마크 PPL 비교(낮을수록 좋음)



[WARP; Web Rephrase Augmented Pre-training]
