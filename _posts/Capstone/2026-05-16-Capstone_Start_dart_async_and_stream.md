---
title: "[Capstone-Start] Dart 학습 — Future, async/await, Stream"
date: 2026-05-16 23:30:00 +0900
categories: [Capstone, Start]
tags: [dev, capstone]
---

**📅 일시:** 2026. 05. 16

**💬 내용:** Dart의 비동기 모델 — `Future<T>`, `async`/`await`, `Stream<T>`와 JavaScript Promise 비교

---

어제는 Dart의 타입 시스템을 정리했고, 오늘은 두 번째 학습 주제로 **비동기 모델**을 공부했습니다. Flutter는 API 호출, 파일 입출력, 애니메이션 등 비동기 코드가 도처에 있고, 핵심 위젯인 `FutureBuilder`/`StreamBuilder`가 비동기 결과를 위젯 트리에 직접 묶어주는 구조라 이 부분을 모르면 한 줄도 못 짭니다.

다행히 Dart의 비동기 모델은 **JavaScript와 거의 동일**합니다. Single-threaded 이벤트 루프 위에서 `Future`와 `Stream`이 돌고, `async`/`await` 문법도 같습니다. 그래서 학습 자체는 어렵지 않았는데, **Dart만의 디테일**(예: `unawaited`, `async*`/`yield`, 에러 전파)을 정리해 둡니다.

## 1. Dart의 동시성 모델 — Single-threaded 이벤트 루프

Dart는 기본적으로 **single isolate 안에서 single-threaded**로 동작합니다. JavaScript의 V8과 같은 모델로, 모든 비동기 작업은 이벤트 큐에 등록되었다가 순차적으로 처리됩니다.

```
┌─────────────────────────────────────┐
│         Dart Isolate                │
│  ┌─────────┐    ┌──────────────┐    │
│  │  Stack  │    │  Event Loop  │    │
│  └─────────┘    └───────┬──────┘    │
│        ▲                ▼           │
│        │     ┌───────────────────┐  │
│        └─────│  Event Queue      │  │
│              │  Microtask Queue  │  │
│              └───────────────────┘  │
└─────────────────────────────────────┘
```

병렬 처리가 필요하면 **Isolate**(별도 프로세스에 가까운 격리 단위)를 띄워야 하는데, 캡스톤 범위에서는 이미지 처리나 무거운 JSON 파싱 정도가 아니면 거의 안 쓴다고 합니다. 일단 single isolate로 충분합니다.

## 2. `Future<T>` — Dart 버전의 Promise

`Future<T>`는 "**언젠가 `T` 타입 값을 반환할 것**"이라는 미래의 약속입니다. JavaScript의 `Promise<T>`와 의미·동작이 거의 동일합니다.

```dart
Future<int> fetchUserId() async {
  await Future.delayed(const Duration(seconds: 1));
  return 42;
}
```

### `then` 체이닝 vs `async`/`await`

`Future`는 `.then` / `.catchError` / `.whenComplete`로도 다룰 수 있지만, 가독성을 위해 거의 모든 코드에서 `async`/`await`을 씁니다.

```dart
// .then 방식 — 가독성 떨어짐
fetchUserId()
  .then((id) => fetchUser(id))
  .then((user) => print(user.name))
  .catchError((e) => print('에러: $e'));

// async/await — sequential하게 읽힘
try {
  final id = await fetchUserId();
  final user = await fetchUser(id);
  print(user.name);
} catch (e) {
  print('에러: $e');
}
```

### 병렬 처리 — `Future.wait`

여러 Future를 동시에 기다리려면 `Future.wait`을 씁니다. TypeScript의 `Promise.all`과 같습니다.

```dart
final results = await Future.wait([
  fetchUser(1),
  fetchUser(2),
  fetchUser(3),
]);
// results는 List<User>
```

병렬 실행이 의미 있는 건 **독립적인 작업**일 때입니다. 서로 의존하면 그냥 `await`을 두 번 부르는 게 맞습니다.

```dart
// ❌ 잘못 — 두 호출이 독립적인데 sequential
final user = await fetchUser(1);
final orders = await fetchOrders(1);

// ✅ 병렬
final results = await Future.wait([fetchUser(1), fetchOrders(1)]);
final user = results[0] as User;
final orders = results[1] as List<Order>;
```

### `unawaited` — 의도적으로 기다리지 않을 때

비동기 함수를 호출하면서 결과를 기다리지 않으면, Dart 분석기가 `unawaited_futures` 경고를 띄웁니다. **로깅이나 fire-and-forget 패턴**처럼 의도적으로 무시할 때는 `unawaited()` 함수로 명시합니다.

```dart
import 'dart:async';

void main() {
  unawaited(logToServer('app_started')); // 결과를 기다리지 않음을 명시
  // 다음 작업으로 진행
}
```

이 작은 wrapper 한 줄이 "버그가 아니다"라는 의도를 코드에 박아주는 역할을 합니다.

## 3. `async`/`await`은 JavaScript와 거의 같다

문법은 JavaScript와 동일합니다. `async`로 표시된 함수는 자동으로 `Future`를 반환하고, `await`은 Future를 풀어서 값을 꺼냅니다.

```dart
Future<User> fetchUser(int id) async {
  final response = await dio.get('/users/$id');
  return User.fromJson(response.data);
}
```

다만 한 가지 다른 점이 있습니다. **JavaScript는 async 함수가 명시적으로 `return`이 없어도 `Promise<void>`를 반환**하지만, Dart는 **`Future<void>`를 명시적으로 선언해야 분석기가 만족**합니다.

