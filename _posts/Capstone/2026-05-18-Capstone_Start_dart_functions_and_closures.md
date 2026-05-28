---
title: "[Capstone-Start] Dart 학습 — 함수 일급 객체와 클로저"
date: 2026-05-18 21:00:00 +0900
categories: [Capstone, Start]
tags: [dev, capstone]
---

**📅 일시:** 2026. 05. 18

**💬 내용:** Dart 함수의 일급 객체 성격, `typedef`, named/optional 파라미터, 클로저 — Flutter Builder 패턴 사전 준비

---

어제(5/17) Flutter로 가는 결정을 내렸고, 내일은 `flutter create`로 부트스트랩에 들어갑니다. 그 전에 마지막 사전 학습 주제로 **Dart의 함수**를 정리합니다. Flutter 위젯 코드는 표면적으로는 클래스 트리지만, 실제로는 **함수를 인자로 받는 패턴**(builder, callback, listener 등)이 어느 위치에나 등장합니다. 그래서 함수 타입을 깨끗하게 다룰 수 있어야 위젯 코드도 깨끗해집니다.

오늘 정리할 핵심 주제는 다음 다섯 가지입니다.

1. 함수는 일급 객체 (first-class)
2. 함수 타입 표기와 `typedef`
3. Named / Positional / Optional / Required 파라미터
4. 클로저
5. Flutter Builder 패턴 사전 학습

## 1. 함수는 일급 객체 (First-Class)

Dart에서 함수는 **값처럼 다룰 수 있는 일급 객체**입니다. 변수에 담고, 다른 함수의 인자로 넘기고, 다른 함수의 반환값으로 받을 수 있습니다. JavaScript / Kotlin / Swift와 동일한 모델입니다.

```dart
int add(int a, int b) => a + b;

void main() {
  // 함수를 변수에 담기
  final op = add;
  print(op(1, 2)); // 3

  // 함수를 인자로 받기
  print(applyTwice(3, op)); // 9
}

int applyTwice(int x, int Function(int, int) f) => f(f(x, x), x);
```

`int Function(int, int)`처럼 **`반환타입 Function(파라미터타입들)`** 형태로 함수 타입을 명시합니다. TypeScript의 `(a: number, b: number) => number`에 해당합니다.

## 2. `typedef` — 함수 타입 alias

함수 타입을 코드 곳곳에 그대로 박으면 길어집니다. `typedef`로 alias를 만들어 재사용합니다.

```dart
typedef IntBinaryOp = int Function(int, int);

int reduceList(List<int> xs, IntBinaryOp op, int seed) {
  var acc = seed;
  for (final x in xs) {
    acc = op(acc, x);
  }
  return acc;
}

void main() {
  print(reduceList([1, 2, 3, 4], (a, b) => a + b, 0)); // 10
}
```

Flutter SDK는 자주 쓰는 콜백 타입을 미리 `typedef`로 정의해 두고 있습니다. 예를 들어 `WidgetBuilder`는 다음처럼 정의돼 있다고 합니다.

```dart
typedef WidgetBuilder = Widget Function(BuildContext context);
```

그래서 `ListView.builder`나 `FutureBuilder` 같은 위젯의 `builder:` 인자가 깔끔하게 표현됩니다.

## 3. Named / Positional / Optional / Required

Dart의 파라미터 종류는 표면적으로 단순한데, **named vs positional**과 **optional vs required**가 직교(orthogonal)로 조합돼서 처음에는 헷갈렸습니다. 정리하면 다음과 같습니다.

### Positional 파라미터

```dart
int sum(int a, int b) => a + b;
sum(1, 2);
```

순서대로 전달. C / Java 스타일.

### Optional Positional 파라미터 — `[ ... ]`

```dart
String greet(String name, [String greeting = '안녕하세요']) {
  return '$greeting, $name!';
}
greet('민수');          // '안녕하세요, 민수!'
greet('민수', 'Hello'); // 'Hello, 민수!'
```

대괄호로 감싼 파라미터는 optional이며 기본값을 줄 수 있습니다.

### Named 파라미터 — `{ ... }`

```dart
Widget buildCard({
  required String title,
  String subtitle = '',
  double padding = 16,
}) {
  // ...
}

buildCard(title: '오늘의 식단', padding: 24);
```

중괄호로 감싼 파라미터는 **이름과 함께 호출**합니다. `required` 키워드를 붙이면 호출 시 반드시 넘겨야 합니다. Flutter 위젯은 거의 모든 생성자가 이 패턴을 씁니다.

```dart
const Padding(
  padding: EdgeInsets.all(16),
  child: Text('Hello'),
)
```

생성자 인자가 10개 넘어가는 위젯도 named 파라미터 덕분에 호출부에서 가독성이 유지됩니다. 직접 위젯을 짤 때도 named + required 조합을 기본으로 가져갈 계획입니다.

### 한눈에 비교

| 종류 | 표기 | 기본값 | 호출 |
| --- | --- | --- | --- |
| Required positional | `(a, b)` | ❌ | `f(1, 2)` |
| Optional positional | `([a, b])` | ✅ | `f(1)` |
| Required named | `({required a})` | ❌ | `f(a: 1)` |
| Optional named | `({a})` 또는 `({a = 0})` | ✅ | `f(a: 1)` 또는 `f()` |

## 4. 익명 함수와 Arrow Function

JavaScript와 거의 동일한 두 가지 표기를 모두 지원합니다.

```dart
// 익명 함수
final square = (int x) {
  return x * x;
};

// Arrow function (단일 표현식만)
final cube = (int x) => x * x * x;
```

Flutter `onPressed`, `onTap`처럼 짧은 콜백은 거의 100% arrow function으로 씁니다.

