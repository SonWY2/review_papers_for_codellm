arxiv: https://arxiv.org/abs/2405.19265v1
release-date: 2024-05-29
github: https://github.com/InternLM/AlchemistCoder

[개요]
** HumanEval/MBPP 비교 모델 성능 차트 ** ![image](https://github.com/user-attachments/assets/5a427d06-3ce2-4f31-a6ea-5350d83464ac)

- 멀티 소스 데이터의 다양한 스타일과 품질의 충돌을 완화하고 통합하는 방법론 제안
  - alchemist prompt: 서로 다른 데이터 소스간의 조화↑ 및 Instruction-Response간 조화↑
- 이 외에도 instruction evol, filtering, code review 3가지 코드 이해 능력을 강화할 수 있는 프로세스 제안
- 다중 소스 데이터에 대해 미세 조정된 코드 생성 및 일반화 기능을 확대하는 AlchemistCoder 릴리즈
  - 동급 크기의 모델 대비 높은 성능 발휘

---

[Method; 방법론]
■ Multi-source data construction; 멀티 소스 데이터 구성
- 일반적으로 파인튜닝을 위해 다양한 오픈소스 데이터를 수집하고 함께 학습
- 이러한 여러 소스 데이터를 통합하는데 어려움 有
  - 질문이나 응답 스타일이 서로 상이
  - 코드 이해능력보다 데이터셋에 존재하는 응답 스타일이 유사한 문제들이 fitting될 수 있음
  - ** 멀티 데이터의 데이터 충돌 예시 ** ![image](https://github.com/user-attachments/assets/b6fe47f9-8001-4876-98e1-4c60e44ebafa)

