# Missing Handling

## 🎯 문제상황
동일한 sample(예: 같은 환자 visit) 내에서 특정 조건(예: 검사 안 받음)으로 인해  
**여러 개 변수에서 동시에 결측**이 발생할 수 있음.

## 🔎 고려사항
- **반복측정 데이터**인 경우: 동일 환자/개체의 이전 또는 이후 시점 데이터를 활용할 수 있음
- **반복측정이 없는 경우**: 전체 모집단의 공변량 구조를 이용한 다변량 보정이 필요함

---

## 🧠 주요 기법 정리

| 기법 이름                        | 설명                                         | 종류                                                    |
| ---------------------------- | ------------------------------------------ | ------------------------------------------------------- |
| **Forward Fill (`ffill`)**   | 과거 값을 결측에 복사                               | 단순 누적 대체 (Last Observation Carried Forward)        |
| **Backward Fill (`bfill`)**  | 미래 값을 결측에 복사                               | Next Observation Carried Backward                       |
| **Interpolate()**            | 앞뒤 값을 기준으로 선형 보간                           | 수학적 보간법 (선형, 다항, 시간 기반 등)                          |
| **Rolling / Expanding Mean** | 과거 일정 기간 평균으로 대체                           | 시계열 smoothing + 보정 (이동 평균, 누적 평균 등)              |
| **Simple Imputer**           | 전체 평균, 중앙값, 최빈값 등으로 결측값 대체                | 단순 통계 기반 대체                                        |
| **KNN Imputer**              | 결측이 없는 관측치들과의 거리 기반 유사한 이웃값으로 대체        | 거리 기반 다변량 대체 (K-Nearest Neighbors)                |
| **Iterative Imputer (MICE)** | 다른 변수들로 결측값을 회귀 기반 반복 추정 (다중 대체 가능)       | 다변량 회귀 기반 다중 보간 (MICE: Multiple Imputation by Chained Equations) |
| **Random Forest Imputer**    | 트리 기반 모델로 결측값 예측                             | 비선형 모델 기반 다변량 대체                                |

---

## 📛 결측 Indicator (Flag) 활용 전략

### ✅ 왜 필요한가?

- 결측 자체가 outcome 또는 treatment와 관련될 수 있는 **MNAR 상황**에서는  
  단순 대체만으로는 **편향된 해석**이 발생할 수 있음
- 해석 중심 모델에서는 **결측 자체가 정보**일 수 있으며,  
  이를 모델에 명시적으로 반영하기 위해 **결측 indicator(flag)**를 함께 포함시켜야 함

### 🔧 어떻게 만드는가?

```python
df['X1_missing'] = df['X1'].isnull().astype(int)
df['X1'] = df['X1'].fillna(df['X1'].median())
```

### 🧪 어떻게 활용하는가?

| 적용 대상               | 활용 방식                                                      |
|------------------------|---------------------------------------------------------------|
| **회귀 / 인과 모형**        | `Y ~ T + X1 + X1_missing + ...`                               |
| **Propensity Score Model** | `T ~ X1 + X1_missing + ...`                                   |
| **Doubly Robust Model**    | PS + Outcome 모델 **모두에** flag 포함                         |
| **감도 분석**              | flag 기준 subgroup 분석<br>예: 결측 있는 경우만 별도 분석 수행 |

---

### 🔎 주의사항

- 결측이 **없는 변수에는 flag를 생성하지 않는다**
- **다중 대체(MICE)**를 사용하더라도, **flag 변수는 별도로 모델에 포함**해야 한다

---

## 🌲 Missing Handling Decision Tree

🔍 데이터에 반복측정 구조(동일 ID에 여러 시점)가 있는가?  
│  
├── ✔ Yes (반복측정 있음: longitudinal)  
│ │  
│ ├── 시계열 순서 정보가 있는가? (날짜, 방문일 등)  
│ │ │  
│ │ ├── ✔ Yes  
│ │ │ ├── 결측값이 개체 내 점진적으로 변화하는 변수인가?  
│ │ │ │  
│ │ │ ├── ✔ Yes → ✅ [Interpolate()], [Forward Fill], [Rolling Mean]  
│ │ │ └── ✘ No → ⚠ 보간은 신중! 단순 대체 + 결측 indicator 고려  
│ │ │  
│ │ └── ✘ No  
│ │ └── 🟡 시간 기준 정렬 불가 → groupby 없이 MICE 또는 Simple 대체  
│ │  
│ └── 시점 간격이 불규칙한가? → interpolate(method='time') 고려  
│  
└── ✘ No (반복측정 없음: cross-sectional)  
│  
├── 전체 sample 수가 충분히 많은가? (n > 200?)  
│ │  
│ ├── ✔ Yes  
│ │ ├── 변수 간 상관성이 뚜렷한가?  
│ │ │  
│ │ ├── ✔ Yes → ✅ [Iterative Imputer (MICE)], [KNN Imputer]  
│ │ └── ✘ No → ⚠ [Simple Imputer] + Flag 사용  
│ │  
│ └── ✘ No (소규모 데이터) → ⚠ [Simple Imputer] or [KNN], but 불확실성 높음  
│  
└── 변수의 수가 너무 많고 복잡한가?  
└── ✔ Yes → 🔁 [Random Forest Imputer] or Feature Selection + Impute  


---

## 📌 실무 팁

- 결측 처리는 "단순히 채운다"가 아니라 **결측 원인을 고려**한 전략이 중요함  
- 반복측정이면 ID 내 시간 흐름, cross-sectional이면 공변량 구조 활용  
- 결측 indicator(flag)를 함께 모델에 포함시키면 MNAR 가능성에도 대응 가능  
- 다중 대체(MICE)는 불확실성을 반영할 수 있는 가장 강력한 방법 중 하나임

---

## 🔗 함께 보면 좋은 자료

- [Missing 보간 Python 예제 보기](./missing_imputation_code.ipynb)
- [Outlier Handling 정리 보기](./outlier_handling.md)
- [Outlier 처리 Python 예제 보기](./outlier_handling_code.ipynb)
