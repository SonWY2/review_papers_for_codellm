release-date: 2024-06-07
arxiv: https://arxiv.org/abs/2402.10110
github: https://github.com/tianyi-lab/Reflection_Tuning


[개요]
- **IFD 데이터 큐레이션 방법을 개선한 후속 연구**
- Selective Reflecion-Tuning
  - Teacher 모델의 reflection과 introspection(자기성찰)을 활용하여 기존 데이터 품질 향상시키고 Student LLM의 데이터 선택 기능을 활용하여 자동으로 기존 instruction dataset을 개선하는 패러다임
  - IFD와 그 반대개념인 reversed-IFD를 사용하여 학습하려는 모델에서 데이터의 품질을 측정하고 저품질 데이터를 Teacher모델을 통해 개선하는 프로세스
    <전체 파이프라인> ![image](https://github.com/user-attachments/assets/e861713c-cf7b-449c-9e69-3fb9577eb53c)
- 단순히 Curation 뿐만 아니라 IFD/r-IFD 기반으로 데이터의 품질을 개선하는 방법 제시
  - → Many To Many 방법과 비교해봐야 할 것 같음. 해당 논문 제목 기억안남.. 재서치 필요

[방법론]
■ 1단계: Selective Reflection on **Instruction**
- Teacher 모델을 통해 원본 데이터셋 (x0, y0)를 개선한 새로운 쌍 (x_ins, y_ins) 을 생성.
  <Instruction Reflection Prompt>
  ```
  System Prompt
  You are a helpful, precise but picky assistant for checking the quality of a given instruction.
  User Prompt
  [Instruction]
  Instruction
  [The Start of Answer]
  Answer
  [The End of Answer]
  We would like you to answer several questions related to the quality of a given instruction.
  1. Why this instruction is not good? First analyze the instruction based on the Complexity of the Topic,
  Level of Detail Required, Knowledge Required, Ambiguity of the Instruction and Logical Reasoning or
  Problem-Solving Involved. Then analyze why this answer is not good for the given instruction based on
  the Helpfulness, Relevance, Accuracy and Level of Details. Finally, analyze why this bad instruction
  leads to a bad answer.
  2. Based on the reason you provided, generate a new and complete instruction that is complex and
  difficult to answer directly. Make sure the new instruction is relevant but independent to the original
  instruction, which can be answered without knowing the original instruction, put the new instruction in
  the format of [New Instruction] your instruction [End]
  3. Answer the newly generated instruction as detailed as possible, in the format of [New Answer] your
  answer [End]
  ```

- 새롭게 생성된 (x_ins, y_ins) 쌍에 대하여 IFD 기준으로 기존 쌍 데이터와 점수를 비교하여 개선 여부를 판단.
  - Difficulty 기준으로 데이터를 선별하는 과정
- 이렇게 개선이 확인된 instruction에 대해서 다시 페어한 데이터 셋 → (x1, y1)
  
■ 2단계: Selective Reflection on **Response**
- 1단계에서 reflection한 (x1, y1)은 response가 개선되지 않아 불와전
- 위와 유사하지만 대상을 response로 하여 아래 프롬프트를 통해 response를 개선 (x1, y1) → (x1, y_res)
  <Response Reflection Prompt>
  ```
  System Prompt
  You are a helpful, precise but picky assistant for checking the quality of the answer to a given instruction.
  User Prompt
  [Instruction]
  Instruction
  [The Start of Answer]
  Answer
  [The End of Answer]
  We would like you to answer several questions related to the quality of the answer to the given
  instruction.
  1. Why this answer is not good for the given instruction? Analyze based on the Helpfulness, Relevance,
  Accuracy, and Level of Details.
  2. Based on the reason you provided, generate a better answer, new and complete, as detailed as possible,
  in the format of [Better Answer] your answer [End]
  ```
- 새로 reflection된 response는 reversed-IFD(rIFD)값을 바탕으로 개선여부를 판단하여 최종 (x2, y2) 데이터 페어 완성
  - Feasibility 기준으로 데이터를 선별
- rIFD 값은 response에 대해 instruction을 평가하라는 지침을 통해 역으로 유추된 instruction의 타당성을 수치화 한 값으로 볼 수 있음
  - "Below is the response to an instruction, please guess the corresponding instruction for the given response. {original response} Generate the instruction for the above response."
  - 해당 값이 작을 수록 해당 지시를 생성하기 쉽다는 것을 의미
    <r-IFD 수> ![image](https://github.com/user-attachments/assets/ddff962a-f4e1-412a-bb1c-4932d6d0b3b6)

[실험 설정]
- 데이터 셋: Alpaca(52k)
- 평가 지표:
  - Pair-wise comparison
  - Alpaca Eval
  - Open LLM Leaderboard
  - MT-bench
  - Human Study
- 모델:
  - IFD/rIFD로 reflection 되어 생성된 데이터 셋으로 학습된 모델(=sRecycled Models)
 
[실험 결과]
- Pair-wise comparison
  <sRecycled WizardLM 7B vs Other Models > ![image](https://github.com/user-attachments/assets/5f565c08-498d-451a-b15b-812586b78e5d)
  - 해당 방법론을 통해 학습한 모델이 크기나 추가적인 RLHF 사용 여부와 관계없이 대부분의 모델 능가
- Alpaca Eval & Open LLM Leaderboard
  - 두드러지는 성과. 경쟁 모델들 대비 소량의 데이터를 사용하여 근접의 성능 발
  < Alpaca Eval Leaderboard > ![image](https://github.com/user-attachments/assets/74039af3-4fe9-4435-9015-257cb4bee50c)
  <Open LLM Leaderboard> ![image](https://github.com/user-attachments/assets/b1c497ff-5d23-48a0-a4ab-3df42d0bfb85)
  <학습 데이터 개수에 대한 모델별 Leaderboard 성능 비교 > ![image](https://github.com/user-attachments/assets/124b7883-5104-4bb4-9551-7ead9ab69847)
  - 별표: ours / 점: 지시튜닝 모델 / 삼각형: RLHF 모델.
  - 파란색 7B, 빨간색 13B, 보라색 13B↑
  <학습 데이터 개수 변화에 대한 벤치마크 점수 변화> ![image](https://github.com/user-attachments/assets/ce039432-6f9f-42c8-b680-fd0b7f7f6748)
  - 상위 k% 데이터 선택에 따른 성능 비교
  - 벤치마크 점수는 데이터 수에 띠라 일정 부분 하락이 발생했으나 LLM-as-Judge 방식에서는 지속적으로 향상되는 양상 발생
  - 특히 2%(1k) 선택 데이터로 학습된 모델을 같은 데이터를 학습한 LIMA 방법론의 모델보다 더 뛰어난 성과 발
    
- 인간 평가
  - sRecycled WizardLM 7B vs WizardLM 7B를 인간 평가자들이 수동으로 평가.
  - 57번 승리 / 23번 무승부 / 28번 패배; 총 108번. 효과적임
 
[Ablation Study]
< sRecycled WizardLM vs >  ![image](https://github.com/user-attachments/assets/dc35eb85-0cd5-4c9f-a3a3-bb19bb919790)

1) Reflection에 대한 ablation
- Reflect on Ins. / Reflect on Res. 각 항목에 대해서만 reflection 된 모델
- Reflect on Ins. + Res. 모든 항목에 대해 reflection 되었지마 선별과정 x
- 원본 WizardLM과 비교했을 때 성능이 크게 향상되었음 확인
    
2) Selection에 대한 ablation
- Select by Randomness(무작위 선택) curation은 모델 성능을 저하시킬 수 있음
- Select by Coherence는 임베딩의 코사인 유사도에 의해 계산된 응집력에 기반하여 선택된 데이터. Instruction과 Response가 가까울 수록 선택될 가능성이 높음
- Select by PPL도 IFD와 유사한 접근방법으로 효과적이었으나 이 논문의 방법론 보다는 아쉬웠음
- IFD, r-IFD의 선택도 효과적이 었으나 reflection이 포함되는 것이 더 효과적

