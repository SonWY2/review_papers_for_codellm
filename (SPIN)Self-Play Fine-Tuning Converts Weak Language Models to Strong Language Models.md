release-date: 2024-01-02
arxiv: https://arxiv.org/abs/2401.01335
github: https://github.com/uclaml/SPIN

[개요]
Can we empower a weak LLM to improve itself without acquiring additional human annotated data?
(사람의 주석이 달린 데이터를 추가로 확보하지 않고도 약한 LLM 스스로 개선할 수 있도록 지원할 수 있을까요?)

- 알파고와 같은 자가학습 방법론에서 영감을 얻은 self-play 방식의 finetuning 기법(SPIN) 소개
- 기존의 SFT 데이터를 학습데이터로 그대로 활용하여 LLM이 자신의 인스턴스를 상대로 플레이하도록 한 후 자체 생성 응답과 SFT 데이터의 y와의 응답 구분을 통해 정책을 개선하는 방식
- DPO를 통해 훈련된 모델보다 더 뛰어난 성능

[SPIN]
- 추가적인 사람 또는 AI 피드백에 의존하지 않고 LLMs의 성능을 향상시킬 수 있는 새로운 finetuning 방법
  - 일반적으로 파인튜닝된 $LLMp$
