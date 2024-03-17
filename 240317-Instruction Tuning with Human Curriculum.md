Naver

release-date: 2023-10-14
arxiv: https://arxiv.org/pdf/2310.09518.pdf
github: x

[요약]
- 그림 1 ![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/859084e3-2e28-4d1b-b77f-729e61eabfcb)
- 인간 교육 프로그램을 참조하여 주제/인지적 엄격성 등을 보강하여 instruction dataset을 합성 생성
- 커리큘럼 학습 방법인 CORGI(Cognitively rigorous instructions) LLM 학습 방법 제시.
- 인간 교육 프로그램의 영감을 받은 인지적으로 점진적인 훈련이 무작위 훈련보다 상당한 장점을 제공한다는 사실 검증
- ChatGPT를 교사 모델로 활용하고, 인간 교육 방식과 유사한 커리큘럼 학습을 통해 LLaMA2 모델에서 MMLU를 3.06 개선
  

- 신경망 아키텍쳐가 인간 뇌를 모방한다는 것을 고려할 때, 인간 교육과 유사한 학습 과정을 채택하는 것은 논리적으로 합당

[CORGI 데이터셋 구성 방식]
- 학생 교육과정의 지식 전체 범위를 포괄하며 상세한 메타 정보를 포함한 "대학 카탈로그"와 "캠브릿지 IGCSE 교육 과정"을 기본 소스로 활용
  - 45개의 독립적인 주제
  - 교육 단계(중등, 학부, 대학원), 주제(수학, 생물학 등), 과목 및 과정 설명(syllabus)를 포함한 풍부한 메타 정보가 포함된 소스
 
![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/e105843c-0a94-4fec-8da6-e80216ac9f32)
Step1. Extract Concepts from Educational Curricula
- 각 과목의 과정 설명을 기반으로 중요한 삭술적 개념을 추출하는 과정
- 관리 용어와 일정 등 불필요한 내용을 제거하고 실제 과정 커버리지를 확대하여 구체적인 설명과 교과서와 같은 형태로 변환하기 위한 전용 프롬프트 사용
  - https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdG3J7U%2FbtsAJgGBMh6%2FFSjUJq0785cwjQi6QPPcYK%2Fimg.png
- 이후 데이터의 다양성/구별성 확보를 위해 all-MiniLM-L12-v2 모델을 활용하여 임베딩 후, cosine simliary 0.67을 기준으로 시맨틱 중복 제거 수행.
- 결과적으로 45개 주제에서 1.8K개의 코스(과목)의 5.6K 분량의 fine-grained된 개념 확보

Step2. Generate Synthetic Instructions
- Bloom's texonomy를 기반으로 실제 instruction 데이터 생성
  - *Bloom: 교육자들에게 중요한 가이드 역할을 제공하는 체계적인 교육 학습 분류 체계
  - 피라미드로 시각화 될 수 있는 여섯 가지 인지 프로세스 계층을 구분함
  - 하위 계층은 간단한 사고(remember, understand 및 apply)로 구성되며, 상위 게층은 보다 복잡한 인지 프로세스(분석, 평가, 창조 등)으로 구성됨
  - 학습자가 정보를 수집하고 사용하며, 원래 지식을 창출하는 방법을 배우도록 보장
- 교사 모델인 ChatGPT에게 각 인지 수준의 자세한 object를 전달하여 단일 개념에 대한 다양한 데이터를 생성.
  - 'remember', 'understand', 'apply' 3개 계층만 활용. 더 상위 계층은 교사 모델에서 편향되거나 너무 주관적인 내용을 포함할 수 있는 우려가 있기 때문.
  - 사전 정의된 19개의 템플릿 활용하며 무작위 시스템 템플릿 삽입
  - TODO 템플릿 삽입
  - 5.6K 분량의 개념을 seed로 활용하여 총 107K개의 데이터셋 생성

Step3. Filter Unclear Instructions
  - 해당 방법은 교사 모델에 크게 의존함. 즉 Q-A 쌍에서의 불일치가 발생할 수 있고 이는 학생 모델의 성능을 크게 저하시킬 우려가 있음.
  - 타사 도구인 Contriever를 사용하여 품질이 낮은 데이터 필터링 수행함.
    - 각 데이터 인스턴스에 대해 Wikipedia에서 관련된 내용(문단)을 256개 단어 수준으로 수집 후 확인 프롬프트를 사용하여 데이터 인스턴스와 수집된 문단 사이의 관련성을 평가하고 충족되는 것만 데이터에 포함
    - 추가적으로 'As an AI...' 와 같은 특정 텍스트 시퀀스가 포함된 거부 데이터 휴리스틱하게 제거
  
[Curriculum Instruction Tuning]
![image](https://github.com/SonWY2/paper_caputred_images_repo/assets/36894403/00d66406-c222-43d3-8539-606c2ffc5dac)
- 두 개의 학습 전략을 비교함. Blocking vs Interleaving
- 초점은 인터리빙 방식에 맞춤. 많은 연구들이 서로 다른 주제를 섞는 방식인 인터리빙 학습이 기존 지식과 새로운 지식을 통합하는데 도움이 된다고 제안.
