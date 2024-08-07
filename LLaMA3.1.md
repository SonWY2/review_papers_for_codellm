[요약]
- 특정기능 (코딩/추론)을 개선하도록 모델 조정 사후 훈련 단계 적용
- 다국어, 코딩, 추론, tool 지원
- 128k context size
- 이전버전 모델 대비 학습데이터의 양과 질 개선
  - 전처리 / 큐레이션 파이프라인 개선
  - LLaMa2 1.8T > LLaMA3 15.6T
- 모델 계산 최적 시간보다 더 오래 학습
  - 같은 크기의 다른 모델들 대비 성능 향상
- MoE 및 복잡한 강화학습 알고리즘 등은 안정성의 이슈로 적용 안함
  - 대신 SFT, RS, DPO같은 강건하고 안전한 방법 적용

[사전학습]
- 8k context 15.6T 학습 후 128K context로 확대
- ~ 2023.12 데이터까지 학습
- web 큐레이션
  - 도메인 기준 안전하지 않은 정보 제거
  - HTML 파서 자체 개발. 수학, 코드 등의 정보도 최대한 보존하도록 노력
  - markdown이 웹데이터로 학습된 모델에 해롭다는 사실 발견. 모든 markdown marker 제거
  - URL중복된 데이터 제거 + minhash 중복 제거 적용
  - 3000만개 문서마다 6번 이상 등장한 같은 text는 삭제
    - 쿠키 경고 등 무의미한 정보
    - 유의미한 정보도 포함되어 있었으나 궁극적으로 성능은 두드러지게 향상됨
  - 휴리스틱 기반 저품질 데이터 제거
    - duplicated n-gram coverage ratio를 도입하여 로깅 등 저품질 데이터 제거
    - 불건전 단어를 카운팅하여 일정 수준 이상의 성인 컨텐츠 제거
    - 토큰 분포를 확인하여 과도한 수 이상의 token 이상값 존재하는 데이터 제거
  - 모델 기반 품질 분류
    - 경량 방식(fasttext): wikipedia에 참조될만한 텍스트인지 검증
    - 계산집약적 방식(roberata): LLaMA2의 예측결과로 학습하여 품질점수 예측
    - LLaMA2 모델 분류기: 정제웹문서 학습하여 품질요구사항에 충족하는지 검증.
      - Distilroberta를 사용했다고 되어 있는데 llama2를 distill한거인지 명확히 설명X
  - code & reasoning 데이터
    - DeepSeek과 유사하게 코드/수학 웹페이지를 추출하는 도메인별 파이프라인 구축
    - LLaMa2에 의해 주석이 달린 웹 데이터로 훈련된 DistilledRoberta모델 사용.
    - 수학 추론, STEM 영역 추론, 자연어가 삽입된 코드가 포함된 웹페이지로 튜닝됨
    - 토큰 분포가 자연어 데이터와 달라 앞선 파이프라인과 별도의 도메인 전용 파이프라인 구축
  - 다국어
    - 위의 파이프라인과 유사하게 안전하지 않은 콘텐츠 웹사이트를 데이터 필터링
    - 추가적으로 FastText 기반 언어 식별 모델을 활용하여 문서를 176개 언어로 분류
    - 각 언어에 대해 데이터 내에서 문서 수준 및 라인 수준 중복 제거
    - 언어별 휴리스틱 및 모델 기반 필터를 적용하여 저품질 데이터 제거
    - llama2 모델로 품질 순위를 매겨 고품질 컨텐츠 우선순위 확보
    - 사전 학습에 사용되는 다국어 토큰 양을 실험적으로 결정하여 영어 vs 다국어 데이터 구성으로 모델 성능 균형 확보
