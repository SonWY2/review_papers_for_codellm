release-date: 2024-03
arxiv: https://arxiv.org/abs/2402.08073v2
gthub: x

Google Deepmind


[개요]
- LLM은 자연어에서 코드 생성에 강점을 보이나, 실제 프로그래밍에서는 추가적인 입출력(I/O) 명세가 필요함
- 그러나 LLM은 자연어 프롬프트와 I/O 명세를 모두 따르는 데 어려움이 있음
- 이를 해결하기 위해 GIFT4CODE라는 새로운 접근 방식 제안:
  - LLM이 생성한 합성 데이터와 실행 기반 피드백 활용
  - 프로그램 I/O 명세 형태의 피드백을 LLM에 제공하여 instruction fine-tuning 수행
- ARCADE와 DS-1000(22.62 → 24.56) 데이터 과학 벤치마크에서 평가:
  - 실행 가능하고 사용자 명세와 일치하는 코드 생성 능력이 크게 향상됨
  - 복잡한 데이터 과학 작업에 대한 코드 생성 품질이 상당히 개선됨
    

[GIFT4CODE]
I/O 명세가 포함된 개발자 의도를 따르도록 코드 LLM을 fine-tuning하는 접근 방식

1. 합성 의도 및 코드 솔루션 생성
- 프로그래밍 맥락 초기화: 실제 코드 저장소에서 프로그래밍 맥락 추출 (예: Github의 데이터 과학 노트북에서 사용된 표 형식 데이터셋)
- 초기 NL 의도 생성: PALM2 LLM을 이용해 주어진 맥락에 대한 자연어 의도 시퀀스 생성
- 코드 솔루션 예측: 코드 LLM을 이용해 각 의도에 대한 여러 후보 코드 솔루션 생성
- 합성 데이터 품질 개선: 실행 가능성 등의 프로그램 속성을 활용해 후보 코드 솔루션의 품질 향상

2. 코드 실행 및 I/O 명세 추론  
- 각 문제에 대해 코드 실행 후 실행 결과로부터 I/O 명세 도출
- 입출력 변수 추적 및 수집
- 3가지 유형의 I/O 명세 조사:
  1) TypeDesc: 변수 타입명
  2) I/O Examples: 구체적인 I/O 예시
  3) I/O Summary: LLM이 생성한 I/O 예시의 자연어 요약

3. I/O 명세를 따르도록 코드 LLM fine-tuning
- I/O 명세가 추가된 의도와 해당 코드 솔루션으로 구성된 합성 훈련 데이터로 fine-tuning
- LLM이 주어진 의도뿐만 아니라 I/O 제약 조건을 준수하는 코드 생성 학습

[GIFT4CODE]
![image](https://github.com/user-attachments/assets/71058f75-b400-4274-b38c-fcf14686c97c)
** 그림 1: 
왼쪽: 사용자가 복잡한 출력(판다 데이터프레임)을 가진 코드를 생성하기 위해 NL 인텐트 및 I/O 사양을 가진 코드 LLM에 메시지를 표시하는 방법의 그림입니다. 바닐라 코드 LLM은 I/O 사양을 이해하지 못합니다. 
오른쪽: 저희가 제안한 인스트럭션 튜닝 접근 방식은 합성 인텐트와 코드 솔루션을 사용하며, 여기서 인텐트는 프로그램 실행에서 파생된 I/O 사양으로 보강됩니다. **

- GIFT4CODE는 I/O 명세가 포함된 개발자 의도를 따르도록 코드 LLM을 fine-tuning하는 접근 방식

1. 합성 의도 및 코드 솔루션 생성
a) 프로그래밍 맥락 초기화:
   - 실제 코드 저장소에서 프로그래밍 맥락 추출 (예: Github의 데이터 과학 노트북에서 사용된 CSV 파일)
   - 각 맥락은 DataFrame 임포트 문과 DataFrame 내용을 설명하는 NL 설명으로 구성
b) 초기 NL 의도 생성:
   - PALM2 LLM을 사용하여 주어진 context(헤더 포함 3줄만 사용)에 대한 자연어 의도 시퀀스 생성
     - 예)
     - Here are a series of contextually dependent data wrangling and exploratory data
      analysis tasks for the dataset:
      Task 1: How many pokemons are there in the Pokedex?
      Task 2: Find the total number of columns that are integers.
      Task 3: Calculate the ratio of mean weight to height for each pokemon
      Task 4: What is the weight of “Snorlax” ?
      Task 5: How many pokemon have the same average base experience as their id?
      Task 6: Find the order for each pokemon that weighs less than 100 pounds
      Task 7: What is the “mean” and “median” of “height” column ?
      Task 8: Show the names of the pokemons with minimum and maximum weight, height
      and base experience.
      Task 9: Show the first 20 and last 10 pokemon with their average base experience.
      Task 10: Create a new column called "size_cat" that has size categories for
      pokemon (child: 1-10, teenager: 11-20, adult: 21+)

   - 각 context에 대해 6개의 의도를 생성하고, 각 의도에 대해 5개의 후보 코드 솔루션 샘플링
   - 다양성을 위해 과제 다양성, 주제 다양성, 지시 다양성을 고려한 프롬프트 생성
c) 코드 솔루션 예측:
   - 코드 LLM을 이용해 각 의도에 대한 여러 후보 코드 솔루션 생성
   - 프롬프트는 프로그래밍 맥락과 의도의 연결, 그리고 추가적인 few-shot 데모로 구성
