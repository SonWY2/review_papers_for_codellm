release-date: 2024-06-07
arxiv: https://arxiv.org/abs/2402.10110
github: https://github.com/tianyi-lab/Reflection_Tuning


[개요]
- **IFD 데이터 큐레이션 방법을 개선한 후속 연구**
- Selective Reflecion-Tuning
  - Teacher 모델의 reflection과 introspection(자기성찰)을 활용하여 기존 데이터 품질 향상시키고 Student LLM의 데이터 선택 기능을 활용하여 자동으로 기존 instruction dataset을 개선하는 패러다임
  - IFD와 그 반대개념인 reversed-IFD를 사용하여 학습하려는 모델에서 데이터의 품질을 측정하고 저품질 데이터를 Teacher모델을 통해 개선하는 프로세스
    <전체 파이프라인> ![image](https://github.com/user-attachments/assets/e861713c-cf7b-449c-9e69-3fb9577eb53c)


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

