WizardLM: Empowering Large Language Models to Follow Complex Instructions
WizardCoder: Empowering Code Large Language Models with Evol-Instruct


[요약]
- WizardLM
  - Data-driven한 접근법 고안
  - Evol-Instruct 데이터 생성법을 제안하여 해당 데이터로 미세 조정한 LLaMA 모델(7B)로 우수한 성능 달성
- WizardCoder
  - WizardLM에서 고안한 Evol-Instruct 기법을 코드 도메인에 적합하도록 수정하여 데이터 생성 및 starcoder 미세조정

---
[Evol-Instruct]
- 기존의 Instruction 데이터셋(예: Alpaca 52k)는 사람이 생성한 데이터셋으로 데이터 셋의 난이도 분포가 쉬움 / 보통에 치우쳐 있음
- 해당 데이터셋으로는 실제 사람이 요구하는 다양한 작업과 명령에 대응하게 학습되기 어려움.
- 따라서 다양한 난이도의 오픈 도메인 명령어를 사람 대신 LLM(ChatGPT-3.5)을 사용해 자동으로 생성하는 Evol-Instruct 방법 도입.
- 초기의 간단한 명령어를 더 복잡하거나(In-depth Evolving) 다양성을 위해 새로운 명령어를 생성함.(In-breadth Evolving)
https://github.com/nlpxucan/WizardLM/raw/main/imgs/git_overall.png
- Alpaca instruction 훈련 데이터(52K) → 4회 Evolving 수행 → 25만개 데이터 생성 → basemodel들과의 비교를 위해 7만개로 샘플링

- Evolving 수행 과정
  - (주어진 Instruction을 Evolving을 위한 Prompt로 LLM 데이터 생성 → 성공적으로 생성되었는지 검증) * 반복
  - In-depth Evolving(심층 진화): 제약 조건 추가, 심화, 구체화, 추론 단계 추가, 입력 복잡화 등의 진화를 수행하여 점진적인 난이도 증가
  - In-Breadth Evolving(광범위 진화): 토픽 커버리지, 스킬 커버리지, 전반적 데이터 세트의 다양성 향성
  - 생성된 response의 길이가 비교적 짧거나(80자 이하), '죄송합니다' 가 포함되어 있거나, equal판단 prompt의 결과를 통해 evolving 실패 판단함.
  - 모든 Evolving이 완료되면 모든 epoch의 데이터를 병합하여 데이터로 활용.
https://github-production-user-asset-6210df.s3.amazonaws.com/36894403/250261065-666d9e55-d2ac-4e1b-8a35-9205c32745d2.png
  - 샘플 prompt
  ```none
  프롬프트 재작성자로 활동해 주세요.
  여러분의 목표는 주어진 프롬프트를 좀 더 복잡한 버전으로 재작성하여 유명한 AI 시스템(예: chatgpt 및 GPT4)이 처리하기 어렵게 만드는 것입니다.
  하지만 재작성된 프롬프트는 합리적이어야 하며 사람이 이해하고 응답할 수 있어야 합니다.
  주어진 프롬프트#:의 표와 코드 등 텍스트가 아닌 부분은 재작성할 때 생략할 수 없습니다. 또한 #Given Prompt#의 입력도 생략하지 마세요.
  다음 방법을 사용하여 주어진 프롬프트를 복잡하게 만들어야 합니다:
  
  #Given Prompt#에 제약 조건/요구 사항을 하나 더 추가하거나 다음과 같이 입력하세요.
  #Given Prompt#에 특정 문제에 대한 문의가 포함되어 있는 경우 문의의 깊이와 폭을 늘릴 수 있습니다.
  일반적인 개념을 보다 구체적인 개념으로 바꿔주세요.
  #Given Prompt#를 몇 가지 간단한 사고 과정만으로 해결할 수 있는 경우, 다단계 추론을 명시적으로 요청하도록 재작성할 수 있습니다.
  
  재작성된 프롬프트#가 장황해지지 않도록 최선을 다해야 하며, #Rewritten Prompt#는 #Given Prompt#.에 10~20개 단어만 추가할 수 있습니다.
  '#Given Prompt#', '#Rewritten Prompt#', 'given prompt' 및 'rewritten prompt'는 #Rewritten Prompt#에 표시할 수 없습니다.
  #Given Prompt#:
  <Here is instruction.>
  #Rewritten Prompt#:
  ```
    - equal 판단 prompt
  ```none
  Here are two Instructions to ChatGPT AI, do you think they are equal to each other, which meet the
  following requirements:
  1. They have same constraints and requirments.
  2. They have same depth and breadth of the inquiry.
  The First Prompt: <Here is first instruction.>
  The Second Prompt: <Here is second instruction.>
  Your Judgement (Just answer: Equal or Not Equal. No need to explain the reason.):
  ```

[WizardCoder에서의 Evolving-Instruct]
https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/66ad2d84-8e9a-43bb-9944-6a868931412c

- 앞선 WizardLM의 Evol-Instruct 기법을 코드 도메인에 고려하여 활용
- code alpaca(20k) 데이터를 초기 데이터로 Evoving하여 starcoder를 fine tuning함.
- Evolving 프롬프트를 코드 도메인에 맞게 몇 가지 수정
  - 기존 Evol-Instruct의 진화 방식에서 deepening(심화), complicating input(입력 복잡화), In-Bradth Evolving(심층진화)을 삭제하여 간소화함
  - Evolutionary Prompt를 단일화 하여 형태 단순화
  - 코드 도메인의 특성을 고려하여 code debugging과 code time-space complexity constraints 진화 명령어 추가.
  - prompt 예시)
  ```
  Please increase the difficulty of the given programming test question a bit. 
  
  You can increase the difficulty using, but not limited to, the following methods: 
  {method} 
  
  {question} 
  ```
  ```
  주어진 프로그래밍 시험 문제의 난이도를 조금 높여주세요. 
  
  다음 방법을 사용하여 난이도를 높일 수 있지만 이에 국한되지는 않습니다: 
  {method} (evolving 유형 5가지 중 하나)
  
  {question} (evolving 해야 하는 기존 명령어)
  ```
  - method에 삽입될 evolving 유형 5가지
    ```
    Add new constraints and requirements to the original problem, adding approximately 10 additional words. 
    
    Replace a commonly used requirement in the programming task with a less common and more specific one. 
    
    If the original problem can be solved with only a few logical steps, please add more reasoning steps. 
    
    Provide a piece of erroneous code as a reference to increase misdirection. 
    
    Propose higher time or space complexity requirements, but please refrain from doing so frequently. 
    ```
    ```
    원래 문제에 새로운 제약 조건과 요구 사항을 추가하여 약 10개의 단어를 추가합니다. 
    
    프로그래밍 작업에서 일반적으로 사용되는 요구 사항을 덜 일반적이고 더 구체적인 요구 사항으로 바꿉니다. 
    
    원래 문제를 몇 가지 논리적 단계만으로 해결할 수 있다면 추론 단계를 더 추가하세요. 
    
    잘못된 코드 조각을 참조로 제공하여 잘못된 방향을 유도합니다. 
    
    더 높은 시간 또는 공간 복잡도 요구 사항을 제안하되, 너무 자주 제안하는 것은 자제해 주세요. 
    ```
  - WizardLM에서와 다르게 각 evolving epoch마다 데이터를 병합하여 Humaneval 벤치마크 pass@1 테스트. pass@1 지표가 감수하는 순간 evolving 중단.
    https://github-production-user-asset-6210df.s3.amazonaws.com/36894403/250257158-dff7f00f-d1d6-42a5-a5f3-19f21974858a.png