- 데이터 mix
  - 데이터 소스의 비율은 knowledge classification과 scaling law 에 따라 실험적 구성
  - knowledge classification: 정보 유형 분류. 웹에서 과도하고 표현되는 카테고리 다운샘플링(예술, 엔터테인먼트)
  - scaling law data mix: 데이터 조합에 대해 여러 작은 모델을 학습시키고 이를 활용하여 해당 조합에 대한 큰 모델의 성능을 예측하는 스케일링 법칙 실험 수행. 이를 토대로 데이터 믹스 후보 선택
  - Data mix summary: 일반 지식 50%, 수학/추론이 25%, 코드가 17%, 다국어 토큰이8%
- Annealing Data(최적화)
  - 경험적으로 소량의 고품질 코드 및 수학 데이터에 대한 어닐링은 성능 향상에 효과적
  - 일부 도메인 고품질 데이터를 업샘플링하는 데이터 믹스 어닐링 수행
    - 8B는 GSM8K 및 Math에서 24.0%, 6.4% 향상
    - 단, 405B와 같이 강력한 모델에선 개선 미미
  - 어닐링을 통한 데이터셋의 가치를 판단
    - 소규모 도메인별 데이터셋의 가치 판단 가능하다는 사실 발견
    - 50%만 훈련된 8B 모델의 40B Token에 대해 0까지 선형적으로 어닐링하여 데이터 세트 가치 측정
    - 실험결과 새로운 데이터셋에 대해 30% 가중치 할당하고 나머지 70%의 가중치는 기본 데이터 믹스에 할당.
 

[아키텍쳐]
- 개요:
  - 성능 향상은 데이터 품질과 다양성 개선, 규모에 의한 것으로 아키텍쳐는 많이 수정X
  - 8개의 key-value head를 가진 GQA 적용
  - 동일한 시퀀스 내의 다른 문서내에 자체 attention을 방지하는 attention 마스크 적용.
    - long context에 대해서 중요하게 작용했음
  - 128K 토큰으로 구성된 어휘 사용.
    - Tiktoken에서 28,000개의 비영어권 토큰을 추가. 
    - LLaMA2에 비해 영어 데이터 샘플 압축률을 토큰당 3.17자에서 3.94자로 개선
  - RoPE base freq 하이퍼파라미터를 500,000으로 늘림. 32,768 컨텍스트 길이에 효과적
- scaling raws:
  - 컴퓨팅 예산에 대해 플래그쉽 모델 최적의 크기를 결정하는 법칙 개발
    - 컴퓨팅 예산 학습 토큰 간의 파워 법칙 관계와 로그 확률, 벤치마크 정확도간의 상관관계를 분석하여 모델의 성능 예측
  - downstream 작업에 대한 컴퓨팅 최적 모델의 negative log-likelihood과 FLOP 간의 상관관계 설정
  - 법칙 모델과 더 높은 FLOP으로 훈련된 이전 모델을 모두 활용하여 다운 스트림 작업의 negative log-likelihood 작업 정확도와 연관시킴. (LLaMA2 활용)

[Infrastructure, Scaling, and Efficiency]
  - skip


[Training Recipe]
- 405B 학습을 위해 초기 사전훈련, 장기간 사전 훈련, 어닐링 세 주요 단계로 구성됨.*8, 70B도 유사
- 초기 pre-training
  - 초기에는 낮은 배치 크기 사용하여 훈련 안정성 개선 후 이후 배치 크기 확대
  - 초기는 배치크기 4M token 과 4,096 시퀀스 사용하고 252M 토큰 학습 후 8M, 8192 시퀀스로 확대. 2.87T 학습 후 또 2배로 늘림
  - 비영어권 비율 확대, 수학 데이터 업샘플링, 저품질 데이터 다운샘플링
  - 사전학습 후반 단게에서 최신 웹 데이터 추가
- Long Context Pre-Training
  - 지원되는 context 길이를 점진적으로 늘려 모델이 증가된 컨텍스트 길이에 성공적으로 적응할 때까지 사전 학습 진행
  - 405B의 경우 최초 8K에서 6단계에 걸쳐 128K 까지 확대.
  - 800B 토큰 사용
