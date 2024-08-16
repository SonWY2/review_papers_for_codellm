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
  - 멀티 소스 데이터의 조화를 통해 일반적인 태스크(언어, 추론) 에서도 성능 저하 없이 높은 성능 발휘

---

[Method; 방법론]
** 전체 프로세스의 개요 ** ![image](https://github.com/user-attachments/assets/1cf2bda7-1b4a-41b3-a41f-ac21df208002)
(a): 멀티 데이터 소스 통합 
(b): Instruction evol 진행
(c): Alchemisty Prompt를 통해 충돌 조정
(d): 코드 이해를 위한 데이터 구축

■ Alchemist Prompt
- 일반적으로 파인튜닝을 위해 다양한 오픈소스 데이터를 수집하고 함께 학습
- 이러한 여러 소스 데이터를 통합하는데 어려움 有
  - 질문이나 응답 스타일이 서로 상이
  - 코드 이해능력보다 데이터셋에 존재하는 응답 스타일이 유사한 문제들이 fitting될 수 있음
  - ** 멀티 데이터의 데이터 충돌 예시 ** ![image](https://github.com/user-attachments/assets/b6fe47f9-8001-4876-98e1-4c60e44ebafa)
- 멀티 소스 데이터의 충돌 완화를 위해 Alchemist Prompt 방법을 통해 각 데이터별 prompt 커스터마이징 할 것을 제안.
- Alchemist prompt와 GPT-4를 사용하여 instruction을 커스터마이징.
  - ** 세부 Alchemist Prompt ** ![image](https://github.com/user-attachments/assets/9b53949f-64c8-47e8-9669-912c66ca4934)
  - 예시) 전체 프로세스 개요의 (c) 그림 참고
- ablation 실험 결과 전체 샘플의 5%만 Alchemist prompt로 보정하여 통합하면 멀티 소스 데이터의 융합으로 인한 다양성 및 도메인 격차 사이의 균형을 맞추고 최적 성능 달성 가능

■ Code comprehension task
- 코드 과제에 대한 이해력 향상을 위해 Instruction evolution, Data Filtering, Code Review 3 가지 코드 과제를 추가로 구성함
- Instruction evolution
  - **프롬프트**
  - GPT-3 채택
  - 다양하고 복잡한 Instruction을 생성한다는 원 논문의 취지보다 본 논문에서는 해당 방법을 통합함으로써 모델이 진화 전후의 차이를 식별함으로써 요구 사항, 복잡도 등 코드 관련 개념에 대한 이해가 확대된다고 주장
- Data Filtering
  - **프롬프트**
  - 품질이 낮은 데이터 범주 식별
  - 기준:
    - Short Responses: 지나치게 짧고 코드가 부족한 응답
    - Uncompilable code: 컴파일에 실패한 코드
    - Unclear code: 명확성이 떨어지는 코드
    - Disorganized code: 함수 형식 구성에 관한 지침의 요구 사항 미준수
  - 필터링 된 데이터는 반대 예제를 제공하여 재활용함
- Code Review
  - **프롬프트**
  - 모델이 코드를 검토하고 정확성 및 명확성에 대해 0~10점의 점수를 할당하도록 함
  - 또한 코드 개선에 대한 제안을 제시하고 개선된 코드를 제시하도록 함

■ Data cleaning and decontamination; 데이터 정리 및 오염 제거
- filtering 기준
  - 지나치게 간략하고 코드가 없는 응답
  - 컴파일 할 수 없거나 테스트 케이스에 실패하는 응답
- 오염 제거
  - N-gram similarity, code embedding의 cosine 유사도, code syntax tree의 edit distance를 사용하여 훈련 데이터와 HumanEval, MBPP 문제간의 유사성을 계산한 후 필터링
  - 약 6%의 데이터가 제거되었음

■ 최종 AlchemistCoder 데이터 세트
** 데이터 분포 분석 ** ![image](https://github.com/user-attachments/assets/22fd39f2-4ac8-44d9-a8f6-0453d8a9067c)
** 데이터 LoC 분석 ** ![image](https://github.com/user-attachments/assets/f6f716e8-9b26-45ce-ac86-2f230c7008b3)
- 총 ~200M token 크기
- 구성:
  - 오픈소스 데이터(Evol-Instruct-code-80k-v1, CodeExercise-Python-27k, evol-codealpaca-v1)
  - gpt3.5 turbo로 생성된 EvolCode 데이터(WizardCoder)
  - AlchemistPrompt로 사용자 정의한 데이터
  - 코드 이해 작업 데이터(instruction evolution, data filtering, code review)

[실험 설정 & 결과]
■ 데이터 구성
- AlchemistCoder 데이터 세트

■ 벤치마크
- HumanEval(+); zero-shot 평가
- HumanEval-X
- MBPP(+); 3-shot 평가
- DS-1000 

■ 학습 설정
- Llama2-7b, CodeLlama-Python-7B, DeepSeek-Coder-Base-6.7B
- A100-80GB * 32
- 2 epoch
- learning rate 1e-4
- AdamW
- micro batch size 2
- sequence length 8,192
 
■ 실험 결과
** HumanEval/MBPP 평가 결과 ** ![image](https://github.com/user-attachments/assets/91b7c72a-a548-4793-acfb-b7b11d70b5ce)
** HumanEval-X 평가 결과 ** ![image](https://github.com/user-attachments/assets/c928d063-3711-473e-a144-cfc54666d79e)
** DS-1000 평가 결과 ** ![image](https://github.com/user-attachments/assets/6d9c98b0-3137-45b8-967b-1ddecaa05809)

[Abalation Study]
■ AlchemistPrompts의 최적 사용법에 대한 실험
- AlchemistPrompts로 커스터마이즈된 데이터 비율을 0%~20% 까지 변화 실험
** CL-7B에 대한 AlchemistPrompts 학습데이터 비율간 성능 변화. 왼) 원본 데이터에 변형 추가 오) 원본 데이터를 변형으로 대체 ** ![image](https://github.com/user-attachments/assets/cfb48e6a-572b-47ad-8112-0b4c8f86391c)
- AlchemistPrompts 학습데이터를 추가하거나 대체하는 방식 모두 효과적이었음
- AlchemistPrompts가 1% ~ 5% 까지 비율을 확대할 동안 지속적으로 성능이 향상되었고 5%에 거의 최적의 성능에 도달함

■ Code comprehension task의 기여도 검증
- 본 논문에서 제시한 3가지 세부 태스크에 대한 효과성 검증 실험 수행
** HumanEval/MBPP에서 평가한 코드 이해 작업의 효과 ** ![image](https://github.com/user-attachments/assets/40ac309b-837a-4010-b06f-1f62428c55a4)
- MBPP에서 5.1%의 개선이 두드러지게 나타나 효과적으로 보임


[Analytical Study - 방법론의 효과성 분석]
■ Instruction-Response 간의 조화 확대
- PPL을 통해 조건부 복잡성 불일치(CPD; Conditional Perplexity Discrepancy) 계산
  - PPL(conditional_instruction + response)와 PPL(response)간의 차이
- AlchemistPrompt를 도입하였을 경우 예측에 유요하고 효과적인 문맥 정보를 제공했음
** CPD의 Kernel Density Estimation ** ![image](https://github.com/user-attachments/assets/5db2425f-00e6-4821-b3f9-b40a71373fca)
- 또한 UMAP 시각화 결과 Instruction과 response간의 간격이 짧아진 것을 확인 가능
** UMAP 시각화 ** ![image](https://github.com/user-attachments/assets/5eeac93b-44d7-4c0f-aa9a-32b08f46eb8c)

■ 학습된 모델의 일반화/정규화 능력 검증
- MMLU와 같은 언어 이해를 위한 벤치마크와 BBH 또는 GSM-8K와 같은 다양한 분야의 벤치마크 추가 측정
** 타 벤치마크 측정 결과 ** ![image](https://github.com/user-attachments/assets/9c454f16-1aaf-4ea0-b27d-8f7c79adcd7b)
- 특히, 특정 벤치마크에서 점수가 하락하여 치명적인 망각이 발생한 다른 모델들과 비교하여, 모든 벤치마크에서 높은 향상을 보여줌으로써 해당 방법론의 다면적이고 포괄적인 기능 제공 입증

■ 오류 사례 분석
- HumanEval과 MBPP의 오류 사례 분석
- Alchemist prompt 도입 전후의 모델과 코드 이해 작업 데이터를 비교
** HumanEval 오류 사례 분석 ** ![image](https://github.com/user-attachments/assets/b128d8f2-111a-44a4-ad1e-c725d95812c5)
- AlchemistPrompts와 코드 이해 태스크 데이터가 컴파일 오류(SyntaxError, NameError, ValueError)를 더 잘 처리하도록 도움
- 응답에 코드가 전혀 없던 경우 개선
** MBPP 오류 사례 분석 ** ![image](https://github.com/user-attachments/assets/f6e0030d-531a-4c55-9e5f-262889a90a5f)
- 잘못된 답변(Wrong Answer)의 오류 사례가 눈에 띄게 감소
- Alchemistcoder 시리즈가 code comprehension 데이터를 도입함으로써 강력한 프로그래밍 로직을 얻었음을 방증





