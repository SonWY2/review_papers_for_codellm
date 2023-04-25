
# paper_caputred_images_repo
논문의 이미지를 캡처 및 보관



## 🎁[PreTrain]

<details>
<summary>[Awesome-totally-open-chatgpt](https://github.com/nichtdax/awesome-totally-open-chatgpt) - ChatGPT LLM 오픈소스 대체제 정보 정리</summary>

```
- ChatGPT는 채팅을 위한 RLHF가 적용된 GPT-3.5
- 채팅 시스템을 위한 다른 언어 모델을 적용한 오픈소스들만 나열
- 4가지 태그로 구분
  - B: Bare(데이터 없음, 모델 가중치 없음, 채팅 시스템 없음)
  - M: Mildly bare(데이터 있음, 모델 가중치 있음, API를 이용한 기본 채팅)
  - F: Full(데이터 있음, 모델 가중치 있음, TUI와 GUI를 포함한 팬시한 채팅 시스템)
  - C: Complicated(세미 오픈소스, 실제론 오픈소스 아님, 클로즈 모델 기반,..)
- 리스트
  - lucidrains/PaLM-rlhf-pytorch : Bare
  - togethercomputer/OpenChatKit : Full
  - oobabooga/text-generation-webui : Full
  - KoboldAI/KoboldAI-Client : Full
  - LAION-AI/Open-Assistant : Full
  - tatsu-lab/stanford_alpaca : Complicated
  - BlinkDL/ChatRWKV : Full
  - THUDM/ChatGLM-6B : Full
  - bigscience-workshop/xmtf : Standard
  - carperai/trlx : Bare
  - databrickslabs/dolly : Standard
  - LianjiaTech/BELLE : Standard
  - ethanyanjiali/minChatGPT : Standard
  - cerebras/Cerebras-GPT : Standard
  - TavernAI/TavernAI : Full
  - Cohee1207/SillyTavern : Full
  - h2oai/h2ogpt : full
  - mlc-ai/web-llm : Full
  - Stability-AI/StableLM : Full
  - clue-ai/ChatYuan : Full
```

</details>

<details>
<summary>상용❌[LLaMA](https://ai.facebook.com/blog/large-language-model-llama-meta-ai/)</summary>
<div markdown="1">
  
  ```
- 7B, 13B, 33B, 65B 의 4가지 사이즈로 공개
- 훨씬 작은 규모지만, 데이터 학습 강화 및 파인 튜닝하여 더 큰 규모의 모델과 비교가능한 효율적인 모델
- 33B/65B는 1조 4천억개의 토큰으로 훈련됨(7B는 1조)
- "13B 모델이 175B인 GPT-3보다 뛰어나고, 65B는 훨씬 더 큰 Chinchilla70B 및 PaLM-540B 와 경쟁 가능"
- 인공지능 연구등 비상업적 용도로만 활용 가능(신청하여 승인 필요)
  ```
</div>
</details>


<details>
<summary>상용🔺[StableLM: Stability AI Language Models](https://github.com/Stability-AI/StableLM)</summary>
<div markdown="1">
  
  ```
- 3B/7B 모델을 공개, 15B/30B/65B 모델도 공개 예정이고 175B까지 계획중. 학습코드는 공개되어 있지 않음.
- 모델은 CC BY-SA-4.0 라이센스로 출처 표기시 상업적 이용 가능
- 오픈 데이터셋인 The Pile에 기반했지만 3배 크기인 1.5T 토큰을 가지는 새로운 데이터셋으로 훈련
- 컨텍스트 길이는 4096 토큰
- PoC로 Alpaca 프로시져를 따라서 파인튜닝한 StableLM-Tuned-Alpha-7B 모델도 공개
- 5개의 대화형 데이터셋을 이용 : Stanford's Alpaca, Nomic-AI's gpt4all, RyokoAI's ShareGPT52K datasets, Databricks labs' Dolly, Anthropic's HH
- 챗봇 데모는 Hugging Face에 공개
  ```
</div>
</details>

<details>
<summary>상용❌[Alpaca](https://crfm.stanford.edu/2023/03/13/alpaca.html) LLaMA + 52,000(davinci-003)</summary>

  ```
-   LLaMA 모델의 큰 약점은 질문-답변을 위한 "명령어-튜닝"이 부족하다는 것
-   OpenAI의 큰 혁신중 하나는 GPT-3에 명령어 튜닝을 추가한 것
-   스탠포드 CRFM에서 메타의 LLaMA 7B를 52K Instruction-Following 데이터를 통해서 파인튜닝
-   OpenAI의 GPT-3.5(text-davinci-003)와 비슷하게 동작하지만, 매우 작고 저렴
-   스탠포드는 여기에 52000개의 훈련 예제를 제공하고 $100 만으로 훈련 가능하게 해줌
-   가장 작은 7B 모델은 이제 라즈베리파이/모바일 폰에서도 도는데, 매우 인상적인 결과를 뽑아내줌
-   하지만, 아직 상업용은 아님(3가지 이유에서 불가능. LLaMA의 라이센스/명령어셋 데이터를 OpenAI 모델에서 만들어냄/안전조치를 설계하지 않음)
-   Alpaca는 52K의 예제와 $100의 비용으로도 7B 모델(4bit 양자화로 4GB로 줄인) 파인 튜닝이 가능하며, 최신 text-davinci-003 과 비슷한 결과를 낼수 있다는 것을 보여줌
  ```
  
</div>
</details>

<details>
<summary>상용❌[CodeAlpaca](https://github.com/sahil280114/codealpaca)</summary>



  ```
-   스탠포드 Alpaca 7B/13B 기반으로 개발자가 코딩 작업에 사용하기 좋게 튜닝한 모델
-   코드 생성에 관련된 20K 짜리 Instruction Following 데이터([`data/code_alpaca_20k.json`](https://github.com/sahil280114/codealpaca/blob/master/data/code_alpaca_20k.json)로 교체 (Self-Instruct 기술 이용)
-   데이터 생성 파이프라인을 일부 수정: 일반 작업이 아닌 코드 생성/편집/최적화에 관련되게 프롬프트를 변경
-   Hugging Face 훈련 코드와 Deepspeed 로 파인 튜닝
  ```
  
</div>
</details>

<details>
<summary>상용❌[GPT4ALL](https://github.com/nomic-ai/gpt4all): LLaMA 7B + 437,605 대화(ChatGPT API)</summary>

```
- 어시스턴트 스타일 대규모 언어모델
- 수집된 데이터, 데이터 수집 프로시져, 훈련 코드, 최종 모델 가중치 등을 모두 공개
- GPT 3.5 Turbo로 생성된 800k 데이터(코드/스토리/대화)로 훈련
- LAION OIG, 스택오버플로우의 코딩 질문, Big-Science/P3 의 명령어 튜닝 등을 기본 데이터 셋으로 활용
- 스탠포드 알파카 등을 참고하고, 데이터를 ATLAS에 올려서 큐레이션 및 클리닝 진행
```
</details>

<details>
<summary>상용❌[Vicuna](https://vicuna.lmsys.org/): LLaMA 13B + ShareGPT (공유된 ChatGPT 대화)</summary>

```
- LLaMA 를 ShareGPT에 공유된 7만개의 사용자 대화로 파인 튜닝
- ChatGPT 및 구글 Bard 대비 90% 이상의 품질을 보여줌
- 훈련/서빙 코드 및 온라인 데모 포함
- Alpaca 에 메모리 최적화 적용 및 다단계 대화를 고려하고, Spot Instance로 비용을 절감(훈련비용은 약 $300 정도)
```
</details>


<details>
<summary>상용❌[KoAlpaca](https://github.com/Beomi/KoAlpaca): LLaMA 7B + Alpaca(52,000한글) / Ployglot-ko 5.8B + Alpaca(52,000한글)</summary>

```
- 스탠포드의 Alpaca 모델 학습방식과 동일한 방식으로 학습
- 백본모델로 Polyglot-ko 5.8B 와 LLaMA 7B 를 이용
- LLaMA는 한국어 데이터셋 학습이 부족해서 한국어 성능이 낮음, 한국어 모델을 추가로 학습
- LLaMA의 52k 명령어 데이터 셋은 DeepL API로 번역
```
</details>

<details>
<summary>상용✅[Dolly2.0](https://www.databricks.com/blog/2023/04/12/dolly-first-open-commercially-viable-instruction-tuned-llm): </summary>

```
- 세계 최초의 진정한 개방형 Instruction-Tuned LLM
- 전체 훈련 코드, 데이터 셋, 모델 가중치를 모두 공개. 즉 개인/회사 누구든 자신의 강력한 LLM을 생성 및 소유 가능
- 사람이 생성한 명령어 databricks-dolly-15k 데이터셋으로 파인 튜닝
- 15000개의 프롬프트/답변 페어. 누구나 변경/확장 가능하며 상업용도로도 사용 가능
  (Alpaca, Koala, GPT4All, Vicuna 등은 모두 상업용 사용 불가)
- 이 데이터는 5천명의 databricks 직원들이 직접 작성한 것
- EleutherAI pythia 12B 파라미터 언어 모델 기반
```
</details>

<details>
<summary>상용✅[Cerebras-GPT](https://www.cerebras.net/blog/cerebras-gpt-a-family-of-open-compute-efficient-large-language-models/): The Pile(800GB, English)</summary>

```
- OpenAI GPT-3을 기반으로 deepmind가 2022.3월 공개
- Chinchilla 방식으로 학습
- 학습 시간이 짧고, 비용이 낮고, 소비 전력이 적은 것이 특징
- 모델, 가중치, 체크포인트 모두 Apache 2.0 라이선스로 공개
- 111m, 256m, 590m, 1.3B, 2.7B, 6.7B, 13B 공개
- 타 모델 대비 높은 효율을 보인다고 주장
```
</details>

<details>
<summary>상용❌[Koala](https://bair.berkeley.edu/blog/2023/04/03/koala/): 학술 연구를 위한 대화형 모델</summary>

```
- Alpaca와 비슷하게 LLaMA를 대화 및 명령 셋으로 훈련한 Koala-13B 모델
- 다양한 쿼리에 대해 Alpaca 보다 선호되는 결과를 생성하며, 적어도 절반 이상의 경우에 ChatGPT와 동일한 응답 생성 가능
- 웹에서 수집한 오픈소스 대화 데이터로 Supervised Fine-Tuning
  - ShareGPT의 60K 대화
  - HC3의 87K 질답 예제
  - Open Instruction Generalist (OIG)
  - Stanford Alpaca 가 공개한 52K 데이터셋
  - Anthropic HH (160K)
  - OpenAI WebGPT (20K))
  - OpenAI Summarization (93K)
```
</details>


<details>
<summary>상용❌[Koala](https://bair.berkeley.edu/blog/2023/04/03/koala/): 학술 연구를 위한 대화형 모델</summary>

```
- Alpaca와 비슷하게 LLaMA를 대화 및 명령 셋으로 훈련한 Koala-13B 모델
- 다양한 쿼리에 대해 Alpaca 보다 선호되는 결과를 생성하며, 적어도 절반 이상의 경우에 ChatGPT와 동일한 응답 생성 가능
- 웹에서 수집한 오픈소스 대화 데이터로 Supervised Fine-Tuning
  - ShareGPT의 60K 대화
  - HC3의 87K 질답 예제
  - Open Instruction Generalist (OIG)
  - Stanford Alpaca 가 공개한 52K 데이터셋
  - Anthropic HH (160K)
  - OpenAI WebGPT (20K))
  - OpenAI Summarization (93K)
```
</details>


<details>
<summary>[SantaCoder](https://huggingface.co/bigcode/santacoder): 1.1B 코드 생성 모델</summary>

```
- Python, Java, Javascript 코드로 학습한 멀티언어 랭귀지 모델
- LTR 생성 및 Infilling에서 페이스북의 InCoder(6.7B) / 세일즈포스의 CodeGen-Multi (2.7B) 보다는 뛰어나다고
- BigCode가 공개했던 The-Stack v1.1(6TB) 데이터셋의 일부를 사용
```
</details>





<br>
<br>

## 🎢[Related Works]

<details>
<summary>[Simple LLaMA Finetuner](https://github.com/lxe/simple-llm-finetuner): LoRA 학습 UI 도구</summary>

```
- 초보자도 쉽게 LLaMA-7B를 파인 튜닝 할수 있게 해주는 간단하고 직관적인 UI 도구
  - 데이터셋 관리, 파라미터 커스터마이징, 모델을 학습하고, 추론 기능을 평가
- PEFT 라이브러로 LoRA를 이용, 일반 NVIDIA GPU에서도 실행 가능
- Linux 또는 WSL, 16GB 이상의 램을 가진 최신 NVIDIA GPU 필요
```
</details>

<details>
<summary>[RedPajama](https://www.together.xyz/blog/redpajama): LLaMA 데이터셋을 재작성하는 오픈소스 프로젝트</summary>

```
- LLaMA, Alpaca, Vicuna 같은 반개방형 모델이 아니라 재현가능하고 완전한 개방형 언어 모델을 만들기 위한 프로젝트
- 3가지 구성요소
  - 높은 품질과 넓은 커버리지를 가진 Pre-Training 데이터
  - 이 데이터 기반으로 대규모로 학습된 베이스 모델
  - 베이스 보델은 안전하고 사용가능하게 만들기 위한 인스트럭션 튜닝 데이터와 모델
- 첫번째 컴포넌트로 RedPajama-Data-1T 데이터셋을 공개
  - LLaMA 논문에 설명된 레시피에 따라서 생성한 1.2조개의 토큰으로 구성된 완전 개방형 데이터 셋
  - HuggingFace를 통해 다운로드 가능. 전체 5TB(3TB로 압축하여 배포)
  - 7개의 데이터 조각으로 구성 : 각각 전처리와 필터링하여 LLaMA 논문과 비슷한 갯수로 구성(전처리 방법 및 필터 역시 GitHub에 공개)
    - CommonCrawl (878b) - 웹 크롤링 데이터
    - C4 (175b) - Colossal, Cleaned version of Common Crawl
    - GitHub (59b) - 라이센스와 품질로 필터링된 GitHub의 데이터
    - arXiv (28b) - 과학 논문과 기사들 (boilerplate 제거)
    - Books (26b) - 콘텐츠 유사성에 따라서 중복을 제거한 공개 서적 Corpus
    - Wikipedia (24b) - 위키피디어의 일부 페이지들 (boilerplate 제거)
    - StackExchange (20b) - 스택익스체인지의 일부 페이지들 (boilerplate 제거)
- 다음 단계는 강력한 베이스모델을 훈련하는 것. 몇주내로 공개 예정
- 명령어 튜닝은 OpenChatkit을 통해서 제공된 것으로 할 예정
```
</details>

<details>
<summary>[LLaMA Academy](https://github.com/danielgross/LlamaAcademy): GPT에게 코딩하는 법 가르치기</summary>

```
- LLaMA, LoRA, LangChain을 활용하여 GPT에게 API 문서를 읽는 법을 학습시키기
- OpenAI Api 활용(GPT 3.5 Turbo, GPT4)
- API 문서에 대해서 LlamaAcademy를 실행하면 새로운 LLaMA 모델이 생성
- 해당 모델을 서버에서 호스팅하면, 사용자들에게 해당 API에 대한 호출 코드를 작성해주는 게 가능
```
![image](https://user-images.githubusercontent.com/36894403/234182487-d4ef1215-0eb0-49b0-b1f4-28fc2da8558b.png)

</details>

<details>
<summary>[Xturing](https://github.com/stochasticai/xturing): 데이터전처리, LLM 파인튜닝 및 평가 자동화</summary>

> xturing provides fast, efficient and simple fine-tuning of LLMs, such as LLaMA, GPT-J, Galactica, and more. By providing an easy-to-use interface for fine-tuning LLMs to your own data and application, xTuring makes it simple to build, customize and control LLMs. The entire process can be done inside your computer or in your private cloud, ensuring data privacy and security.

```
- LLaMA, GPT-J, GPT-2 같은 LLM을 빠르고, 효율적으로 파인 튜닝하는 오픈소스 도구
- 다양한 소스에서 데이터를 가져와서 LLM들이 이해할 수 있는 포맷으로 전처리
- 빠른 튜닝을 위해 싱글~멀티 GPU 활용
- 메모리 효율적인 기술(LoRA 같은)을 활용하여 비용 절감
- 다양한 파인 튜닝 방법들을 평가헤서 최적의 모델 찾기
- 잘 정의된 메트릭을 통해 파인 튜닝 모델들을 평가
```

</details>

<details>
<summary>[TurboPilot](https://github.com/ravenscroftj/turbopilot): local 에서 셀프호스트 가능한 Copilot 클론모델</summary>

```
- llama.cpp 를 이용해서 6b 파라미터 Salesforce Codegen 모델을 4GiB 램에서 실행
- 미리 변환해둔 모델(2B/6B) 모델 제공 (C/C++/Go/Java/JavaScript/Python으로 사전 학습) 또는 자신이 직접 변환 가능
- TurboPilot 서버를 바이너리로 실행하거나 도커로 실행
- 멀티코어중 몇개의 CPU를 사용할 것인지 지정 가능
- VSCode용 FauxPilot Plugin 이용
```

</details>

<details>
<summary>[Deepspeed Chat](https://github.com/microsoft/DeepSpeedExamples/tree/master/applications/DeepSpeed-Chat): RLHF를 이용한 chatGPT 구현 훈련용 프레임워크</summary>

```
- 빠르고 저렴하며 확장 가능한 개방형 시스템 프레임워크
- End-to-End RLHF(Reinforcement Learning Human Feedback)를 통해 모든 규모의 고품질 ChatGPT 스타일 모델을 생성 가능
- 1클릭으로 48GB 메모리가 장착된 NVIDIA A6000 GPU 한대로 1.3B 파라미터 ChatGPT 모델을 1.36시간내에 훈련, 생성 및 서빙 가능
- Databricks Dolly, CarperAI-TRLX, Huggingface-PEFT 등이 이용중
```

</details>


<details>
<summary>[ColossalChat](https://medium.com/@yangyou_berkeley/colossalchat-an-open-source-solution-for-cloning-chatgpt-with-a-complete-rlhf-pipeline-5edf08fb538b): RLHF를 이용한 chatGPT 구현 훈련용 프레임워크-2</summary>

```
- LLaMA 모델을 기반
  - Supervised 데이터 수집
  - Supervised 파인 튜닝
  - Reward 모델 학습
  - Reinforcement Learning 파인 튜닝
- 포함하는 콘텐츠
  - 온라인에서 실행하는 인터랙티브 데모
  - 7B/13B 모델을 포함하는 완전한 RLHF 훈련코드 오픈소스
  - 중국어/영어로 구성된 104k bilingual 데이터셋
  - 7B모델의 4-bit 양자화. 4GB GPU 메모리만 필요
  - 모델 가중치 포함. 싱글 서버에서 간단히 재생산 가능
  - 대형 모델/데이터셋/최적화 등도 계속 추가 에정
```

</details>



<details>
<summary>[ChatGPT 학습과정 관련]</summary>

```
1) 📄[챗GPT는 어떻게 학습되었을까 - Human Feedback Reinforcement Learning (RLHF)]
   - https://littlefoxdiary.tistory.com/111
2) 🎞[긴급 세미나. 2부 (실전) ChatGPT-replica 만들기 코드실습 (전자통신부설연구소 고우영 선임연구원)]
   - https://www.youtube.com/watch?v=Iq8erq62s8c&ab_channel=AI%ED%94%84%EB%A0%8C%EC%A6%88
```

</details>









<br>
<br>

## 💾[Dataset]

- [ShareGPT52K](https://github.com/zilliztech/GPTCache](https://huggingface.co/datasets/RyokoAI/ShareGPT52K)
 
 
 
 
 
 
 
 
 
<br>
<br>

## 🎬[Production] 

<details>
<summary> [GPT LLM 질의를 캐싱](https://github.com/zilliztech/GPTCache)</summary>
</details>
<details>
<summary> [Alpaca .cpp](Alpaca .cpp) c++ native 구현을 통한 inference 가속 </summary>
</details>
<details>
<summary>[quantization LLaMA: Int8](https://github.com/tloen/llama-int8)</summary>

```
- Meta의 LLaMA-13B를 24 GiB램만으로 돌릴 수 있게 해주는 포크 버전
- 즉, RTX4090/3090 한대만으로 운영이 가능
- 이론상 LLaMA-65B 를 80GB A100 하나로 운영 가능
- 변경 내역
  - 병렬 처리 구조체 제거
  - 호스트 머신의 Weights를 정량화
  - 메모리 문제 방지를 위해 Weights를 점진적으로 로드
  - bitsandbytes 와 tqdm 이용
  - 반복 페널티 설정(기본값 1.15)
  - RTX4090 + 64GB Ubuntu 머신에서 모델 로드하고 정량화 하는데 약 25초 소요
```
</details>






 
