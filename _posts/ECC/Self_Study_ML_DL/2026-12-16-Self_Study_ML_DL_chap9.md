---
title: "[혼자 공부하는 머신러닝+딥러닝] 9장. 텍스트를 위한 인공 신경망"
date: 2026-12-16 21:00:00 +0900
categories: [Study, Machine Learning]
tags: [ml, dl, python, ecc]
---

["혼자 공부하는 머신러닝+딥러닝(개정판)" 도서 바로가기](https://www.hanbit.co.kr/store/books/look.php?p_code=B7077594897)

예제 코드 (저자 제공): [09-2](https://colab.research.google.com/github/rickiepark/hg-mldl2/blob/main/09-2.ipynb)

# 9장. 텍스트를 위한 인공 신경망


## 09-1. 순차 데이터와 순환 신경망

### 순차 데이터

: **순서에 의미가 있는** 데이터. 텍스트, 시계열 데이터(날씨, 주가) 등이 여기에 해당한다

- 지금까지 다룬 데이터(생선, 과일, 패션 아이템)는 **순서와 상관이 없었다**
    - 그래서 훈련 세트를 마음대로 섞어도 문제가 없었다
- 텍스트는 단어의 순서가 바뀌면 의미가 달라진다 → 이전에 입력한 데이터를 **기억하고 있어야** 한다
- **피드포워드 신경망(FFNN)**: 완전 연결 신경망이나 합성곱 신경망처럼 입력이 **한 방향으로만** 흐르는 신경망

### 순환 신경망

: 이전 샘플의 출력을 **자기 자신에게 다시 입력**하는 신경망 (Recurrent Neural Network, RNN)

- 뉴런의 출력이 다시 자기 자신으로 돌아가는 **순환**하는 고리가 있다
- **셀(cell)**: 순환 신경망에서의 층. 한 셀에는 여러 개의 뉴런이 있지만 보통 뉴런을 모두 표시하지 않는다
- **은닉 상태(hidden state)**: 셀의 출력. 다음 타임스텝에 다시 입력으로 사용된다
- **타임스텝(timestep)**: 샘플을 처리하는 한 단계
- 활성화 함수로는 주로 **하이퍼볼릭 탄젠트(tanh)** 를 사용한다 (−1 ~ 1 사이 값)

### 셀의 가중치와 입출력

- 순환층의 가중치는 두 종류다
    - **입력에 곱해지는 가중치**: (입력 특성 개수 × 뉴런 개수)
    - **은닉 상태에 곱해지는 가중치**: (뉴런 개수 × 뉴런 개수)
    - 절편은 뉴런마다 하나씩
- 순환층은 **모든 타임스텝에서 같은 가중치**를 사용한다
- 입력은 **3차원 배열**(샘플 수, 타임스텝 수, 단어 표현 크기)이다
- 출력
    - 마지막 타임스텝의 은닉 상태만 출력하면 **2차원 배열**(샘플 수, 뉴런 개수)
    - 순환층을 여러 개 쌓으려면 앞 층이 **모든 타임스텝의 은닉 상태**를 출력해야 한다 (3차원 배열)

### 순환 신경망으로 순환 데이터 처리

이번 절에서 배운 개념을 정리하면 다음과 같다.

| 용어 | 의미 |
| --- | --- |
| 순차 데이터 | 순서에 의미가 있는 데이터 (텍스트, 시계열) |
| 순환 신경망 | 이전 타임스텝의 출력을 다시 입력으로 사용하는 신경망 |
| 셀 | 순환 신경망의 층 |
| 은닉 상태 | 셀의 출력이자 다음 타임스텝의 입력 |
| 타임스텝 | 샘플을 처리하는 한 단계 |

- 순환 신경망은 이전 정보를 은닉 상태에 담아 다음 계산에 사용하므로 **순서를 기억**할 수 있다

<hr>

## 09-2. 순환 신경망으로 IMDB 리뷰 분류하기

### IMDB 리뷰 데이터셋

: 영화 리뷰 5만 개를 긍정(1)과 부정(0)으로 분류해 놓은 데이터셋. 훈련 세트와 테스트 세트가 각 2만 5천 개다

- **자연어 처리(NLP)**: 컴퓨터가 사람의 언어를 처리하는 분야. 음성 인식, 기계 번역, 감성 분석 등
- **말뭉치(corpus)**: 자연어 처리 분야에서 사용하는 훈련 데이터
- **토큰(token)**: 텍스트를 나눈 단위. 영어는 보통 공백으로 나눈 단어 하나가 토큰이다
- **어휘 사전(vocabulary)**: 훈련 세트에 등장하는 고유한 단어의 집합. 각 단어에 정수를 매겨 사용한다

```python
# 실행마다 동일한 결과를 얻기 위해 케라스에 랜덤 시드를 사용하고 텐서플로 연산을 결정적으로 만듭니다.
import keras
import tensorflow as tf

keras.utils.set_random_seed(42)
tf.random.set_seed(42)
tf.config.experimental.enable_op_determinism()
```

```python
from keras.datasets import imdb

(train_input, train_target), (test_input, test_target) = imdb.load_data(
    num_words=200)
```

- `num_words=200`: 가장 자주 등장하는 단어 **200개만** 사용한다

```python
print(train_input.shape, test_input.shape)
```

```
(25000,) (25000,)
```

- 훈련 세트와 테스트 세트에 리뷰가 각각 25,000개씩 들어 있다
- 1차원 배열인 이유: 리뷰마다 길이가 달라 2차원 배열로 만들 수 없기 때문이다

```python
print(len(train_input[0]))
```

```
218
```

```python
print(len(train_input[1]))
```

```
189
```

- 리뷰마다 길이가 다르기 때문에 배열이 아니라 **리스트의 배열**로 되어 있다

```python
print(train_input[0])
```

- 첫 번째 리뷰의 내용을 출력했다
- 이미 **정수로 변환**되어 있다. 0은 패딩, 1은 문장의 시작, 2는 어휘 사전에 없는 단어를 뜻한다

```python
print(train_target[:20])
```

```
[1 0 0 1 0 0 1 0 1 0 1 0 0 0 0 0 1 1 0 1]
```

- 타깃은 0(부정)과 1(긍정)이다

```python
from sklearn.model_selection import train_test_split

train_input, val_input, train_target, val_target = train_test_split(
    train_input, train_target, test_size=0.2, random_state=42)
```

- 훈련 세트의 20%(5,000개)를 검증 세트로 떼어 낸다

```python
import numpy as np

lengths = np.array([len(x) for x in train_input])
```

```python
print(np.mean(lengths), np.median(lengths))
```

```
239.00925 178.0
```

- 리스트 내포로 리뷰마다 길이를 구했다
- 평균(239)이 중간값(178)보다 훨씬 크다 → 일부 아주 긴 리뷰가 평균을 끌어올린 **한쪽으로 치우친 분포**다

```python
import matplotlib.pyplot as plt

plt.hist(lengths)
plt.xlabel('length')
plt.ylabel('frequency')
plt.show()
```

- 대부분의 리뷰가 300단어 이하다
- 그래서 여기서는 리뷰 길이를 **100**으로 맞춰 사용한다

<!-- 캡처: 리뷰 길이 히스토그램
<!-- ![사진1](/assets/img/posts/Self_Study_ML_DL/Self_Study_ML_DL_chap9_1.png) -->
-->

```python
from keras.preprocessing.sequence import pad_sequences

train_seq = pad_sequences(train_input, maxlen=100)
```

- `pad_sequences()`: 시퀀스 길이를 **모두 같게** 맞춘다
    - 길면 잘라 내고, 짧으면 **0으로 채운다(패딩)**

```python
print(train_seq.shape)
```

```
(20000, 100)
```

- 샘플 20,000개가 모두 길이 100으로 맞춰져 2차원 배열이 되었다

```python
print(train_seq[0])
```

```python
print(train_input[0][-10:])
```

- 패딩된 첫 번째 샘플의 끝부분과 원본 샘플의 마지막 10개 토큰이 같다

- 100개보다 긴 리뷰는 **앞부분이 잘린다**. 뒷부분의 정보가 더 유용할 것으로 기대하기 때문

```python
print(train_seq[5])
```

- 짧은 리뷰는 **앞쪽이 0으로 채워진다**. 뒤쪽에 패딩이 있으면 순환층이 마지막에 의미 없는 값을 보게 되기 때문

```python
val_seq = pad_sequences(val_input, maxlen=100)
```

- 검증 세트도 같은 길이로 맞춘다
- `truncating='post'`, `padding='post'`로 지정하면 뒤쪽을 자르거나 채울 수도 있다

### 순환 신경망 만들기

```python
import keras

model = keras.Sequential()
model.add(keras.layers.Input(shape=(100,200)))
model.add(keras.layers.SimpleRNN(8))
model.add(keras.layers.Dense(1, activation='sigmoid'))
```

- `Input(shape=(100,200))`: 타임스텝 100개, 각 토큰을 길이 200인 원-핫 벡터로 표현한다
- `SimpleRNN`: 케라스의 기본 순환층. 뉴런 8개를 사용한다
- 이진 분류이므로 출력층은 뉴런 1개에 **시그모이드** 활성화 함수를 쓴다

```python
train_oh = keras.utils.to_categorical(train_seq)
```

```python
print(train_oh.shape)
```

```
(20000, 100, 200)
```

- `to_categorical()`: 정수 배열을 원-핫 인코딩한다. 토큰마다 200개 원소의 배열이 되었다

```python
print(train_oh[0][0][:12])
```

```python
print(np.sum(train_oh[0][0]))
```

```
1.0
```

- 첫 번째 토큰의 원-핫 벡터는 해당 단어 위치 하나만 1이므로 합이 1이다

- 정수값을 그대로 넣으면 **큰 숫자가 더 중요하다고 오해**하게 된다 → **원-핫 인코딩**으로 바꾼다

```python
val_oh = keras.utils.to_categorical(val_seq)
```

```python
model.summary()
```

```
 Total params: 1,681 (6.57 KB)
```

- 파라미터 개수: (200 × 8) + (8 × 8) + 8 = 1,672개 + 출력층 9개

### 순환 신경망 훈련하기

```python
model.compile(optimizer='adam', loss='binary_crossentropy',
              metrics=['accuracy'])

checkpoint_cb = keras.callbacks.ModelCheckpoint('best-simplernn-model.keras',
                                                save_best_only=True)
early_stopping_cb = keras.callbacks.EarlyStopping(patience=3,
                                                  restore_best_weights=True)

history = model.fit(train_oh, train_target, epochs=100, batch_size=64,
                    validation_data=(val_oh, val_target),
                    callbacks=[checkpoint_cb, early_stopping_cb])
```

- 이진 분류이므로 손실 함수는 `binary_crossentropy`를 사용한다
- `batch_size=64`: 미니배치 크기를 기본값 32 대신 64로 늘렸다
- 에포크는 100으로 크게 잡았지만, `patience=3` 조기 종료 콜백이 적절한 시점에 멈춘다

```python
plt.plot(history.history['loss'], label='train')
plt.plot(history.history['val_loss'], label='val')
plt.xlabel('epoch')
plt.ylabel('loss')
plt.legend()
plt.show()
```

- 훈련 손실은 꾸준히 줄지만 검증 손실은 어느 순간부터 줄지 않아 조기 종료되었다

<!-- 캡처: SimpleRNN 모델의 훈련/검증 손실 곡선
<!-- ![사진2](/assets/img/posts/Self_Study_ML_DL/Self_Study_ML_DL_chap9_2.png) -->
-->

```python
print(train_seq.nbytes, train_oh.nbytes)
```

```
8000000 3200000000
```

- 원-핫 인코딩을 하면 데이터 크기가 **400배**로 늘어난다. 단어가 많아지면 감당할 수 없다

### 단어 임베딩을 사용하기

: 각 단어를 **고정 크기의 실수 벡터**로 바꾸는 방법. 단어 사이의 관계를 표현할 수 있다

```python
(train_input, train_target), (test_input, test_target) = imdb.load_data(
    num_words=500)
train_input, val_input, train_target, val_target = train_test_split(
    train_input, train_target, test_size=0.2, random_state=42)
train_seq = pad_sequences(train_input, maxlen=100)
val_seq = pad_sequences(val_input, maxlen=100)
```

- 이번에는 어휘 사전 크기를 **500**으로 늘려 데이터를 다시 준비했다

```python
model_emb = keras.Sequential()
model_emb.add(keras.layers.Input(shape=(100,)))
model_emb.add(keras.layers.Embedding(500, 16))
model_emb.add(keras.layers.SimpleRNN(8))
model_emb.add(keras.layers.Dense(1, activation='sigmoid'))
```

- `Embedding(500, 16)`: 어휘 사전 크기 500, 임베딩 벡터 크기 16
- 임베딩 벡터는 **훈련하면서 함께 학습**되므로 원-핫 인코딩보다 의미를 잘 담는다
- 정수를 그대로 입력할 수 있어 메모리도 훨씬 적게 쓴다

```python
model_emb.summary()
```

```
 Total params: 8,209 (32.07 KB)
```

- 임베딩 층: 500 × 16 = 8,000개
- 순환층: (16 × 8) + (8 × 8) + 8 = 200개, 출력층: 9개
- 원-핫 인코딩보다 입력 크기는 훨씬 작은데, 어휘 사전은 오히려 더 크다

```python
model_emb.compile(optimizer='adam', loss='binary_crossentropy',
                  metrics=['accuracy'])
checkpoint_cb = keras.callbacks.ModelCheckpoint('best-embedding-model.keras',
                                                save_best_only=True)
early_stopping_cb = keras.callbacks.EarlyStopping(patience=3,
                                                  restore_best_weights=True)

history = model_emb.fit(train_seq, train_target, epochs=100, batch_size=64,
                        validation_data=(val_seq, val_target),
                        callbacks=[checkpoint_cb, early_stopping_cb])
```

```python
plt.plot(history.history['loss'], label='train')
plt.plot(history.history['val_loss'], label='val')
plt.xlabel('epoch')
plt.ylabel('loss')
plt.legend()
plt.show()
```

- 원-핫 인코딩 모델과 비슷한 성능을 내면서도, 메모리와 파라미터 효율이 훨씬 좋다

<!-- 캡처: 임베딩 층을 사용한 모델의 손실 곡선
<!-- ![사진3](/assets/img/posts/Self_Study_ML_DL/Self_Study_ML_DL_chap9_3.png) -->
-->

### 케라스 API로 순환 신경망 구현

이번 절의 문제 해결 과정을 정리하면 다음과 같다.

1. **데이터 준비**: IMDB 리뷰를 불러오고 `pad_sequences()`로 길이를 100으로 맞춤
2. **첫 시도**: 정수를 원-핫 인코딩해 `SimpleRNN`에 입력 → 동작은 하지만 **메모리가 400배**로 늘어남
3. **개선**: `Embedding` 층으로 단어를 실수 벡터로 표현 → 정수를 그대로 입력할 수 있고 메모리 절약
4. **훈련**: 체크포인트/조기 종료 콜백으로 최적 지점에서 훈련 종료

<hr>

## 📌 정리

- **순차 데이터**는 순서에 의미가 있어, 이전 정보를 은닉 상태로 기억하는 **순환 신경망**을 사용한다
- IMDB 리뷰를 `pad_sequences()`로 길이를 맞추고, 원-핫 인코딩 대신 **임베딩 층**을 써서 메모리를 크게 줄였다

**핵심 용어**: 순차 데이터 / 순환 신경망 / 셀 / 은닉 상태 / 타임스텝 / 말뭉치 / 토큰 / 패딩 / 단어 임베딩
