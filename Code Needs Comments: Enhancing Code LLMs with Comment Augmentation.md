release_date: 2024-02-20
arxiv: https://arxiv.org/pdf/2402.13013
github: -


[요약]
- LLM의 코드 사전 훈련 과정에서 코드의 주석 밀도가 LLM downstream 성능에 상당한 영향을 끼친다고 주장
- 위의 가설을 위해 코드에 퀄리티 있는 주석을 추가하는 새로운 데이터 증강 파이프라인 제안
- LLaMA2, CodeLLaMA, InternLM2를 활용하여 가설의 효과 검증

[방법론]
1) Instrution Tuning for Comment Generation
- 주석 생성 Student 모델(CodeLLaMA-7B)을 학습하기 위한 퀄리티 있는 주석 데이터 인스턴스 생성 방법
- Instruction Tuning 데이터셋 구축
  - 샘플 선택: StarCoder 데이터셋에서 4000개 이상의 seed 샘플을 선택.
  - GPT-4 모델 사용: 선택한 샘플에 주석을 추가하여 광범위한 Instruction 데이터셋 생성.
  - 데이터 정제: 수동 검토를 통해 4394개의 고품질 Instruction 데이터 인스턴스를 유지.
  - Markdown 형식 변환: Instruction 데이터셋(prompt 및 코드)을 Markdown 형식으로 변환.
  - 추가 데이터셋 통합: 특정 테스크에 오버피팅되는 것을 방지하기 위하여 CodeAlpaca 및 Evol-Instruct-Code-80k 데이터셋 통합.
- Implicit filter; True-Negative 데이터 통합 및 필터링
  - 사전 필터링 이후에도 교육적 가치가 없는 데이터 존재.(단순 클래스 정의, 버전 명세 등)
  - 해당 데이터는 추가 필터링 되거나 True-Negative 데이터의 형태로 Instruction 데이터에 재통합됨.
  - True-Negative 데이터
    - 모델이 주석 생성 과정 전반에서 고품질 코드 데이터를 인식하는 능력 부여를 위한 설계.
    - 입력으로 교육적 가치가 없는 데이터가 주어지면 출력으로 단순히 `<|EOT|>` 토큰을 출력.   
    ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/3f10e6ec-ce88-4e3c-84de-dd94033e814b)
- 정제된 데이터셋을 사용하여 CodeLlama-7b 모델을 미세 조정하여 코드 주석 생성기 생성.

2) NL-Aligned Code Data Generation
- 원시 코드의 보존을 보장하고 주석 생성 과정에서 가속도를 높이기 위해 제한된 생성 방법을 도입
- 각 코드를 라인별로 구분한 뒤, 정규 표현식을 사용하여 코드인지 주석인지를 판별
- 주석을 생성할 코드가 한 라인씩 입력되며, LLM으로 부터 생성된 것이 주석일 때만 현재 내용에 추가하여 다음 생성을 요청하고, 코드인 경우는 원래 코드로 추가하여 다음 생성 요청.
- 필터링:
  - 생성된 주석이 마크다운 형식을 준수하지 않거나,
  - 입력으로 주어진 코드와의 길이 차이가 100%를 초과하는 경우 데이터에서 제외  

[실험]
1) 데이터셋
- 데이터 출처: 실험을 위해 StarCoder Python(SP) 데이터를 사용하였음. 
- 데이터 필터링:
  - Implicit Filter: 모델 출력에 <|EOT|>이 포함된 데이터를 제외함.
  - Explicit Filter: 마크다운 형식을 따르지 않거나 생성된 코드와 원래 코드 길이 차이가 100%를 초과하는 경우 제외함.
- 데이터셋 분류:
  - SP(StarCoder Python)/Remove: 필터를 통과하지 못한 원래 Python 데이터셋.
  - SP(StarCoder Python)/Absent: 모든 주석을 제거한 순수 코드 데이터셋.
  - CP(Comment Pack)/Remove: 필터를 통과하지 못한 데이터를 제거하여 생성된 고품질 데이터셋.
  - CP(Comment Pack)/Restore: 필터를 통과하지 못한 데이터에 원래 코드를 대체하여 생성된 저품질 데이터셋.
  ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/826e6f75-35dc-4608-bcce-1c4422c711f5)
- 학습 하이퍼파리미터:
  -  # TODO


[결과]
![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/4b654594-87ad-4b75-8338-7c3dcad281ae)

- 주석 밀도 효과:
  - LLaMA2-7B 모델을 기준으로 주석 밀도가 0%인 SP/Absent에서 주석 밀도가 증가함에 따라 HumanEval 데이터셋에서 16.46에서 23.17로, MBPP 데이터셋에서 19.00에서 29.20으로 성능이 향상됨.
  - 또한 동일한 수의 토큰으로 학습할 때, 주석 비율이 높은 데이터가 더 나은 결과를 보임.
  (Llama 2 7B에서 추가 사전 훈련 결과) ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/364d3a6e-a84a-4ffd-8d61-a4a54dc4e9a8)
  (CodeLLaMA 7B에 대한 추가 사전 훈련 결과) ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/3973c55c-0c9f-4d66-9a46-ce0b0597da9a)
- 필터링된 데이터의 효과:
  - 명시적/암시적 필터에 의해 제거된 데이터를 원본 데이터로 대체해도 다운스트림 작업 성능이 크게 향상되지 않음.
  - 그러나 오히려 필터링된 데이터를 완전히 제거하면 HumanEval 평가 세트에서 어느 정도 성능이 향상됨.
- 주석 추가의 효과:
  - 동일한 필터링된 데이터에 더 포괄적인 주석을 추가하면 HumanEval에서 성능이 크게 향상됨.
  - MBPP에서는 데이터 구조와 주석 통합 방식이 달라 큰 향상을 이루지 못함.
  - 하지만, Code Llama가 추가 학습을 거친 후 MBPP 데이터셋에서 5.4%의 pass@1 개선을 보임.


