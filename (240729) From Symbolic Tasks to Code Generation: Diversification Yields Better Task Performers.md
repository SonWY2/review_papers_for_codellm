release-date: 2024-05-31
arxiv: https://arxiv.org/abs/2405.19787
github: x

- 각 작업에 대한 sample이 매우 적음에도 불구하고 충분히 다양한 task 집합이 제공되면 훈련 분포에 대한 일반화 및 견고성 확대

Rule = 규칙 = Instruction
Sample = Examples

[실험 설정]
- 다양한 task와 sample이 주어지는 상황을 시뮬레이션 하기 위해 "문자열 대체" 라는 실험세팅으로 시뮬레이션
- 문자열을 바꾸는 규칙 R 을 제공(예: aa→bac 규칙; caaba => cbacba)하여 여기서 규칙R을 하나의 task로 가정.
  <sample> ![image](https://arxiv.org/html/2405.19787v2/x2.png)
- 규칙 R에 해당하지 않는 입력은 입력 그대로 출력하도록 하여 추가적인 하나의 task로 정의(no-op으로 명명)
  
1) 다양한 Instruction(=규칙)의 효과를 위한 실험 설계
- GPT-2 모델 사용 (256차원, 6층, 4개의 어텐션 헤드). 초기화된 weight부터 학습 시작
- 훈련 세트: S × I 입력 시퀀스 (I: 다른 대체 규칙R 수, S: 각 규칙당 입력 시퀀스 sample 수)
- 테스트: 105개의 예제, 모두 훈련에서 보지 않은 rule 포함
- 실험 결과:
  - a. 지시사항 다양성의 중요성:
    - 300개 미만의 규칙으로 훈련된 모델은 일반화 실패
    - 1000개 이상의 규칙으로 훈련된 모델은 항상 일반화 성공
    - 약 400~500 개의 지시사항에서 급격한 상전이 발생
    < Instruction(=Rule)과 Example(=Sample) 수에 따른 정확도 변화 > ![image](https://arxiv.org/html/2405.19787v2/x3.png)
  - b. No-Op 상황에서의 일반화:
    - 규칙이 적용되지 않는 경우를 포함한 더 복잡한 시나리오
    - No-Op 빈도를 10%에서 50%까지 변경하며 실험
    - 일반화 패턴은 이전 실험과 유사하나, 일반화에 필요한 지시사항 수가 약간 감소 (500에서 400으로)
    < No-op이 포함된 경우의 정확도 변화 > ![image](https://arxiv.org/html/2405.19787v2/x4.png)
  - c. 불균형 분포의 영향:
    - 멱법칙 분포를 사용하여 지시사항의 불균형한 분포 생성
    - 10,000개 이상의 지시사항으로 훈련된 모델은 분포에 관계없이 강건함
    - 1,000개 지시사항 모델의 경우, 형태 매개변수가 0.2를 초과하면 일반화 정확도가 급격히 감소
    < 규칙 수 및 Sample 분포에 따른 정확도 변화 > ![image](https://arxiv.org/html/2405.19787v2/x6.png)
    < 각 Rule의 불균형 분포 설정 > ![image](https://arxiv.org/html/2405.19787v2/x5.png)

2) Semantic Diversification Boosts Task Performance; 명령어의 의미적 다양성
- 지시사항의 '의미적 다양성'을 고려하기 위해 3가지의 패턴 도입
  - 캐릭터 반복: k값 만큼 캐릭터 반복. 예) k=3 aaabbbccc
  - 주기적 반복: k값 만큼 문자열이 반복. 예) k=2 abcabc
  - 미러링: k값 만큼 k값 만큼 문자열이 미러링. 예) k=3 abccbaabc
