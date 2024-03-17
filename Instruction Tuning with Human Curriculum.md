Naver

release-date: 2023-10-14
arxiv: https://arxiv.org/pdf/2310.09518.pdf
github: x

[요약]
- 인간 교육 프로그램을 참조하여 주제/인지적 엄격성 등을 보강하여 instruction dataset을 합성 생성
- 커리큘럼 학습 방법인 CORGI(Cognitively rigorous instructions) LLM 학습 방법 제시.
- 인간 교육 프로그램의 영감을 받은 인지적으로 점진적인 훈련이 무작위 훈련보다 상당한 장점을 제공한다는 사실 검증
- ChatGPT를 교사 모델로 활용하고, 인간 교육 방식과 유사한 커리큘럼 학습을 통해 LLaMA2 모델에서 MMLU를 3.06 개선
  

- 신경망 아키텍쳐가 인간 뇌를 모방한다는 것을 고려할 때, 인간 교육과 유사한 학습 과정을 채택하는 것은 논리적으로 합당

[CORGI]
- 학생 교육과정의 지식 전체 범위를 포괄하며 상세한 메타 정보를 포함한 "대학 카탈로그"와 "캠브릿지 IGCSE 교육 과정"을 기본 소스로 활용
  - 45개의 독립적인 주제
  - 교육 단계(중등, 학부, 대학원), 주제(수학, 생물학 등), 과목 및 과정 설명(syllabus)를 포함한 풍부한 메타 정보가 포함된 소스
Step1. Extract Concepts from Educational Curricula
- 각 과목의 과정 설명을 기반으로 중요한 삭술적 개념을 추출하는 과정
- 관리 용어와 일정 등 불필요한 내용을 제거하고 실제 과정 커버리지를 확대하여 구체적인 설명과 교과서와 같은 형태로 변환하기 위한 전용 프롬프트 사용
  - https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdG3J7U%2FbtsAJgGBMh6%2FFSjUJq0785cwjQi6QPPcYK%2Fimg.png
- 이후 데이터의 다양성/구별성 확보를 위해 all-MiniLM-L12-v2 모델을 활용하여 임베딩 후, cosine simliary 0.67을 기준으로 시맨틱 중복 제거 수행.
- 결과적으로 45개 주제에서 1.8K개의 코스(과목)의 5.6K 분량의 fine-grained된 개념 확보

- 
