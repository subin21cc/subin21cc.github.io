---
title: "[Capstone-Start] Dart 학습 — 변수 선언과 Null Safety"
date: 2026-05-15 22:00:00 +0900
categories: [Capstone, Start]
tags: [dev, capstone]
---

**📅 일시:** 2026. 05. 15

**💬 내용:** Flutter 사전 학습 시작 — `var` / `final` / `const` / `late`와 Sound Null Safety 정리

---

피그마 프로토타입 1차 작업을 마무리하고, 다음 단계로 **실제 동작하는 프로토타입**을 어떤 기술 스택으로 만들지 검토하는 중입니다. 후보는 React Native와 Flutter 두 가지인데, Flutter 쪽이 단일 코드베이스로 웹까지 자연스럽게 나오는 점이 매력적이라 우선 Dart 언어부터 손에 익혀보기로 했습니다.

학부 4년 동안 C / C++ / Java / Python / TypeScript는 한 번씩 다 써봤지만 Dart는 처음입니다. 다행히 표면적인 문법은 TypeScript와 굉장히 비슷해서 진입 자체는 어렵지 않았는데, **변수 선언 키워드 4종**과 **Sound Null Safety**가 다른 언어와 미묘하게 달라서 첫 학습 노트로 정리해 두려 합니다.

## 1. 왜 Dart를 공부하게 됐는지

Flutter는 Google이 만든 크로스 플랫폼 UI 프레임워크이고, 그 위에서 동작하는 언어가 Dart입니다. Dart는 다음과 같은 특징을 가집니다.

- **AOT(Ahead-of-Time)와 JIT(Just-in-Time) 컴파일을 동시에 지원**합니다. 개발 시점에는 JIT로 Hot Reload를 빠르게 돌리고, 릴리즈 빌드는 AOT로 컴파일해서 네이티브 수준의 성능을 냅니다.
- **Sound Null Safety**를 기본으로 강제합니다. 타입 시스템이 nullable과 non-nullable을 명확히 구분합니다.
- **Single-threaded 이벤트 루프**입니다. JavaScript와 동일한 동시성 모델이라 `async`/`await`이 친숙합니다.

캡스톤 일정 안에서 새 언어를 처음부터 배워야 한다는 점이 부담이긴 하지만, **TypeScript와 문법이 80% 비슷**해서 큰 진입 장벽은 없었습니다. 다만 같아 보여서 더 헷갈리는 부분(예: `const`의 의미)이 있어서 그런 부분 위주로 정리합니다.

## 2. 변수 선언 키워드 4종 — `var` / `final` / `const` / `late`

Dart의 변수 선언 키워드는 표면적으로 단순한데, 각 키워드가 **언제 값이 정해지는지(compile-time / runtime)** 와 **재할당 가능 여부**에서 미묘하게 다릅니다.

| 키워드 | 재할당 | 값이 정해지는 시점 | 비고 |
| --- | --- | --- | --- |
| `var` | ✅ | runtime | 타입은 컴파일 시점에 추론 |
| `final` | ❌ (한 번만 할당) | runtime | 초기화는 늦게 해도 됨 |
| `const` | ❌ | **compile-time** | canonicalize됨(동일 값이면 동일 인스턴스) |
| `late` | (수식어) | runtime | 초기화를 첫 사용 시점까지 연기 |

### `var` vs `final`

`var`는 단순히 "타입을 추론하라"는 표시입니다. TypeScript의 `let`과 비슷합니다.

```dart
var count = 0;
count = count + 1; // OK — 재할당 가능
```

`final`은 한 번 할당된 후 재할당이 불가능합니다. JavaScript의 `const`나 Java의 `final`과 같습니다.

```dart
final user = User(id: 1, name: '민수');
user = User(id: 2, name: '지수'); // ❌ 컴파일 에러
```

다만 `final`이 가리키는 **객체의 내부 필드**까지 immutable로 만들어 주지는 않습니다. 객체 자체를 immutable로 만들려면 `freezed` 패키지나 직접 setter를 막아야 합니다.

```dart
final list = <int>[1, 2, 3];
list.add(4); // OK — list 참조는 final이지만 내용은 mutable
```

### `const` — Dart에서 가장 헷갈렸던 키워드

JavaScript / TypeScript의 `const`와는 의미가 다릅니다. Dart의 `const`는 **compile-time에 값이 정해져야 하고**, 같은 값을 가진 const 인스턴스는 메모리 상에서 동일한 객체로 canonicalize됩니다.

```dart
const pi = 3.14;
const greeting = 'Hello'; // OK

// 컴파일 타임에 정해지지 않는 값은 const가 안 됩니다.
const now = DateTime.now(); // ❌ DateTime.now()는 runtime 값
```

Flutter 위젯에서 `const Text('홈')`처럼 위젯 앞에 `const`를 붙이면, 같은 위치에 같은 위젯이 다시 그려질 때 **새로 인스턴스를 만들지 않고 재사용**합니다. 빌드 비용을 줄이는 중요한 최적화 포인트라고 합니다. Flutter 코드에서 `const`가 도배되는 건 이 이유 때문입니다.

```dart
// rebuild 때마다 새 Text 인스턴스를 만들지 않음
const Text('홈')
```

### `late` — null safety의 안전판

`late`는 키워드 자체보다는 **수식어**에 가깝습니다. "이 변수는 nullable이 아니지만, 초기화는 첫 사용 시점까지 연기하겠다"는 뜻입니다.

```dart
late final Database db;

void main() async {
  db = await Database.connect(); // 첫 사용 전에 반드시 초기화
}

void query() {
  db.execute('SELECT 1'); // 여기서 db가 아직 초기화 안 됐으면 LateInitializationError
}
```

