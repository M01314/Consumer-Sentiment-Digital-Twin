# Consumer Sentiment Digital Twin

## AI Agent 기반 소비자동향조사 시뮬레이션 시스템 (Hybrid Architecture)

---

## 1. 프로젝트 목적

경제지표 및 뉴스 데이터를 입력으로 받아
다수의 가상 Agent가 소비자동향조사 응답을 생성하도록 설계하고,

이를 실제 조사 결과와 비교·보정하여
미래 소비심리지표(기대인플레이션, 주택가격전망 등)를 사전 예측하는 시스템을 구축한다.

---

## 2. 설계 철학

본 프로젝트는 다음 원칙을 따른다:

1. **텍스트 해석과 의사결정을 분리한다.**
2. LLM은 “텍스트 → 구조화된 신호(feature)” 변환에만 사용한다.
3. Agent의 기대 형성과 응답 생성은 **파라메트릭·구조화된 함수 모델**로 수행한다.
4. 분포 기반 Loss로 실제 조사 응답 구조를 직접 보정한다.

즉,

> LLM은 인지(해석) 모듈
> Agent는 결정(행동) 모듈

---

## 3. 전체 시스템 구조

```
뉴스 텍스트
    ↓
LLM 기반 구조화 모듈
    ↓
정량 Feature 벡터 z_t
    ↓
경제지표 x_t 와 결합
    ↓
Agent 의사결정 함수 f_i
    ↓
개별 응답 생성 y_i,t
    ↓
집계 → 지표 산출
    ↓
실제 조사값과 비교 → Calibration
```

---

## 4. 입력 데이터

### 4.1 경제 데이터 (Structured)

* CPI
* 금리
* 실업률
* 주택가격지수
* 유가
* 소득지표 등

→ 벡터 ( x_t )

---

### 4.2 뉴스 데이터 (Unstructured → Structured)

LLM을 활용하여 다음과 같은 구조화 feature 생성:

예시 출력 벡터:

```
z_t = [
    inflation_risk_score,
    housing_heat_score,
    labor_market_uncertainty,
    financial_instability_score,
    policy_uncertainty_score
]
```

※ LLM은 감성 점수 및 토픽 강도를 수치화하는 역할만 수행
※ Agent는 텍스트를 직접 읽지 않음

---

## 5. Agent 설계

### 5.1 Agent 구성 요소

각 Agent i는 다음 파라미터를 가진다:

* α_i : 기억 지속성 계수
* β_i : 경제지표 민감도 벡터
* γ_i : 뉴스 민감도 벡터
* θ_i : 위험 성향/편향 파라미터

---

### 5.2 기대 형성 모델

잠재 기대 상태 ( s_{i,t} ) 정의:

[
s_{i,t} = \alpha_i s_{i,t-1} + \beta_i^\top x_t + \gamma_i^\top z_t + \epsilon_{i,t}
]

* ( x_t ) : 경제지표
* ( z_t ) : 뉴스 구조화 feature
* ( \epsilon_{i,t} ) : 확률적 오차

---

### 5.3 응답 생성 모델

#### (A) 범주형 문항 (상승/보합/하락)

[
P(y_{i,t}=k) = \text{softmax}(W_k s_{i,t})
]

---

#### (B) 순서형 문항 (매우 나쁨 ~ 매우 좋음)

Ordered logit / probit 구조 사용 가능

---

## 6. Aggregation (집계)

[
Y_{sim,t} = \frac{1}{N} \sum_{i=1}^{N} y_{i,t}
]

또는 항목별 응답 분포:

[
\hat p_t(k) = \frac{1}{N} \sum_{i=1}^{N} \mathbf{1}(y_{i,t}=k)
]

---

## 7. Calibration (보정)

### 7.1 Loss 함수

#### (1) 분포 기반 Loss (권장)

* Jensen-Shannon Divergence
* Cross-Entropy

[
\mathcal{L}_{dist} = JS(p_t | \hat p_t)
]

---

#### (2) 지표 기반 Loss

[
\mathcal{L}*{index} = (Y*{sim,t} - Y_{real,t})^2
]

---

#### (3) 결합 Loss

[
\mathcal{L} = \lambda_1 \mathcal{L}*{index} + \lambda_2 \mathcal{L}*{dist}
]

---

## 8. 학습 루프

1. 과거 경제 + 뉴스 입력
2. Agent 응답 생성
3. 집계
4. 실제 조사값과 비교
5. Loss 계산
6. Agent 파라미터 업데이트

---

## 9. 예측 단계

현재 시점의:

* 경제지표
* 뉴스 구조화 feature

를 입력하여:

* 기대인플레이션
* 주택가격전망
* 소비지출전망

등을 사전 예측

---

## 10. 아키텍처 선택 이유

### LLM을 의사결정에 직접 사용하지 않는 이유

* 재현성 문제
* 캘리브레이션 불명확성
* 정책 활용 시 설명 가능성 부족

### 하이브리드 구조의 장점

* 텍스트 이해력 확보
* 의사결정 구조의 안정성 확보
* 파라미터 해석 가능
* 백테스트 용이

---

## 11. 장기 목표

> 소비자 기대의 “구조적 디지털 트윈” 구축

경제 상황을 입력하면
집단 심리 반응을 확률적으로 시뮬레이션하는 시스템 완성

---

## 12. 디렉토리 구조 (예시)

```
/data
    economic/
    news/
    survey/

/nlp
    llm_feature_extractor.py

/agents
    agent_model.py
    expectation_update.py
    response_model.py

/simulation
    run_simulation.py
    aggregate.py

/calibration
    loss.py
    optimize.py

/validation
    backtest.py
    event_test.py
```

---

# 현재 상태

개념 설계 및 하이브리드 구조 확정 단계

---

# 요약 (핵심 구조 정리)

✔ 뉴스는 LLM으로 구조화
✔ Agent 의사결정은 롤 기반 파라메트릭 함수
✔ 분포 기반 Loss로 실제 조사와 보정
✔ 집계 후 지표 예측

---

## 질문에 대한 확인

당신이 정리한 이해는 정확하다:

> 사전 데이터를 LLM으로 구조화(토픽·리스크 점수화)
> 이후 각 Agent는 정의된 구조화 함수로 의사결정
> 그 결과를 집계하여 지표를 생성

이 접근이 **정책·연구·백테스트 관점에서 가장 안정적인 구조**다.
