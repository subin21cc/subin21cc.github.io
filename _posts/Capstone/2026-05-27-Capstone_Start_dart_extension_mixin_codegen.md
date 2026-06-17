---
title: "[Capstone-Start] Dart 학습 — Extension Methods, Mixin, Code Generation"
date: 2026-05-27 23:30:00 +0900
categories: [Capstone, Start]
tags: [dev, capstone]
---

**📅 일시:** 2026. 05. 27

**💬 내용:** Extension methods, Mixin, `build_runner` 코드 생성 — Flutter 개발에서 자주 보는 메타 패턴 정리

---

시연 환경 안정화는 어제(5/26)까지 끝났고, 오늘은 캡스톤 코드를 다시 한 번 훑으면서 발견한 **Dart의 메타 패턴 3가지 — Extension methods, Mixin, build_runner를 통한 코드 생성**을 정리합니다.

이 세 가지는 처음 Flutter 코드를 볼 때 가장 낯설게 느껴졌던 키워드들입니다. `with WidgetsBindingObserver`, `extension StringX on String`, `*.g.dart` 자동 생성 파일이 어떤 메커니즘인지 처음에는 한 번씩 멈춰서 찾아봐야 했습니다. 검색해서 들어오신 분들께 도움이 될 만한 정리로 남깁니다.

## 1. Extension Methods — 타입에 외부에서 메서드 추가

**Extension**은 기존 타입(클래스/인터페이스/built-in)에 **소스 코드를 수정하지 않고 메서드를 추가**하는 기능입니다. C#의 extension method, Kotlin의 확장 함수, Swift의 extension과 동일한 개념입니다.

### 기본 사용

```dart
extension StringExtensions on String {
  String get reversed => split('').reversed.join();
  bool get isBlank => trim().isEmpty;
}

void main() {
  print('Hello'.reversed); // 'olleH'
  print('  '.isBlank);     // true
}
```

`extension <이름> on <타입>` 구문으로 추가합니다. 메서드는 마치 그 타입의 원래 메서드인 것처럼 점 표기로 호출됩니다.

### 캡스톤에서 사용한 예 — `DateTime` 확장

식단 페이지에서 일요일~토요일 주간 스트립을 그릴 때, Dart의 `DateTime.weekday`가 **월요일이 1, 일요일이 7**인 ISO 기준이라 매번 `weekday % 7`을 부르는 게 거슬렸습니다. 그래서 `shared/extensions/`에 `date_x.dart`로 확장을 묶었습니다.

```dart
extension DateTimeX on DateTime {
  /// 일요일을 0으로 두는 weekday (Diet 페이지 주간 스트립용)
  int get sundayIndex => weekday % 7;

  /// 해당 주의 일요일 0시
  DateTime startOfWeek() {
    final sundayOffset = sundayIndex;
    return DateTime(year, month, day - sundayOffset);
  }

  /// 시간을 0:00으로 잘라낸 같은 날짜
  DateTime get atMidnight => DateTime(year, month, day);
}
```

이렇게 묶어두니 호출부가 깔끔해집니다.

```dart
final monday = today.startOfWeek().add(const Duration(days: 1));
final index = selectedDate.sundayIndex;
```

### 주의점 — 정적 분석에서만 동작

Extension은 **컴파일러가 정적으로 호출 위치를 보고** 어떤 extension의 메서드인지 결정합니다. 런타임 다형성은 적용되지 않습니다.

```dart
extension A on Object { String name() => 'A'; }
extension B on String { String name() => 'B'; }

Object x = 'hello';
print(x.name()); // 'A' — 정적 타입은 Object
print((x as String).name()); // 'B'
```

이 동작은 Java의 정적 메서드 디스패치와 유사합니다. extension을 "런타임 메서드 추가"로 오해하면 함정에 빠질 수 있습니다.

### Generic Extension

