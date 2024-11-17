release-date: 2024-10-05
arxiv: https://arxiv.org/abs/2410.01215
github: https://github.com/YerbaPage/MGDebugger


[개요]
![image](https://github.com/user-attachments/assets/1631c4a1-9f0f-4c43-8d71-4b2a7ecff20b)
** MGDebugger system architecture **
- LLM 코드 생성의 pass rate가 미묘한 오류들로 인해 제한되는 문제를 해결하기 위한 계층적 디버깅 방법론 제안
- 기존 LLM 기반 디버깅 시스템들의 한계
  - 생성된 프로그램을 단일 단위로 취급
  - 다양한 수준의 오류(구문 오류부터 알고리즘 결함까지)를 적절히 처리하지 못함
- Multi-Granularity Debugger (MGDebugger)
  - 문제가 있는 코드를 서브함수들의 계층적 트리 구조로 분해
  - 각 레벨이 특정 수준의 오류를 담당
  - 각 서브함수를 독립적으로 분석하고 bottom-up 방식으로 버그를 해결

[방법론]
■ MGDebugger의 주요 구성요소

1) Hierarchical Code Decomposition
- 입력받은 LLM 생성 함수를 서브함수들의 트리 구조로 분해
- 분해 원칙:
  - 각 서브함수는 특정 목적을 가진 최소 재사용 가능 단위
  - 상위 레벨 함수가 하위 레벨 함수를 호출하여 복잡한 기능 구현
  - 전체 구조가 독립적 테스팅과 디버깅을 용이하게 함

2) Test Case Generation for Subfunctions
- 각 서브함수 fi에 대해 테스트 케이스 Ti 생성
- 메인 함수의 공개 테스트 케이스 Tpub를 활용하여 서브함수 테스트 케이스 도출
- LLM을 활용한 테스트 케이스 생성 과정:
  - 서브함수가 메인 함수 내에서 어떻게 사용되는지 분석
  - 공개 테스트 케이스의 입출력을 분석하여 서브함수의 예상 입출력 도출

3) Debugging Subfunctions with LLM-Simulated Execution
- LLM 시뮬레이션 기반 코드 실행기 도입:
  - LLM이 Python 인터프리터 역할을 수행
  - 코드 실행 과정을 시뮬레이션하고 핵심 변수들의 상태 추적
  - 실패한 테스트 케이스에 대한 상세 분석 수행

4) Bottom-up Debugging 
- 전체 디버깅 워크플로우:
  - 계층 구조를 깊이 우선 탐색으로 순회
  - 각 서브함수를 재귀적으로 디버깅
  - 수정된 내용을 상위 함수들에 전파
- 가장 세부적인 레벨부터 시작하여 상위 레벨로 진행하는 체계적 접근

[실험 설정]
■ 모델
- 코드 생성과 디버깅을 위한 3가지 LLM 사용:
  - CodeQwen1.5 (7B)
  - DeepSeek-Coder-V2-Lite (16B)
  - Codestral (22B)

■ 데이터셋
- HumanEval: 164개 문제
- MBPP: 500개 문제  
- HumanEvalFix: 6가지 버그 유형을 포함한 164개 버그 함수

■ 평가 지표
- Accuracy: 디버깅 후 모든 private 테스트를 통과한 코드 비율
- Repair Success Rate (RSR): 수정된 코드 샘플 비율

■ 베이스라인
8가지 최신 디버깅 방법과 비교:
- Simple Feedback: 코드가 틀렸다는 정보만 제공
- Self-Edit: 실패한 테스트케이스의 실행결과를 llm에 제공
- Self-Debugging (2가지 변형)
  - explanation: 코드를 라인별로 설명하면서 디버깅
  - trace: 코드를 직접 실행하면서 변수 상태를 추적하고 문제를 식
- LDB: 코드를 블록/함수/라인 단위로 분할하여 각 블록 실행 후 변수 값을 추적하며 디버깅
- Reflexion: 코드 실행 결과에 대한 "반성"과 메모리 버퍼를 활용하여 점차적으로 개선하는 방

[실험 결과]
1) 주요 결과
- 모든 모델과 데이터셋에서 기존 방법들 대비 우수한 성능:
  - HumanEval: accuracy 15.3~18.9% 향상
  - MBPP: accuracy 11.4~13.4% 향상
  - HumanEvalFix: 97.6%의 높은 repair 성공률
- DeepSeek-Coder-V2-Lite와 Codestral에서 HumanEval 94.5% accuracy 달성
![image](https://github.com/user-attachments/assets/6076a14e-63ab-41b7-a4e0-fb9e4509f93d)
** Humaneval, MBPP에 대한 디버깅 방법들의 결과 **

2) Ablation Study 결과
- 계층적 디버깅 전략이 가장 큰 영향:
  - 제거 시 HumanEval RSR 76.3%→52.6% 하락
  - MBPP RSR 39.0%→33.5% 하락
- LLM 시뮬레이션 실행과 테스트 케이스 생성도 중요한 역할
![image](https://github.com/user-attachments/assets/670b8678-3674-493f-8879-03d341a9780c)
**ablation study 결과**

3) 버그 유형별 성능
- HumanEvalFix의 6가지 버그 유형에서 일관되게 우수한 성능
- DeepSeek-Coder 사용 시 value misuse를 제외한 모든 카테고리에서 100% 성공률
- 특히 missing logic과 excess logic 같은 하위 레벨 버그 처리에 강점
![image](https://github.com/user-attachments/assets/16033636-847d-422b-a338-0b55df470e90)
**각 bug 유형에 대한 디버깅 방법별 성능 비교**
  
4) 코드 길이에 따른 분석
- 코드 길이가 증가함에 따라 대부분의 방법들의 성능이 저하
- MGDebugger는 긴 코드에서도 일관된 성능 유지
![image](https://github.com/user-attachments/assets/7407edd6-27a7-46a8-b55c-b66196381cad)
**code 길이변화에 따른 수정 성공률 변화**

5) 디버깅 시도 횟수의 영향
- 반복적 디버깅을 통한 지속적인 성능 향상
- 다른 방법들은 초기 몇 번의 시도 후 성능 정체
- ![image](https://github.com/user-attachments/assets/ecf48938-4cff-42d5-8004-6623a14bc96d)
**debug 시도별 누적 성공률**

[의의]
- 계층적 분해를 통한 체계적 디버깅 방법론 제시
- 다양한 수준의 버그를 효과적으로 처리할 수 있는 프레임워크 제공
- 코드 길이나 복잡도에 관계없이 일관된 성능 유지
