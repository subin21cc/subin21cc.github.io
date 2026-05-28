---
title: "[Capstone-Start] Dart 학습 — Records, Patterns, Switch Expression"
date: 2026-05-26 22:30:00 +0900
categories: [Capstone, Start]
tags: [dev, capstone]
---

**📅 일시:** 2026. 05. 26

**💬 내용:** Dart 3의 새 기능 — Records, Pattern Matching, Switch Expression이 실제 코드에 어떻게 들어갔는지 정리

---

캡스톤 시연 환경이 어제(5/25) 안정화된 후, 오늘은 그동안 짠 코드를 한 번 훑으며 정리하는 시간을 가졌습니다. 그러면서 깨달은 게, **Dart 3에서 새로 들어온 records / patterns / switch expression이 코드 곳곳에 자연스럽게 박혀 있다**는 점입니다. Stage 8에서 `(_, _)` wildcard를 쓰거나, Stage 2에서 `Result<T>` switch matching을 깔거나, fl_chart 데이터에 `(double x, double y)` records를 쓰는 등의 흔적이 남아 있었습니다.

처음에 이 기능들을 쓸 때는 "Kotlin 비슷한 것"이라는 직관으로 박았는데, 오늘 다시 정리하면서 정식 학습 노트로 남겨 둡니다. 검색해서 들어오신 분께 도움이 될 수 있도록 **Dart 2 시절 코드와 Dart 3 코드의 before/after**를 같이 보여드리는 형태로 정리합니다.

## 1. Records — 즉석에서 만드는 tuple

**Record**는 여러 값을 하나의 immutable 묶음으로 만드는 경량 타입입니다. Python의 tuple, Kotlin의 Pair/Triple, TypeScript의 anonymous object와 비슷한 개념입니다.

### Positional Records

```dart
(int, String) pair = (1, 'apple');

print(pair.$1); // 1
print(pair.$2); // apple
```

필드명이 없는 record는 `$1`, `$2` 식으로 위치 인덱스로 접근합니다. **함수가 여러 값을 반환**해야 할 때 별도 클래스를 만들지 않고 즉석에서 쓸 수 있어 편리합니다.

```dart
(int, int) divmod(int a, int b) => (a ~/ b, a % b);

final (q, r) = divmod(17, 5); // q = 3, r = 2
```

좌변에 `(q, r)` 패턴을 쓰면 한 줄에 destructuring이 됩니다.

### Named Records — `({ ... })`

```dart
({int x, int y}) point = (x: 10, y: 20);

print(point.x); // 10
print(point.y); // 20
```

필드명이 있는 record는 점 표기로 접근할 수 있고, 시그니처가 더 명확해집니다. 캡스톤의 fl_chart 래퍼에서는 좌표를 `({double x, double y})` 형태의 record로 받도록 했습니다.

```dart
class AppLineChart extends StatelessWidget {
  const AppLineChart({super.key, required this.points, ...});
  final List<({double x, double y})> points;
}

// 호출
AppLineChart(points: [
  (x: 0, y: 70),
  (x: 1, y: 71),
  (x: 2, y: 72.5),
]);
```

차트 라이브러리 시그니처에 도메인 타입을 노출하지 않으면서도, `point.x`/`point.y` 접근이 가능한 가독성을 얻었습니다. **임시 데이터 묶음**에는 record가 거의 항상 답입니다.

### Records를 클래스 대신 쓰는 가이드

다만 record가 만능은 아닙니다. 다음 기준으로 record vs class를 결정합니다.

- **이름이 의미를 가지면 class** — `User` / `Meal`처럼 도메인 의미가 있는 객체는 클래스.
- **필드가 3개 이상이고 재사용된다면 class** — record는 시그니처가 길어지면 가독성이 떨어집니다.
- **메서드를 붙이고 싶으면 class** — record는 메서드를 가질 수 없습니다.
- **로컬·임시·반환값 묶음이면 record** — 짧게 살아 있을 데이터에 적합.

## 2. Patterns — destructuring + 조건 매칭

Dart 3의 **패턴**은 두 가지 역할을 동시에 합니다.

