# DLcompetition

## 프로젝트 개요
본 프로젝트는 고객의 금융 및 신용 관련 정보를 바탕으로 `Credit_Score`를 예측하는 **다중분류 문제**를 다룬다.  
목표 변수는 `Good`, `Poor`, `Standard`의 3개 클래스이며, 표 형식(tabular) 데이터에 적합한 모델을 활용하여 분류 성능을 향상시키는 것을 목표로 하였다.

본 분석은 다음 흐름으로 진행되었다.

1. 데이터 구조 확인 및 전처리  
2. EDA를 통한 변수 해석  
3. TabNet 기반 분류 모델 학습 및 튜닝  
4. Validation/Test 성능 비교 및 최종 모델 선정  

---

## 데이터 설명

| 항목 | 내용 |
|------|------|
| 데이터 파일 | `train2.csv` |
| 목표 변수 | `Credit_Score` |
| 문제 유형 | Multiclass Classification |
| 클래스 | `Good`, `Poor`, `Standard` |

`Credit_Score`는 고객의 신용 상태를 나타내는 범주형 변수이며, 각 고객의 부채, 이자율, 연체 여부, 결제 습관, 월 잔액 등 다양한 금융 관련 정보가 입력 변수로 사용되었다.

---

## 전처리

전처리는 다음과 같은 원칙으로 진행하였다.

| 처리 항목 | 내용 |
|------|------|
| 식별자 제거 | `ID`, `Customer_ID`, `Name`, `SSN` 제거 |
| 변수 유형 구분 | 범주형 / 수치형 변수 분리 |
| 결측치 처리 | 수치형은 중앙값, 범주형은 최빈값으로 대체 |
| 인코딩 | 범주형 변수와 타깃 변수 라벨 인코딩 |
| 데이터 분할 | train / validation / test 분리 |
| 분할 방식 | `stratify` 적용으로 클래스 비율 유지 |

식별자 변수는 예측 일반화에 직접적인 도움이 적고, 과적합 또는 데이터 누수 가능성이 있어 제거하였다.  
또한 결측치 처리와 범주형 변수 인코딩을 수행한 뒤, 데이터 누수를 방지하기 위해 학습/검증/평가 데이터를 분리하여 실험을 진행하였다.

---

## EDA 요약

EDA는 단순 시각화보다 **신용등급과 관련성이 높은 변수 해석**에 초점을 두고 수행하였다.

### 주요 해석
- `Credit_Score`는 `Standard` 비중이 가장 높아 **약한 클래스 불균형**이 존재하였다.
- `Outstanding_Debt`, `Interest_Rate`, `Delay_from_due_date`, `Num_of_Delayed_Payment`는  
  `Good < Standard < Poor` 순으로 증가하는 경향을 보여, 신용등급을 설명하는 핵심 변수로 해석할 수 있었다.
- `Monthly_Balance`는 `Good`에서 가장 높고 `Poor`에서 가장 낮아, **잔액 여유와 신용상태 간의 양의 관련성**을 확인할 수 있었다.
- `Payment_of_Min_Amount`의 분포를 통해 범주형 변수 역시 신용등급 분류에 유의미한 정보를 제공할 가능성을 확인하였다.
- 상관관계 히트맵을 통해 일부 변수 간 높은 상관관계를 확인하였고, 변수 구조를 이해하는 참고 자료로 활용하였다.

---

## 대표 시각화

### 1. Credit Score 분포
<img width="755" height="502" alt="image" src="https://github.com/user-attachments/assets/ec72f8fa-3b66-4d17-9fc9-724e1258d638" />

`Standard` 클래스 비중이 가장 높아 데이터가 완전한 균형 구조는 아님을 확인할 수 있다.  
따라서 단순 정확도뿐 아니라 클래스별 균형 성능도 함께 고려할 필요가 있었다.

### 2. Outstanding Debt Boxplot
<img width="726" height="477" alt="image" src="https://github.com/user-attachments/assets/a871f592-1d15-4856-9b60-dd02a93636f8" />

`Outstanding_Debt`는 `Good`, `Standard`, `Poor` 순으로 분포가 증가하는 경향을 보여  
부채 규모가 신용등급과 밀접한 관련이 있음을 시사한다.

