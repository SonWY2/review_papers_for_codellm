release-date: 2024-01-04
arxiv: https://arxiv.org/pdf/2401.02415.pdf
github: https://github.com/TencentARC/LLaMA-Pro


# 요약
- block expansion이라는 방법 제시
  - 모델이 가지고 있던 지식을 손상하지 않으면서 새로운 지식(예: domain adaptation)을 효율적이고 효과적으로 학습할 수 있는 아키텍쳐
  - 기존 모델의 weight를 고정하고 identical한 transformer block을 교차방식으로 추가하여 해당 block들만 학습하여 새로운 지식을 parameterize하는 방식.
- LLaMA2-7B 모델에 코드와 수학 domain 지식을 추가학습한 LLaMA Pro-8.3B 제공
- LLaMA2-7B 베이스 모델에서 코드 지식을 futher pretraining한 CodeLLaMA 보다 기존 자연어 지식의 망각없이 더 탁월한 성능 발휘
  ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/a6fde2de-9fc6-48f1-a65c-a33658cf83b2)


# Block Expansion (LLaMA 아키텍쳐를 중심으로 설명)
![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/c1363f56-ecda-4b90-adfb-2d9070b9f40d)

- 기존 모델의 transformer block을 복제하고 추가하여 기존 LLM을 확장하는 방식
- 원래 모델의 각 블록 뒤에 indentity block을 통합(인터리빙-교차하여 추가)하여 확장된 모델이 확장 후에도 동일한 출력을 유지하도록 보장
  ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/427b7287-fe7b-4661-ab29-bce497a5ad83)
- 이 때, Linear-layer를 0으로 초기화하는 방식 활용
- 이전 연구에서 Norm layer의 스케일 파라미터를 0으로 초기화할 것이 제안되었으나, LLaMA의 아키텍쳐에서 사용중인 RMSNorm은 스케일 파라미터를 0으로 초기화 할 경우 출력값이 0이되고, backward 중에 기울기도 0이 되어 학습 X
  - RMSNorm(x')에 대해서 RMSNorm(x') = 0 일 때, 해당 layer의 출력이 항상 0이 되어 layer를 통과하는 데이터가 무의미해짐. 정규화 능력 상실.
  - RMSNorm과 달리 Batch/Layer Norm의 경우는 β가 존재.
- LLaMA 아키텍쳐의 특성상 MHSA(RMSNorm(x)) = 0 및 FFN(RMSNorm(x')) = 0이 보장되면 동일성 얻을 수 있음. 그림을 참고했을 때, bias 항이 없고 잔차 연결만 남기 때문.
- 따라서 위 그림과 같이 초기화할 경우, 입력 x에 대해서 블록을 통과한 뒤에도 출력이 x로 동일하게 보장됨.
- 이러한 아키텍쳐 확장 후 기존 블록은 freezing 한 후에 추가한 block만 미세조정하는 과정을 통해 일반적인 능력을 유지함

* 기존 Adapter 형식과의 차이(개인적인 생각)
  - Adapter는 layer 추가시 identical함을 보장하지 않고 랜덤 초기화 됨.
    - 모델의 출력이 어느 형태로든 간섭받는 형태에서 학습 시작
    - 그러나 해당 방법론은 identical함을 보장하여 입력과 출력의 동일성이 보장된 상태에서 추가된 parameter만 학습 보장
  - Adapter 방법의 규모 확장에는 제한이 있음.
    - 따라서 학습할 새로운 지식 domain의 규모에 따라 제한이 있을 수 밖에 없음.
    - 이러한 구조적 한계에 따라 Adapter는 지식 alignment에 좀 더 타겟팅하고 있음.
    - Transformer block의 경우 그 아키텍쳐의 특성 상 깊이 쌓으면 쌓을수록 지식 보존에 유리하다는 것이 입증된 상황. (150B 까지 모델을 키워도 모델의 성능은 지속적으로 상승함)
    - 이에 따라 새로운 지식을 학습할 block 확장에 제한이 없음.

# 실험
- 코드 및 수학에 집중한 데이터 세트에 domain adaptation 추가 학습 수행
- Pretrain for LLaMA-Pro
  - 데이터:
    - 코드: stack-dedup 중 Python 분할
    - 수학: 55B 크기의 토큰으로 구성된 Proof-pile-2 활용
  - 기본 LLaMA2-7B를 교차방식으로 기존 32개 블록을 40개로 확장.
  - 각 그룹이 4개 블록에서 5개 블록으로 확장되는 8개 블록으로 변환
  - 하이퍼 파라미터: batch size 1024, seq len 4096, warmup-ratio 0.06, learning-rate 2e-4, cosine lr scheduler, bf16, decay 0.1, 1.0 grad clipping, flash-attention
  - H800 16개에서 총 15,900 step까지 2830시간/per GPU 소요.
- Instruction tuning(SFT) for LLaMA-Pro-Instruct
  - 데이터: 5개의 데이터 소스 결합. 총 1M 샘플.
    - ShareGPT, WizardLM Evol, Code Alpaca, MetaMath, SlimOrca(500K의 GPT-4 완성 문제만 통합)
  - 하이퍼파라미터: batch size 128, seq len 4096, warmup-ratio 0.03, learning_rate 2e-5, cosine lr scheduler, bf16
 
# 평가
- 평가 데이터:
  - AI@ Reasoning Challenge
  - HellaSwag
  - MMLU
  - TruhfulQA
  - Winogrande
  - GSM8k
  - HumanEval
  - MBPP
- Pretrain result
![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/f23bfd5e-bb16-48ab-8451-97689eef5adc)
  - 자연어 처리와 코딩 기능의 균형을 효과적으로 유지함
  - LLaMA2-7B의 일반적인 성능을 유지할 뿐만 아니라 평균 성능에선 오히려 상승
  - CodeLLaMA-7B가 코드 능력을 위해 일반 자연어 성능을 희생한것과 비교됨.
![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/5d042624-0506-4acc-896a-4db2a84d209e)
  - 코드 LLM과 동등한 수준의 코드 성능도 동시에 보여줌.
  - 파레토 프론티어에 위치한 LLaMA Pro는 LLaMA2에서 80B의 토큰을 추가하여 finetuning했으며, 코드 작업 평균 성능을 두배 이상 향상시킴.
    * 파레토 프론티어: 여러 목표 간의 최적의 균형을 이룬 최적 지점
  - 대조적으로 CodeLLaMA는 500B의 토큰으로 미세조정됨.

- SFT Result
  - 위 pretrain result 표 참고.
  - LLaMA Pro-Instruct가 각 목적 특화 모델인 WizardCoder 또는 wizardMath보다 더 뛰어난 성능 발휘.
  - LLaMa2-7B-chat의 자연어 성능 대비 13.81%, CodeLLaMA-7B-inst 대비 코드 능력 14.50% 향상 시킴.
  - 일반적인 LLM 학습 파이프라인(pretrain-SFT Tuning)이 해당 방법론에도 적용될 수 있음을 증명
  - Vicua에서 제시된 GPT-4 자동 채점 기능의 MT-Bench 점수를 통한 종합적인 대화 서능에서도 기존 챗봇들을 능가함
    ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/7a9434ce-7e38-44ac-a48d-45dd931d421a)
  - 멀티턴 상호작용 능력 평가를 위한 MINT-Bench 에서도 비슷한 크기의 모델 대비 높은 성능 달성
    ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/5ae35019-3bd9-45f9-a644-da0f2990a960)

