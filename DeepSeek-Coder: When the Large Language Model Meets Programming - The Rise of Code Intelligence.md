release-date: 2024-01-26
arxiv: https://arxiv.org/pdf/2401.14196.pdf
github: https://github.com/deepseek-ai/DeepSeek-Coder

# 요약
- 16k context length의 DeepSeek-coder-Base와 DeepSeek-Coder-Instruct 모델 제공 (1.3B, 6.7B, 33B)
- 2T 토큰의 87개 프로그래밍 언어 학습
- NTP(Next Token Prediction)과 FIM(Fill-In-The-Middle)로 학습
- DeepSeek-Coder-Base 6.7B는 CodeLlama-33B와 비교하여 경쟁력 있는 성능 보여줌
- DeepSeek-LLM-7B 에서 continous pre-training을 수행하여 학습한 DeepSeek-Coder-v1.5는 기존 DeepSeek-Coder-6.7B보다 강력함.
  - 기존 자연어 LLM에 추가 학습하는 코드 모델이 가장 강력함을 시사
- 모델은 연구 및 제한 없는 상업적 사용을 허용하는 라이선스로 제공

# 데이터 수집
![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/e91ef968-b88e-4ae0-9061-bdb86a5132a0)

- 데이터셋 구성: 87% 소스코드, 10% 영어 + 코드 관련 자연어 말뭉치, 3% 코드와 관련없는 중국어 자연어 말뭉치
  ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/1fe35a3b-4cc2-42da-a813-966f89ad5d6d)
  - 영어 말뭉치는 Github의 Markdown과 StackExchange로 구성
  - 중국어는 뉴스 기사로 구성

- STEP1. GitHub 데이터 크롤링 및 필터링(Data Crawling & Rule Filtering):
  - 2023년 2월 이전에 생성된 GitHub의 공개 저장소에서 데이터 수집
  - 총 87개 프로그래밍 언어를 대상으로 함
  - StarCoder 프로젝트에서 사용된 필터링 규칙을 적용하여 낮은 품질의 코드를 사전에 필터링
  - 필터링 규칙에는 평균 줄 길이가 100자를 초과하거나 최대 줄 길이가 1000자를 넘는 파일 제거, 알파벳 문자가 25% 미만인 파일 제거, XSLT를 제외한 "<?xml version=" 문자열이 처음 100자 안에 나타나는 파일 제거 포함
  - HTML 파일의 경우, 보이는 텍스트가 코드의 최소 20%를 차지하고 100자 이상일 때만 보존
  - JSON 및 YAML 파일은 문자 수가 50자에서 5000자 사이인 경우만 보존하여 데이터가 많은 파일 제거​​​​
 
- STEP2. 의존성 파싱(Dependency Parsing):
  - 기존 대규모 언어 모델은 파일 수준의 소스 코드에 주로 사전 훈련되어 프로젝트 내 다른 파일 간의 의존성을 무시
  - 실제 응용 프로그램에서는 프로젝트 수준의 코드 시나리오를 효과적으로 처리하는 데 어려움이 있음
  - 이를 해결하기 위해 프로젝트 수준의 코드 의존성을 고려하여 데이터 구성​​
  - 파일의 호출관계(종속성, 의존성)에 따라 파일을 정렬함으로써 현재 파일 앞에 놓인 컨텍스트를 확보할 수 있도록 함.
  - 프로젝트 내 종속성 분석(위상 정렬)
    ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/20467691-acd6-4553-bf11-995b2ca8ad0c)

    
- STEP3. repository 수준 중복 제거(Repo-Level Deduplication):
  - 대규모 언어 모델의 훈련 데이터 세트에서 중복 제거를 통해 성능 향상
  - repository 수준에서 중복 제거(near-deduplication)를 수행하여 파일 수준의 중복 제거가 저장소 구조를 방해하는 것을 방지
  - repository 레벨에서 연결된 코드를 단일 샘플로 처리하고 동일한 near-deduplication 알고리즘 적용​​
 