d) 합성 데이터 품질 개선:
   - 실행 가능성 등의 프로그램 속성을 활용해 후보 코드 솔루션의 품질 향상
   - 약 60%의 코드 샘플이 실행 가능했으며, 실행 가능성과 API 다양성을 기반으로 필터링 적용
   - 최종적으로 약 20K개의 합성 예제를 instruction fine-tuning에 사용

2. 코드 실행 및 I/O 명세 추론
a) 코드 실행:
   - 각 문제에 대해 원래의 프로그래밍 맥락을 실행한 후 생성된 코드 실행
   - 실행 과정에서 입력 및 출력 변수의 집합 {v} 추적 및 수집
b) I/O 명세 도출:
   - 실행 결과로부터 3가지 유형의 I/O 명세 조사:
     1) TypeDesc: 변수 타입명 (예: "Generate a variable with name df and type pandas.DataFrame")
     2) I/O Examples: 구체적인 I/O 예시 (예: DataFrame의 실제 내용 일부 표시)
     3) I/O Summary: LLM이 생성한 I/O 예시의 자연어 요약 (예: "The output dataframe has columns such as Delhi, Mumbai, Chennai")
   - I/O Summary 생성을 위해 PALM2 LLM에 few-shot 프롬프팅 적용
   - 프롬프트에는 프로그래밍 맥락, 의도, 코드 솔루션, I/O 변수 정보 포함
c) 의도 업데이트:
   - 생성된 I/O 명세 z를 원래의 의도 x에 추가하여 업데이트

3. I/O 명세를 따르도록 코드 LLM fine-tuning
a) 훈련 데이터 구성:
   - 각 예제 <c, x, y>는 프로그래밍 맥락 c, I/O 명세가 추가된 의도 x, 해당 코드 솔루션 y로 구성
b) Fine-tuning 목표:
   - LLM이 주어진 의도뿐만 아니라 I/O 제약 조건을 준수하는 코드 생성 학습
   - P_LLM(y | c, x) 최적화
c) 학습 과정:
   - 합성 데이터와 기존 Python 데이터를 혼합한 데이터셋으로 fine-tuning
   - 약 1,500 instruction tuning 단계 수행 (배치 크기 64)
   - 약 1.5 에폭의 합성 데이터셋 소비
d) 테스트 시 노이즈 I/O 명세 시뮬레이션:
   - 테스트 시 I/O Summary 생성 과정에서 구체적인 입출력 변수 상태 {v} 제거
   - 사용자가 노이즈가 있는 I/O 명세를 제공하는 시나리오 시뮬레이션



[실험 설정]
- 데이터셋: 
  1) ARCADE: 상호작용 데이터 과학 노트북의 자연어-코드 생성 벤치마크
  2) DS-1000: Stack Overflow에서 추출한 데이터 과학 문제 벤치마크
- 기본 코드 LLM: 62B 파라미터의 PALM 기반 디코더 전용 모델
- 학습 방법: 다양한 I/O 명세 유형에 대해 베이스라인 및 instruction-tuned 모델 평가
- 평가 지표: pass@k (k개 샘플 중 하나 이상 정답인 문제 비율)

[주요 실험 결과]
ARCADE 데이터셋 결과 (pass@20):
- 베이스라인(Code LLM 62B): 37.47%
- GIFT4CODE (no spec): 46.94% 
- GIFT4CODE (I/O Summary): 55.47%

DS-1000 데이터셋 결과 (pass@1):
- 베이스라인(Code LLM 62B): 22.62%
- GIFT4CODE (no spec): 24.56%
- GIFT4CODE (I/O Summary): 29.34%

주요 발견:
- I/O 명세가 더 상세할수록 pass@k 성능이 일반적으로 향상됨
- 맥락이 부족할 때 추가 명세의 개선 효과가 더 두드러짐
- 실행 가능성 휴리스틱으로 필터링된 합성 데이터로 fine-tuning하면 I/O 명세 없이도 성능 향상
- 자연어 I/O 요약을 사용한 instruction fine-tuning이 두 데이터셋에서 가장 좋은 결과를 보임

[Ablation Study]
1. 훈련 데이터 크기
- SPIN은 데이터 크기가 커질수록 성능이 지속적으로 향상되는 반면, SFT는 오히려 성능이 저하되는 현상 발생

2. 반복 훈련 vs 더 많은 Epoch 훈련
- Epoch 수를 늘리는 것보다 반복적인 학습 방법이 더 효과적
- 단일 반복으로는 달성할 수 있는 성능에 내재적인 한계가 있음을 시사

3. 다른 벤치마크 추가 조사
- MT-bench, Big-bench, OpenBookQA 등 다양한 과제에 대해서도 추가 조사
- SPIN은 대부분의 다양한 작업에서 반복을 진행하며 성능이 향상됨

[결론]
- GIFT4CODE는 실행 기반 명세를 활용해 코드 LLM의 instruction fine-tuning을 수행하는 프레임워크
- 사용자 제공 명세를 따르는 생성 코드의 품질을 향상시켜 ARCADE와 DS-1000 벤치마크에서 정확도를 크게 개선
- 데이터 과학 도메인에 초점을 맞췄지만, GIFT4CODE의 기본 방법론은 일반적이며 정확한 작업 설명을 위해 명세가 필요한 다른 도메인에도 적용 가능
