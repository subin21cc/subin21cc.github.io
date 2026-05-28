---
title: "[Capstone-Start] Dart 학습 — Generics, Sealed Class, freezed"
date: 2026-05-21 19:30:00 +0900
categories: [Capstone, Start]
tags: [dev, capstone]
---

**📅 일시:** 2026. 05. 21

**💬 내용:** Generics와 Dart 3의 sealed class, `freezed` 코드 생성으로 만든 `Result<T>`와 `AppError` 코드 회고

---

**코어 인프라의 에러 모델 부분**을 학습 노트로 정리했습니다. 구체적으로는 `core/errors/`에 만들어 둔 `AppError` sealed 클래스와 `Result<T>` 타입이 **Dart Generics와 Dart 3의 sealed class, 그리고 freezed 코드 생성**이라는 세 가지 기능을 합쳐 만든 결과물이라, 각 요소를 정리해 둡니다.

검색해서 들어오신 분들께 도움이 될 만한 순서로 정리합니다.

1. Generics — bounded와 variance
2. sealed class (Dart 3)
3. freezed 패키지 — 왜 쓰는가
4. 실제 코드: `AppError` + `Result<T>`
5. 회고

## 1. Dart의 Generics

C++ 템플릿이나 Java/Kotlin의 generic처럼 **타입 파라미터**를 선언해 코드를 재사용할 수 있게 만드는 기능입니다. Dart의 generic은 **invariant**가 기본이고, 키워드로 covariance를 약화시킬 수 있다는 점이 다른 언어와 비슷합니다.

### 기본 사용

```dart
class Box<T> {
  Box(this.value);
  final T value;
}

final intBox = Box<int>(42);
final stringBox = Box<String>('hello');
```

### Bounded generics — `extends`

타입 파라미터에 상한을 둘 수 있습니다. Java의 `T extends Comparable<T>`와 같은 의미입니다.

```dart
T maxOf<T extends Comparable<T>>(T a, T b) {
  return a.compareTo(b) >= 0 ? a : b;
}

maxOf(3, 5);          // OK — int는 Comparable<int>
maxOf('apple', 'b');  // OK — String은 Comparable<String>
```

### Generic 함수와 추론

타입 파라미터는 호출 시점에 인자로부터 추론됩니다. 명시적으로 박는 경우는 추론이 안 될 때만입니다.

```dart
List<T> wrap<T>(T value) => [value];

final ints = wrap(1);        // List<int>로 추론
final strings = wrap<String>(null as dynamic); // 명시 필요
```

## 2. Sealed Class — Dart 3의 Sum Type

`sealed`는 Dart 3에서 정식 도입된 키워드로, **하위 클래스를 같은 라이브러리(파일) 안에서만 정의할 수 있게** 제한합니다. Kotlin의 `sealed class`, Rust의 `enum`(데이터를 가질 수 있는 enum), TypeScript의 discriminated union과 같은 sum type 개념입니다.

### 왜 필요한가

일반 추상 클래스는 외부에서 임의로 상속될 수 있어서, **모든 경우를 한 자리에서 망라하는 패턴 매칭**이 컴파일러 차원에서 보장되지 않습니다. sealed는 하위 종류가 닫혀 있어 `switch`로 분기할 때 컴파일러가 **exhaustiveness**(모든 케이스를 다 처리했는지)를 검사해 줍니다.

```dart
sealed class Shape {}
class Circle extends Shape { final double r; Circle(this.r); }
class Square extends Shape { final double a; Square(this.a); }

double area(Shape s) => switch (s) {
  Circle(:final r) => 3.14 * r * r,
  Square(:final a) => a * a,
  // 케이스를 빠뜨리면 컴파일 에러
};
```

여기서 `switch (s) { ... }` 형태는 **switch expression**입니다(내일 별도 글로 정리할 예정입니다). 핵심은 `Circle`/`Square` 두 케이스를 모두 다루지 않으면 컴파일러가 막아준다는 점입니다.

## 3. freezed — 왜 쓰는가

`AppError` 같은 sealed 계층을 직접 손으로 짜면 보일러플레이트가 많습니다.

- 각 변형(variant)마다 별도 클래스 선언
- `==` / `hashCode` 직접 구현
- `copyWith` 직접 구현
- `toString` 가독성용 override
- JSON 직렬화가 필요하면 fromJson/toJson