- STEP4. 품질 검사 및 오염 제거(Quality Screening and Decontamination):
  - STEP1에서 언급된 필터링 규칙 외에도 컴파일러와 quality 모델을 결합한 휴리스틱 규칙을 사용하여 낮은 품질의 데이터를 추가로 필터링.
  - syntax 오류, 가독성이 낮은 코드, 모듈성이 낮은 코드를 포함하여 제거.
  - GitHub에서 벤치마크 데이터 세트(예: HumanEval, MBPP, GSM8K 등)의 정보가 포함될 수 있으므로 n-gram 필터링 프로세스를 구현하여 훈련 데이터가 오염되는 것을 방지.
    - 문제, docstring, 솔루션을 10-gram으로 동일시 데이터에서 제외. 10-gram 보다 짧지만 3-gram보다는 작지 않은 경우 exact match로 재필터링.
  - 특정 기준에 맞는 코드 세그먼트를 제거하는 과정 포함​​.

![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/3baa78c0-b4f1-4343-a3b8-d805100ff248)


# Training Policy
- Pre-training 학습 전략
  - Next Token Prediction
  - Fill-in-the-middel
    - DeepSeek-Coder-Base 1.3B 모델로 실험 시 100% FIM 비율로 학습했을 때, FIM 벤치마크인 HumaEval-FIM에서 최고 성능을 달성하지만 code generation 성능이 저하됨
    - 따라서 50%의 비율로 한 PSM(prefix-suffix-middel)을 기본 훈련 정책으로 선택.
    - `<|fim_start|>`prefix`<|fim_hole|>`suffix`<|fim_end|>`middle
    - FIM 방법을 packing 프로세스 이전의 문서 레벨에서 구현 -? (Bavarian 논문 참고 필요)
    ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/5331e344-f110-49a7-bbef-8ca3a8c0a5fa)
    * MSP(Masked Span Prediction) T5에서 제시된 방식



# Model Architecture
- DeekSeek-AI-LLM 프레임워크 기반으로 구축
- RoPE, Grouped-Query-Attention, FlashAttention v2 사용
![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/29c9e305-7303-463a-b983-5707b14ec40b)

- Tokenizer: BPE 토크나이저 32,000 vocab size로 구성
- Optimization: AdamW 𝛽1=0.9 𝛽2=0.95
  - warmup 10%, 최종 lr은 최초 lr의 10%, lr은 step마다 √︃1/10 만큼 작아짐
- Environment
  - HAI-LLM 프레임워크 사용: Tensor Parallel, Zero data parallel, PipeDream 파이프라인 병렬화 등
  - A100, H800 사용. 각 노드는 A100은 NVLink, H800은 NVLink 및 NVSwitch로 연결됨. A100클러스터와 H800클러스터는 InfiniBand 인터커넥트 사용
- Long Context
  - repository 수준 코드 처리와 같은 context를 위해 RoPE 매개변수 다시 구성
  - linear scaling strategy를 사용하여 caling factor를 1→4로 증가시키고 base frequency를 10,000 → 100,000으로 변경
    *Note; Scaling Factor 증가시키면 모델이 위치 정보를 더 강조하게 됨. base frequence가 증가하면 세밀한 위치 차이 구별할 수 있으나 과적합 우려
  - 추가적으로 1000 step 학습시키고 배치크기는 512, 시퀀스 길이는 16k로 설정됨
  - 이론적으로 이러한 수정을 통해 최대 64K토큰까지 처리 가능
 
# Instruct Tuning
- DeepSeek-Coder-Base에서 DeepSeek-Coder-Instruct 생성
- Alpaca instruction 형식으로 학습. 각 대화 턴을 구분하기 위해 고유한 구분자 토큰을 사용하여 각 세그먼트 종료 표시
- 100step의 warmup 및 최초 lr 1e-5를 가진 코사인 스케쥴 사용
- 4M의 batch size와 총 2B 토큰 사용