- 실험 결과:
  - 하나의 패턴으로만 학습된 모델: 새로운 패턴에 적응 X
  - 두 가지 패턴 혼합: 여전히 새로운 패턴 적응 X
  - 세 가지 패턴 혼합: 새로운 패턴에 잘 적응함
  - Rule(=Instruction)이 많을 수록 더 좋은 결과
  - 단, K값이 너무 커서 제한적인 패턴으로 훈련하면 일반화가 어려워짐
  - 단순히 많은 지시사항을 주는 것보다 다양한 유형(의미적으로 다양한) 및 구조로 지시사항을 주어야 함
    < 각 Rule의 불균형 분포 설정 > ![image](https://arxiv.org/html/2405.19787v2/x7.png)

3) Generalization Across Input Semantics; 입력 시퀀스의 다양성
- 입력 sequence의 정해진 규칙이 몇 번 등장하게 구성하는 지에 대한 설정
- 실험 결과:
  - 발생 빈도를 다양하게 가져갈수록 일반화 수행 가능
  - 단, 앞선 실험들과 공통적으로 일반화가 시작되는 지점은 500개 이상의 Instruciton이 확보될 때 였음
    < 등장 빈도o와 Instruction 수에 따른 정확도 > ![image](https://github.com/user-attachments/assets/981f624a-0554-4ba8-abad-0a2d1beaf581)

4) 사전 훈련된 모델
- GPT2가 아닌 사전 학습된 LLaMA2-7B에 LoRA 튜닝을 수행하여 실험 수행
- 단, 사전학습 모델의 강력한 퍼포먼스를 완화하기 위해 단순 문자열 교체에서 교체 규칙에 암호화를 추가하여 더 어렵게 변형
- 실험 결과:
  - 지시사항의 다양성이 모델의 일반화에 도움
  - 적은 수의 지시사항으로 훈련된 모델은 no-op 케이스만 해결하고 "대체 후 암호화" 작업을 제대로 수행하지 못함
  - 규칙이 롱테일 분포를 따르더라도 데이터셋의 규칙 다양화로 완화될 수 있음
  - 균일 분포가 롱테일 분포보다 성능이 뛰어나지만 룰이 향상될 수록 그 차이가 감소함
    < 사전학습 모델의 Instruction 개수에 따른 정확도 변화 > ![image](https://arxiv.org/html/2405.19787v2/x8.png)
    < 데이터(규칙) 분포에 따른 정확도 변화 > ![image](https://arxiv.org/html/2405.19787v2/x9.png)

5) 코드 생성 태스크에 대한 실험
- 벤치마크 데이터:
  - HumanEval, MBPP, EvalPlus 벤치마크 사용하여 측정
- 학습 데이터
  - OSS-Instruct 데이터셋에서 20,000개의 코드 생성 지시사항 샘플링
  - Alpaca 데이터셋에서 일반 도메인 지시사항 추가
  - CoT collection과 같은 비도메인 데이터(abaltion)
- 평가 모델
  - DeepSeek-Coder-6.7B-Base와 CodeQwen-7B-Base 모델 사용
- 결과:
  - 일반 도메인 데이터 포함이 코드 생성 성능을 향상시킴
  - 그러나 일반 도메인 데이터 비율이 증가할수록 코드 생성 도메인 특화성이 감소
  - 최적의 혼합 비율이 존재함 (LLaMA3.1 에서도 비율의 중요성에 대해 언급)
    < OSS와 Alpaca 혼합 비율에 대한 벤치마크 향상 > ![image](https://arxiv.org/html/2405.19787v2/x13.png)
  - Alpaca뿐만 아니라 CoT-Collection 데이터셋을 추가로 혼합 실험 수행
  - 3-way 혼합(OSS-Instruct + Alpaca + CoT-Collection)이 2-way 혼합보다 더 나은 성능을 보임
  - 혼합 비율에 따라 성능 향상 정도가 다름
    < OSS와 Alpaca + CoT 혼합 비율에 대한 벤치마크 향상 > ![image](https://github.com/user-attachments/assets/423bdc0a-9bbb-41a7-b570-fa62adc42c4a)