`extension`도 generic을 받을 수 있습니다. 캡스톤에서는 `Iterable`에 자주 쓰는 헬퍼를 묶어 두었습니다.

```dart
extension IterableX<T> on Iterable<T> {
  T? firstOrNull(bool Function(T) test) {
    for (final e in this) {
      if (test(e)) return e;
    }
    return null;
  }
}

final firstHigh = scores.firstOrNull((s) => s > 90);
```

표준 라이브러리에 `firstWhere`가 있지만, 조건에 맞는 게 없으면 throw하는 정책이 불편할 때 위 헬퍼가 유용합니다.

## 2. Mixin — `with` 키워드로 합성하는 행위

**Mixin**은 클래스에 **공통 행위(메서드/필드)를 합성**하는 메커니즘입니다. Dart는 다중 상속을 지원하지 않지만, mixin으로 비슷한 효과를 안전하게 얻을 수 있습니다.

### 기본 사용

```dart
mixin Loggable {
  void log(String message) {
    print('[$runtimeType] $message');
  }
}

class UserController with Loggable {
  void load() {
    log('loading user'); // mixin이 제공한 메서드
  }
}
```

`with`로 mixin을 합성하면 그 클래스에 mixin의 멤버가 그대로 박힙니다. 여러 mixin을 합성할 수도 있습니다.

```dart
class HomeController with Loggable, ChangeNotifier { ... }
```

### Flutter에서 자주 보는 mixin

Flutter SDK는 mixin을 굉장히 많이 씁니다. 다음은 캡스톤에서 직접 마주친 mixin들입니다.

| Mixin | 용도 |
| --- | --- |
| `ChangeNotifier` | listener 패턴 (`addListener` / `notifyListeners`) |
| `WidgetsBindingObserver` | 앱 라이프사이클 (`didChangeAppLifecycleState`) |
| `SingleTickerProviderStateMixin` | 애니메이션 컨트롤러용 `vsync` 제공 |
| `AutomaticKeepAliveClientMixin` | 탭 전환 시 위젯 상태 유지 |

```dart
class _ExercisePageState extends State<ExercisePage>
    with SingleTickerProviderStateMixin {
  late final TabController _tabController =
      TabController(length: 2, vsync: this);
  // ...
}
```

`vsync: this`로 자신을 ticker provider로 넘기는 패턴인데, mixin이 제공한 `createTicker` 메서드 덕에 가능합니다.

### `on` 키워드 — mixin 사용처 제한

mixin은 특정 클래스의 하위에서만 쓰일 수 있도록 `on`으로 제약할 수 있습니다. mixin이 의존하는 메서드/필드가 부모 클래스에 있을 때 유용합니다.

```dart
mixin ScrollLogger on State<StatefulWidget> {
  void logScroll(double offset) {
    debugPrint('[$widget] offset=$offset');
  }
}
```

위 mixin은 `State<StatefulWidget>` 또는 그 하위 클래스에만 합성될 수 있습니다.

### Extension vs Mixin — 언제 무엇을 쓸까

같은 듯 다른 두 기능을 다음 기준으로 구분합니다.

| 기준 | Extension | Mixin |
| --- | --- | --- |
| 대상 | 기존 타입 (수정 불가) | 새 클래스를 만들 때 합성 |
| 상태 | 가질 수 없음 | 가질 수 있음 |
| `this` 접근 | extension 안에서 `this` 가능 | 인스턴스의 일부가 됨 |
| 다형성 | 정적 디스패치 | 동적 디스패치 |
| 충돌 | 타입에 같은 이름 있으면 원래 우선 | 합성 순서로 결정 |

요약하면, **이미 있는 타입에 가벼운 유틸리티를 붙이고 싶으면 extension**, **새 클래스를 만들 때 행위/상태를 합성하고 싶으면 mixin**입니다.

## 3. Code Generation — `build_runner`로 보일러플레이트 자동화