# Experiment and Result
- 코드 생성, FIM 코드 코드 완성, cross-file 코드 완성, 프로그램 기반 수학 추론 네 가지 작업에서 평가
  - 비교 대상; CodeGeeX2, StartCoder, CodeLlama, code-cushman-001, GPT3.5/4

 ## 코드 생성
- HumanEval / MBPP
   ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/56278238-6436-4a7e-b6d7-ff22d0c4c23b)
  - 특히, HumanEvaal의 경우 C++, Java, PHP, TypeScript, C#, Bash, Javascript로 확장하여 검증.
  - Greedy Decoding으로 측정
- DS-1000
  ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/f2b1037a-8d83-46e7-aaf8-542d3bfcbd44)
  - 모든 라이브러리에서 높은 정확도 달성
  - 실제 워크플로우에서 라이브러리 더 정확하게 사용할 수 있음 증명 
- LeetCode
  ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/867c6fb0-0c06-42b1-b669-e79e7e9bc32a)
  - 최신 문제 수집하여 오염 방지
  - Instruction ("\nPlease complete the code below to solve the above problem:\n") 사용하여 수행
  - 33B 모델로 유일하게 GPT-3.5를 넘어선 오픈소스 모델임
- 해당 능력을 테스트한 결과 CoT(Chain-of-Thought) 프롬프팅 구현이 모델의 능력을 현저히 향상시킴
- 초기 프롬프트 뒤에 `"You need first to write a step-by-step outline and then write the code."` 라는 지치믈 추가하여 성능이 더 향상됨
- 상세한 코드 설명을 작성하는 프로세스가 모델이 코딩 작업의 로직과 종속성의 복잡성, 특히 더 높은 복잡성 작업을 더 효과적으로 이해하고 처리하는데 도움이 되었던 것.

## FIM 코드 완성
- SantaCoder, StarCoder, CodeLlama 와 비교 수행
- Single-Line inflling 벤치마크로 측정
![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/a1b3442d-6c35-4ed3-a7a1-0ba0d524d782)
- 타 모델 대비 우수한 성능
- pre-tranining 데이터의 품질이 우수한 부분에서 기인했다 판단됨

## Cross-file 코드 완성
- Cross-file 코드 완성은 모델이 여러 파일에 걸쳐 있는 repository에 접근하고 이를 이해해야 하는 태스크.
- CrossCodeEval 벤치마크 사용(https://crosscodeeval.github.io/)
![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/ca0ee03c-0026-4587-b6e6-5e19af23292c)
- Exact Match와 Edit similarity 사용하여 평가함.
- Repo 수준보다 File 수준으로 학습했을 때, Java, TypeScript 및 C# 등에서 성능이 저하되는 것 확인됨

## 프로그램 기반 수학 추론
- Program-Aided Math Reasoning (PAL) 방법 사용.
- 7가지 벤치마크를 통해 적용됨.(GSM8K, MATH, GSMHard, SVAMP, TabMWP, ASDiv, MAWPS)
- 자연어로 솔루션 단계를 설명하고 그 단계를 코드로 실행하도록 교대로 안내되는 프로세스
![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/ea036bc0-aa7f-405b-8b4c-9d6422f905f2)
- 모든 벤치마크에서 놀라운 성능 발휘.

# Continue Pre-Training From General LLM
- scratch가 아닌 DeepSeek-LLM-7B Base의 자연어 모델에 2T 토큰을 추가 학습시켜 DeepSeek-Coder-v1.5 생성
- 2T 토큰 상세:
  ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/47d31944-67c5-4ca1-8060-45b2264d0c67)
- DeepSeek-Coder와 달리 FIM 학습은 포함하지 않음.
- DeepSeek-Coder-6.7B-Base와 DeepSeek-Coder-1.5v-7B의 비교 실시
  ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/b53731f2-66f6-4ab6-8fd2-33488242f1b7)
- 결론: 가장 효과적인 코드 LLM은 강력한 자연어 일반 LLM을 기반으로 구축된 모델


