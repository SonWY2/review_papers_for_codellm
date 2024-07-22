release-date: 2024-05-27
arxiv: https://arxiv.org/abs/2405.17057
github: https://github.com/SenseLLM/ReflectionCoder

[요약]
![image](https://github.com/user-attachments/assets/28b807ce-3d2b-4d61-9426-4b3e202daba8)

- ReflectionCoder라는 새로운 접근 방식을 제시함
  - 컴파일러 피드백을 통합하여 구성된 반영 시퀀스를 활용하여 일회성 코드 생성 성능을 향상시키는 방법
  - Reflection Sequence를 효과적으로 활용하기 위해 반영 자기 증류(reflection self-distillation)와 동적 마스크 증류(dynamically masked distillation)를 제안.
- ReflectionCoder 학습을 위한 데이터 구축 및 공개(Apache-2.0 라이센스)
  - 반영 시퀀스와 코드 생성으로 구성된 두 라운드의 대화 데이터셋
- ReflectionCoder-DeepSeek-Coder-33B 모델로 SOTA 수준의 점수 달성
  - HumanEval (+): 82.9 (76.8)
  - MBPP (+): 84.1 (72.0)
---

[Methodlogy]
■ Reflection Sequence 데이터셋 구축
   - GPT-4 Code Interpreter를 활용하여 2단계 프롬프트 방식으로 데이터셋 구축
   - 1단계에서 생성한 sequence의 interpreter 검증하고 안내하여 더 나은 생성 결과 수집
   - 첫 번째 단계: Reflection Sequence 생성
     - 주어진 문제에 대해 코드 생성, 테스트 케이스 작성, 실행
     - 오류 발생 시 원인 분석 및 수정 반복
     - Prompt:
```
Here is a programming problem for you to tackle:
(1) Write a Python function that solves the specified problem with craft test cases
using assert statements and execute it. Pay special attention to edge cases to thoroughly
validate your solution’s correctness.
(2) If your code fails any of the tests, carefully examine the root cause of the fail-
ure. Make the necessary corrections to your code and then retest to confirm the fixes.
Note: At this phase, your primary goal is to ensure the reliability of your code.
There’s no need to delve into in-depth problem analysis or strive for code optimization.

# Programming Problem
{prompt}
```
   - 두 번째 단계: 최종 코드 생성
     - 앞선 Reflection Sequence(코드 interpreter의 결과)를 바탕으로 반복적 수행을 통해 전체 코드 생성
     - 이전 반영 과정에 대한 언급 없이 독립적인 일회성 코드 생성 시뮬레이션
     - Prompt:
```
Then, your task is to create a precise solution for the given programming problem.

Your answer should be complete and standalone, avoiding references to external re-
sources or past exercises, and omit phrases such as "correct version".

There is no requirement to execute the code or provide any test/usage example.
Just present the code for the given problem between "```python" and "```".
```
   - 결과적으로 [Reflection Instruction, Reflection Sequence, Re-answer Instruction, Final code] 형태의 데이터 구성

2. 반영 시퀀스 활용 학습 방법
   a) 반영 자기 증류 (Reflection Self-Distillation)
      - 교사 샘플: [Reflection Instruction, Reflection Sequence, Instruction, Final Code]
      - 학생 샘플: [Instruction, Final Code]
      - 교사 샘플의 최종 코드는 반영 시퀀스를 기반으로 낮은 PPL로 생성 가능
      - 증류 손실 함수:
        ![image](https://github.com/user-attachments/assets/0e2319d5-61e0-4e13-93af-d216bffa98c2)
        (t_c: 최종 코드 토큰, t_ri: Reflection Instruction 토큰, t_rs: Reflection Sequenc 토큰, t_i: Instruction 토큰)

   b) 동적 마스크 증류 (Dynamically Masked Distillation)
      - 지난 연구에 따르면 교사-학생 모델의 차이가 너무 크면 증류에 부정적 영향이 발생할 수 있음
      - 교사-학생 샘플 간 맥락 차이로 인한 부정적 영향 완화하기 위해 동적 마스크 증류 도입.
      - 학습 과정에서 반영 시퀀스의 마스크 비율을 점진적으로 증가시켜 난이도 조절
      - 증류 손실 함수:
        ![image](https://github.com/user-attachments/assets/0d4fd1ec-997b-40f9-a826-f3fee37a93ac)
        (t_prs: 부분적으로 마스킹된 반영 시퀀스 토큰)
      - 세 가지 동적 마스킹 전략 제안:
        1) 랜덤 마스크: 마스크 비율에 따라 무작위로 블록 선택
        2) 순차 마스크: 왼쪽부터 순차적으로 마스크 범위 확장
        3) 블록 마스크: 마스크 비율에 따라 실행 블록, 생성 블록, 분석 블록 순으로 마스킹
        ![image](https://github.com/user-attachments/assets/4fc1caaa-bc84-4f1c-974f-bac3fe7c1961)

[실험 설정]
  - DeepSpeed와 Flash Attention 2를 활용하여 효율적인 학습 수행
  - 16 NVIDIA A800 80GB GPU 사용
  - 7B 모델은 3.5시간, 34B 모델은 25시간 소요
  - 데이터:
    - CodeFeedback-filtered-Instruction에서 10k를 무작위 선택하여 GPT-4 Code interpreter를 활용해 Reflection sequence 데이터 구축
    - 10K reflection sequence데이터 + 156k code instruction tuning 데이터를 활용하여 DeepSeek-Coder 33B 튜닝.
    - 앞서 학습한 DS 33B 모델을 활용하여 12k의 추가적인 Reflection sequence 데이터 생성.
    - 최종적으로 22K의 Reflection sequence data 2배 업샘플링 + 156k code관련 instruction tuning 데이
  - 코드 지시 튜닝 데이터에 대해서는 다음 토큰 예측만 수행
  - 반영 시퀀스 데이터에 대해서는 제안된 방법으로 손실 계산
  - 블록 마스크 전략만 사용 (실행 블록, 분석 블록, 코드 블록 순서로 마스킹)

[실험 결과]
■ 기준 모델과의 비교
![image](https://github.com/user-attachments/assets/0a1789bf-1c2c-4e06-8591-461a8a678afb)

   - HumanEval (+)와 MBPP (+) 벤치마크에서 평가
   - 오픈소스 모델 중 모든 기본 모델에서 최고 성능 달성
   - ReflectionCoder-CodeLlama-7B가 WizardCoder-CodeLlama-34B 능가
   - OpenCodeInterpreter와 비교 시 다양한 기본 모델에서 우수한 성능
   - 닫힌 소스 모델과 비교 시 Gemini Pro, Mistral Large, Claude-3-sonnet 능가
   - GPT-3.5-Turbo 및 Claude-3-opus와 동등한 성능 달성
   - Multiple-E(Java, JavaScript, C++, PHP, Swift, Rust 6개 언어) 평가결과 이전 연구중 최고 성능 발휘
     ![image](https://github.com/user-attachments/assets/88de8ffc-4c5c-472a-bd0a-ba07377bd523)
   - DS-1000의 경우 reflexion data의 여부가 개선에 큰 영향 X. 데이터 리소스에 부족이라 추측. 단, reflexion data의 여부와 상관없이 성능제한은 발생하지 않음
     ![image](https://github.com/user-attachments/assets/497f3246-4e46-49d3-b235-6c728a8e5dc5)



[상세 분석]
■ 제거 연구 (Ablation Study)
  - 각 구성 요소의 기여도 확인
  - 성능 순위: ReflectionCoder > 동적 마스크 없음 > 증류 없음 > 반영 데이터 없음
    ![image](https://github.com/user-attachments/assets/0d56b944-5a10-4b37-b724-b646489cac9d)
  - GPT-4 데이터와 DeepSeek-Coder 데이터 모두 성능 향상에 기여
  - 세 가지 마스킹 전략 모두 효과적이나 완전히 호환되지는 않음

■ 블록 마스크 순서 효과
  - 실행 블록을 먼저 마스킹하는 순서가 가장 효과적
  - 코드 블록을 마지막에 마스킹하는 순서도 우수한 성능
    ![image](https://github.com/user-attachments/assets/32e793ab-a992-4a83-9c6e-d60bb91f5066)
    E: Execution, A: Analysis, C:code block

■ 업샘플링 요인 효과
  - 대부분의 벤치마크에서 2배 업샘플링이 최적 성능
  - 과도한 업샘플링은 과적합으로 인한 성능 저하 가능성

■ 회전 위치 임베딩 효과
  - 회전 위치 임베딩을 사용하는 모델(예: Code Llama)에서 더 효과적
  - 절대 위치 임베딩 모델(예: StarCoder)에서는 효과 제한적

■ 토큰 수준 동적 마스킹 전략
  - 블록(reflection sequence의 유형으로서의 블록. code, execution 등. transformer block이 아님) 수준 마스킹이 토큰 수준 마스킹보다 우수한 성능
  - 토큰 수준 마스킹은 텍스트나 코드의 완전성을 해칠 수 있음


[결론 및 향후 연구]
- ReflectionCoder: 반영 시퀀스를 활용한 효과적인 일회성 코드 생성 방법 제안
- 반영 자기 증류와 동적 마스크 증류 기법을 통해 반영 시퀀스의 지식을 효과적으로 활용
- 다양한 벤치마크에서 최첨단 성능 달성


[한계점]
- GPT-4 Code Interpreter에 의존한 데이터 구축
 - 비용이 많이 들어 방법의 접근성과 확장성 제한
- 회전 위치 임베딩에 대한 의존성
 - 일반화 가능성과 적응성 제한 가능성