1. **Destructuring** — 복합 구조에서 값을 꺼내기
2. **Matching** — 어떤 형태인지 검사

```dart
// 1. Variable pattern — 그냥 변수에 담기
final user = User(id: 1, name: '민수');
final User(:id, :name) = user;  // id = 1, name = '민수'

// 2. List pattern — 리스트 형태와 동시에 분해
final list = [1, 2, 3];
if (list case [final first, _, final last]) {
  print('$first, $last'); // 1, 3
}

// 3. Map pattern
final json = {'type': 'circle', 'r': 5};
if (json case {'type': 'circle', 'r': final r}) {
  print(r); // 5
}
```

### `:` 단축 (shorthand)

`Object(:fieldName)`은 `Object(fieldName: fieldName)`의 단축 표기입니다. 같은 이름의 변수에 바인딩할 때 한 글자라도 줄이려는 syntactic sugar입니다.

```dart
// 풀어쓴 형태
final Circle(r: r) = circle;

// 단축형
final Circle(:r) = circle;
```

### Wildcard `_` 패턴 — Stage 8에서 실제 사용

이 부분이 캡스톤 코드에 가장 빈번하게 등장합니다. `separatorBuilder` 같은 콜백은 인자 두 개를 받지만 둘 다 안 쓰는 경우가 많습니다. 예전(Dart 2)에는 다음처럼 썼습니다.

```dart
// Dart 2 — 미사용 인자에 이름을 붙여야 했음
ListView.separated(
  itemCount: items.length,
  separatorBuilder: (context, index) => const Divider(),
  itemBuilder: ...,
)
```

Dart 3에서는 wildcard `_`로 "안 쓴다"는 의도를 명시할 수 있습니다.

```dart
// Dart 3 — 미사용 인자를 wildcard로
ListView.separated(
  itemCount: items.length,
  separatorBuilder: (_, _) => const Divider(),
  itemBuilder: (_, i) => ListTile(title: Text(items[i].name)),
)
```

Stage 8에서 lint가 잡아서 일괄로 정리했던 흔적이 있는데, 그게 이 패턴입니다.

```
style: drop deprecated activeColor + use Dart 3 wildcards (phase 8.8 fixup)
style(notification): use Dart 3 wildcard `(_, _)` in separatorBuilder
```

## 3. Switch Expression — 식(expression)으로서의 switch

Dart 2 시절의 `switch`는 **statement**라 변수에 직접 할당할 수 없었습니다. 다음처럼 if 사슬을 쓰거나, 함수로 감싸야 했습니다.

```dart
// Dart 2
String label;
if (status == Status.success) {
  label = '성공';
} else if (status == Status.failure) {
  label = '실패';
} else {
  label = '진행 중';
}
```

Dart 3의 **switch expression**은 식(expression)이라 변수에 직접 할당할 수 있고, **모든 케이스를 처리하지 않으면 컴파일 에러**가 발생합니다.

```dart
final label = switch (status) {
  Status.success => '성공',
  Status.failure => '실패',
  Status.inProgress => '진행 중',
};
```

`break`도 없고, `:`이 아니라 `=>` 화살표 표기를 씁니다. 표현식 한 줄이라 가독성이 매우 좋습니다.

### sealed class와 결합

지난 글에서 정리한 `AppError` sealed와 결합하면 진가가 드러납니다.

```dart
String describe(AppError e) => switch (e) {
  NetworkError(:final message) => '네트워크 오류: $message',
  NotFoundError(:final resource) => '$resource을(를) 찾을 수 없습니다',
  UnauthorizedError() => '로그인이 필요합니다',
  UnknownError(:final error) => '알 수 없는 오류: $error',
};
```

새 variant를 `AppError`에 추가하면 위 switch가 **컴파일 에러**로 막혀서, 사용처를 강제로 업데이트하게 됩니다. 도메인 변경 시 누락을 컴파일러가 잡아주는 안전망이 됩니다.

### `if-case` — 단일 패턴 매칭

전체 분기가 아니라 **특정 패턴만 매칭**하고 싶으면 `if-case` 구문을 씁니다.