```dart
Future<void> sendLog(String event) async {
  await dio.post('/logs', data: {'event': event});
}
```

### 에러 전파

`async` 함수 안에서 throw하면 그대로 `Future`가 reject됩니다. `try`/`catch`로 잡을 수 있습니다.

```dart
Future<User> fetchUser(int id) async {
  try {
    final response = await dio.get('/users/$id');
    return User.fromJson(response.data);
  } on DioException catch (e) {
    // dio가 던진 4xx/5xx
    throw NetworkError(e.message ?? 'unknown');
  } catch (e) {
    // 그 외 모든 예외
    throw UnknownError(e);
  }
}
```

`on TypeName catch (e)` 구문은 Java의 `catch (TypeName e)`와 같습니다. **타입별로 catch 블록을 분기**할 수 있어 가독성이 좋습니다.

## 4. `Stream<T>` — 여러 값을 시간차로 emit

`Future`는 "한 번 값이 오고 끝"이지만, `Stream`은 "값이 여러 번 시간차로 emit된다"는 의미입니다. RxJS의 Observable이나 Kotlin의 Flow와 비슷한 개념입니다.

### Stream 구독

```dart
final clock = Stream.periodic(
  const Duration(seconds: 1),
  (count) => count,
);

await for (final tick in clock) {
  print('tick: $tick');
  if (tick >= 3) break;
}
```

또는 `.listen`을 직접 호출할 수도 있습니다.

```dart
final subscription = clock.listen((tick) {
  print('tick: $tick');
});

// 나중에 구독 해제
await subscription.cancel();
```

### `async*` / `yield` — Stream 생성

함수 자체가 Stream을 emit하도록 만들려면 `async*`와 `yield`를 씁니다. Python의 generator와 같은 메커니즘입니다.

```dart
Stream<int> countTo(int n) async* {
  for (var i = 1; i <= n; i++) {
    await Future.delayed(const Duration(milliseconds: 500));
    yield i;
  }
}

void main() async {
  await for (final v in countTo(5)) {
    print(v); // 1, 2, 3, 4, 5
  }
}
```

### Broadcast Stream

기본 `Stream`은 **single-subscription**이라 한 번에 한 명만 구독할 수 있습니다. 여러 곳에서 같은 이벤트를 보려면 `.asBroadcastStream()`으로 broadcast로 바꿔야 합니다. 알림 패널, 위치 추적 같은 곳에서 자주 쓰입니다.

```dart
final broadcast = sourceStream.asBroadcastStream();
broadcast.listen((e) => print('subscriber A: $e'));
broadcast.listen((e) => print('subscriber B: $e'));
```

## 5. 캡스톤에서 미리 그려본 사용 시나리오

아직 코드를 짜진 않았지만, 학습한 내용을 캡스톤 화면에 어떻게 매핑할지 머릿속으로 그려봤습니다.

| 화면 / 기능 | 비동기 패턴 | Future / Stream |
| --- | --- | --- |
| 식단 분석 API 호출 | 한 번 호출 후 결과 | `Future<DietAnalysis>` |
| 대시보드 데이터 fetch | 여러 도메인 병렬 호출 | `Future.wait` |
| 알림 패널 실시간 갱신 | 시간차로 도착하는 이벤트 | `Stream<Notification>` |
| 운동 타이머 / 스트릭 카운터 | 1초마다 emit | `Stream.periodic` |
| 위치 추적 (Place 화면) | 위치가 바뀔 때마다 emit | `Stream<Position>` |

Flutter에서는 `FutureBuilder<T>`와 `StreamBuilder<T>`가 이 두 패턴을 위젯 트리에 직접 묶어주는 위젯이라고 합니다. `snapshot.connectionState`로 로딩/완료/에러 상태를 분기하는 흐름이 표준 패턴입니다.

## 6. 학습 후 알게 된 것 / 내일 계획

오늘 정리하면서 알게 된 가장 중요한 포인트는 다음입니다.

1. **Dart 비동기는 JavaScript와 모델·문법이 거의 동일**합니다. `Future` = `Promise`, `async`/`await`도 같습니다. 학습 비용이 거의 없다는 게 큰 강점입니다.
2. **`Stream`은 한 번이 아니라 여러 번 emit되는 비동기**입니다. RxJS Observable과 같은 개념이라 위젯에 실시간 데이터를 묶을 때 자연스럽습니다.
3. **`unawaited()`로 fire-and-forget을 명시**합니다. 의도하지 않은 누락된 await를 잡기 위한 안전장치입니다.

내일은 Dart의 **함수 — 일급 객체, typedef, 클로저**를 공부할 계획입니다. Flutter의 핵심 패턴인 Builder(`ListView.builder`, `FutureBuilder` 등)가 모두 "함수를 인자로 받는 위젯"이라, 함수 타입을 잘 다루지 못하면 위젯 코드가 깨끗하게 안 나옵니다.

---

## 📌 요약

Dart의 비동기 모델은 JavaScript와 거의 동일한 single-threaded 이벤트 루프 위의 `Future<T>` + `Stream<T>` 패턴입니다. `Future`는 한 번 값, `Stream`은 여러 번 값입니다. `Future.wait`로 병렬 처리, `unawaited`로 의도적 무시를 표현하고, `async*`/`yield`로 Stream을 직접 emit할 수 있습니다. Flutter에서는 이 두 비동기 타입이 `FutureBuilder`/`StreamBuilder`로 위젯 트리에 직접 연결되는 구조라 비동기 모델 이해가 화면 코드 작성의 기초가 됩니다.
