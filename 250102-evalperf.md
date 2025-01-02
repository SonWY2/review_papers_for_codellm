release-date: 2024-08-12
arxiv: https://arxiv.org/pdf/2408.06450
github: https://github.com/evalplus/evalplus/blob/master/docs/evalperf.md

---

1. 개요
- 코드 생성의 효율성 평가라는 새로운 차원 제시
- DPE 프레임워크 제안 (metric: DPS)
- 121개의 성능 테스트 코딩 과제를 포함한 EVALPERF 벤치마크 제작

2. 배경
- LLM이 코드 생성에 널리 사용되면서 이들을 종합적으로 평가하는 것이 중요해짐
- 기존 연구들은 주로 코드의 기능적 정확성(functional correctness) 평가에 집중
- 코드 효율성도 고품질 소프트웨어 개발에 매우 중요한 요소
- Program-aided Language Models의 실행 병목 현상 해결을 위해서도 효율적인 코드 생성이 필요

2. 기존 코드 효율성 평가의 한계점
(1) 경량 연산 (Light computation)
- 기존 코딩 과제들은 작은 테스트 입력과 단순한 제어 흐름만 사용
- 경량 연산은 시스템 노이즈에 민감하여 결과의 신뢰성이 떨어짐
- 작은 규모에서는 모든 복잡도가 비슷하여 효율성 차이를 구분하기 어려움

(2) 부적절한 메트릭 (Inadequate metric) 
- Runtime speedup이 표준 메트릭으로 사용됨
- 단일 최적화 대상에는 적합하나, 여러 작업에 대한 평균 speedup은 해석이 모호함
- 예: 모델 A가 99개 작업에서 2배 느리고 1개 작업에서 100배 빠른 경우, 평균 speedup은 1.495배로 사용자 인식과 불일치

3. 본 논문 방법론: Differential Performance Evaluation (DPE)
- 성능 평가를 위한 프로그래밍 과제를 선별하고 효과적인 코드 효율성 평가 수행
- 성능 평가 메트릭: DPS(Differential Performance Score) 제시
  - 0%~100% 사이의 점수
  - 높은 점수 = 더 효율적인 코드
  - 상대적 성능 순위 기반 평가
- 테스트 데이터셋 구성 방법
  - 1) 초기 데이터 수집 (seed data: HumanEval과 MBPP 총 563개)
  - 2) 솔루션 수집
    - pass@1 점수 50 이상 21개의 오픈소스 LLM 활용하여 각 모델당 50개의 솔루션 생성
  - 3) 입력 생성기(SAS)
  - 4) 과제 필터링을 통해 121개 선정
- 성능 측정
  - 1) 정확성 검증
  - 2) 성능 프로파일링 (PMU 카운터)
    - PMU: assembly 로 변환하여 명령어(instruction) 수를 직접 카운팅
  - 3) 참조 솔루션 비교
  - 4) DPS 계산 (4개 클러스터)
    