- Annealing; LLaMA2 부터 사용된 학습률을 점진적으로 줄여 모델의 성능을 최적화하는 기법
  - 최종 40M 토큰 사전 학습 동안 learning rate를 0으로 선형적 어닐링하여 128K 토큰의 컨텍스트 길이를 유지.
  - 이 단게에서 데이터 믹스를 조정하여 매우 높은 품질의 데이터 소스를 업샘플링 수행.
  - 어닐링 중의 모델 체크포인트 평균을 계산하여 최종 모델 생성.

[Post-Training]
![image](https://github.com/user-attachments/assets/5cca9c3e-f038-4ba5-8470-fdf2fe90aee8)

- 여러 차례 사후 학습 및 피드백 정렬 수행(SFT, DPO)
- Modeling
  - Chat dialog format
    - LLaMA3에서 추가된 Tool 지원기능을 위해 다양한 특수 헤더 토큰 및 종료 토큰 프로토콜(템플렛) 설계.
  - 보상 모델링
    - 데이터 확장 후 개선이 감소하는 것을 관찰하기 위해 loss에서 margin 제거 외 LLaMA2와 동일
    - 유사한 응답을 가진 샘플을 필터링 후 모든 선호도 데이터를 보상 모델 학습에 사용
    - (chosen, rejected) 외에도 일부 프롬프트에 대해 (edited)를 추가 생성. 이 쌍에서 chosen 응답은 개선을 위해 추가로 편집됨. (edited > chosen > rejected)
    - 훈련 중에는 response를 무작위로 섞어 프롬프트와 여러 개의 응답을 하나의 행으로 연결(훈련 효율 향상)
  - SFT
    - 앞선 보상 모델을 활용하여 rejection sampling 수행.
    - 프롬프트 토큰은 마스킹하고 합성 데이터 및 거부 샘플링 데이터를 활용하여 사전학습 모델 미세 조정
    - 405B 모델은 8.5K~9K를 걸쳐 1e-5 학습률로 미세 조정 됨. 이러한 설정이 경험적으로 다양한 라운드와 데이터 믹스에서 잘 작동했음.
  - 강화학습(DPO)
    - PPO도 검토했지만 대규모 모델에서는 DPO의 리소스 사용이 더 적고 IFEval과 같은 벤치마크에서 더 나은 성능을 보여 DPO로 채택
    - 1e-5의 learning rate, β는 0.1
    - DPO loss에서 포맷 토큰 masking 적용
      - loss 계산에서 chosen, rejected 모두에서 header 및 eos 토큰을 포함한 특수 토큰들을 마스킹하여 학습 안정화
    - NLL 손실에 따른 정규화
      - 선택된 시퀀스에 0.2 스케일링 계수를 갖는 NLL loss 항 추가. 이는 원하는 형식을 유지하고 응답의 로그 확률 감소 방지하여 DPO 안정화 기여.
  - 모델 평균화
    - 다양한 버전의 데이터 및 하이퍼 파라미터를 활용하여 얻은 모델을 RM, SFT, DPO 단계에서 평균화 수행
  - 반복 round 
    - 위의 방법을 6차례에 걸쳐 적용. 각 주기마다 최신 모델에서 합성 데이터를 새로 샘플링하여 새로운 선호도 annotation과 SFT 데이터 수집

[Post-training Data]
- LLaMA2와 유사한 방식. 서로 다른 데이터 조합과 정렬 레시피로 훈련된 모델의 각 라운드(학습 단계) 후 여러 모델을 배포하여 annotation 작업 수행
- 사용자 프롬프트 마다 두 개의 다른 모델에서 두 개의 응답을 샘플링
- 주석자들이 선호도를 4단계로 평가: 현저히 좋음, 더 좋음, 약간 더 좋음, 미세하게 더 좋음 으로 구분
- 선호도 순위 지정 후 편집 단계를 통합하여 주석자가 선호하는 응답을 더욱 개선할 수 있도록 장려 => 직접 수정 또는 개별적 피드백 추가를 활용해 모델의 새로운 출력 생성
  -  (edited > chosen > rejected )
- 수집된 데이터를 엄격하게 평가하기 위해 품질 분석 및 인적 평가 프로세스를 구현하여 프롬프트를 개선하고 annotator에게 체계적이고 실행 가능한 피드백을 제공.
- 각 라운드 마다 LLaMA3가 개선됨에 따라 모델이 지연되는(취약한) 목표 영역의 프롬프트 복잡도를 높이는 방법으로 학습 개선
- 각 라운드에서 보상 모델링에는 당시 사용 가능한 모든 선호도 데이터를 사용하고, DPO 훈련에는 다양한 기능의 최신 배치만 사용됨.
- DPO와 reward model 학습시에는 chosen 응답이 rejected 대비 현저히 좋거나 더 좋은 샘플만 사용하고 유사한 응답은 제외

- SFT 데이터
  - 크게 3가지의 소스 사용
    - 거부 샘플링된 응답이 포함된 인간 주석 컬렉션의 프롬프트
    - 특정 기능을 타겟팅 하는 합성 데이터
    - 사람이 직접 선별한 소량 데이터
  - 훈련 후 라운드 진행마다 대규모 데이터 수집에 LLaMA3 변형 사용
- Rejection Sampling
  - 주석자들이 입력한 프롬프트에 대해 여러 모델 응답을 생성하고, 가장 적합한 응답을 선택하는 과정
  - 과정
    - 주석된 각 프롬프트에 대해 최신 채팅 모델 정책(일반적으로 이전 후속 학습 반복에서 가장 성능이 좋은 체크포인트 또는 특정 능력에 대한 가장 성능이 좋은 체크포인트)으로부터 K개의 응답 샘플링. K는 보통 10에서 30 사이
    - 샘플링된 K개의 응답 중에서 보상 모델을 사용해 가장 적합한 후보를 선택
    - 후속 학습의 후반 라운드에서는 시스템 프롬프트를 도입하여, 거부 샘플링 응답이 원하는 톤, 스타일 또는 형식에 맞게 조정되도록 유도. 다양한 능력에 대해 서로 다른 요구사항 반영 가능
  - 효율을 위해 PagedAttention 채택.
- 전체 데이터 구성
  - ![image](https://github.com/user-attachments/assets/cb2e32bd-5ece-4795-bd10-87caa952baa8)
 
- 데이터 처리 및 품질관리
  - 문제 데이터 필터링을 위해 휴리스틱 기반 전략 적용. (예: I apologize와 같은 문구를 가진 샘플 비율 조절)
  - Data cleansing
    - 이모지, 이모티콘 과도한 특수문자와 같은 패턴들 가진 데이터 제거. I'm sorry와 같이 과도하게 사용되는 문구 식별 후 비율 조
  - Data pruning
    - 토픽 분류:
      - 8B 모델을 토픽 분류기로 미세조정하여 데이터 분류. 대략적인 주제와 세부 주제를 분류(예: 대분류-수학적 추론, 세부분류-기하학과 삼각법)
    - 품질 스코어링:
      - RM을 사용하여 품질 점수 측정 후 상위 사분위수 데이터를 고품질로 간주.
      - LLaMA3 를 사용하여 도메인별 기준 평가
        -  영어 데이터: 정확성, 지시 사항 준수 여부, 톤/프레젠테이션 등
        -  코딩 데이터: 버그 식별, 사용자 의도 등
        -  두 신호 간 불일치율이 높았지만 이를 결합하면 내부 테스트에서 가장 높은 recall 보임
      - 난이도 스코어링:
        - LLaMA 70B 사용 복잡한 예제 우선을 위해 Instag와 LLaMA기반 점수를 사용하여 데이터 평가
        - 1~3점으로 난이도도 측정시킴
    - 의미 중복 제거
      - RoBERTa를 사용하여 대화 클러스터링 후, 클러스터 내에서 (품질 점수 x 난이도 점수)를 기준으로 정렬. 그 후 코사인 유사도가 threshold보다 낮은 예제만 유지

[capabilities]
1) code
- 우선순위가 높은 프로그래밍 언어에 대한 코드 생성, 문서화, 디버깅 및 검토 기능을 개선하고 평가하는 것을 목표로 했음
  - Python, Java, Javascript, C/C++, Typescript, Rust, PHP, HTML/CSS, SQL, bash/shell
  - 코드 전문가 양성, SFT용 합성 데이터 생성, 시스템 프롬프트 스티어링을 통한 서식 개선, 학습 데이터에서 불량 샘플을 제거하는 품질 필터 생성 등 적용
  - Expert training
    - 코드 전문가 모델 학습: 기본 사전 훈련 뒤 85% 이상 코드 데이터로 구성된 1T 토큰 혼합에 대해 사전 훈련을 계속하여 획득(≒ CodeLLaMA)
    - 마지막 수천 step 에서는 Long context fine tuning을 수행하여 고품질의 repo 수준의 코드 데이터 조합에 대해 16K로 학습
    - 모델링 레시피에 따라 모델을 조정할 때 주로 코드를 대상으로 하는 SFT 및 DPO 데이터 믹스를 사용
    - 이렇게 얻은 모델은 코드 관련 데이터의 rejection sampling에도 활용
  - 합성 데이터 생성(execution feedback)
    - LLaMA3와 Code Expert를 SFT 데이터를 합성하는 데 활용하여 2.7M 이상의 합성 예제를 SFT 학습단계에 사용.
    - 8, 70B 는 더 큰 모델에서 생성한 데이터로 학습할 때 성능이 크게 향상되었지만 405B는 자체 생성 데이터로 훈련하는게 도움되지 않았음.
    - 실행 피드백을 a soure of truth로 함께 활용
    - 프로세스:
      - a. 문제 설명 생성
        - long tail 분포를 포함한 다양한 주제의 대규모 프로그래밍 problem descriptions 생성.
        - magicoder 방식 활용
      - b. 솔루션 생성
        - "좋은 프로그래밍의 일반적인 규칙"을 프롬프트에 추가하여 솔루션 생성 → 솔루션 품질 향상
      - c. 정확성 분석
        - 잘못된 솔루션을 미세 조정 데이터에 포함시키면 모델 품질 저하될 수 있음
        - 완전한 정확성은 아니지만 근사치를 구하기 위하여 솔루션의 소스 코드를 추출하여 정적/동적 분석 기법을 조합하여 정확성을 테스트하는 방법 적용
          - 정적분석: 구문 정확성 보장 목표. linter와 구문 분석기를 통해 구문 오류, 초기화 되지 않은 변수, non-imported 함수, 코드 스타일, 타이핑 오류 등을 탐지
          - 단위 테스트 생성 및 실행: 문제와 솔루션에 대해 모델 기반 단위 테스트 생성 후 샌드박스 환경에서 실행하여 오류 포착
      - d. 오류 비드백 및 반복적 자가 수정
        - 솔루션 코드가 실패햇을 때. (problem description, faulty solution, 파서/린터/테스터의 피드백-stdout, stderr, return code)로 프롬프트 재구축
        - unit text 실패시에는 ①코드를 수정하거나 ②코드에 맞게 unit test를 수정할 수 있음.
        - 모든 검사를 통과한 데이터만 SFT에 활용.
      - d. 미세 조정 / 반복 개선
        - 여러 라운드에 걸쳐 fine tuning 됨. 각 라운드는 이전 라운드를 기반으로 하며 각 라운드가 끝나면 다음 라운드를 위한 더 높은 품질의 합성 데이터 생성 가능
  - 합성 데이터 생성(programming language translation)
    - 주요 언어 대비 덜 일반적인 언어(예: TS, PHP)의 성능 격차 발견. 데이터의 부족 때문.
    - 이를 보완하기 위해 주요 언어 데이터를 일반적인 언어로 번역하여 데이터로 사용. 큰 효과 있었음
       ![image](https://github.com/user-attachments/assets/de1977a6-15df-42e2-9b3d-6dee2ada6230)
  - 합성 데이터 생성(backtranslation)
    - execution feedback이 적용되지 않는 코드 태스크(예: 문서, 설명) 개선을 위한 다단계 접근 방식.
    - 120만개 데이터 생성
    - ① 목표 태스크에 대한 데이터 생성(code snippet에 docstring 생성하기)
      ② 역번역 실시(예) 설명에서 코드만 생성하도록 요청)
      ③ 원본 코드를 참조하여 모델 기반 출력의 품질 판단.(예: 역번역된 코드가 원본에 얼마나 충실한가요) 가장 높은 점수의 샘플만 학습에 활용
  - 기타
    - 거부 샘플링 중 시스템 프롬프트 조정: RS 중에 코드 가독성, 문서화, 철저함, 구체성 개선을 위해 코드 특화 시스템 프롬프트 사용.(정확히 뭔진 알 수 없음)
      (시스템 프롬프트를 통한 생성 코드 품질 개선) ![image](https://github.com/user-attachments/assets/1e689139-83fd-4980-bb39-f4ff4ac85137)
    - 실행 및 모델 신호로 데이터 필터링
      - 합성 데이터와 달리 RS 데이터에서는 자연어와 코드가 혼재되어 있어 품질 문제 판단하기 어려움.
      - 이를 해결하기 위해 "코드 정확성", "코드 스타일" 이라는 두 기준을 이진 점수(0/1)로 평가하는 모델 평가 접근 방식 활용
      - 만점(2점)을 받은 샘플만 유지
      - 처음에는 엄격한 필터링으로 challenging한 프롬프트가 불균형하게 제거되는 문제가 있었고 오히려 성능이 퇴보하기도 함
      - 가장 challenging한 것으로 분류된 문제들을 LLaMA의 판단 기준에 부합할때까지 전략적으로 응답을 수정함(어떻게?)

2. 다국어(Multilinguality)
- 독일어, 프랑스어, 이탈리아어, 포르투갈어, 힌디어, 스페인어, 태국어 등 학습
- 사전 학습 모델에서 분기하여 90%의 다국어 토큰으로 구성된 데이터 믹스에 대한 사전 학습 진행 후 사후학습까지 수행하여 "다국어 전문가 모델" 생성
- 이 모델을 활용하여 비영어권 언어로 된 고품질 데이터 수집
- 데이터 분포:
  - 다국어 SFT 사람 주석 2.4% 다른 NLP 작업 44.2%, rejection sampling 데이터 18.8%, 번역된 reasoning 데이터 34.6%
  - 사람 주석
    - 언어학자 및 원어민으로 부터 수집된 고품질 데이터
    - 대부분 실제 사용 사례를 나타내는 open-ended prompt로 구성
  - 다른 NLP 작업
    - 다른 작업의 다국어 학습 데이터를 대화 형식으로 재작성(예: exams-qa, Conic10k)
    - 저품질 데이터 제거를 위해 LID 기반 필터링과 Blaser2.0 사용.
    - 병렬 텍스트 데이터의 경우 bitext pair를 직접 사용하는 대신 다국어 템플릿을 적용
  - 거부 샘플링 데이터
    - 수정 최소화를 위해 인간 주석이 달린 프롬프트에 거부 샘플링 적용
    - post-training 마지막 라운드에서 temperature 0.6을 사용. 응답 형식, 구조, 일반적인 가독성 개선을 위해 특수 시스템 프롬프트 사용
    - 데이터 선별 시 보상모델의 판별에 앞서 언어 일치율 보장을 위한 다국어별 검사 구현(동일언어로 response가 생성되었는지 판별)
  - 번역된 데이터
    - 번역 시 영어 문화적 맥락에 편향되지 않도록 주의 기울임. reasoning 데이터 번역하여 합성 데이터 사용함.

3. 수학/추론(Math and Reasoning)
- 프롬프트 부족 해결
  - 수학적 맥락 관련 사전 학습 데이터를 확보하고 QA 형식으로 변환하여 SFT 데이터로 활용
  - "수학적 기술 분류체계"를 구성하여 그에 따라 모델이 부족한 부분을 파악하고 데이터 수동 작성
- 단계별 추론을 통한 데이터 보강
  - LLaMA3를 통해 프롬프트에 대해 다양한 수의 단계별 솔루션 생성
  - 정답을 기준으로 생성결과 필터링
  - 또한 LLaMA3의 자체 검증을 통해 단계별 솔루션이 유효한지 확인하여 추가 필터링(잘못된 추론 추적 필터링)
    - 결과와 중간 추론 단계가 올바른지 판단하는 보상모델을 훈련하여 평가
    - 어려운 프롬프트에 대해 몬테카를로 트리 서치(MCTS)를 통해 유효한 추론 추적을 생성하여 보강
- code 및 텍스트 추론 결합
  - 텍스트 추론과 파이썬 코드 조합을 통해 문제 해결하도록 유도
  - 추론 단계가 유효하지 않은 경우 제거
- 피드백
  - 사람의 피드백을 시뮬레이션 하기 위해 잘못된 생성과 피드백을 활용하여 오류 수정
  - 피드백 참고 방식:
    - LEMA(Learning from mistakes makes llm better reasoner): GPT-4를 통해 잘못된 단계를 식별하고 수정하도록 요청
      ![image](https://github.com/user-attachments/assets/008caf40-2f1c-4b00-91e4-f4c599024f5a)
    - Generating sequences by learning to self-correct: corrector 모델 훈련을 위해 오류 생성물에 대한 스칼라/자연어 피드백을 사용할 수 있는 online 훈련 절차에 관련된 논문
    - Self-refine: Iterative refinement with self-feedback: Single LLM으로 initial generation을 수행하고 generated output에 대해 Feedback과 Refine의 과정을 iteration하여 output quality를 향상시키는 방법론

4. Long Context
- 미세 조정 과정에서 짧은 컨텍스트와 긴 컨텍스트의 기능 균형을 위해 레시피를 신중히 조정해야 함 발견
- 짧은 문맥 데이터만 가지고 SFT를 수행하면 긴 문맥 기능이 크게 퇴보하였음.
- 전체 데이터의 0.1% 정도를 긴 문맥 데이터로 혼합하면 짧은 데이터, 긴 데이터 모두에서 성능이 좋았음.
- DPO에서는 짧은 컨텍스트 학습 데이터만 사용해도 SFT가 잘되어 있다면 성능에 부정적인 영향을 끼치지 않았음.
- LLaMA3를 활용하여 (QA, 긴 문서 요약, 코드 저장소 추론) 등 긴 문맥 사용 사례를 기반으로 합성 데이터 생성
  - QA
    - 사전 학습 데이터에서 긴 문서 세트를 큐레이션.
    - 8K 토큰 청크로 분할하고 LLaMA3로 조건부 QA쌍을 생성. 단, 훈련중에는 전체 문서가 컨텍스트로 사용됨
  - 요약
    - 8K 길이의 청크를 요약한다음 그 요약을 다시 요약하는 방식으로 긴 컨텍스트 문서에 계층적 요약 적용
    - 훈련 중엔 전체 문서를 입력으로 모델에 "모든 중요한 세부 사항을 보존하면서 문서를 요약하라" 라는 메세지를 표시
    - 또한 문서 요약을 기반으로 QA 를 생성하고 긴 문서를 전체적으로 이해해야 하는 질문을 모델에 제시
  - Long Context 코드 추론
    - 파이썬 파이릉ㄹ 파싱하여 import 구문을 식별하고 종속성 결정
    - 최소 다른 5개 파일에서 참조된 파일들을 가장 많이 의존하는 파일로 선택
    - 이러한 주요 파일 중 하나를 저장소에서 제거하고 모델에 누락된 파일에 의존하는 파일을 식별하여 누락된 코드를 생성하도록 요청
   
5. Tool Use
![image](https://github.com/user-attachments/assets/b70787fb-2c76-409a-823a-36f548da5567)
- 검색 엔진 또는 코드 인터프리터와 같은 도구 사용 방법을 LLM에 가르침
- 대화에서 주어진 문맥에서 사용자 쿼리를 통해 올바른 도구 호츌 하도록 학
- 종류
  - 검색 엔진: 웹에서 검색해야 하는 경우 Brave Search를 사용하도록 훈련
  - 파이썬 인터프리터: 사용자가 업로드한 파일을 읽고, 질문 답변, 요약, 데이터 분석 또는 시각화와 같은 작업 수행
  - 수학 계산 엔진: Wolfram Alpha API를 사용하여 수학,과학 문제를 정확하게 해결
- 구현
  - 다양한 메소드를 가진 Python 객체로 구현됨
  - 함수 정의와 호출은 JSON 형식으로 변환하여 웹 API 호출 등에 사용
  - 모든 Tool 호출은 Python 인터프리터에 의해 실행되며 시스템 프롬프트로 컨트롤됨.
- Data collection
  - 사람의 주석과 선호도에 의지함
  - post-training 파이프라인과 크게 2가지 차이점 존재
    - 메세지 수준에서 주석을 달아 세분화된 피드백 수집
    - rejection sampling 수행 x
- Tool datasets
  - Single-step tool
    - few-shot으로 tool 중 하나를 호출해야 하는 합성 사용자 프롬프트로 시작
    - tool 수행 결과를 컨텍스트로 추가
    - 모델에 다시 프롬프트를 수행해 tool 출력을 기반으로 사용자 쿼리에 대한 최종 답변 생성
    - (시스템 프롬프트, 사용자 프롬프트, 도구 호출, 도구 출력, 최종 답변)
  - Multi-step use
    - 최소 두개의 도구 호출이 필요한 사용자 프롬프트를 생성하도록 LLaMA3 사용.
    - ReACT와 유사하게 추론단계와 도구 호출이 교차로 구성된 솔루션을 생성하도록 few-shot 생성
  - 파일 업로드
    - txt, docx, pdf, pptx, xlsx, csv, tsv, py, json, jsonl, html, xml
   
[Factuality]
- 모델이 "자신이 아는 것을 아는 능력"을 향상 시키는데 중점. 즉, 아는 것과 모르는 것을 구별할 수 있게 만들고자 함
- 방법
  - 모델이 알고 있는 정보에서 질문을 만듬
  - 모델에게 질문하여 답한 정보의 사실성과 유용성 판단
  - 만약 틀린 답을 할경우 "모르겠습니다"라고 대답하도록 데이터 생성 및 학습

[조종 가능성-Steerability]
- 모델이 해동과 결과를 조절할 수 있는 능력을 향상시키는데 중점
- 주로 응답 길이, 형식, 어조, 캐릭터/페르소나 등
- 다양한 시스템 프롬프트를 설계하여 LLaMA3를 통해 평가하고 모델이 지시를 잘 따르는지 평가
- 데이터 수집
  - 시스템 프롬프트 설계: AI 역할, 톤, 전문 지식 영역 등을 지정한 다양한 시스템 프롬프트 생성
  - 대화 시뮬레이션: 앞서 설계한 프롬프트로 LLaMA3와 대화
  - 일관성 평가: 모델이 얼마나 시스템 프롬프트를 잘 따르는지 관찰하여 일관성 평가
- 수집한 데이터로 보상 모델링, RS, SFT, DPO에 활용
- 













 