```dart
if (result case Ok(:final value)) {
  print('성공: $value');
}
```

이게 굉장히 유용한 패턴입니다. `Result<T>`에서 성공 케이스만 분기하고 실패는 다른 흐름으로 보내고 싶을 때 한 줄로 가능합니다.

## 4. Guard 조건 — `when`

패턴 매칭에 추가 조건을 붙이고 싶을 때 `when` 키워드를 씁니다. 패턴이 매칭되더라도 `when` 조건이 false면 다음 케이스로 넘어갑니다.

```dart
String classify(int score) => switch (score) {
  >= 90 => 'A',
  >= 80 => 'B',
  >= 70 => 'C',
  _ when score < 0 => 'invalid',
  _ => 'F',
};
```

위에서는 **relational pattern**(`>= 90`)도 쓰고 있습니다. Dart 3가 깔아 놓은 패턴 종류가 워낙 많아서, 같은 표현식 하나에 여러 패턴을 조합할 수 있습니다.

## 5. 실제 캡스톤 코드 예시 모음

학습 정리 차원에서 캡스톤에서 이 기능들이 실제로 어디에 들어갔는지 한 자리에 모아 봤습니다.

### Records — fl_chart 좌표

```dart
final List<({double x, double y})> weightPoints = [
  (x: 0, y: 70.5),
  (x: 1, y: 70.8),
  (x: 2, y: 71.2),
  // ...
];
```

`Map<String, double>`이나 별도 클래스보다 훨씬 가볍습니다.

### Switch expression — Result 매칭

```dart
state = switch (result) {
  Ok(:final value) => AsyncData(value),
  Err(:final error) => AsyncError(error, StackTrace.current),
};
```

repository가 반환한 `Result<DietDay>`를 Riverpod의 `AsyncValue`로 한 줄에 변환합니다.

### Wildcard — 미사용 콜백 인자

```dart
ListView.separated(
  itemCount: notifications.length,
  separatorBuilder: (_, _) => const Divider(height: 1),
  itemBuilder: (_, i) => NotificationTile(item: notifications[i]),
)
```

### `if-case` — Stream 이벤트 분기

```dart
notificationStream.listen((event) {
  if (event case PushArrived(:final notification)) {
    inAppCenter.add(notification);
  }
});
```

`PushArrived`가 sealed family의 한 variant라면, if-case로 자연스럽게 unpack됩니다.

## 6. 정리 / 다음 학습 계획

Dart 3의 records / patterns / switch expression은 처음 봤을 때는 "Kotlin이나 Rust 같은 것"이라고만 느꼈는데, 실제로 코드에 박아 보니 다음 효과가 분명했습니다.

1. **임시 데이터 묶음**의 보일러플레이트가 사라집니다. record로 끝.
2. **sealed class와 결합**할 때 exhaustiveness 검사가 컴파일러 안전망이 됩니다.
3. **콜백 시그니처가 깔끔**해집니다. `_`로 무관 인자를 명시하면 의도가 분명히 드러납니다.

다음 학습은 캡스톤 코드를 다시 한 번 훑으면서 발견한 또 다른 Dart 메타 패턴 — **Extension methods, Mixin, build_runner를 통한 코드 생성**을 정리할 예정입니다. 학기 마지막 학습 노트가 될 것 같습니다.

---

## 📌 요약

Dart 3의 **records / patterns / switch expression**은 캡스톤 코드에 자연스럽게 녹아든 핵심 기능들입니다. records는 임시 데이터 묶음(차트 좌표 등)에서 보일러플레이트를 없애고, switch expression은 sealed class와 결합해 모든 variant를 망라하지 않으면 컴파일 에러를 띄워 도메인 확장 시 안전망이 됩니다. `_` wildcard 패턴은 콜백의 미사용 인자를 명시적으로 표현해 가독성을 높이고, `if-case`로 단일 패턴 매칭이 한 줄에 가능합니다. 적은 문법 추가로 표현력이 크게 늘어난, Dart 3에서 가장 만족스러운 변화였습니다.
