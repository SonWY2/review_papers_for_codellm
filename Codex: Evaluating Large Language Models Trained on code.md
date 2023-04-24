# Codex: Evaluating Large Language Models Trained on code

## 요약
- 새로운 평가 세트인 HumanEval 제공
-- 164개의 샘플 Python 함수 데이터
-- 12B Codex 
- 올바른 솔루션을 생성하기 위해 샘플링을 반복하는 방법론 제시
-- pass@k 메트릭, 
-- Codex-s의 경우 100개의 샘플 생성 시 HumanEval 77.5% 정확도.
-- mean log-probability가 가장 높은 샘플링이 가장 정확할 가능성이 높으므로 이를 배포시 활용하는 것이 좋음

## 데이터 수집
2020년 5월에 github에서 호스팅되는 5,400만개의 repository에서 수집.
- 1mb 미만의 고유한 python 파일 179gb 수집.
- 필터링
-- 자동 생성 가능성이 높은 코드
-- 평균 줄 길이가 100보다 큰 코드
-- 최대 줄 길이가 1000보다 큰 코드
-- 영숫자 문자가 일부 포함된 파일(contained a small percentage of alphanumeric characters.) → 외국어도 함께 포함된 파일을 의미하는 듯
필터 링 후 최종 데이터 세트 크기는 159gb

## 모델
사전학습된 GPT-3에서 fine-tuning 하는 경우데이터세트가 너무 커서 개선효과가 관찰되지 않음.
그러나 fine-tuning된 모델이 더 빠르게 수렴하므로 GPT-3를 fine-tuning 후 사용함.



- Codex: 12B 모델의 GPT 모델을 fine-tuning.
-- HyperParams
```
-- 175 step linear warmup
-- cosine learning rate decay
-- same learning rate with gpt3(175B 모델의 경우 0.00006)
-- Adam optimizer with β1 = 0.9, β2 = 0.95,  = 10−8 , and a weight decay coefficient of 0.1
-- 1,000 억 개 tokens 학습
```
- Codex-S 
-- Docstring에서 함수를 합성하는 작업(humaneval 형태)으로 fine-tuning한 모델
-- 독립형 함수로만 구성. 아닐 경우 성능 저하됨.
-- 프로그래밍 대회 사이트에서 데이터 10,000개 수집. 숨겨진 테스트케이스는 잘못된 솔루션을 의도적으로 제출하여 추출.
-- 오픈 소스 프로젝트에서 프로그래밍 문제 40,000개 수집. sys.setprofile을 활용하여 통합 테스트 중 호출된 모든 함수와 입력 추출. CI툴을 활용하는 github repo를 대상으로 함.
-- 수집된 데이터 중 codex-12b 사용하여 100개의 샘플링을 수행하고 단위 테스트가 통과되지 않을 경우 필터링으로 제외함.
-- codex fine-tuning learning rate의 1/10
-- codex보다 더 좁은 분포를 포착하는 것처럼 보임(k=1 t=0 / k=100 t=1)
-- pass@1 에서 기존 codex보다 6.5% 우수. 

- Codex-D
-- docstring 생성 모델
-- 수작업 채점. 코드 본문을 고유하고 정확하게 지정하는 경우 올바른 docstring으로 간주
-- humaneval에 대해 T=0.8의 codex-D-12B에서 문제당 10개의 샘플만 채점함
-- codex-d의 합격률은 낮지만 codex-s와 비슷. docstring의 품질 때문이라고 추측

## HumanEval: Hand-Written Evaluation Set
- 164개의 프로그래밍 문제 세트(Python)
- 문제당 평균 7.7개의 테스트케이스 존재
- 모델에서 ``'\nclass'``, ``'\ndef'``, ``'\n#'``, ``'\nif'``, ``'\nprint'``  token을 중지 sequence로 사용.

### Evaluation Framework
- 코드 평가에 대해 활용하는 pass@k
![](https://user-images.githubusercontent.com/36894403/233528258-42d2b72a-e967-4cf6-a7a8-f9f3a352e62b.png)
- 기존 사용되던 BLEU 점수는 실제 생성된 함수의 기능적 정확성을 평가하는데 어려움
- Humaneval 데이터에 포함된 test를 통과하는 샘플 개수에 따라 함수의 기능성 평가

## 결과
### 1) 샘플 수 / 온도 변수간의 관계
![](https://user-images.githubusercontent.com/36894403/233838488-67ee2a27-ca94-431a-9bcc-5f79d3ff4377.png)
- 샘플 수 k와 Temperature에 대한 pass@k를 최적화 하는 것이 중요
- k가 클수록 Temperature도 높을 수록 최적(예: k=1 T=0.2 / k=100 T=0.8)
![](https://user-images.githubusercontent.com/36894403/233838583-2aa65ac6-f1a7-417f-b5a5-35f31a444596.png)

### 2) 샘플링 중 최선의 솔루션 선택 방법
![](https://user-images.githubusercontent.com/36894403/233839022-df7007c8-871a-46c6-afa2-61fe7f4d7326.png)
- mean logp가 가장 높은 샘플을 평가하는 것이 효율적. 

### 3) 경쟁모델 비교
- GPT-Neo, GPT-J : 8%의 GitHub 코드가 포함된 데이터 세트인 The Pile(Gao et al., 2020)을 기반으로 학습
- GPT-Neo-2.7B ≒ Codex-85M
- GPT-J ≒ Codex-300M
- ![](https://user-images.githubusercontent.com/36894403/233839155-548d3bc4-a552-4beb-8987-964b06a40b84.png)

### 4) Apps Dataset에 대한 결과
- Apps Dataset
-- 언어 모델의 코딩 과제 능력 측정을 위한 데이터셋
-- train: 5000개 / test: 5000개
-- 각 예제에는 단위 테스트 세트 포함
- 1000개의 솔루션을 샘플링하고 단위 테스트를 통과한 솔루션만 필터링하여 활
- 모두 올바른 솔루션을 찾았지만 효율적이지 않은 경우는 있었음.
- Codex12B가 Apps에 대해 미세조정된 GPT-Neo 모델과 비슷![](https://user-images.githubusercontent.com/36894403/233839818-0eface9f-9cc7-452b-987d-b43b60d498f2.png)

## 한계
- 구문상 잘못되거나 정의되지 않은 코드를 추천하는 경우 有
- 코드베이스 범위를 벗어난 함수, 변수, 속성을 호출하기도 함
- docstring의 연산과 변수 수가 많은 경우 문제 발생