Dart는 매크로(컴파일 시점에 코드를 변형시키는 기능)가 정식 진입하기 전이라, 그동안의 대안으로 **`build_runner`** 기반 코드 생성을 광범위하게 씁니다. 어노테이션을 붙인 소스 파일을 보고 `.g.dart` 또는 `.freezed.dart`를 생성해 주는 방식입니다.

### 캡스톤에서 쓰는 codegen 도구들

`pubspec.yaml`의 `dev_dependencies`를 보면 한눈에 정리됩니다.

```yaml
dev_dependencies:
  build_runner: ^2.4.13
  riverpod_generator: ^2.6.5
  freezed: ^3.0.0
  json_serializable: ^6.8.0
  drift_dev: ^2.20.3
```

각 도구가 만들어 주는 것은 다음과 같습니다.

| 도구 | 입력 어노테이션 | 생성 파일 | 만드는 코드 |
| --- | --- | --- | --- |
| `freezed` | `@freezed` | `*.freezed.dart` | sealed 계층, `==`, `hashCode`, `copyWith`, `toString` |
| `json_serializable` | `@JsonSerializable` | `*.g.dart` | `fromJson` / `toJson` |
| `riverpod_generator` | `@riverpod` | `*.g.dart` | Provider 정의 |
| `drift_dev` | `@DriftDatabase` | `*.g.dart` | DB 클래스, DAO, 쿼리 헬퍼 |

### 실제 모델 예시

캡스톤의 `DietDay` 모델은 위 도구들을 다음처럼 결합합니다.

```dart
import 'package:freezed_annotation/freezed_annotation.dart';

part 'diet_day.freezed.dart';
part 'diet_day.g.dart';

@freezed
class DietDay with _$DietDay {
  const factory DietDay({
    required DateTime date,
    required int calories,
    required int sodium,
    required List<MealEntry> meals,
  }) = _DietDay;

  factory DietDay.fromJson(Map<String, dynamic> json) =>
      _$DietDayFromJson(json);
}
```

`part` 지시어로 생성 파일을 같은 라이브러리에 포함시키고, freezed가 sealed 계층 + copyWith를, json_serializable이 fromJson/toJson을 자동으로 만들어 줍니다. 손으로 짤 코드가 거의 0줄에 가까워집니다.

### `build_runner` 사용법

```bash
# 한 번 생성
dart run build_runner build --delete-conflicting-outputs

# 파일 변경 감지하며 자동 재생성
dart run build_runner watch --delete-conflicting-outputs
```

`--delete-conflicting-outputs`는 이전 생성 파일이 남아 있을 때 충돌을 자동 해결합니다. CI에서는 매번 깨끗하게 generation하기 위해 build를 한 번 돌립니다.

### `part` / `part of` 지시어

코드 생성을 쓰면 자주 마주치는 키워드가 `part` / `part of`입니다.

```dart
// diet_day.dart
part 'diet_day.freezed.dart';
part 'diet_day.g.dart';
```

```dart
// diet_day.freezed.dart (자동 생성)
part of 'diet_day.dart';
// ...
```

`part`로 묶인 파일들은 **같은 라이브러리**로 취급됩니다. 즉 별도 import 없이 서로의 private 멤버에 접근할 수 있습니다. 자동 생성 파일이 private `_$DietDay` 같은 멤버를 만들 수 있는 이유가 이것입니다.

### 코드 생성을 쓸 때 주의점

- **생성 파일을 직접 수정하지 않는다** — 다음 빌드에서 덮어쓰입니다.
- **lint 룰에 생성 파일을 제외한다** — `analysis_options.yaml`에 `exclude:`로 `**/*.g.dart`, `**/*.freezed.dart`를 박습니다.
- **`.gitignore`에 넣지 않는다** — 일반적으로 생성 파일을 커밋에 포함시키는 게 권장됩니다. CI에서 build_runner 실패가 머지 후 발견되는 것보다 사전에 잡히는 게 안전합니다.

