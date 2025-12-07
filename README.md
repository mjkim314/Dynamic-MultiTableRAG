# MultiTableRAG: 토큰 효율적인 멀티 테이블 추론 (Token-Efficient Multi-Table Reasoning)

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Model](https://img.shields.io/badge/Model-Mistral--Nemo-green)](https://huggingface.co/mistralai/Mistral-Nemo-Instruct-2407)
[![Dataset](https://img.shields.io/badge/Dataset-Spider%20%7C%20MMQA-yellow)](https://github.com/taoyds/spider)

> **강화학습(RL) 기반의 동적 토큰 예산 분배를 통한 멀티 테이블 QA 최적화**

## 📖 개요 (Overview)
**MultiTableRAG**는 다중 테이블(Multi-Table) 환경의 질의응답(QA) 시스템에서 발생하는 비효율적인 토큰 사용 문제를 해결하기 위해 설계된 프레임워크입니다.

기존 TableRAG 구조를 확장하여 **강화학습(Reinforcement Learning) 컨트롤러**를 도입했습니다. 이 컨트롤러는 질문의 난이도와 복잡성을 분석해 토큰 예산(Token Budget)을 동적으로 할당함으로써, 불필요한 토큰 낭비를 줄이면서도 실행 정확도(Execution Accuracy)를 극대화합니다.

## 💡 연구 배경 및 문제 정의 (Motivation & Challenges)

### 1. 멀티 테이블 환경에서의 토큰 폭증 (Token Explosion)
여러 테이블을 참조하거나 조인(Join)해야 하는 복잡한 질문을 처리할 때, 참조해야 할 스키마와 데이터의 양이 늘어나면서 LLM에 입력되는 컨텍스트의 길이가 급격히 증가합니다. 이는 연산 비용 증가와 응답 속도 저하로 이어집니다.

### 2. 정적 검색 전략의 비효율성 (Inefficiency of Static Retrieval)
기존 방식은 모든 질문에 대해 고정된 개수(**Top-k**)의 정보만 검색합니다.
* **쉬운 질문:** 필요 이상의 정보를 가져와 토큰을 낭비합니다.
* **어려운 질문:** 정보가 부족하여 정답을 맞히지 못합니다.
* **트레이드오프:** k를 줄이면 정확도가 떨어지고, k를 늘리면 비용이 증가하는 딜레마가 존재합니다.

## 🚀 제안 방법 (Proposed Method)

우리는 질문마다 최적의 컨텍스트 길이($T_{max}$)와 스키마/셀 정보의 비율($\alpha$)이 다르다는 점에 착안하여, 이를 스스로 결정하는 **Dynamic Token Budget Controller**를 제안합니다.

### 모델 구조 (Architecture)
전체 파이프라인은 다음과 같은 단계로 구성됩니다:

1. **질문 분석 및 상태 인코딩 (Question Analyzer & State Encoder):**
   자연어 질문을 분석하여 조인 필요 여부, 집계 함수 사용 등 복잡도(State)를 파악합니다.
2. **RL 예산 컨트롤러 (RL Budget Controller):**
   학습된 에이전트가 현재 질문 상태에 맞춰 최적의 행동(Action)을 결정합니다.
   * **$T_{max}$:** 사용할 총 토큰 예산 (예: 256, 384, 512, 768)
   * **$\alpha_{col}, \alpha_{cell}$:** 스키마(Schema)와 셀(Cell) 데이터 간의 할당 비율
3. **적응형 검색 (Adaptive Retrieval):**
   할당받은 예산 범위 내에서 질문과 가장 연관성이 높은 테이블, 컬럼, 셀 정보를 선별적으로 검색합니다.
4. **SQL 생성 (SQL Generation):**
   최적화된 컨텍스트를 바탕으로 LLM이 최종 SQL 쿼리를 생성합니다.

### 보상 함수 (Reward Function)
정확도와 효율성 사이의 균형을 맞추기 위해, 강화학습 에이전트는 아래의 보상 함수를 통해 학습됩니다.

$$Reward = EX + \beta PM - \mu Tok(q, b; f)$$

* **EX (Execution Accuracy):** 정답 실행 여부 (성공 시 1, 실패 시 0)
* **PM (Partial Match):** 정답 SQL과의 부분 일치율
* **Tok:** LLM이 실제로 사용한 총 토큰 수 (토큰 사용량이 적을수록 보상 증가)

## 👥 팀원 및 참고 자료 (Team & References)

* **Team 8:** 권준호, 김민재, 박영진
* **Base Framework:** [TableRAG: Million-token table understanding with language models (NeurIPS 2024)](https://arxiv.org/abs/2410.04739)
* **Dataset:** [Spider](https://yale-lily.github.io/spider), [MMQA](https://github.com/deepaknlp/MMQA?tab=readme-ov-file)
