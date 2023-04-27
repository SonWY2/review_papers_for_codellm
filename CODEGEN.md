
# CODEGEN: AN OPEN LARGE LANGUAGE MODEL FOR CODE WITH MULTI-TURN PROGRAM SYNTHESIS
## 요약 / contribute

 - 최대 16.1B의 대규모 언어 모델인 Codegen 공개
 - 대규모 모델 병렬 훈련 라이브러리인 JAXFORMER 오픈소스화
 - multi-turn 합성 패러다임 도입 및 프로그래밍 벤치마크(MTPB) 구축 및 공개


## 학습 데이터
  - THEPILE (825.18 GB)
  - BIGQUERY (6가지 프로그래밍 언어: C, C++, Go, Java, Javascript, Python)
  - BIGPYTHON (2010년 10월까지의 Python 데이터)
![](https://user-images.githubusercontent.com/36894403/234484388-e5aa7971-71a3-4190-9291-24eb96398cf8.png)
- 데이터 전처리 (THEPILE 논문에 제시된 방법대로 수행하는 듯)
  - Filtering: 파일 확장자로 파일을 필터링, avg code len < 100, max code len > 1000, code line의 포함된 10진수 / 16진수 등 숫자 > 90% 인 내용 삭제
  - deduplication: SHA-256 hash 기반으로 정확한 중복 제거
  - tokenization: GPT-2 의 BPE tokenizer 기반 tab/space 를 특수 토큰으로 대체, Codegen-Multi의 경우 프로그래밍 언어를 포함하는 prefix 를 추가
  - shuffling: 각 연도의 데이터를 무작위 셔플링 
  - concatenation: 특수 토큰을 구분 기호로 사용하여 2,048 sequence 길이를 충족하기 위해 시퀀스 연결

## 모델
- 350M, 2.7B, 6.1B, 16B 의 다양한 사이즈 모델 생성
- 모델 사이즈에 따라 토큰 수 조절하여 학습
- RoPE(Rotary Position Embedding) 채택. attention과 feed-forward 계산 병렬 수행
- 모델 타입
	- Codegen-NL : THEPILE 데이터로 훈련
	- Codegen-Multi: Codegen-NL로 초기화되고 BIGQUERY로 추가 훈련
	- Codegen-Mono: Codegen-Multi로 초기화되고 BIGPYTHON으로 추가 훈련
![](https://user-images.githubusercontent.com/36894403/234485586-6da4c130-6814-405f-986d-b86bbbaa7fa1.png)
- Google의 TPU-v4 하드웨어에서 효율적으로 훈련하기 위한 JAXFORMER 개발 및 도입하여 훈련
 
 ## 모델 평가
 **1. Single-turn evaluation(HumanEval)**
 
    - Codex 및 학습데이터(THEPILE)와 모델 크기가 유사한 GPT-NEO, GPT-J와 비교
    - t ∈ {0.2, 0.6, 0.8}, k ∈ {1, 10, 100}
    - ![](https://user-images.githubusercontent.com/36894403/234498834-71a566de-5eb4-497b-a705-66a4e332c670.png)
    - Codegen 모델이 뛰어나거나 동등한 성능 발휘
    - multi-mono 성능을 비교해 봄에 따라 데이터 양과 모델 크기가 늘어날 때 성능이 전반적으로 증가하는 것 확인 가능
    - Codex와 비교하였을 때, 동등  또는 다소 우위. Codegen-Mono-6.1B의 경우 codex 12B에 근접하는 성능 발휘.
    - 추가적인 실험으로 pass한 문제와 non-pass한 문제를 perplexity 차원에서 비교해보았을 때, non-pass 문제들이 상대적으로 perplexity가 높음을 확인함
    - ![](https://user-images.githubusercontent.com/36894403/234499721-9c2ab735-d8d4-452b-8d19-e6b802849845.png)

**2. Multi-Turn 평가**

- 새로 구축한 MTPB: Multi-Turn 평가 벤치마크 데이터 셋 활용
- 115개의 문제 세트. 
- 자연어로 점진적으로 목표를 제공하면서 여러 단계에 걸쳐 목표 코드를 완성하는 접근법.
- 길고 복잡한 사양을 여러 단계로 나누면 모델의 이해가 쉬워져 성능이 향상된다고 고려 
- 문제가 3개 이상의 턴으로 분해되고 한 번의 턴으로 문제를 해결 할 수 없도록 구성되어 있음
![](https://user-images.githubusercontent.com/36894403/234481545-931a88a7-e938-482a-869e-20516932dc64.png)
 - 성능은 humaneval과 유사하게 전문가가 작성한 테스트 케이스 통과율로 측정됨
 - 테스트 시 몇 가지 보정 추가
	- 예: 마지막 결과 출력을 위한 Print()문이 누락되었을 시, AST 분석 후 추가 주입, list-tuple 처럼 유사한 타입에 대한 동등성 검사, 부동 소수점 허용 오차 임계값(ε = 1e -6) 추가 등
 - 샘플 데이터 포맷:
 
 ```
{"prompts":  ["Assign the string \"{A}\" to a variable named \"my_string\".", 
       "Lowercase the given string \"my_string\".", 
       "Assign the distinct characters of the string to a variable named \"chars\".", 
       "Sort these characters in alphabetical order.", 
       "Print the resulting list of characters."  ], 
```
 -  결과
	 - 모델 간 pass rate 비교
![](https://user-images.githubusercontent.com/36894403/234732581-6769a453-600d-4d97-8936-fc564c236e23.png)
    - 실제 실험 결과 multi-turn의 prompt를 single-turn으로 변환하여 모델에 테스트한 결과 single-turn의 평균 perplexity가 높게 측정됨. → multi-turn이 모델에서 더 쉽게 이해될 수 있음
![](https://user-images.githubusercontent.com/36894403/234733803-2d08c948-0b93-4cc5-92a4-50bbdd78e122.png)   		- 문제를 easy/medium/hard 로 난이도를 분리해서 single/multi 간 perplexity를 비교한 결과 medium 이상의 어려운 문제에서 특히 multi-turn 접근의 pass rate가 두드러지게 상승함 확인. 모델 크기가 클수록 쉬운 문제는 개선이 없는 것이 특이점.
![](https://user-images.githubusercontent.com/36894403/234735473-0e14f7ec-ab38-4ee5-b336-bfc78c5cd7ac.png)
- 또 다른 특이점은, 큰 모델이 유연하지 않다는 점을 관찰한 것. Codegen-Mono 16.1B 모델이 prompt를 문자 그대로 받아들여 2.7B에 비해 성능이 현저히 떨어지는 사례가 종종 관찰됨.
- ![image](https://user-images.githubusercontent.com/36894403/234757856-858ab5f0-a516-47bd-afe1-f4bf9abec845.png)
==그러나 논문에서 다양한 해당 사례를 공유하지 않았고, 정확한 원인 파악 및 추가연구까지 진행하고 있지는 않음.==

** 3. Addintional Benchmark 결과**

 - Mostly Basic Python Problems (MBPP) 데이터 셋
![](https://user-images.githubusercontent.com/36894403/234744619-165cea83-8782-4bb7-b88c-2c8eb4393aed.png)
