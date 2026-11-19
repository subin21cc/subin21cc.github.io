---
title: "[혼자 공부하는 머신러닝+딥러닝] 8장. 이미지를 위한 인공 신경망"
date: 2026-11-11 21:00:00 +0900
categories: [Study, Machine Learning]
tags: [ml, dl, python, ecc]
---

["혼자 공부하는 머신러닝+딥러닝(개정판)" 도서 바로가기](https://www.hanbit.co.kr/store/books/look.php?p_code=B7077594897)

예제 코드 (저자 제공): [08-2](https://colab.research.google.com/github/rickiepark/hg-mldl2/blob/main/08-2.ipynb), [08-3](https://colab.research.google.com/github/rickiepark/hg-mldl2/blob/main/08-3.ipynb)

# 8장. 이미지를 위한 인공 신경망


## 08-1. 합성곱 신경망의 구성 요소

### 합성곱

: 입력 데이터에 **도장을 찍듯** 마치 훑으면서 유용한 특성만 뽑아내는 연산

- 밀집층은 뉴런마다 입력 개수만큼의 가중치를 가지고 **전체를 한 번에** 계산한다
- 합성곱은 **일부(예: 3 × 3)** 에만 가중치를 곱하고, 한 칸씩 이동하며 같은 가중치를 반복해서 사용한다
- **필터(filter) = 커널(kernel)**: 합성곱에 사용하는 가중치 묶음. 뉴런 하나에 해당한다
    - 케라스에서는 뉴런 개수를 `filters`, 가중치 크기를 `kernel_size`로 지정한다
- **특성 맵(feature map)**: 합성곱 계산으로 만들어진 출력 배열
    - 필터를 여러 개 쓰면 특성 맵이 여러 장 쌓여 **3차원 배열**이 된다
- 합성곱 신경망에서는 1차원으로 펴지 않고 **2차원 그대로** 입력을 사용하므로 이미지의 공간 정보가 유지된다

### 케라스 합성곱 층

- `keras.layers.Conv2D(10, kernel_size=(3,3), activation='relu')`
    - 2차원 이미지에 적용하는 합성곱이라 `Conv2D`다
    - 첫 번째 매개변수는 **필터 개수**, `kernel_size`는 커널 크기, 활성화 함수는 보통 렐루
    - 커널 크기는 보통 (3, 3) 또는 (5, 5)를 사용한다
- **패딩(padding)**
    - **밸리드 패딩(valid)**: 패딩 없이 계산 → 특성 맵이 입력보다 작아진다
    - **세임 패딩(same)**: 입력 주위를 0으로 채워 **입력과 같은 크기**의 특성 맵을 만든다
    - 패딩을 쓰는 이유: 패딩이 없으면 **모서리 픽셀이 중앙 픽셀보다 적게 계산에 참여**하기 때문
- **스트라이드(stride)**: 커널을 이동하는 칸 수. 기본값은 1이고, 보통 1을 그대로 쓴다
- **풀링(pooling)**: 특성 맵의 크기를 줄이는 연산
    - **최대 풀링(max pooling)**: 영역에서 가장 큰 값을 고른다
    - **평균 풀링(average pooling)**: 영역의 평균을 계산한다. 특성이 희석될 수 있어 최대 풀링을 더 많이 쓴다
    - 풀링에는 **가중치가 없고**, 영역이 겹치지 않도록 스트라이드가 풀링 크기와 같다
    - `keras.layers.MaxPooling2D(2)`

### 합성곱 신경망의 전체 구조

- 일반적인 구조: **(합성곱 층 → 풀링 층) 반복 → Flatten → 밀집 은닉층 → 출력층**
- 층을 지날수록 **특성 맵의 너비·높이는 줄고, 깊이(필터 개수)는 늘어난다**
- 마지막에는 특성 맵을 일렬로 펴서 밀집층에 넣고, 소프트맥스로 클래스 확률을 만든다
- 컬러 이미지는 **RGB 3개의 채널**을 가지므로 입력이 3차원이 되고, 커널도 깊이가 3이 된다

### 합성곱 층과 풀링 층 이해하기

이번 절에서 배운 개념을 정리하면 다음과 같다.

| 용어 | 의미 |
| --- | --- |
| 합성곱 | 입력의 일부에 가중치를 곱하며 훑는 연산 |
| 필터(커널) | 합성곱에 사용하는 가중치. 필터 개수 = 뉴런 개수 |
| 특성 맵 | 합성곱의 출력 배열 |
| 패딩 | 입력 주위를 0으로 채워 출력 크기를 유지(세임 패딩) |
| 스트라이드 | 커널을 이동하는 칸 수 |
| 풀링 | 특성 맵의 크기를 줄이는 연산(최대 풀링/평균 풀링) |

- 밀집층과 달리 합성곱은 **입력의 공간 구조를 유지**하면서 특성을 뽑아내기 때문에 이미지에 특히 잘 맞는다

<hr>

## 08-2. 합성곱 신경망을 사용한 이미지 분류

### 패션 MNIST 데이터 불러오기

```python
# 실행마다 동일한 결과를 얻기 위해 케라스에 랜덤 시드를 사용하고 텐서플로 연산을 결정적으로 만듭니다.
import keras
import tensorflow as tf

keras.utils.set_random_seed(42)
tf.random.set_seed(42)
tf.config.experimental.enable_op_determinism()
```

```python
import keras
from sklearn.model_selection import train_test_split

(train_input, train_target), (test_input, test_target) = \
    keras.datasets.fashion_mnist.load_data()

train_scaled = train_input.reshape(-1, 28, 28, 1) / 255.0

train_scaled, val_scaled, train_target, val_target = train_test_split(
    train_scaled, train_target, test_size=0.2, random_state=42)
```

- 7장과 달리 1차원으로 펴지 않고 **(28, 28, 1)** 크기의 3차원 배열로 만든다
    - 마지막 1은 흑백 이미지의 **채널 차원**이다
    - `Conv2D` 층은 (높이, 너비, 채널) 3차원 입력을 기대하기 때문이다
- 픽셀값을 255로 나눠 0~1로 정규화하고, 20%를 검증 세트로 나눈 것은 7장과 같다

### 합성곱 신경망 만들기

```python
model = keras.Sequential()
model.add(keras.layers.Input(shape=(28,28,1)))
model.add(keras.layers.Conv2D(32, kernel_size=3, activation='relu',
                              padding='same'))
```

- 필터 32개, 커널 3 × 3, 세임 패딩 → 출력 특성 맵 크기는 (28, 28, 32)
    - `kernel_size=3`처럼 정수 하나만 주면 (3, 3)과 같다

```python
model.add(keras.layers.MaxPooling2D(2))
```

- 최대 풀링으로 절반 크기가 된다 → (14, 14, 32)

```python
model.add(keras.layers.Conv2D(64, kernel_size=(3,3), activation='relu',
                              padding='same'))
model.add(keras.layers.MaxPooling2D(2))
```

- 두 번째 합성곱은 필터를 **64개로 늘린다** → (14, 14, 64) → 풀링 후 (7, 7, 64)

```python
model.add(keras.layers.Flatten())
model.add(keras.layers.Dense(100, activation='relu'))
model.add(keras.layers.Dropout(0.4))
model.add(keras.layers.Dense(10, activation='softmax'))
```

- 특성 맵을 펴서(7 × 7 × 64 = 3136) 밀집 은닉층 → 드롭아웃 → 출력층으로 연결한다

```python
model.summary()
```

```
 Total params: 333,526 (1.27 MB)
```

- 첫 번째 합성곱 층: 3 × 3 × 1 × 32 + 32 = 320개
- 두 번째 합성곱 층: 3 × 3 × 32 × 64 + 64 = 18,496개
- 은닉층: 3136 × 100 + 100 = 313,700개, 출력층: 100 × 10 + 10 = 1,010개
- 풀링, Flatten, 드롭아웃 층은 파라미터가 없다. 파라미터 대부분이 밀집 은닉층에 몰려 있다

<!-- 캡처: 합성곱 신경망의 model.summary() 출력
<!-- ![사진1](/assets/img/posts/Self_Study_ML_DL/Self_Study_ML_DL_chap8_1.png) -->
-->

```python
keras.utils.plot_model(model)
```

```python
keras.utils.plot_model(model, show_shapes=True)
```

- `plot_model()`: 모델 구조를 그림으로 그려 준다. `show_shapes=True`면 입출력 크기도 함께 표시된다

<!-- 캡처: plot_model()로 그린 모델 구조도
<!-- ![사진2](/assets/img/posts/Self_Study_ML_DL/Self_Study_ML_DL_chap8_2.png) -->
-->

### 모델 컴파일과 훈련

```python
model.compile(optimizer='adam', loss='sparse_categorical_crossentropy',
              metrics=['accuracy'],
              # 현재 텐서플로 버전에서는 최대 풀링에 대한 결정적인 XLA 구현이 없기 때문에
              # enable_op_determinism() 호출로 인해 오류가 발생합니다.
              # 이를 피하기 위해 jit_compile 옵션을 False로 지정합니다.
              # https://github.com/tensorflow/tensorflow/issues/69417
              jit_compile=False)
checkpoint_cb = keras.callbacks.ModelCheckpoint('best-cnn-model.keras',
                                                save_best_only=True)
early_stopping_cb = keras.callbacks.EarlyStopping(patience=2,
                                                  restore_best_weights=True)
history = model.fit(train_scaled, train_target, epochs=20,
                    validation_data=(val_scaled, val_target),
                    callbacks=[checkpoint_cb, early_stopping_cb])
```

- 7장에서 배운 `ModelCheckpoint`와 `EarlyStopping` 콜백을 그대로 사용한다
- 합성곱 층이 추가된 것 말고는 `compile()`, `fit()` 사용법이 7장과 같다

```python
import matplotlib.pyplot as plt

plt.plot(history.history['loss'])
plt.plot(history.history['val_loss'])
plt.xlabel('epoch')
plt.ylabel('loss')
plt.legend(['train', 'val'])
plt.show()
```

- 검증 손실이 가장 낮아진 뒤 2 에포크 동안 나아지지 않아 조기 종료되었다

<!-- 캡처: 합성곱 신경망의 훈련/검증 손실 곡선
<!-- ![사진3](/assets/img/posts/Self_Study_ML_DL/Self_Study_ML_DL_chap8_3.png) -->
-->

```python
model.evaluate(val_scaled, val_target)
```

```
[0.225491002202034, 0.9190833568572998]
```

- 검증 정확도 약 **0.92**. 7장의 밀집층 모델(약 0.885)보다 확실히 높다

```python
plt.imshow(val_scaled[0].reshape(28, 28), cmap='gray_r')
plt.show()
```

- `imshow()`는 흑백 이미지에 채널 차원이 있으면 그리지 못하므로 (28, 28)로 바꿔 출력한다
- 검증 세트의 첫 번째 이미지는 가방이다

```python
preds = model.predict(val_scaled[0:1])
print(preds)
```

- `[0:1]`로 슬라이싱해 (1, 28, 28, 1) 크기를 유지한다. 케라스는 항상 배치 차원이 있는 입력을 기대한다

```python
plt.bar(range(1, 11), preds[0])
plt.xlabel('class')
plt.ylabel('prob.')
plt.show()
```

- 아홉 번째 클래스의 확률만 1에 가깝고 나머지는 거의 0이다

<!-- 캡처: 예측 확률 막대그래프
<!-- ![사진4](/assets/img/posts/Self_Study_ML_DL/Self_Study_ML_DL_chap8_4.png) -->
-->

```python
classes = ['티셔츠', '바지', '스웨터', '드레스', '코트',
           '샌달', '셔츠', '스니커즈', '가방', '앵클 부츠']
```

```python
import numpy as np

print(classes[np.argmax(preds)])
```

```
가방
```

- `np.argmax()`로 가장 큰 확률의 인덱스를 찾아 클래스 이름 리스트에서 꺼냈다. 실제로 가방이므로 맞게 예측했다

```python
test_scaled = test_input.reshape(-1, 28, 28, 1) / 255.0
```

- 테스트 세트도 훈련 세트와 똑같이 채널 차원을 추가하고 255로 나눈다

```python
model.evaluate(test_scaled, test_target)
```

```
[0.2488059252500534, 0.9160000085830688]
```

- 마지막으로 테스트 세트로 **딱 한 번** 평가한다. 실전에서 기대할 수 있는 성능은 약 0.92다

### 케라스 API로 합성곱 신경망 구현

이번 절의 문제 해결 과정을 정리하면 다음과 같다.

1. **입력 형태 변경**: 1차원으로 펴지 않고 (28, 28, 1) 3차원 배열로 준비
2. **모델 구성**: (Conv2D 32 + MaxPooling2D) → (Conv2D 64 + MaxPooling2D) → Flatten → Dense 100 → Dropout → Dense 10
3. **훈련**: Adam 옵티마이저 + 체크포인트/조기 종료 콜백
4. **검증**: 검증 정확도 약 0.919 (7장 밀집층 모델보다 향상)
5. **예측·평가**: 샘플 하나의 확률을 확인하고, 마지막에 테스트 세트로 평가 → 약 0.916

<hr>

## 08-3. 합성곱 신경망의 시각화

### 가중치 시각화

```python
# 실행마다 동일한 결과를 얻기 위해 케라스에 랜덤 시드를 사용하고 텐서플로 연산을 결정적으로 만듭니다.
import keras
import tensorflow as tf

keras.utils.set_random_seed(42)
tf.random.set_seed(42)
tf.config.experimental.enable_op_determinism()
```

```python
# 코랩에서 실행하는 경우에는 다음 명령을 실행하여 best-cnn-model.keras 파일을 다운로드받아 사용하세요.
!wget https://github.com/rickiepark/hg-mldl2/raw/main/best-cnn-model.keras
```

```python
import keras
model = keras.models.load_model('best-cnn-model.keras')
```

- 2절에서 체크포인트 콜백으로 저장한 최상의 모델을 불러온다

```python
model.layers
```

- `layers`: 모델에 추가한 층이 순서대로 들어 있는 리스트
    - Conv2D, MaxPooling2D, Conv2D, MaxPooling2D, Flatten, Dense, Dropout, Dense 순서다

```python
conv = model.layers[0]
print(conv.weights[0].shape, conv.weights[1].shape)
```

```
(3, 3, 1, 32) (32,)
```

- `weights[0]`은 가중치, `weights[1]`은 절편이다
- 첫 번째 합성곱 층의 가중치는 (3, 3, 1, 32), 절편은 필터마다 하나씩 32개다
    - 커널 크기 3 × 3, 입력 채널(깊이) 1, 필터 32개라는 뜻이다

```python
conv_weights = conv.weights[0].numpy()

print(conv_weights.mean(), conv_weights.std())
```

```
-0.017617775 0.22690606
```

- `weights`는 텐서 객체이므로 `numpy()`로 넘파이 배열로 바꾼다
- 가중치 평균은 0에 가깝고, 표준편차는 약 0.23이다

```python
import matplotlib.pyplot as plt

plt.hist(conv_weights.reshape(-1, 1))
plt.xlabel('weight')
plt.ylabel('count')
plt.show()
```

- `hist()`에 넣기 위해 가중치를 1차원으로 폈다. 0을 중심으로 종 모양에 가깝게 분포한다

```python
fig, axs = plt.subplots(2, 16, figsize=(15,2))
for i in range(2):
    for j in range(16):
        axs[i, j].imshow(conv_weights[:,:,0,i*16 + j], vmin=-0.5, vmax=0.5)
        axs[i, j].axis('off')
plt.show()
```

- 32개 필터를 2행 16열로 그렸다. `vmin`, `vmax`로 색의 기준 범위를 고정해 필터끼리 밝기를 비교할 수 있게 한다
- 훈련된 필터를 이미지로 그리면 밝은 부분이 **큰 가중치**다. 필터마다 주목하는 패턴이 다르다

<!-- 캡처: 훈련된 32개 필터의 가중치 이미지
<!-- ![사진5](/assets/img/posts/Self_Study_ML_DL/Self_Study_ML_DL_chap8_5.png) -->
-->

```python
no_training_model = keras.Sequential()
no_training_model.add(keras.layers.Input(shape=(28,28,1)))
no_training_model.add(keras.layers.Conv2D(32, kernel_size=3, activation='relu',
                                          padding='same'))
```

```python
no_training_conv = no_training_model.layers[0]
print(no_training_conv.weights[0].shape)
```

```
(3, 3, 1, 32)
```

```python
no_training_weights = no_training_conv.weights[0].numpy()
print(no_training_weights.mean(), no_training_weights.std())
```

```
0.0053191613 0.08463709
```

- 훈련하지 않은 합성곱 층을 같은 구조로 만들어 비교한다
- 평균은 역시 0에 가깝지만 표준편차가 0.085로, 훈련된 가중치(0.23)보다 훨씬 작다
    - 케라스는 가중치를 **글로럿 균등 분포**로 작은 값에서 랜덤하게 초기화하기 때문이다

```python
plt.hist(no_training_weights.reshape(-1, 1))
plt.xlabel('weight')
plt.ylabel('count')
plt.show()
```

```python
fig, axs = plt.subplots(2, 16, figsize=(15,2))
for i in range(2):
    for j in range(16):
        axs[i, j].imshow(no_training_weights[:,:,0,i*16 + j], vmin=-0.5, vmax=0.5)
        axs[i, j].axis('off')
plt.show()
```

- 훈련하지 않은 필터는 값이 **좁은 범위에 고르게 퍼져** 있고 그림에도 뚜렷한 패턴이 없다
    - 즉, 훈련을 통해 필터가 **의미 있는 패턴을 학습**했다는 것을 확인할 수 있다

<!-- 캡처: 훈련 전 필터와 훈련 후 필터의 가중치 분포 비교
<!-- ![사진6](/assets/img/posts/Self_Study_ML_DL/Self_Study_ML_DL_chap8_6.png) -->
-->

### 함수형 API

: 층을 함수처럼 호출해 모델을 만드는 방법. 입력이 여러 개거나 중간 출력을 꺼내야 할 때 사용한다

```python
inputs = keras.Input(shape=(784,))
dense1 = keras.layers.Dense(100, activation='relu')
dense2 = keras.layers.Dense(10, activation='softmax')

hidden = dense1(inputs)
outputs = dense2(hidden)

func_model = keras.Model(inputs, outputs)
```

- 층 객체를 함수처럼 호출하면 그 층의 출력이 반환된다. 이 출력을 다음 층에 다시 넣어 연결한다
- `keras.Model(inputs, outputs)`: 입력과 출력을 지정해 모델을 만든다
- `Sequential`은 층을 순서대로만 쌓을 수 있지만, 함수형 API는 **입력과 출력을 직접 지정**할 수 있다

```python
print(model.inputs)
```

- `Sequential` 모델도 내부적으로는 함수형 API로 만들어지므로 `inputs` 속성으로 입력을 얻을 수 있다

```python
conv_acti = keras.Model(model.inputs[0], model.layers[0].output)
```

- 모델의 입력은 그대로 두고, **첫 번째 합성곱 층의 출력**을 출력으로 하는 새 모델을 만들었다
    - 이렇게 하면 중간 층이 만든 특성 맵을 꺼내 볼 수 있다

### 특성 맵 시각화

```python
(train_input, train_target), (test_input, test_target) =\
    keras.datasets.fashion_mnist.load_data()
```

```python
plt.imshow(train_input[0], cmap='gray_r')
plt.show()
```

- 훈련 세트의 첫 번째 샘플은 앵클 부츠다

```python
ankle_boot = train_input[0:1].reshape(-1, 28, 28, 1)/255.0
feature_maps = conv_acti.predict(ankle_boot)
```

```python
print(feature_maps.shape)
```

```
(1, 28, 28, 32)
```

- 세임 패딩과 필터 32개를 사용했으므로 (28, 28, 32) 특성 맵이 나온다. 앞의 1은 샘플 개수(배치 차원)다

```python
fig, axs = plt.subplots(4, 8, figsize=(15,8))
for i in range(4):
    for j in range(8):
        axs[i, j].imshow(feature_maps[0,:,:,i*8 + j])
        axs[i, j].axis('off')
plt.show()
```

- 특성 맵 32개를 4행 8열로 그렸다
- 첫 번째 합성곱 층의 특성 맵은 **앵클 부츠의 형태가 그대로 보인다**. 시각적인 정보를 감지한다

<!-- 캡처: 첫 번째 합성곱 층의 특성 맵 32개
<!-- ![사진7](/assets/img/posts/Self_Study_ML_DL/Self_Study_ML_DL_chap8_7.png) -->
-->

```python
conv2_acti = keras.Model(model.inputs, model.layers[2].output)
```

- `layers[2]`는 두 번째 합성곱 층이다 (`layers[1]`은 첫 번째 풀링 층)

```python
feature_maps = conv2_acti.predict(ankle_boot)
```

```python
print(feature_maps.shape)
```

```
(1, 14, 14, 64)
```

- 첫 번째 풀링 층에서 크기가 절반이 되었고, 필터가 64개이므로 (14, 14, 64)다

```python
fig, axs = plt.subplots(8, 8, figsize=(12,12))
for i in range(8):
    for j in range(8):
        axs[i, j].imshow(feature_maps[0,:,:,i*8 + j])
        axs[i, j].axis('off')
plt.show()
```

- 두 번째 합성곱 층의 특성 맵은 **무엇을 나타내는지 알아보기 어렵다**
    - 층이 깊어질수록 시각적인 형태가 아니라 **추상적인 정보**를 학습하기 때문이다

<!-- 캡처: 두 번째 합성곱 층의 특성 맵 64개
<!-- ![사진8](/assets/img/posts/Self_Study_ML_DL/Self_Study_ML_DL_chap8_8.png) -->
-->

### 시각화로 이해하는 합성곱 신경망

이번 절의 문제 해결 과정을 정리하면 다음과 같다.

1. **모델 불러오기**: 앞 절에서 저장한 `best-cnn-model.keras`를 복원
2. **가중치 시각화**: 훈련된 필터와 훈련하지 않은 필터의 분포·이미지를 비교 → 훈련으로 패턴을 학습했음을 확인
3. **함수형 API**: 중간 층의 출력을 꺼내는 모델을 따로 구성
4. **특성 맵 시각화**: 첫 번째 층은 형태가 보이고, 두 번째 층은 추상적 → **층이 깊어질수록 추상화된다**

<hr>

## 📌 정리

- **합성곱**은 입력의 일부에 가중치를 곱하며 훑어 특성 맵을 만든다. 패딩·스트라이드·풀링으로 크기를 조절한다
- (합성곱 + 풀링) → Flatten → 밀집층 구조로 패션 MNIST를 분류해 **정확도 약 0.92** 로 7장보다 향상되었다
- 층을 지날수록 특성 맵의 **너비·높이는 줄고 깊이(필터 개수)는 늘어난다**
- 가중치와 특성 맵을 시각화하면 첫 층은 시각적 형태를, 깊은 층은 **추상적인 정보**를 학습한다는 것을 볼 수 있다

**핵심 용어**: 합성곱 / 필터(커널) / 특성 맵 / 패딩 / 스트라이드 / 풀링 / 합성곱 신경망 / 함수형 API / 가중치 시각화 / 특성 맵 시각화

## 💬 스터디 토의

### 토의 주제 1. 합성곱 신경망에서 특성 맵의 크기를 줄이면서 필터의 개수를 늘리는 이유는 무엇일까? (08-1)

**CNN은 일반적으로 층이 깊어질수록 특성 맵의 너비와 높이는 감소하고, 필터의 개수는 증가하는 구조를 가진다. 그렇다면 공간적 정보가 줄어드는데도 더 복잡한 특징을 학습할 수 있는 이유는 무엇일까?**
- **특성 맵 크기를 줄이는 이유**
  - 특성 맵이 줄어들면 정보가 압축된다
  - 당장은 세밀한 위치를 잃는 것처럼 보이지만, 한 칸이 더 넓은 영역을 보게 된다
  - 그래서 깊은 층에서는 작은 선이 아니라 더 큰 부분과 전체 모양을 볼 수 있다
  - 계산량과 파라미터 수도 크게 줄어든다
- **필터 개수를 늘리는 이유**
  - 층이 깊을수록 표현해야 할 특징이 많아진다
  - 조합의 경우의 수가 훨씬 많으므로 필터도 더 필요하다

**필터를 거치면 압축되는데, 숫자로 이미지를 어떻게 표현할까?**
- **다시 펼쳐서 분류만 하면 된다**
  - CNN의 목적은 이미지를 다시 그리는 것이 아니라 분류하는 것이다
- **압축해도 필요한 정보는 다 있다. 숫자의 의미가 달라진다**
  - 밝기 → 모양이 있는 정도 → 추상적 특징이 있는 정도