`late`가 있어야 하는 이유는, Dart의 sound null safety가 **non-nullable 변수는 선언 시점에 값이 있어야 한다**고 요구하기 때문입니다. 의존성 주입이나 lazy init 같은 패턴을 쓰려면 `late`가 필수입니다.

## 3. Sound Null Safety — `?`, `!`, `??`, `?.`

Dart의 null safety는 **sound**합니다. "sound"는 타입 시스템이 약속한 것을 런타임에 반드시 지킨다는 의미로, 컴파일러가 null이 아니라고 판단한 변수는 실행 중 null이 절대 들어올 수 없습니다. TypeScript의 strict mode와 유사하지만, TS는 unsound한 부분(예: `as` 캐스팅)이 있는 반면 Dart는 더 엄격합니다.

### Nullable 타입 표기 — `?`

```dart
String name = 'Subin';      // non-nullable
String? maybeName = null;   // nullable — null 허용
```

`String`과 `String?`은 **다른 타입**입니다. nullable 변수를 non-nullable에 직접 대입할 수 없습니다.

```dart
String? maybe = '...';
String required = maybe; // ❌ 타입 에러
String required2 = maybe ?? '기본값'; // OK — null이면 기본값 사용
String required3 = maybe!; // OK — 단, null이면 런타임 에러
```

### 핵심 연산자

| 연산자 | 의미 | 예 |
| --- | --- | --- |
| `?` | 타입 뒤에 붙어 nullable 표시 | `String?` |
| `!` | null assertion — null이면 즉시 throw | `maybe!` |
| `??` | null이면 우측 값 사용 | `maybe ?? '기본값'` |
| `??=` | null이면 우측 값 할당 | `cache ??= compute()` |
| `?.` | null이면 호출 건너뜀, 결과는 null | `user?.name` |

### `!` 사용 가이드

`!`는 강력하지만 **`as` 캐스팅처럼 안전성을 깨뜨릴 수 있는 도구**라 가능한 한 줄이는 게 좋다고 합니다. 다음 두 가지 경우에만 쓰는 걸 원칙으로 잡았습니다.

1. 컴파일러가 추론하지 못하는 **invariant**가 있을 때 (예: `if (x != null)` 직후의 다른 변수 참조)
2. 외부 라이브러리가 잘못된 시그니처를 반환했고 우회 수단이 없을 때

그 외에는 `??`, `if (x != null)`, 패턴 매칭 등으로 가능한 한 안전하게 처리합니다.

## 4. `dynamic`은 왜 피해야 하는가

Dart에는 `dynamic`이라는 타입이 있습니다. TypeScript의 `any`와 비슷하게 **모든 타입을 허용하고 모든 메서드 호출을 허용**합니다.

```dart
dynamic x = 'hello';
print(x.length); // OK — String의 length
x = 42;
print(x.length); // 컴파일 통과, 런타임에 NoSuchMethodError
```

`dynamic`을 쓰는 순간 **sound null safety가 깨집니다**. 정말 타입을 모를 때(JSON 파싱 직후 등)만 쓰고, 가능한 즉시 구체 타입으로 좁히는 게 정석이라고 합니다. JSON 디코딩 시에는 `Map<String, dynamic>`을 받자마자 `freezed`로 만든 모델로 변환해서 dynamic 노출을 한 줄로 끝내려 합니다.

## 5. TypeScript와의 비교

|  | TypeScript | Dart |
| --- | --- | --- |
| 변수 선언 | `let` / `const` | `var` / `final` / `const` |
| 타입 시스템 | structural | nominal |
| Null safety | strictNullChecks 옵션 | 기본 강제 (sound) |
| const의 의미 | 재할당 불가 | compile-time canonicalize |
| 안전성 | unsound (as, any 등) | sound (dynamic은 예외) |

타입 시스템의 가장 큰 차이는 **structural vs nominal**입니다. TypeScript는 "필드 구성이 같으면 같은 타입으로 간주"하는 반면, Dart는 "선언된 클래스 이름이 같아야 같은 타입"입니다. Duck typing이 안 되는 대신, 타입 추론이 더 명확하다는 장점이 있습니다.

## 6. 학습 후 알게 된 것 / 다음 학습 계획

오늘 정리하면서 가장 인상 깊었던 건 다음 세 가지입니다.

1. **`const`가 단순한 "재할당 불가"가 아니라 컴파일타임 canonicalize**라는 것. Flutter 위젯 앞에 `const`를 박는 이유를 이제 알았습니다.
2. **`late`는 lazy init이자 null safety의 escape hatch**라는 것. DI 주입 시점이 늦어지는 패턴을 위해 필요합니다.
3. **null safety가 sound하다**는 것. TypeScript의 `any` 같은 우회 통로가 거의 없어서 학습 곡선은 조금 있지만, 런타임 안정성에 큰 도움이 될 것 같습니다.

내일은 Dart의 **비동기 모델 — `Future`, `async`/`await`, `Stream`**을 공부할 계획입니다. Flutter는 비동기 위젯(`FutureBuilder`, `StreamBuilder`)이 핵심 위젯이라 이 부분을 잘 알아야 본격적인 개발에 들어갈 수 있을 것 같습니다.

---

## 📌 요약

Dart의 변수 선언 키워드 4종(`var` / `final` / `const` / `late`)과 Sound Null Safety의 핵심 연산자(`?`, `!`, `??`, `??=`, `?.`)를 정리했습니다. 특히 `const`는 단순 immutability가 아니라 **컴파일타임 canonicalize**라는 의미를 가져서, Flutter 위젯 트리에서 const 위젯이 rebuild 비용을 줄이는 최적화 단위가 됩니다. `dynamic`은 sound null safety를 깨뜨리므로 JSON 파싱 같은 경계 영역에서만 쓰고 즉시 구체 타입으로 좁혀야 합니다.