**[freezed](https://pub.dev/packages/freezed)** 패키지는 어노테이션을 붙이면 위의 보일러플레이트를 모두 코드 생성으로 만들어 줍니다. 캡스톤에서는 도메인 모델과 상태(state) 표현 거의 모두에 freezed를 적용할 계획입니다.

### 의존성 추가

```yaml
dependencies:
  freezed_annotation: ^3.0.0
  json_annotation: ^4.9.0

dev_dependencies:
  build_runner: ^2.4.13
  freezed: ^3.0.0
  json_serializable: ^6.8.0
```

`*_annotation`은 런타임 의존성(어노테이션 클래스만 들어 있음), 본체는 `dev_dependencies`로 들어가서 빌드 시점에만 동작합니다.

### 코드 생성 실행

```bash
dart run build_runner build --delete-conflicting-outputs
```

`*.g.dart`, `*.freezed.dart` 파일이 자동으로 생성됩니다. watch 모드(`build_runner watch`)로 두면 파일이 바뀔 때마다 자동 재생성됩니다.

## 4. 실제 코드 — `AppError` + `Result<T>`

어제 짠 `core/errors/`의 핵심 두 타입을 정리합니다. 두 타입 모두 sealed + generics + freezed의 조합입니다.

### `AppError` — 도메인 에러 sealed union

```dart
import 'package:freezed_annotation/freezed_annotation.dart';
part 'app_error.freezed.dart';

@freezed
sealed class AppError with _$AppError {
  const factory AppError.network(String message) = NetworkError;
  const factory AppError.notFound(String resource) = NotFoundError;
  const factory AppError.unauthorized() = UnauthorizedError;
  const factory AppError.unknown(Object error, [StackTrace? trace]) =
      UnknownError;
}
```

각 `factory`는 별도의 **variant**가 됩니다. `NetworkError`, `NotFoundError`, `UnauthorizedError`, `UnknownError`가 모두 `AppError`의 하위 타입이고, freezed가 자동으로 만들어 줍니다.

호출부에서는 다음처럼 변형을 패턴 매칭으로 분기합니다.

```dart
String describe(AppError e) => switch (e) {
  NetworkError(:final message) => '네트워크 오류: $message',
  NotFoundError(:final resource) => '$resource을(를) 찾을 수 없습니다',
  UnauthorizedError() => '로그인이 필요합니다',
  UnknownError(:final error) => '알 수 없는 오류: $error',
};
```

새 variant를 추가하면 이 switch가 컴파일러 에러로 막혀서 모든 사용처를 강제로 업데이트하게 됩니다. **에러 처리 누락**을 컴파일러가 잡아주는 패턴이라 안전합니다.

### `Result<T>` — `Either`의 슬림 버전

Either / Result 패턴은 함수형 언어에서 자주 보는 "성공 또는 실패를 값으로 표현" 트릭입니다. Dart에는 표준 `Result` 타입이 없어서, sealed + generics로 직접 만들었습니다.

```dart
sealed class Result<T> {
  const Result();
}

class Ok<T> extends Result<T> {
  const Ok(this.value);
  final T value;
}

class Err<T> extends Result<T> {
  const Err(this.error);
  final AppError error;
}
```

여기서 `T`는 generic 타입 파라미터입니다. `Result<User>`, `Result<List<Meal>>`처럼 임의의 도메인 타입을 담을 수 있습니다.

### Controller(Riverpod notifier)에서의 사용

```dart
class DietController extends StateNotifier<AsyncValue<DietDay>> {
  DietController(this._repository) : super(const AsyncLoading());

  final DietRepository _repository;

  Future<void> load() async {
    state = const AsyncLoading();
    final result = await _repository.fetchToday();
    state = switch (result) {
      Ok(:final value) => AsyncData(value),
      Err(:final error) => AsyncError(error, StackTrace.current),
    };
  }
}
```

`switch`로 두 케이스를 한 자리에서 분기하니, repository가 던질 수 있는 모든 결과를 controller에서 명시적으로 다루게 됩니다. `try/catch`로 풀면 어디서 어떤 예외가 어디로 흘러가는지 추적이 어려운데, sealed + Result로 명시하면 데이터 흐름이 한눈에 보입니다.

## 5. 회고 — sum type이 헬스케어 도메인에 잘 맞는 이유

캡스톤의 데이터 모델을 보면 sum type이 자연스럽게 등장하는 자리가 많습니다.

- **운동 종류** — 유산소 / 근력 / 스트레칭 (세 가지가 모두 다른 필드를 가짐)
- **알림 종류** — 식사 / 운동 / 정기 점검 (각 페이로드가 다름)
- **건강 위험도** — 정상 / 경계 / 위험 (구간별 메시지 다름)
- **분석 결과 상태** — Pending / Success / Failure

이 모든 케이스를 `String` 필드 한 줄로 표현하면, 코드 곳곳에 `if (kind == 'cardio') ...`가 흩어지면서 새 종류를 추가했을 때 무엇을 수정해야 하는지가 한눈에 안 보입니다. sealed class로 모델링하면 컴파일러가 빠뜨린 곳을 콕 집어주기 때문에, 도메인이 확장될 때 리스크가 낮아집니다.

## 6. 정리 / 다음 학습 계획

오늘 정리한 핵심은 다음 세 가지입니다.

1. **Generics**는 invariant가 기본이고, `extends`로 상한을 둘 수 있습니다. `Result<T>`처럼 임의의 도메인 타입을 담는 컨테이너 만들 때 필수입니다.
2. **sealed class**는 Dart 3의 sum type이고, switch 분기에서 컴파일러가 exhaustiveness를 검사해 줍니다. 도메인 에러 / 알림 종류 / 운동 카테고리 같은 닫힌 집합 표현에 적합합니다.
3. **freezed**는 sealed + immutable data class + copyWith + 동등성 비교를 한 번에 자동 생성해 줍니다. 캡스톤의 모든 도메인 모델에 적용할 계획입니다.

다음 학습은 오늘 코드에 살짝 등장한 **`switch expression`과 패턴 매칭** — Dart 3에서 새로 들어온 기능들을 본격적으로 정리하려 합니다. 캡스톤 후반에 본격적으로 쓰게 될 거라 미리 학습 노트로 남겨 둘 예정입니다.

---

## 📌 요약

`AppError` sealed와 `Result<T>` generic 타입을 만들면서 학습한 내용을 정리했습니다. **Generics**는 임의 도메인 타입을 담는 컨테이너를 만들 때 쓰고, **sealed class**는 컴파일러가 exhaustiveness를 검사해 주는 닫힌 합 타입이며, **freezed**는 어노테이션 한 줄로 immutable data class + copyWith + 동등성을 자동 생성합니다. 세 가지를 합치면 헬스케어 도메인의 닫힌 변형들(에러·운동 종류·위험도 등)을 안전하게 모델링할 수 있고, 새 변형을 추가했을 때 컴파일러가 사용처를 강제로 업데이트하게 만들어 줍니다.
