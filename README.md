# paper_caputred_images_repo
논문의 이미지를 캡처 및 보관



## [PreTrain]
- [StableLM: Stability AI Language Models](https://news.hada.io/topic?id=9003) https://github.com/Stability-AI/StableLM
<details>
<summary>접기/펼치기</summary>
  ```
+ 3B/7B 모델을 공개, 15B/30B/65B 모델도 공개 예정이고 175B까지 계획중
+ 모델은 CC BY-SA-4.0 라이센스로 출처 표기시 상업적 이용 가능
+ 오픈 데이터셋인 The Pile에 기반했지만 3배 크기인 1.5T 토큰을 가지는 새로운 데이터셋으로 훈련
+ 컨텍스트 길이는 4096 토큰
+ PoC로 Alpaca 프로시져를 따라서 파인튜닝한 StableLM-Tuned-Alpha-7B 모델도 공개
+ 5개의 대화형 데이터셋을 이용 : Stanford's Alpaca, Nomic-AI's gpt4all, RyokoAI's ShareGPT52K datasets, Databricks labs' Dolly, Anthropic's HH
+ 챗봇 데모는 Hugging Face에 공개
  ```
</details>




- [Alpaca](https://simonwillison.net/2023/Mar/13/alpaca/)
--   LLaMA 모델의 큰 약점은 질문-답변을 위한 "명령어-튜닝"이 부족하다는 것
--   OpenAI의 큰 혁신중 하나는 GPT-3에 명령어 튜닝을 추가한 것
--   스탠포드는 여기에 52000개의 훈련 예제를 제공하고 $100 만으로 훈련 가능하게 해줌
--   가장 작은 7B 모델은 이제 라즈베리파이/모바일 폰에서도 도는데, 매우 인상적인 결과를 뽑아내줌
--   하지만, 아직 상업용은 아님(3가지 이유에서 불가능. LLaMA의 라이센스/명령어셋 데이터를 OpenAI 모델에서 만들어냄/안전조치를 설계하지 않음)
--   Alpaca는 52K의 예제와 $100의 비용으로도 7B 모델(4bit 양자화로 4GB로 줄인) 파인 튜닝이 가능하며, 최신 text-davinci-003 과 비슷한 결과를 낼수 있다는 것을 보여줌
 
 
## [Production] 
- [GPT LLM 질의를 캐싱](https://github.com/zilliztech/GPTCache)
- [Alpaca .cpp](Alpaca .cpp) c++ native 구현을 통한 inference 가속 
