---
title: "[혼자 공부하는 머신러닝+딥러닝] 5장. 트리 알고리즘"
date: 2026-10-07 21:00:00 +0900
categories: [ECC, Team-Study-MLDL, Self_Study_ML_DL]
tags: [dev, study, ml, dl, python]
---

["혼자 공부하는 머신러닝+딥러닝(개정판)" 도서 바로가기](https://www.hanbit.co.kr/store/books/look.php?p_code=B7077594897)

예제 코드 (저자 제공): [05-1](https://colab.research.google.com/github/rickiepark/hg-mldl2/blob/main/05-1.ipynb), [05-2](https://colab.research.google.com/github/rickiepark/hg-mldl2/blob/main/05-2.ipynb), [05-3](https://colab.research.google.com/github/rickiepark/hg-mldl2/blob/main/05-3.ipynb)

# 5장. 트리 알고리즘
  

## 05-1. 결정 트리

### 로지스틱 회귀로 와인 분류하기

- 상황: 알코올 도수·당도·pH만 보고 **레드 와인인지 화이트 와인인지** 구분하기

```python
import pandas as pd

wine = pd.read_csv('https://bit.ly/wine_csv_data')
```

- 판다스로 와인 샘플 데이터를 데이터프레임으로 불러온다

```python
wine.head()
```

```
   alcohol  sugar    pH  class
0      9.4    1.9  3.51    0.0
1      9.8    2.6  3.20    0.0
2      9.8    2.3  3.26    0.0
3      9.8    1.9  3.16    0.0
4      9.4    1.9  3.51    0.0
```

- `class`가 타깃. 0이면 레드 와인, 1이면 화이트 와인이다 (화이트 와인이 양성 클래스)

```python
wine.info()
```

- `info()`: 각 열의 데이터 타입과 **누락된 값**이 있는지 확인할 수 있다
    - 총 6497개 샘플, 4개 열 모두 실수형(`float64`)이고 누락된 값이 없다
    - 누락된 값이 있다면 그 행을 버리거나 평균값으로 채워야 한다. 이때도 **훈련 세트의 통계값**으로 테스트 세트를 채워야 한다

```python
wine.describe()
```

- `describe()`: 평균, 표준편차, 최소·최대, 사분위수 등 통계 요약을 보여 준다
- 세 특성의 스케일이 서로 다르므로 표준화가 필요하다
    - 예: 알코올 도수는 8~15 정도인데 당도는 0.6~65.8로 범위가 훨씬 넓다

```python
data = wine[['alcohol', 'sugar', 'pH']]
target = wine['class']
```

- 처음 3개 열은 입력 데이터로, `class` 열은 타깃으로 나눈다

```python
from sklearn.model_selection import train_test_split

train_input, test_input, train_target, test_target = train_test_split(
    data, target, test_size=0.2, random_state=42)
```

```python
print(train_input.shape, test_input.shape)
```

```
(5197, 3) (1300, 3)
```

- `test_size=0.2`: 기본값 25% 대신 **20%** 만 테스트 세트로 나눈다. 샘플이 6497개로 충분히 많기 때문

```python
from sklearn.preprocessing import StandardScaler

ss = StandardScaler()
ss.fit(train_input)

train_scaled = ss.transform(train_input)
test_scaled = ss.transform(test_input)
```

- 4장과 같은 방법으로 훈련 세트의 평균·표준편차로 두 세트를 표준화한다

```python
from sklearn.linear_model import LogisticRegression

lr = LogisticRegression()
lr.fit(train_scaled, train_target)

print(lr.score(train_scaled, train_target))
print(lr.score(test_scaled, test_target))
```

```
0.7808350971714451
0.7776923076923077
```

- 두 점수 모두 낮다 → **과소적합**
    - 규제 매개변수 C를 바꾸거나, 다항 특성을 추가하는 방법으로 개선해 볼 수 있다

```python
print(lr.coef_, lr.intercept_)
```

```
[[ 0.51268071  1.67335441 -0.68775646]] [1.81773456]
```

- 계수와 절편을 보여 줘도 "왜 이 와인이 화이트 와인인지" **설명하기 어렵다**
    - 당도 계수(1.67)가 가장 커서 당도가 높을수록 화이트 와인일 가능성이 높은 것 같지만, 정확히 어떤 의미인지 말로 설명하기는 어렵다
    - 다항 특성까지 추가하면 설명은 더 어려워진다

### 결정 트리

: 예/아니오 질문을 이어 가며 데이터를 나누는 **스무고개 같은** 모델. 결과를 설명하기 쉽다

```python
from sklearn.tree import DecisionTreeClassifier

dt = DecisionTreeClassifier(random_state=42)
dt.fit(train_scaled, train_target)

print(dt.score(train_scaled, train_target))
print(dt.score(test_scaled, test_target))
```

```
0.996921300750433
0.8592307692307692
```

- `DecisionTreeClassifier`: 사이킷런의 결정 트리 분류 클래스. 사용법은 다른 모델과 같다
    - 결정 트리는 노드를 나눌 때 특성 순서를 무작위로 섞기 때문에, 책과 같은 결과를 얻으려고 `random_state`를 지정한다
- 훈련 세트 점수가 매우 높고 테스트 세트 점수는 낮다 → **과대적합**

```python
import matplotlib.pyplot as plt
from sklearn.tree import plot_tree

plt.figure(figsize=(10,7))
plot_tree(dt)
plt.show()
```

- `plot_tree()`: 결정 트리를 이해하기 쉬운 트리 그림으로 그려 준다
- 그림이 매우 복잡하다. 맨 위에서 아래로 가지를 치며 내려가는 거꾸로 된 나무 모양이다

![사진1](/assets/img/posts/Self_Study_ML_DL/Self_Study_ML_DL_chap5_1.png)

```python
plt.figure(figsize=(10,7))
plot_tree(dt, max_depth=1, filled=True,
          feature_names=['alcohol', 'sugar', 'pH'])
plt.show()
```

- `max_depth=1`: 루트 노드 아래로 한 단계만 그린다. `feature_names`로 특성 이름을 표시한다
- **루트 노드(root node)**: 맨 위의 노드, **리프 노드(leaf node)**: 맨 아래 끝 노드
- 노드 하나에는 **테스트 조건**(예: sugar ≤ −0.239), **불순도**(gini), **샘플 수**(samples), **클래스별 샘플 수**(value)가 적혀 있다
    - 조건을 만족하면 왼쪽(Yes), 만족하지 않으면 오른쪽(No) 가지로 간다
    - 리프 노드에서 **가장 많은 클래스**가 예측 클래스가 된다
- `filled=True`: 클래스 비율이 높아질수록 노드 색이 진해진다
- **불순도(impurity)**: 노드에 섞여 있는 정도를 나타내는 값
    - **지니 불순도(gini)** = 1 − (음성 클래스 비율² + 양성 클래스 비율²)
    - 한 클래스만 있으면 0(순수 노드), 반반이면 0.5로 최악
- **정보 이득(information gain)**: 부모와 자식 노드의 **불순도 차이**. 결정 트리는 이 값이 최대가 되도록 나눈다
    - `criterion='entropy'`로 엔트로피 불순도를 쓸 수도 있다

![사진2](/assets/img/posts/Self_Study_ML_DL/Self_Study_ML_DL_chap5_2.png)

```python
dt = DecisionTreeClassifier(max_depth=3, random_state=42)
dt.fit(train_scaled, train_target)

print(dt.score(train_scaled, train_target))
print(dt.score(test_scaled, test_target))
```

```
0.8454877814123533
0.8415384615384616
```

- **가지치기(pruning)**: 트리의 최대 깊이를 제한해 과대적합을 막는 것
- `max_depth=3`: 루트 노드 아래로 최대 3개의 노드까지만 자라게 한다
- 훈련 세트 점수는 0.997 → 0.845로 낮아졌지만 테스트 세트 점수(0.842)와 거의 같아졌다

```python
plt.figure(figsize=(20,15))
plot_tree(dt, filled=True, feature_names=['alcohol', 'sugar', 'pH'])
plt.show()
```

- 깊이 3의 트리를 그려 보면, 왼쪽 끝에서 세 번째 노드만 **음성 클래스(레드 와인)** 가 더 많다
    - 당도가 −0.239 이하이면서 −0.802 초과, 알코올 도수가 0.454 이하인 와인이 레드 와인으로 분류된다
- 그런데 표준화된 값이라 "당도 −0.802"가 실제로 얼마인지 알기 어렵다

```python
dt = DecisionTreeClassifier(max_depth=3, random_state=42)
dt.fit(train_input, train_target)

print(dt.score(train_input, train_target))
print(dt.score(test_input, test_target))
```

```
0.8454877814123533
0.8415384615384616
```

- 결정 트리는 불순도를 기준으로 나누기만 하므로 **특성값의 스케일에 영향을 받지 않는다**
- 표준화하지 않은 원본 데이터를 써도 점수가 똑같고, 대신 트리 그림의 기준값을 **그대로 읽을 수 있다** (예: 당도 ≤ 1.625)

![사진3](/assets/img/posts/Self_Study_ML_DL/Self_Study_ML_DL_chap5_3.png)

```python
print(dt.feature_importances_)
```

```
[0.12345626 0.86862934 0.0079144 ]
```

- **특성 중요도(feature importance)**: 각 노드의 정보 이득과 샘플 수를 곱해 계산한 값. 여기서는 **당도**가 가장 중요하다
    - 출력 순서는 알코올, 당도, pH 순이다. 모두 더하면 1이 된다
    - 트리 그림에서도 루트 노드와 깊이 1의 노드에서 당도를 기준으로 나눴던 것과 일치한다
- 결정 트리는 이렇게 **어떤 특성이 중요한지** 알려 줄 수 있어 특성 선택에도 활용할 수 있다

```python
dt = DecisionTreeClassifier(min_impurity_decrease=0.0005, random_state=42)
dt.fit(train_input, train_target)

print(dt.score(train_input, train_target))
print(dt.score(test_input, test_target))
```

```
0.8874350586877044
0.8615384615384616
```

- `min_impurity_decrease`: 정보 이득이 이 값보다 작으면 더 이상 나누지 않는다. 깊이 대신 불순도로 가지치기하는 방법
- 이렇게 하면 트리가 좌우 대칭이 아니라 정보 이득이 큰 쪽으로만 깊어지고, 테스트 점수도 0.862로 조금 올라간다

### 이해하기 쉬운 결정 트리 모델

이번 절의 문제 해결 과정을 정리하면 다음과 같다.

1. **첫 시도**: 로지스틱 회귀 → 점수가 낮고(과소적합) 계수만으로는 설명하기 어려움
2. **결정 트리 적용**: 기본값으로 훈련 → 훈련 0.997 / 테스트 0.859로 **과대적합**
3. **가지치기**: `max_depth=3`으로 제한 → 두 점수가 비슷해지고 트리를 눈으로 이해할 수 있게 됨
4. **전처리 생략**: 결정 트리는 스케일에 영향받지 않으므로 표준화 없이 훈련 → 기준값을 그대로 해석 가능
5. **설명**: "당도가 −0.239보다 크고 …" 처럼 **모델이 왜 그렇게 판단했는지 설명**할 수 있다

<hr>

## 05-2. 교차 검증과 그리드 서치

### 검증 세트

: 테스트 세트를 사용하지 않고 모델을 평가하기 위해, **훈련 세트에서 다시 떼어 낸** 세트

- 테스트 세트로 여러 번 성능을 확인하면 결국 **테스트 세트에 맞춰진 모델**이 된다
- 테스트 세트는 마지막에 딱 한 번만 사용해야 한다

```python
import pandas as pd

wine = pd.read_csv('https://bit.ly/wine_csv_data')
```

```python
data = wine[['alcohol', 'sugar', 'pH']]
target = wine['class']
```

```python
from sklearn.model_selection import train_test_split

train_input, test_input, train_target, test_target = train_test_split(
    data, target, test_size=0.2, random_state=42)
```

```python
sub_input, val_input, sub_target, val_target = train_test_split(
    train_input, train_target, test_size=0.2, random_state=42)
```

- 테스트 세트는 그대로 두고, **훈련 세트**를 다시 `train_test_split()`으로 나눠 20%를 검증 세트로 만든다
- `sub_input`, `sub_target`: 실제 훈련에 쓸 데이터, `val_input`, `val_target`: 검증 세트

```python
print(sub_input.shape, val_input.shape)
```

```
(4157, 3) (1040, 3)
```

- 원래 훈련 세트 5197개가 훈련 4157개, 검증 1040개로 나뉘었다

```python
from sklearn.tree import DecisionTreeClassifier

dt = DecisionTreeClassifier(random_state=42)
dt.fit(sub_input, sub_target)

print(dt.score(sub_input, sub_target))
print(dt.score(val_input, val_target))
```

```
0.9971133028626413
0.864423076923077
```

- 검증 세트로 평가했더니 훈련 0.997 / 검증 0.864로 과대적합되어 있다. 매개변수를 바꿔 가며 이 검증 점수를 높이면 된다

### 교차 검증

: 검증 세트를 떼어 내는 과정을 **여러 번 반복**해 점수를 평균 내는 방법

- 검증 세트를 한 번만 떼면 훈련 데이터가 줄고, 점수도 어떻게 나뉘었는지에 따라 흔들린다
- **k-폴드 교차 검증**: 훈련 세트를 k개로 나눠, 한 덩어리씩 돌아가며 검증 세트로 쓴다 (보통 5-폴드, 10-폴드)

```python
from sklearn.model_selection import cross_validate

scores = cross_validate(dt, train_input, train_target)
print(scores)
```

- `cross_validate()`: 기본 5-폴드 교차 검증을 수행하고 `fit_time`, `score_time`, `test_score`를 반환한다
    - 각 키에 5개 폴드의 값이 하나씩 담겨 있다
    - 검증 세트를 따로 떼어 내지 않고 **전체 훈련 세트**(`train_input`)를 전달하면 된다

```python
import numpy as np

print(np.mean(scores['test_score']))
```

```
0.855300214703487
```

- 5개 폴드의 `test_score`를 평균한 값이 **교차 검증 점수**다. 이름은 test_score지만 실제로는 검증 폴드의 점수다

```python
from sklearn.model_selection import StratifiedKFold

scores = cross_validate(dt, train_input, train_target, cv=StratifiedKFold())
print(np.mean(scores['test_score']))
```

```
0.855300214703487
```

- `cross_validate()`는 훈련 세트를 섞지 않는다. 분류 모델에는 **`StratifiedKFold`(층화 k-폴드)** 가 기본으로 적용된다

```python
splitter = StratifiedKFold(n_splits=10, shuffle=True, random_state=42)
scores = cross_validate(dt, train_input, train_target, cv=splitter)
print(np.mean(scores['test_score']))
```

```
0.8574181117533719
```

- `shuffle=True`: 폴드를 나누기 전에 데이터를 섞는다
- `n_splits=10`: 10-폴드 교차 검증을 수행한다. 분할기 객체를 `cv` 매개변수로 전달한다

![사진4](/assets/img/posts/Self_Study_ML_DL/Self_Study_ML_DL_chap5_4.png)

### 하이퍼파라미터 튜닝

: 사람이 지정해야 하는 값(하이퍼파라미터)을 바꿔 가며 **가장 좋은 조합**을 찾는 과정

- 매개변수가 여러 개면 서로 영향을 주기 때문에, 하나씩 따로 최적값을 찾으면 안 된다 → 조합을 한꺼번에 탐색해야 한다
- **그리드 서치(grid search)**: 지정한 값들의 **모든 조합**을 교차 검증으로 시도한다

```python
from sklearn.model_selection import GridSearchCV

params = {'min_impurity_decrease': [0.0001, 0.0002, 0.0003, 0.0004, 0.0005]}
```

- 탐색할 매개변수 이름을 키로, 시도할 값의 리스트를 값으로 하는 **딕셔너리**를 만든다

```python
gs = GridSearchCV(DecisionTreeClassifier(random_state=42), params, n_jobs=-1)
```

- `n_jobs=-1`: 사용할 CPU 코어 수. −1이면 **모든 코어**를 사용한다
- 결정 트리 객체와 `params`를 전달해 그리드 서치 객체를 만든다

```python
gs.fit(train_input, train_target)
```

- 일반 모델처럼 `fit()`을 호출하면, 값 5개 × 5-폴드 = **25개의 모델**을 훈련한다

```python
dt = gs.best_estimator_
print(dt.score(train_input, train_target))
```

```
0.9615162593804117
```

```python
print(gs.best_params_)
```

```
{'min_impurity_decrease': 0.0001}
```

```python
print(gs.cv_results_['mean_test_score'])
```

```
[0.86819297 0.86453617 0.86492226 0.86780891 0.86761605]
```

- 5개 값 각각의 평균 교차 검증 점수다. 첫 번째(0.0001)가 가장 크다

```python
print(gs.cv_results_['params'][gs.best_index_])
```

- `best_index_`는 가장 높은 점수의 인덱스다. 이 인덱스로 `params`를 찾으면 `best_params_`와 같은 값이 나온다

- `best_estimator_`: 가장 좋은 조합으로 **전체 훈련 세트를 다시 훈련한 모델**
- `best_params_`: 최적의 매개변수 조합, `cv_results_`: 각 조합의 교차 검증 점수

```python
params = {'min_impurity_decrease': np.arange(0.0001, 0.001, 0.0001),
          'max_depth': range(5, 20, 1),
          'min_samples_split': range(2, 100, 10)
          }
```

- 이번에는 매개변수 3개를 한꺼번에 탐색한다
    - `np.arange(0.0001, 0.001, 0.0001)`: 0.0001부터 0.001 전까지 0.0001씩 → 9개
    - `max_depth`: 5부터 19까지 → 15개
    - `min_samples_split`: 노드를 나누기 위한 최소 샘플 수. 2부터 92까지 10씩 → 10개
- `np.arange()`는 실수 간격도 만들 수 있고, 파이썬의 `range()`는 정수만 가능하다

```python
gs = GridSearchCV(DecisionTreeClassifier(random_state=42), params, n_jobs=-1)
gs.fit(train_input, train_target)
```

```python
print(gs.best_params_)
```

```
{'max_depth': 14, 'min_impurity_decrease': np.float64(0.0004), 'min_samples_split': 12}
```

```python
print(np.max(gs.cv_results_['mean_test_score']))
```

```
0.8683865773302731
```

- `np.max()`로 가장 높은 교차 검증 점수를 확인했다

- 매개변수 3개의 조합 9 × 15 × 10 = 1350개를 5-폴드로 검증하므로 총 6750개의 모델을 훈련한다

```python
from scipy.stats import uniform, randint
```

```python
rgen = randint(0, 10)
rgen.rvs(10)
```

- `randint(0, 10)`: 0~9 사이의 정수를 뽑는 분포 객체. `rvs(10)`으로 10개를 뽑는다

```python
np.unique(rgen.rvs(1000), return_counts=True)
```

- 1000개를 뽑아 숫자별 개수를 세어 보면 0~9가 각각 100개 안팎으로 **고르게** 나온다

```python
ugen = uniform(0, 1)
ugen.rvs(10)
```

- `uniform(0, 1)`: 0~1 사이의 실수를 고르게 뽑는다

- `randint`: 정수 범위에서, `uniform`: 실수 범위에서 값을 **균등하게 샘플링**하는 사이파이 확률 분포

```python
params = {'min_impurity_decrease': uniform(0.0001, 0.001),
          'max_depth': randint(20, 50),
          'min_samples_split': randint(2, 25),
          'min_samples_leaf': randint(1, 25),
          }
```

- 값 목록 대신 분포 객체를 넣었다
    - `min_samples_leaf`: 리프 노드가 되기 위한 최소 샘플 수. 나눴을 때 자식 노드의 샘플이 이보다 적으면 나누지 않는다

```python
from sklearn.model_selection import RandomizedSearchCV

rs = RandomizedSearchCV(DecisionTreeClassifier(random_state=42), params,
                        n_iter=100, n_jobs=-1, random_state=42)
rs.fit(train_input, train_target)
```

- **랜덤 서치(random search)**: 값의 목록 대신 **확률 분포**를 전달하면, 그 안에서 지정한 횟수만큼 무작위로 뽑아 시도한다
    - 매개변수 값의 범위가 넓거나 연속적일 때 그리드 서치보다 효율적이다
- `n_iter=100`: 100번 샘플링해서 교차 검증한다

```python
print(rs.best_params_)
```

```
{'max_depth': 39, 'min_impurity_decrease': np.float64(0.00034102546602601173), 'min_samples_leaf': 7, 'min_samples_split': 13}
```

```python
print(np.max(rs.cv_results_['mean_test_score']))
```

```
0.8695428296438884
```

- 랜덤 서치로 찾은 최고 교차 검증 점수(0.8695)가 앞의 그리드 서치(0.8684)보다 조금 높다

```python
dt = rs.best_estimator_

print(dt.score(test_input, test_target))
```

```
0.86
```

- 마지막으로 최적 모델을 **테스트 세트**로 한 번만 평가한다. 검증 점수보다 조금 낮은 것이 일반적이다

![사진5](/assets/img/posts/Self_Study_ML_DL/Self_Study_ML_DL_chap5_5.png)

```python
gs = RandomizedSearchCV(DecisionTreeClassifier(splitter='random', random_state=42), params,
                        n_iter=100, n_jobs=-1, random_state=42)
gs.fit(train_input, train_target)
```

```python
print(gs.best_params_)
print(np.max(gs.cv_results_['mean_test_score']))

dt = gs.best_estimator_
print(dt.score(test_input, test_target))
```

```
{'max_depth': 43, 'min_impurity_decrease': np.float64(0.00011407982271508446), 'min_samples_leaf': 19, 'min_samples_split': 18}
0.8458726956392981
0.786923076923077
```

- `splitter='random'`: 노드를 나눌 때 최선이 아니라 **무작위로** 나눈다. 여기서는 점수가 더 낮아졌다

### 최적의 모델을 위한 하이퍼파라미터 탐색

이번 절의 문제 해결 과정을 정리하면 다음과 같다.

1. **문제 인식**: 테스트 세트로 여러 번 평가하면 테스트 세트에 맞춰진 모델이 된다
2. **검증 세트 분리**: 훈련 세트에서 20%를 검증 세트로 떼어 냄
3. **교차 검증**: `cross_validate()`로 폴드를 바꿔 가며 안정적인 점수를 얻음
4. **그리드 서치**: 매개변수 조합 전체를 교차 검증으로 탐색 → 최적 조합 자동 선택
5. **랜덤 서치**: 확률 분포에서 매개변수를 샘플링해 더 넓은 범위를 효율적으로 탐색 → 테스트 점수 0.86

<hr>

## 05-3. 트리의 앙상블

### 정형 데이터와 비정형 데이터

- **정형 데이터(structured data)**: CSV, 데이터베이스, 엑셀처럼 **행과 열로 정리되는** 데이터
- **비정형 데이터(unstructured data)**: 텍스트, 사진, 음악처럼 규칙성을 표로 나타내기 어려운 데이터
- **앙상블 학습(ensemble learning)**: 여러 모델을 만들어 그 예측을 합치는 방법. **정형 데이터에서 가장 뛰어난 성능**을 낸다
    - 대부분 결정 트리를 기반으로 만들어진다
    - 비정형 데이터에는 신경망(딥러닝)이 강하다

### 랜덤 포레스트

: 결정 트리를 여러 개 만들어 **각 트리의 예측을 모아** 최종 예측을 만드는 앙상블 모델

- **부트스트랩 샘플(bootstrap sample)**: 훈련 세트에서 **중복을 허용해** 훈련 세트 크기만큼 뽑은 샘플
- 노드를 나눌 때 전체 특성이 아니라 **일부 특성만 무작위로 골라** 그중 최선을 찾는다
    - 분류는 특성 개수의 제곱근만큼, 회귀는 전체 특성을 사용한다
- 이렇게 무작위성을 주기 때문에 훈련 세트에 과대적합되는 것을 막아 준다

```python
import numpy as np
import pandas as pd
from sklearn.model_selection import train_test_split

wine = pd.read_csv('https://bit.ly/wine_csv_data')

data = wine[['alcohol', 'sugar', 'pH']]
target = wine['class']

train_input, test_input, train_target, test_target = train_test_split(
    data, target, test_size=0.2, random_state=42)
```

- 5-1절과 같은 와인 데이터를 훈련 세트와 테스트 세트로 나눈다. 트리 기반 모델이므로 표준화는 하지 않는다

```python
from sklearn.model_selection import cross_validate
from sklearn.ensemble import RandomForestClassifier

rf = RandomForestClassifier(n_jobs=-1, random_state=42)
scores = cross_validate(rf, train_input, train_target, return_train_score=True, n_jobs=-1)

print(np.mean(scores['train_score']), np.mean(scores['test_score']))
```

```
0.9973541965122431 0.8905151032797809
```

- `RandomForestClassifier`: 기본적으로 결정 트리 **100개**를 사용한다. `n_jobs=-1`로 모든 CPU 코어를 쓰면 빠르다
- `return_train_score=True`: 훈련 세트 점수도 함께 반환해 과대적합 여부를 볼 수 있다
- 훈련 0.997 / 검증 0.891로 훈련 세트에 다소 과대적합되어 있다. 이 데이터는 특성이 3개뿐이라 매개변수를 바꿔도 크게 달라지지 않는다

```python
rf.fit(train_input, train_target)
print(rf.feature_importances_)
```

```
[0.23167441 0.50039841 0.26792718]
```

- 결정 트리 하나만 썼을 때(당도 0.87)보다 **당도의 중요도가 낮아지고 나머지 특성의 중요도가 올라갔다**
    - 특성 일부만 무작위로 사용하다 보니 **더 많은 특성이 훈련에 기여**하기 때문. 과대적합을 줄이는 데 도움이 된다

```python
rf = RandomForestClassifier(oob_score=True, n_jobs=-1, random_state=42)

rf.fit(train_input, train_target)
print(rf.oob_score_)
```

```
0.8934000384837406
```

- **OOB(out of bag) 샘플**: 부트스트랩 샘플에 **뽑히지 않은** 나머지 샘플. 검증 세트처럼 쓸 수 있어 교차 검증을 대신할 수 있다
- `oob_score=True`로 지정하면 각 트리의 OOB 점수를 평균해 `oob_score_`에 저장한다. 교차 검증 점수(0.891)와 비슷하다

### 엑스트라 트리

: 랜덤 포레스트와 비슷하지만 **부트스트랩 샘플을 사용하지 않고**, 노드를 나눌 때 **무작위로** 분할하는 모델

```python
from sklearn.ensemble import ExtraTreesClassifier

et = ExtraTreesClassifier(n_jobs=-1, random_state=42)
scores = cross_validate(et, train_input, train_target,
                        return_train_score=True, n_jobs=-1)

print(np.mean(scores['train_score']), np.mean(scores['test_score']))
```

```
0.9974503966084433 0.8887848893166506
```

- 랜덤 포레스트와 비슷한 결과다. 기본 트리 개수도 100개로 같다

```python
et.fit(train_input, train_target)
print(et.feature_importances_)
```

```
[0.20183568 0.52242907 0.27573525]
```

- 무작위로 나누기 때문에 트리 하나하나의 성능은 낮지만, 많이 모으면 과대적합을 막고 검증 점수를 높인다
- 최적의 분할을 찾지 않으므로 **계산 속도가 빠르다**

### 그레이디언트 부스팅

: **깊이가 얕은 결정 트리**를 계속 추가하면서, 앞선 트리의 **오차를 보완**해 나가는 방식

- 기본적으로 깊이 3인 트리 100개를 사용한다. 깊이가 얕아 과대적합에 강하다
- **경사 하강법**을 사용해 트리를 추가한다. 분류는 로지스틱 손실, 회귀는 평균 제곱 오차를 사용한다

```python
from sklearn.ensemble import GradientBoostingClassifier

gb = GradientBoostingClassifier(random_state=42)
scores = cross_validate(gb, train_input, train_target,
                        return_train_score=True, n_jobs=-1)

print(np.mean(scores['train_score']), np.mean(scores['test_score']))
```

```
0.8881086892152563 0.8720430147331015
```

- 훈련 0.888 / 검증 0.872로 과대적합이 거의 없다

```python
gb = GradientBoostingClassifier(n_estimators=500, learning_rate=0.2,
                                random_state=42)
scores = cross_validate(gb, train_input, train_target,
                        return_train_score=True, n_jobs=-1)

print(np.mean(scores['train_score']), np.mean(scores['test_score']))
```

```
0.9464595437171814 0.8780082549788999
```

- `n_estimators`: 트리 개수, `learning_rate`: 학습률(기본값 0.1)
- 트리를 5배로 늘렸는데도 과대적합을 잘 억제하고 있다

```python
gb.fit(train_input, train_target)
print(gb.feature_importances_)
```

```
[0.15887763 0.6799705  0.16115187]
```

- 랜덤 포레스트보다 **당도에 더 집중**한다 (0.68)
- 일반적으로 랜덤 포레스트보다 성능이 좋지만, 트리를 **순서대로** 추가하므로 훈련 속도가 느리다 (`n_jobs`가 없다)

### 히스토그램 기반 그레이디언트 부스팅

: 입력 특성을 **256개 구간으로 나눠** 최적의 분할을 빠르게 찾는 그레이디언트 부스팅

- 구간 중 하나를 누락된 값을 위해 사용하므로 **전처리가 거의 필요 없다**

```python
from sklearn.ensemble import HistGradientBoostingClassifier

hgb = HistGradientBoostingClassifier(random_state=42)
scores = cross_validate(hgb, train_input, train_target,
                        return_train_score=True, n_jobs=-1)

print(np.mean(scores['train_score']), np.mean(scores['test_score']))
```

```
0.9321723946453317 0.8801241948619236
```

- 과대적합을 잘 억제하면서 그레이디언트 부스팅보다 조금 더 높은 검증 점수를 얻었다

```python
from sklearn.inspection import permutation_importance

hgb.fit(train_input, train_target)
result = permutation_importance(hgb, train_input, train_target, n_repeats=10,
                                random_state=42, n_jobs=-1)
print(result.importances_mean)
```

```
[0.08876275 0.23438522 0.08027708]
```

- **순열 중요도(permutation importance)**: 특성을 하나씩 **무작위로 섞어** 성능이 얼마나 떨어지는지로 중요도를 계산한다
    - 사이킷런의 어떤 모델에도 사용할 수 있다
    - `n_repeats=10`: 특성을 섞는 횟수. 10번 반복한 평균을 `importances_mean`에 저장한다
- 훈련 세트에서도 당도(0.234)가 가장 중요하다

```python
result = permutation_importance(hgb, test_input, test_target, n_repeats=10,
                                random_state=42, n_jobs=-1)
print(result.importances_mean)
```

```
[0.05969231 0.20238462 0.049     ]
```

- 테스트 세트로도 계산할 수 있다. 실전에서 모델이 어떤 특성에 기대어 예측하는지 확인할 때 쓴다

```python
hgb.score(test_input, test_target)
```

```
0.8723076923076923
```

- 테스트 세트 점수 0.872로, 앙상블 모델 중 하나를 최종 모델로 쓰면 5-2절의 결정 트리(0.86)보다 좋은 성능을 얻는다

![사진6](/assets/img/posts/Self_Study_ML_DL/Self_Study_ML_DL_chap5_6.png)

```python
from xgboost import XGBClassifier

xgb = XGBClassifier(tree_method='hist', random_state=42)
xgb._estimator_type = "classifier"
scores = cross_validate(xgb, train_input, train_target,
                        return_train_score=True, n_jobs=-1)

print(np.mean(scores['train_score']), np.mean(scores['test_score']))
```

```
0.9567059184812372 0.8783915747390243
```

- **XGBoost**: 코랩에 설치되어 있는 그레이디언트 부스팅 라이브러리. 사이킷런의 `cross_validate()`와 함께 쓸 수 있다
- `tree_method='hist'`로 지정하면 히스토그램 기반 그레이디언트 부스팅을 사용한다

```python
from lightgbm import LGBMClassifier

lgb = LGBMClassifier(random_state=42)
scores = cross_validate(lgb, train_input, train_target,
                        return_train_score=True, n_jobs=-1)

print(np.mean(scores['train_score']), np.mean(scores['test_score']))
```

```
0.935828414851749 0.8801251203079884
```

- **LightGBM**: 마이크로소프트에서 만든 히스토그램 기반 그레이디언트 부스팅 라이브러리. 빠르고 성능이 좋다

- 사이킷런 외에도 **XGBoost**, **LightGBM** 같은 히스토그램 기반 그레이디언트 부스팅 라이브러리가 널리 쓰인다

### 앙상블 학습을 통한 성능 향상

이번 절에서 살펴본 앙상블 모델을 정리하면 다음과 같다.

| 모델 | 특징 |
| --- | --- |
| 랜덤 포레스트 | 부트스트랩 샘플 + 특성 일부 무작위 선택. 가장 대표적이고 안정적 |
| 엑스트라 트리 | 부트스트랩 없이 무작위 분할. 속도가 빠름 |
| 그레이디언트 부스팅 | 얕은 트리로 앞 트리의 오차를 보완. 성능이 좋지만 순차 훈련이라 느림 |
| 히스토그램 기반 그레이디언트 부스팅 | 특성을 256개 구간으로 나눠 빠르고 성능도 높음 |

- 앙상블은 **정형 데이터**에서 가장 좋은 성과를 내는 방법이며, 결정 트리 하나보다 훨씬 높은 검증 점수를 얻었다
- 다음 6장부터는 타깃이 없는 **비지도 학습**으로 넘어간다

<hr>

## 📌 정리

- **결정 트리**는 예/아니오 질문으로 데이터를 나누며, 불순도(지니)와 정보 이득을 기준으로 분할한다
- 스케일에 영향받지 않아 **전처리가 필요 없고**, 특성 중요도로 판단 근거를 설명할 수 있다
- 테스트 세트를 반복해 쓰지 않도록 **검증 세트와 교차 검증**을 사용하고, 그리드·랜덤 서치로 하이퍼파라미터를 탐색한다
- 정형 데이터에서는 **앙상블**(랜덤 포레스트, 엑스트라 트리, 그레이디언트 부스팅)이 가장 좋은 성능을 낸다

**핵심 용어**: 결정 트리 / 불순도 / 정보 이득 / 가지치기 / 특성 중요도 / 검증 세트 / 교차 검증 / 그리드 서치 / 랜덤 서치 / 앙상블 / 랜덤 포레스트 / 엑스트라 트리 / 그레이디언트 부스팅