### 3. Correlation Heatmap
<img width="730" height="572" alt="image" src="https://github.com/user-attachments/assets/38eb160f-b529-429b-b62f-0c6128540a5b" />

수치형 변수 간 상관관계를 시각적으로 확인하여 변수 구조를 파악하고,  
신용등급과 관련된 주요 변수 해석의 참고 자료로 활용하였다.

### 4. Payment of Min Amount Ratio
<img width="728" height="472" alt="image" src="https://github.com/user-attachments/assets/52d966e4-8e3f-49e4-8b83-b7bf98dd4f71" />

최소금액 납부 여부에 따라 `Credit_Score` 분포 차이가 나타나며,  
범주형 변수 역시 신용등급 분류에 유의미한 정보를 제공할 수 있음을 보여준다.

---

## 모델 선택

최종 모델은 **TabNet 기반 다중분류 모델**을 사용하였다.

### TabNet을 선택한 이유
- 표 형식 데이터(tabular data)에 적합한 구조
- 범주형 정보와 수치형 정보를 함께 효과적으로 반영 가능
- 변수 간 비선형 관계를 학습할 수 있음
- 기존 기본 신경망(MLP)보다 성능 개선 가능성이 높다고 판단

---

## 하이퍼파라미터 튜닝

여러 하이퍼파라미터 조합을 비교하였고,  
**Validation Accuracy**를 우선 기준으로 최종 설정을 선택하였다.  
동일한 accuracy일 경우에는 **Macro F1**를 함께 고려하였다.

### 최종 선택된 설정

| 하이퍼파라미터 | 값 |
|------|------|
| `n_d` | 64 |
| `n_a` | 64 |
| `n_steps` | 5 |
| `gamma` | 1.3 |
| `lr` | 0.01 |
| `batch_size` | 512 |
| `virtual_batch_size` | 128 |
| `mask_type` | `sparsemax` |

또한 `max_epochs=50`, `patience=10`으로 설정하여  
early stopping을 적용하고, validation 성능이 더 이상 향상되지 않으면 학습을 종료하도록 하였다.

---

## 최종 성능

| Metric | Score |
|------|------|
| Best Validation Accuracy | **0.7829** |
| Best Validation Macro F1 | **0.7684** |
| Test Accuracy | **0.7803** |
| Test Macro F1 | **0.7658** |

### Classification Report

| Class | Precision | Recall | F1-score |
|------|------:|------:|------:|
| Good | 0.68 | 0.73 | 0.70 |
| Poor | 0.79 | 0.80 | 0.79 |
| Standard | 0.81 | 0.79 | 0.80 |

전체적으로 `Poor`와 `Standard` 클래스는 비교적 안정적으로 분류되었으며,  
`Good` 클래스는 상대적으로 구분이 어려운 범주로 나타났다.

---

## 결과 해석
Confusion Matrix를 보면 `Good` 클래스가 `Standard`로 잘못 분류되는 경우가 비교적 많았다.  
이는 두 클래스 간 경계가 완전히 분리되지 않았음을 시사하며,  
추후에는 클래스 간 경계를 더 잘 반영할 수 있는 추가 feature engineering 또는 하이퍼파라미터 조정이 가능할 것으로 보인다.

그럼에도 불구하고 최종 모델은 **Test Accuracy 0.7803**, **Test Macro F1 0.7658**을 기록하여  
기본 신경망 모델 대비 향상된 성능을 보였다.

---

## 파일 설명

| 파일 | 설명 |
|------|------|
| `컴페티션_이나경.ipynb` | 전체 분석 및 모델링 코드 |

---

## 프로젝트 요약
본 프로젝트는 고객 금융 데이터를 활용한 신용등급 다중분류 문제를 다루었으며,  
EDA를 통해 주요 변수의 해석 가능성을 확인하고, TabNet 모델을 적용하여 성능을 개선하였다.  
최종적으로 Validation과 Test 모두에서 안정적인 성능을 확보하였고,  
표 형식 데이터 분류 문제에 대해 적절한 모델 선택과 튜닝 과정을 수행하였다.