## 4. 종합 — 캡스톤에서의 메타 패턴 활용 패턴

이 세 가지를 합쳐 캡스톤에서 자주 등장한 패턴을 정리하면 다음과 같습니다.

### A. 모델 정의의 표준 템플릿

```dart
import 'package:freezed_annotation/freezed_annotation.dart';

part 'meal_entry.freezed.dart';
part 'meal_entry.g.dart';

@freezed
class MealEntry with _$MealEntry {
  const factory MealEntry({
    required String id,
    required String name,
    required int calories,
    required int sodium,
    required DateTime takenAt,
  }) = _MealEntry;

  factory MealEntry.fromJson(Map<String, dynamic> json) =>
      _$MealEntryFromJson(json);
}
```

### B. Controller 정의의 표준 템플릿 (Riverpod codegen)

```dart
@riverpod
class DietController extends _$DietController {
  @override
  AsyncValue<DietDay> build() => const AsyncLoading();

  Future<void> load() async {
    final result = await ref.read(dietRepositoryProvider).fetchToday();
    state = switch (result) {
      Ok(:final value) => AsyncData(value),
      Err(:final error) => AsyncError(error, StackTrace.current),
    };
  }
}
```

### C. 도메인 유틸리티는 extension으로

```dart
extension DietDayX on DietDay {
  bool get isSodiumOverLimit => sodium > 2000;
  int get mealCount => meals.length;
}
```

### D. 위젯 상태 라이프사이클 훅은 mixin으로

```dart
class _DashboardPageState extends ConsumerState<DashboardPage>
    with WidgetsBindingObserver {
  // ...
}
```

각 도구가 **자기 잘 하는 자리**에 들어가니, 모델 / 컨트롤러 / 유틸 / 라이프사이클이 깔끔하게 분리됩니다.

## 5. 학기 회고 — Dart를 한 학기 써본 소감

이번 학기 캡스톤을 진행하면서 새 언어를 처음부터 익히는 게 부담이긴 했지만, Dart는 다음 세 가지 면에서 캡스톤에 잘 맞는 언어였습니다.

1. **TypeScript 경험을 그대로 가져갈 수 있었습니다.** 비동기 모델, async/await, 클로저 등이 거의 같아서 진입이 빨랐습니다.
2. **Sound null safety와 sealed class가 도메인 모델링에 잘 맞았습니다.** 헬스케어 데이터처럼 변형이 많은 도메인에서 컴파일러가 잡아주는 안전망이 큽니다.
3. **build_runner 기반 코드 생성으로 보일러플레이트가 거의 없었습니다.** 모델 / 컨트롤러 정의에 손으로 짜는 코드가 5줄 미만인 경우가 대부분이었습니다.

다음 학기에는 캡스톤 후속으로 **실제 백엔드 연동, 카메라 권한·촬영, AI Coach 응답을 진짜 API에 붙이는 작업**이 남았는데, 그땐 또 Dart의 다른 영역(`Isolate`, FFI 등)을 공부해야 할 듯합니다. 학습 노트는 그때 다시 이어가려 합니다.

---

## 📌 요약

Dart의 메타 패턴 3종을 정리했습니다. **Extension methods**는 기존 타입에 정적으로 메서드를 추가해 유틸리티를 도메인 친화적으로 묶을 수 있고, **Mixin**은 `with` 키워드로 행위/상태를 합성해 다중 상속 없이도 라이프사이클·로깅·티커 같은 cross-cutting concern을 깔끔히 분리합니다. **`build_runner` 기반 코드 생성**은 freezed / json_serializable / riverpod_generator / drift_dev를 결합해 모델·컨트롤러·DB 보일러플레이트를 거의 0줄로 줄여 줍니다. 캡스톤에서 이 세 가지가 각자의 자리에서 효율을 만들어내는 걸 직접 체감했습니다.
