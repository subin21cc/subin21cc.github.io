---
title: "[혼자 공부하는 머신러닝+딥러닝] 6장. 비지도 학습"
date: 2026-10-24 17:00:00 +0900
categories: [Study, Machine Learning]
tags: [ml, dl, python, ecc]
---

["혼자 공부하는 머신러닝+딥러닝(개정판)" 도서 바로가기](https://www.hanbit.co.kr/store/books/look.php?p_code=B7077594897)

예제 코드 (저자 제공): [06-1](https://colab.research.google.com/github/rickiepark/hg-mldl2/blob/main/06-1.ipynb), [06-2](https://colab.research.google.com/github/rickiepark/hg-mldl2/blob/main/06-2.ipynb), [06-3](https://colab.research.google.com/github/rickiepark/hg-mldl2/blob/main/06-3.ipynb)

# 6장. 비지도 학습


## 06-1. 군집 알고리즘

### 타깃을 모르는 비지도 학습

- 상황: 고객이 보낸 과일 사진을 종류별로 모아야 하는데, **어떤 과일인지 미리 알 수 없다**
- **비지도 학습(unsupervised learning)**: 타깃이 없을 때 데이터에 있는 패턴이나 구조를 스스로 찾는 머신러닝
- 여기서는 사과·파인애플·바나나 사진 300장을 **비슷한 것끼리 모으는** 문제를 다룬다

### 과일 사진 데이터 준비하기

```python
!wget https://bit.ly/fruits_300_data -O fruits_300.npy
```

- 코랩에서 `!`로 시작하면 파이썬 코드가 아니라 **리눅스 셸 명령**으로 실행된다
- `wget`: 원격 주소에서 파일을 내려받는 명령. `-O`로 저장할 파일 이름을 지정한다

```python
import numpy as np
import matplotlib.pyplot as plt
```

```python
fruits = np.load('fruits_300.npy')
```

- `np.load()`: 넘파이 배열이 저장된 `.npy` 파일을 읽어 배열로 만든다

```python
print(fruits.shape)
```

```
(300, 100, 100)
```

- 샘플 300개, 이미지 크기는 100 × 100 픽셀이다
    - 첫 번째 차원이 샘플 개수, 두 번째가 이미지 높이, 세 번째가 이미지 너비다
    - 흑백 사진이라 픽셀마다 값이 하나뿐이다 (컬러라면 차원이 하나 더 필요하다)
- 첫 100장은 사과, 다음 100장은 파인애플, 마지막 100장은 바나나다

```python
print(fruits[0, 0, :])
```

- `fruits[0, 0, :]`: 첫 번째 이미지의 **첫 번째 행** 픽셀 100개를 출력한다
- 이미지는 **0~255 사이의 정숫값**으로 이루어진 넘파이 배열이다

```python
plt.imshow(fruits[0], cmap='gray')
plt.show()
```

- `imshow()`: 넘파이 배열을 이미지로 그린다. 흑백 이미지이므로 `cmap='gray'`로 지정한다
- 0에 가까울수록 검게, 255에 가까울수록 밝게 표시된다

```python
plt.imshow(fruits[0], cmap='gray_r')
plt.show()
```

- 이 데이터는 흑백 사진을 **반전**시킨 것이다 (사진의 배경이 검고 사과가 밝다)
    - 컴퓨터는 값이 큰 픽셀에 주목하므로, 관심 대상인 과일이 큰 값을 갖도록 바꾼 것
- `cmap='gray_r'`: 다시 반전시켜 사람 눈에 익숙한 형태로 보여 준다

![사진1](/assets/img/posts/Self_Study_ML_DL/Self_Study_ML_DL_chap6_1.png)

```python
fig, axs = plt.subplots(1, 2)
axs[0].imshow(fruits[100], cmap='gray_r')
axs[1].imshow(fruits[200], cmap='gray_r')
plt.show()
```

- `subplots()`: 여러 개의 그래프를 한 번에 배치한다
    - `subplots(1, 2)`는 1행 2열로 그래프 2개를 만든다. 반환된 `axs`에 각 그래프가 배열로 담긴다
- 인덱스 100은 파인애플의 첫 사진, 200은 바나나의 첫 사진이다

![사진2](/assets/img/posts/Self_Study_ML_DL/Self_Study_ML_DL_chap6_2.png)

### 픽셀값 분석하기

```python
apple = fruits[0:100].reshape(-1, 100*100)
pineapple = fruits[100:200].reshape(-1, 100*100)
banana = fruits[200:300].reshape(-1, 100*100)
```

```python
print(apple.shape)
```

```
(100, 10000)
```

- 계산하기 쉽도록 100 × 100 이미지를 길이 10000인 **1차원 배열**로 폈다
    - 슬라이싱으로 과일별 100장씩 고른 뒤, `reshape(-1, 100*100)`으로 두 번째·세 번째 차원을 하나로 합쳤다

```python
print(apple.mean(axis=1))
```

- `axis=1`: 열을 따라 계산 → **샘플 하나(사진 한 장)의 평균**
- 사과 사진 100장 각각의 픽셀 평균값 100개가 출력된다

```python
plt.hist(apple.mean(axis=1), alpha=0.8, label='apple')
plt.hist(pineapple.mean(axis=1), alpha=0.8, label='pineapple')
plt.hist(banana.mean(axis=1), alpha=0.8, label='banana')
plt.legend()
plt.show()
```

- `hist()`: 값이 나타나는 빈도를 막대그래프로 그린다. `alpha`로 투명도를 줘 겹친 부분도 보이게 한다
- `label`과 `legend()`로 어떤 색이 어떤 과일인지 범례를 표시한다
- **히스토그램**으로 보면 바나나는 픽셀 평균값이 확실히 낮다 (사진에서 차지하는 면적이 작기 때문)
- 하지만 사과와 파인애플은 평균값이 겹쳐서 **사진 전체의 평균만으로는 구분할 수 없다**

![사진3](/assets/img/posts/Self_Study_ML_DL/Self_Study_ML_DL_chap6_3.png)

```python
fig, axs = plt.subplots(1, 3, figsize=(20, 5))
axs[0].bar(range(10000), apple.mean(axis=0))
axs[1].bar(range(10000), pineapple.mean(axis=0))
axs[2].bar(range(10000), banana.mean(axis=0))
plt.show()
```

- `axis=0`: 행을 따라 계산 → **픽셀별 평균**
- `bar()`: 막대그래프를 그린다. x축은 픽셀 번호(0~9999), y축은 그 픽셀의 평균값이다
- 픽셀별로 보면 세 과일의 그래프 모양이 확실히 다르다
    - 사과는 아래쪽으로 갈수록 값이 높고, 파인애플은 비교적 고르게 높으며, 바나나는 중앙 픽셀값이 높다

![사진4](/assets/img/posts/Self_Study_ML_DL/Self_Study_ML_DL_chap6_4.png)

```python
apple_mean = apple.mean(axis=0).reshape(100, 100)
pineapple_mean = pineapple.mean(axis=0).reshape(100, 100)
banana_mean = banana.mean(axis=0).reshape(100, 100)

fig, axs = plt.subplots(1, 3, figsize=(20, 5))
axs[0].imshow(apple_mean, cmap='gray_r')
axs[1].imshow(pineapple_mean, cmap='gray_r')
axs[2].imshow(banana_mean, cmap='gray_r')
plt.show()
```

- 픽셀 평균을 다시 100 × 100으로 되돌려 이미지로 그리면 **과일의 대표 이미지**가 나온다
- 이 평균 이미지와 가까운 사진을 고르면 같은 과일끼리 모을 수 있지 않을까?

![사진5](/assets/img/posts/Self_Study_ML_DL/Self_Study_ML_DL_chap6_5.png)

### 평균값과 가까운 사진 고르기

```python
abs_diff = np.abs(fruits - apple_mean)
abs_mean = np.mean(abs_diff, axis=(1,2))
print(abs_mean.shape)
```

```
(300,)
```

- `np.abs()`: 절댓값을 계산한다
- 300장 모두에서 사과 평균 이미지와의 **절댓값 오차 평균**을 구했다
    - `fruits`(300, 100, 100)에서 `apple_mean`(100, 100)을 빼면 브로드캐스팅으로 300장 모두에서 뺄셈이 된다
    - `axis=(1,2)`: 두 번째·세 번째 차원(픽셀)에 대해 평균을 내므로 사진마다 값 하나가 남는다

```python
apple_index = np.argsort(abs_mean)[:100]
apple_index = apple_index.reshape(10, 10)
fig, axs = plt.subplots(10, 10, figsize=(10,10))
for i in range(10):
    for j in range(10):
        axs[i, j].imshow(fruits[apple_index[i, j]], cmap='gray_r')
        axs[i, j].axis('off')
plt.show()
```

- `np.argsort()`: 작은 것에서 큰 순서대로 **인덱스**를 반환한다
- 고른 인덱스 100개를 (10, 10)으로 바꾸고, 10 × 10 격자에 이미지를 하나씩 그렸다
    - `axis('off')`: 좌표축을 그리지 않아 이미지만 깔끔하게 보이게 한다
- 오차가 가장 작은 100장을 골랐더니 **모두 사과**였다

![사진6](/assets/img/posts/Self_Study_ML_DL/Self_Study_ML_DL_chap6_6.png)

```python
abs_diff = np.abs(fruits - banana_mean)
abs_mean = np.mean(abs_diff, axis=(1,2))

banana_index = np.argsort(abs_mean)[:100]
banana_index = banana_index.reshape(10, 10)
fig, axs = plt.subplots(10, 10, figsize=(10,10))
for i in range(10):
    for j in range(10):
        axs[i, j].imshow(fruits[banana_index[i, j]], cmap='gray_r')
        axs[i, j].axis('off')
plt.show()
```

- 같은 방법으로 바나나 평균 이미지와 가까운 100장을 골라 본다
- 대부분 바나나지만 사과나 파인애플이 몇 장 섞여 나온다. 평균값만으로는 완벽하지 않다

![사진7](/assets/img/posts/Self_Study_ML_DL/Self_Study_ML_DL_chap6_7.png)

### 비슷한 샘플끼리 모으기

- **군집(clustering)**: 비슷한 샘플끼리 그룹으로 모으는 대표적인 비지도 학습 작업
- **클러스터(cluster)**: 군집으로 모은 샘플 그룹
- 이번 절에서는 사과·파인애플·바나나가 각각 100장씩 있다는 걸 **이미 알고** 평균 이미지를 만들었다
    - 실제 비지도 학습에서는 어떤 과일이 몇 장 있는지조차 모른다
    - → 타깃을 모른 채로 평균을 찾아내는 방법이 다음 절의 **k-평균** 알고리즘이다

<hr>

## 06-2. k-평균

### k-평균 알고리즘 소개

: 클러스터의 중심(**센트로이드**)을 찾아가며 샘플을 나누는 군집 알고리즘

- 작동 방식
    1. 무작위로 **k개의 클러스터 중심**을 정한다
    2. 각 샘플에서 가장 가까운 클러스터 중심을 찾아 그 클러스터로 지정한다
    3. 클러스터에 속한 샘플의 **평균값으로 클러스터 중심을 옮긴다**
    4. 클러스터 중심에 변화가 없을 때까지 2~3번을 반복한다

### KMeans 클래스

```python
!wget https://bit.ly/fruits_300_data -O fruits_300.npy
```

```python
import numpy as np

fruits = np.load('fruits_300.npy')
fruits_2d = fruits.reshape(-1, 100*100)
```

- 사이킷런 모델에 넣으려면 (샘플 개수, 특성 개수)인 **2차원 배열**이어야 하므로 (300, 10000)으로 바꾼다

```python
from sklearn.cluster import KMeans

km = KMeans(n_clusters=3, random_state=42)
km.fit(fruits_2d)
```

- `n_clusters=3`: 클러스터 개수 k를 3으로 지정한다
- 비지도 학습이므로 `fit()`에 **타깃을 전달하지 않는다**

```python
print(km.labels_)
```

```python
print(np.unique(km.labels_, return_counts=True))
```

```
(array([0, 1, 2], dtype=int32), array([112,  98,  90]))
```

- `labels_`: 각 샘플이 어떤 클러스터에 속하는지 담고 있다. 레이블 값 자체에는 의미가 없다
    - n_clusters=3이므로 레이블은 0, 1, 2 중 하나다
- `np.unique(..., return_counts=True)`로 레이블별 샘플 개수를 세었다. 첫 번째 클러스터에 112개, 두 번째에 98개, 세 번째에 90개가 모였다

```python
import matplotlib.pyplot as plt

def draw_fruits(arr, ratio=1):
    n = len(arr)    # n은 샘플 개수입니다
    # 한 줄에 10개씩 이미지를 그립니다. 샘플 개수를 10으로 나누어 전체 행 개수를 계산합니다.
    rows = int(np.ceil(n/10))
    # 행이 1개 이면 열 개수는 샘플 개수입니다. 그렇지 않으면 10개입니다.
    cols = n if rows < 2 else 10
    fig, axs = plt.subplots(rows, cols,
                            figsize=(cols*ratio, rows*ratio), squeeze=False)
    for i in range(rows):
        for j in range(cols):
            if i*10 + j < n:    # n 개까지만 그립니다.
                axs[i, j].imshow(arr[i*10 + j], cmap='gray_r')
            axs[i, j].axis('off')
    plt.show()
```

- 클러스터별 이미지를 출력하기 위한 함수 `draw_fruits()`를 만든다
    - 한 줄에 10개씩 그리며, 필요한 행 개수는 샘플 수를 10으로 나눠 올림(`np.ceil`)해서 계산한다
    - `ratio`로 그림 크기를 조절할 수 있다
    - `squeeze=False`: 행이나 열이 1개여도 `axs`를 항상 2차원 배열로 받아 같은 방식으로 인덱싱할 수 있게 한다

```python
draw_fruits(fruits[km.labels_==0])
```

- `km.labels_==0`은 레이블이 0인 샘플만 True인 불리언 배열이다. 이것으로 해당 클러스터의 사진만 골라 그린다

```python
draw_fruits(fruits[km.labels_==1])
```

```python
draw_fruits(fruits[km.labels_==2])
```

- 레이블 1은 바나나만, 레이블 2는 파인애플만 모였고, 레이블 0에는 사과에 파인애플 몇 개가 섞였다
- 타깃 없이도 **거의 완벽하게 세 과일을 구분**했다

<!-- 캡처: 클러스터별로 모인 과일 사진
![사진8](/assets/img/posts/Self_Study_ML_DL/Self_Study_ML_DL_chap6_8.png)
-->

### 클러스터 중심

```python
draw_fruits(km.cluster_centers_.reshape(-1, 100, 100), ratio=3)
```

- `cluster_centers_`: 클러스터 중심. 앞 절에서 직접 만든 **평균 이미지와 매우 비슷하다**
    - `fruits_2d`로 훈련했기 때문에 중심도 길이 10000인 1차원 배열이다. 이미지로 그리려면 (100, 100)으로 바꿔야 한다

<!-- 캡처: k-평균이 찾은 클러스터 중심 이미지 3개
![사진9](/assets/img/posts/Self_Study_ML_DL/Self_Study_ML_DL_chap6_9.png)
-->

```python
print(km.transform(fruits_2d[100:101]))
```

```
[[3400.24197319 8837.37750892 5279.33763699]]
```

- `transform()`: 샘플에서 각 클러스터 중심까지의 **거리**를 반환한다
    - 인덱스 100인 샘플(파인애플 사진)을 넣었다. `[100:101]`로 슬라이싱해 2차원 배열을 유지한다
    - 첫 번째 클러스터(레이블 0)까지의 거리가 3400으로 가장 가깝다

```python
print(km.predict(fruits_2d[100:101]))
```

```
[0]
```

- `predict()`: 가장 가까운 클러스터 중심을 예측 클래스로 출력한다. `transform()`에서 거리가 가장 짧았던 레이블 0이다

```python
draw_fruits(fruits[100:101])
```

- 이 샘플을 그려 보면 파인애플이다. 레이블 0 클러스터에 파인애플이 일부 섞여 있던 것과 같은 경우다

```python
print(km.n_iter_)
```

```
4
```

- `n_iter_`: 클러스터 중심을 옮기며 반복한 횟수
- 지금은 정답을 알기 때문에 결과를 확인할 수 있었지만, 실전에서는 k를 몇으로 할지부터 정해야 한다

### 최적의 k 찾기

- k-평균의 단점: 클러스터 개수를 **미리 지정해야** 한다. 하지만 실전에서는 몇 개인지 알 수 없다
- **이너셔(inertia)**: 클러스터 중심과 샘플 사이 거리의 제곱 합. 클러스터가 얼마나 조밀한지를 나타낸다
- **엘보우(elbow) 방법**: k를 늘려 가며 이너셔를 그래프로 그리면 **꺾이는 지점**이 나타난다. 그 지점이 적절한 k다

```python
inertia = []
for k in range(2, 7):
    km = KMeans(n_clusters=k, random_state=42)
    km.fit(fruits_2d)
    inertia.append(km.inertia_)

plt.plot(range(2, 7), inertia)
plt.xlabel('k')
plt.ylabel('inertia')
plt.show()
```

- k를 2부터 6까지 바꿔 가며 훈련하고, 각 모델의 `inertia_` 값을 리스트에 저장해 그래프로 그렸다
- k가 커질수록 클러스터마다 샘플이 적어져 이너셔는 계속 줄어든다
- k = 3에서 그래프의 기울기가 바뀐다 → 클러스터 3개가 적절하다
    - 꺾이는 지점이 아주 뚜렷하지는 않지만 조금 꺾인 것을 볼 수 있다

<!-- 캡처: k에 따른 이너셔 그래프(엘보우)
![사진10](/assets/img/posts/Self_Study_ML_DL/Self_Study_ML_DL_chap6_10.png)
-->

### 과일을 자동으로 분류하기

이번 절의 문제 해결 과정을 정리하면 다음과 같다.

1. **문제 인식**: 1절에서는 사과·파인애플·바나나가 각각 100장이라는 정답을 이미 알고 평균을 구했다
2. **해결**: `KMeans`로 타깃 없이 클러스터 중심을 직접 찾게 함 → 레이블별로 과일이 거의 정확히 모임
3. **중심 확인**: `cluster_centers_`가 1절의 평균 이미지와 거의 같다
4. **k 결정**: 이너셔를 k에 따라 그려 엘보우 지점(k = 3)을 찾음

<hr>

## 06-3. 주성분 분석

### 차원과 차원 축소

- **차원(dimension)**: 데이터가 가진 **특성의 개수**. 과일 사진은 픽셀 10000개 = 10000개의 차원
- 특성이 많으면 저장 공간도 많이 들고, 과대적합되기도 쉽다
- **차원 축소(dimensionality reduction)**: 데이터를 가장 잘 나타내는 일부 특성을 골라 **크기를 줄이면서 성능은 유지**하는 방법
    - 줄인 데이터에서 원본을 **복원**할 수도 있다

### 주성분 분석 소개

: 데이터에 있는 **분산이 큰 방향**을 찾는 차원 축소 방법 (Principal Component Analysis, PCA)

- **주성분(principal component)**: 분산이 가장 큰 방향을 나타내는 벡터
    - 원본 데이터와 **차원 개수가 같다**
    - 첫 번째 주성분에 수직이면서 분산이 가장 큰 방향이 두 번째 주성분이 된다
- 주성분에 데이터를 투영하면 특성 개수를 줄일 수 있다

### PCA 클래스

```python
!wget https://bit.ly/fruits_300_data -O fruits_300.npy
```

```python
import numpy as np

fruits = np.load('fruits_300.npy')
fruits_2d = fruits.reshape(-1, 100*100)
```

- 6-2절과 같이 과일 데이터를 내려받아 (300, 10000) 배열로 만든다

```python
from sklearn.decomposition import PCA

pca = PCA(n_components=50)
pca.fit(fruits_2d)
```

- `n_components`: 주성분 개수를 지정한다
- PCA도 비지도 학습이므로 **타깃을 전달하지 않는다**

```python
print(pca.components_.shape)
```

```
(50, 10000)
```

- `components_`: PCA가 찾은 주성분. 50개의 주성분이 있고, 각 주성분은 원본 데이터와 같은 **10000개 차원**이다

```python
import matplotlib.pyplot as plt

def draw_fruits(arr, ratio=1):
    n = len(arr)    # n은 샘플 개수입니다
    # 한 줄에 10개씩 이미지를 그립니다. 샘플 개수를 10으로 나누어 전체 행 개수를 계산합니다.
    rows = int(np.ceil(n/10))
    # 행이 1개 이면 열 개수는 샘플 개수입니다. 그렇지 않으면 10개입니다.
    cols = n if rows < 2 else 10
    fig, axs = plt.subplots(rows, cols,
                            figsize=(cols*ratio, rows*ratio), squeeze=False)
    for i in range(rows):
        for j in range(cols):
            if i*10 + j < n:    # n 개까지만 그립니다.
                axs[i, j].imshow(arr[i*10 + j], cmap='gray_r')
            axs[i, j].axis('off')
    plt.show()
```

```python
draw_fruits(pca.components_.reshape(-1, 100, 100))
```

- 주성분을 이미지로 그려 보면 데이터에서 **가장 분산이 큰 방향**, 즉 과일의 특징적인 모양이 나타난다

<!-- 캡처: 50개 주성분을 이미지로 그린 결과
![사진11](/assets/img/posts/Self_Study_ML_DL/Self_Study_ML_DL_chap6_11.png)
-->

```python
print(fruits_2d.shape)
```

```
(300, 10000)
```

```python
fruits_pca = pca.transform(fruits_2d)
```

```python
print(fruits_pca.shape)
```

```
(300, 50)
```

- `transform()`: 원본 데이터를 주성분에 투영해 차원을 줄인다
- 10000개였던 특성이 **50개로 줄었다** (1/200 크기)

### 원본 데이터 재구성

```python
fruits_inverse = pca.inverse_transform(fruits_pca)
print(fruits_inverse.shape)
```

```
(300, 10000)
```

- 50개 특성이 다시 10000개로 복원되었다

```python
fruits_reconstruct = fruits_inverse.reshape(-1, 100, 100)
```

- 이미지로 출력하기 위해 (300, 100, 100)으로 바꾸고, 과일별 100장씩 그려 본다

```python
for start in [0, 100, 200]:
    draw_fruits(fruits_reconstruct[start:start+100])
    print("\n")
```

- `inverse_transform()`: 줄인 데이터에서 **원본 차원으로 복원**한다
- 50개 특성만으로도 원본 이미지를 거의 그대로 되살릴 수 있다
    - 일부는 흐리고 번진 부분이 있지만 전체적으로 과일 모양은 잘 복원됐다

<!-- 캡처: 50개 주성분으로 복원한 과일 이미지
![사진12](/assets/img/posts/Self_Study_ML_DL/Self_Study_ML_DL_chap6_12.png)
-->

### 설명된 분산

```python
print(np.sum(pca.explained_variance_ratio_))
```

```
0.9215496908553521
```

- **설명된 분산(explained variance)**: 주성분이 원본 데이터의 분산을 얼마나 잘 나타내는지의 비율
- `explained_variance_ratio_`: 각 주성분의 설명된 분산 비율이 담겨 있다. 모두 더하면 50개 주성분의 총 분산 비율이 된다
- 50개 주성분이 원본의 분산 **92%** 를 설명한다. 그래서 복원이 잘 되는 것

```python
plt.plot(pca.explained_variance_ratio_)
plt.show()
```

- 앞쪽 10개 주성분이 대부분의 분산을 설명한다
    - 그래프가 처음에 급격히 떨어지고 그 뒤로는 거의 0에 가깝다. 나머지 주성분은 설명하는 분산이 작다

<!-- 캡처: 주성분별 설명된 분산 그래프
![사진13](/assets/img/posts/Self_Study_ML_DL/Self_Study_ML_DL_chap6_13.png)
-->

### 다른 알고리즘과 함께 사용하기

```python
from sklearn.linear_model import LogisticRegression

lr = LogisticRegression()
```

- 차원 축소 데이터로 지도 학습 모델을 훈련하면 어떤 차이가 있는지 확인하기 위해 로지스틱 회귀를 사용한다

```python
target = np.array([0] * 100 + [1] * 100 + [2] * 100)
```

- 지도 학습과 비교하기 위해 정답 타깃을 직접 만든다 (사과 0, 파인애플 1, 바나나 2)

```python
from sklearn.model_selection import cross_validate

scores = cross_validate(lr, fruits_2d, target)
print(np.mean(scores['test_score']))
print(np.mean(scores['fit_time']))
```

```
0.9966666666666667
1.4621670246124268
```

- 원본 데이터(특성 10000개)로 교차 검증한 결과다. 정확도는 매우 높지만 폴드당 훈련 시간이 1.46초 걸렸다
- `fit_time`: 각 폴드의 훈련 시간이 담겨 있다

```python
scores = cross_validate(lr, fruits_pca, target)
print(np.mean(scores['test_score']))
print(np.mean(scores['fit_time']))
```

```
0.9966666666666667
0.0270662784576416
```

- PCA로 줄인 데이터(특성 50개)로 같은 교차 검증을 했다
- 정확도는 같은데 훈련 시간은 **50배 이상 빨라졌다**

```python
pca = PCA(n_components=0.5)
pca.fit(fruits_2d)
```

```python
print(pca.n_components_)
```

```
2
```

- `n_components`에 0~1 사이 **비율**을 지정하면, 그 분산 비율을 만족하는 주성분 개수를 자동으로 찾는다
- 분산의 50%를 설명하는 데 주성분 **2개**면 충분하다

```python
fruits_pca = pca.transform(fruits_2d)
print(fruits_pca.shape)
```

```
(300, 2)
```

```python
scores = cross_validate(lr, fruits_pca, target)
print(np.mean(scores['test_score']))
print(np.mean(scores['fit_time']))
```

```
0.99
0.054932785034179685
```

- 특성 2개만으로도 정확도 99%를 얻었다

```python
from sklearn.cluster import KMeans

km = KMeans(n_clusters=3, random_state=42)
km.fit(fruits_pca)
```

```python
print(np.unique(km.labels_, return_counts=True))
```

```
(array([0, 1, 2], dtype=int32), array([110,  99,  91]))
```

- 차원 축소한 데이터로 k-평균을 훈련했다. 원본 데이터로 했을 때(112, 98, 90)와 비슷하게 나뉘었다

```python
for label in range(0, 3):
    draw_fruits(fruits[km.labels_ == label])
    print("\n")
```

- 클러스터별 사진을 출력해 보면 원본으로 찾은 클러스터와 비슷한 결과가 나온다

```python
for label in range(0, 3):
    data = fruits_pca[km.labels_ == label]
    plt.scatter(data[:,0], data[:,1])
plt.legend(['apple', 'banana', 'pineapple'])
plt.show()
```

- 특성이 2개뿐이라 **산점도로 시각화**할 수 있다. 세 클러스터가 뚜렷하게 나뉘어 있다
- 사과와 파인애플의 경계가 가까운 것도 눈으로 확인할 수 있다

<!-- 캡처: 2개 주성분으로 그린 클러스터 산점도
![사진14](/assets/img/posts/Self_Study_ML_DL/Self_Study_ML_DL_chap6_14.png)
-->

### 주성분 분석으로 차원 축소

이번 절의 문제 해결 과정을 정리하면 다음과 같다.

1. **문제 인식**: 과일 사진의 특성이 10000개라 저장 공간과 훈련 시간이 많이 든다
2. **PCA 적용**: 주성분 50개로 차원을 축소 → 원본 분산의 92% 유지
3. **복원 확인**: `inverse_transform()`으로 원본과 거의 같은 이미지를 복원
4. **성능 비교**: 로지스틱 회귀의 정확도는 그대로인데 훈련 시간은 크게 단축
5. **시각화**: 주성분 2개로 줄여 산점도로 클러스터를 눈으로 확인

<hr>

## 📌 정리

- 타깃 없이 데이터의 구조를 찾는 **비지도 학습**을 과일 사진 300장으로 실습했다
- 픽셀 평균으로 직접 비슷한 사진을 골라 본 뒤, **k-평균**으로 타깃 없이 클러스터 중심을 찾았다
- **이너셔와 엘보우 방법**으로 적절한 클러스터 개수 k를 정한다
- **PCA**로 10000개 특성을 50개로 줄여도 분산의 92%가 유지되고, 훈련 속도는 50배 이상 빨라졌다

**핵심 용어**: 비지도 학습 / 군집 / 클러스터 / k-평균 / 클러스터 중심 / 이너셔 / 엘보우 방법 / 차원 축소 / 주성분 분석 / 설명된 분산