# Ablation Study
- TRACE 벤치마크(continual learning 능력 평가를 위해 설계된 벤치마크)를 활용하여 다양한 훈련 전략 평가.
  - 도메인별 작업, 다국어 능력, 코드 생성 및 수학적 추론과 같이 까다로운 작업을 포괄하는 8개의 개별 데이터 셋으로 구성됨
- OP(Overall Performance), BWT(Backward Transfer)로 평가. 이전에 학습한 작업들에 대한 모델의 기억 정도를 평가함.
  ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/3135b11c-b74d-4761-adc5-f51d20a6748e)
  - OP: t번째 작업 후에 이전에 학습한 모든 작업들에 대한 성능 평가.
  - BWT: 새로운 작업을 학습함에 따라 이전 작업들에 대한 성능 변화를 측정. (각 작업을 학습한 직후 성능 - 해당 작업에 대해 최종적으로 가지는 성능)의 평균값. 양수면 개선, 음수면 망각 발생
- LoRA 및 순차적 FineTuning과의 비교
  ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/46b2c686-4093-470b-8df4-059eda301a39)
  - 상대적으로 작업별 적응성이 뛰어남 확인
- 추가되는 블록 수에 대한 성능 및 확장성 평가. 
  ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/89a68fa0-63e6-428d-97b4-33ed3f405a07)
  - 추가되는 블록에 관계없이 loss 지속적 감소 관찰 가능. 모델이 커질 수록 더욱 빠르게 감소
  - 더 큰 모델과 더 많은 데이터에 대해 강력한 확장성을 보인다는 것을 시사함.
  - MoE의 loss는 블록 4개 추가한 정도와 비슷
  ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/80049ebf-8f15-43cb-8729-b1f515c226ee)
  - 추가된 블록 개수에 따라 평가해보았을 때, 모든 사이즈에서 초기 모델의 일반적인 기능을 효과적으로 보존함.
  - 도메인별 작업의 경우는 모델이 커질수록 성능이 향상됨. 단, 8개의 블록을 추한 경우가 최적의 성능 포인트로 보여 해당 방식 채택함.
  - 또한 identity 블록을 어디에 추가하는 지에 대한 실험 결과, 하단의 경우 모델 전체에 오류가 전파되어 성능이 저하되었으며, 상단의 경우 인터리빙한 방식보다는 효과가 낮았음.
- pretrain에 획득한 지식의 양 실험
  ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/b02b9361-03e1-495c-95a8-b7731fe1f17b)
  - 같은 tuning 데이터에 대해서 LLaMA2-7B와 LLaMa-Pro를 학습한 결과 LLaMA-Pro가 더 우수한 결과 발휘
 

  



- 