```dart
ElevatedButton(
  onPressed: () => print('탭됨'),
  child: const Text('확인'),
)
```

## 5. 클로저 — Capture by Reference

Dart의 함수는 **선언된 스코프의 변수를 캡처**합니다. JavaScript의 클로저와 동일하게 by-reference로 캡처되므로, 캡처 후 외부에서 변수가 바뀌면 클로저 안에서도 바뀐 값이 보입니다.

```dart
Function makeCounter() {
  var count = 0;
  return () {
    count++;
    return count;
  };
}

void main() {
  final counter = makeCounter();
  print(counter()); // 1
  print(counter()); // 2
  print(counter()); // 3
}
```

`count`는 `makeCounter` 호출이 끝나도 클로저에 의해 살아 있고, 호출마다 같은 변수를 공유합니다. 이 메커니즘은 Riverpod이나 ChangeNotifier 같은 상태관리 라이브러리의 내부 구현에서 빈번하게 쓰이는 패턴입니다.

### for-loop에서의 흔한 함정

JavaScript의 `var` 스코프 버그와 비슷한 함정이 Dart에도 있을지 확인해 봤는데, **Dart는 for-loop 변수가 매 iteration마다 새로 바인딩**되므로 안전합니다.

```dart
final closures = <int Function()>[];
for (var i = 0; i < 3; i++) {
  closures.add(() => i);
}
print(closures.map((f) => f()).toList()); // [0, 1, 2]
```

JavaScript의 `var i`였다면 `[3, 3, 3]`이 나왔을 텐데, Dart는 의도대로 동작합니다. 안전한 기본 동작이라 좋습니다.

## 6. Flutter Builder 패턴 사전 학습

Flutter의 핵심 위젯 다수가 "**함수를 인자로 받아 위젯을 만드는**" 패턴을 씁니다. 표준 패턴 몇 가지를 미리 정리해 둡니다.

### `ListView.builder`

```dart
ListView.builder(
  itemCount: meals.length,
  itemBuilder: (context, index) {
    final meal = meals[index];
    return ListTile(title: Text(meal.name));
  },
)
```

`itemBuilder`는 `Widget Function(BuildContext, int)` 타입의 함수입니다. ListView가 화면에 보이는 영역의 index에 대해서만 builder를 호출해서 위젯을 만들기 때문에 **lazy rendering**이 자동으로 됩니다.

### `FutureBuilder`

```dart
FutureBuilder<User>(
  future: fetchUser(),
  builder: (context, snapshot) {
    if (snapshot.connectionState == ConnectionState.waiting) {
      return const CircularProgressIndicator();
    }
    if (snapshot.hasError) {
      return Text('에러: ${snapshot.error}');
    }
    return Text('이름: ${snapshot.data?.name}');
  },
)
```

어제 공부한 `Future`와 오늘의 함수 타입이 만나는 지점입니다. `builder`는 Future의 상태를 받아 위젯을 반환하는 함수입니다.

### Callback 콜백을 받는 커스텀 위젯 만들기

직접 위젯을 만들 때도 콜백을 받는 패턴이 자주 등장합니다. `typedef`로 콜백 타입을 정의해 두면 인자 시그니처가 명확해집니다.

```dart
typedef OnMealSelected = void Function(Meal meal);

class MealList extends StatelessWidget {
  const MealList({
    super.key,
    required this.meals,
    required this.onSelected,
  });

  final List<Meal> meals;
  final OnMealSelected onSelected;

  @override
  Widget build(BuildContext context) {
    return ListView.builder(
      itemCount: meals.length,
      itemBuilder: (context, i) => ListTile(
        title: Text(meals[i].name),
        onTap: () => onSelected(meals[i]),
      ),
    );
  }
}
```

캡스톤의 식단 페이지에서 위와 같은 구조로 `MealList`를 만들 계획입니다. 호출자는 `onSelected:`에 동작만 넘기면 되고, 리스트 위젯 자체는 데이터에만 관심이 있는 구조입니다.

## 7. 학습 후 알게 된 것 / 내일 계획

오늘 정리한 핵심은 다음 세 가지입니다.

1. **함수는 일급 객체**이고, `typedef`로 함수 타입을 alias할 수 있습니다. Flutter SDK는 `WidgetBuilder`처럼 자주 쓰는 콜백 타입을 미리 정의해 둡니다.
2. **Named + required 파라미터**가 Flutter 위젯 생성자의 표준입니다. 인자가 많아도 호출부 가독성이 유지됩니다.
3. **클로저는 by-reference 캡처**이고, for-loop 변수는 매 iteration마다 새 바인딩이라 JS의 `var` 함정이 없습니다.

내일은 드디어 `flutter create .`을 칩니다. 빈 앱이 Android·iOS·Web 모두 빌드 통과하는 상태까지가 내일의 목표입니다. 코어 인프라(라우터·상태관리·테마·네트워크·저장소)는 그 다음 날부터 본격적으로 들어갑니다.

---

## 📌 요약

Dart의 함수는 일급 객체이고, 함수 타입은 `반환타입 Function(파라미터타입)` 형태로 표기합니다. `typedef`로 함수 타입에 이름을 붙이면 Flutter SDK처럼 `WidgetBuilder` 같은 깔끔한 시그니처를 만들 수 있습니다. 파라미터는 **positional/named × required/optional** 직교 조합으로 표현되며, Flutter 위젯은 named + required 조합을 기본으로 채택해 인자가 많아도 호출부 가독성을 유지합니다. 클로저는 by-reference 캡처라 카운터 같은 상태 유지 패턴이 자연스럽게 동작합니다.
