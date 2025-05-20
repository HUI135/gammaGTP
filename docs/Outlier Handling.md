# Outlier Handling

## 🎯 문제상황
동일한 sample(예: 같은 환자 visit) 내에서 **특정 변수 값이 비정상적으로 크거나 작게 측정**되는 경우  
**해석 모형에서 편향을 유발**할 수 있음. 단순 제거는 정보 손실을 초래할 수 있기 때문에  
상황에 맞는 완화(transform, trimming) 또는 정교한 보정 전략이 필요함.

## 🔎 고려사항

- 반복측정 데이터 여부
- 이상값이 단순 입력 오류인지, 의미 있는 극단값인지
- outcome이나 treatment에 영향을 줄 수 있는지 (감도 분석 필요)
- 분포의 비대칭성(왜도), 이상값의 빈도 및 위치

---

## 🧠 주요 기법 정리

| 기법 이름                   | 설명                                                      | 종류                                                  |
|----------------------------|-----------------------------------------------------------|-------------------------------------------------------|
| **Z-score**                | 평균으로부터 표준편차 몇 배 이상 벗어난 값을 탐지                             | 통계 기반 스코어링 (보통 |z| > 3 기준)                  |
| **IQR Rule**               | 사분위수(Q1, Q3)를 기반으로 외부 범위 벗어난 값 탐지                         | Q1 - 1.5×IQR, Q3 + 1.5×IQR 범위 외                    |
| **Winsorizing**            | 상위/하위 극단값을 경계값으로 절단                                   | outlier 제거 없이 완화                                  |
| **Log / Box-Cox Transform**| 분포의 왜도(skew)를 줄이기 위한 변환                                 | 이상값 영향을 줄이고 정규성 개선                        |
| **Robust Scaler**          | 중앙값과 IQR 기준으로 스케일링                                     | 이상값에 강건한 전처리 방식                            |
| **Isolation Forest**       | 트리 기반 비지도 학습으로 이상값 탐지                                 | 고차원 데이터에서도 가능                               |
| **Local Outlier Factor (LOF)** | 밀도 기반 이상치 탐지 기법                                           | 주변 이웃과의 밀도 차이를 기반으로 판단                  |
| **Time-series Smoothing**  | rolling mean, exponential smoothing 등으로 이상 패턴 탐지                | 반복측정 시계열 데이터에서 유용                         |

---

## 📛 이상값 처리 전략 정리

### ✅ 반복측정 데이터가 있는 경우

> 동일 ID 내 여러 시점 존재 → 개체 내부 패턴을 활용한 이상값 탐지 가능

- **방법**:
  - `groupby('id')` 후 Z-score 또는 IQR 기준 이상값 탐지
  - 이동 평균(`rolling mean`) 또는 보간 결과와의 편차 비교
- **처리**:
  - 이상값 → 로그 변환, winsorizing, robust scaling
  - 원시 값을 보존하고 `*_outlier` flag 변수 생성하여 모델 입력

---

### ✅ 반복측정이 없는 경우 (1회성 관측)

> 전체 분포 기반으로 판단

- **방법**:
  - Z-score, IQR, 또는 boxplot 활용 탐지
  - 변수 분포 시각화 (histogram, violinplot 등)
- **처리**:
  - 삭제보다 변환(log, sqrt 등) 또는 절단(Winsorizing)
  - 해석 모델에서는 변환 + flag 조합이 권장됨

---

## 🌲 Outlier Handling Decision Tree

🔍 데이터에 반복측정 구조(동일 ID에 여러 시점)가 있는가?  
│  
├── ✔ Yes (longitudinal)  
│ ├── 시계열 순서 정보가 있는가?  
│ │ ├── ✔ Yes → ✅ rolling mean, z-score by ID  
│ │ └── ✘ No → ⚠ global 기준으로만 판단  
│ └── 이상값이 개체마다 다르게 정의되는가? → ✔ Yes → per ID 기준으로 판단  
│  
└── ✘ No (cross-sectional)  
├── 수치형 변수인가?  
│ ├── ✔ Yes  
│ │ ├── 정규분포 가정 가능 → ✅ z-score  
│ │ ├── 왜도 있음 → ✅ log/Box-Cox transform  
│ │ └── 극단값 많음 → ✅ winsorizing, robust scaling  
│ └── ✘ No (범주형) → 이상치 대신 rare category로 판단  
└── 이상값이 outcome에 영향을 미치는가?  
├── ✔ Yes → 감도 분석 필수 (포함 vs 제외)  
└── ✘ No → 변환만 적용해도 충분  


---

## 📌 실무 팁

- 이상값 제거는 최후 수단. 가능하면 변환(log, sqrt), trimming(winsorizing) 또는 flag 생성이 우선
- 해석 모형에서는 `X_outlier_flag` 같이 **이상 여부를 변수화**하는 것이 더 안전
- 반복측정 데이터에서는 동일 개체 내 정상 범위에서 벗어난 경우만 이상값으로 판단
- 이상값 처리 전후의 분석 결과 비교는 **감도 분석(sensitivity analysis)**으로 필수

---

## 🔗 함께 보면 좋은 자료

- [Outlier 처리 Python 예제 보기](./outlier_handling_example.ipynb)
- [Missing Handling 정리 보기](./missing_handling.md)
- [Missing 보간 Python 예제 보기](./missing_imputation_example.ipynb)
