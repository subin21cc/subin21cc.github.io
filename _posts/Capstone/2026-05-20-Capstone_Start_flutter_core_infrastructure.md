---
title: "[Capstone-Start] Flutter 코어 인프라"
date: 2026-05-20 23:00:00 +0900
categories: [Capstone, Start]
tags: [dev, capstone]
---

**📅 일시:** 2026. 05. 20

**💬 내용:** go_router 셸 · Riverpod · 테마 토큰 · dio · drift · 에러 모델 · l10n 골격 구축

---

Stage 1에서 빈 앱이 빌드 통과까지 갔지만, 이 상태로는 화면 하나 띄울 수 없다. 오늘은 **Stage 2 — Core Infrastructure** 7개 phase를 한 번에 밀어 BottomNav 셸에서 4개 빈 페이지로 이동하고, 더미 API/DB 호출이 동작하고, 다국어 토글이 살아 있는 상태까지 만드는 것이 목표다.

각 phase를 작은 PR로 쪼개기엔 의존이 너무 얽혀 있고, 한 줄짜리 보일러플레이트가 많아서 **하나의 feature 브랜치(`feature/fe-flutter-core-infra`)**에 phase별 커밋만 분리해서 쌓기로 했다.

## 1. 라우터 골격 — go_router + StatefulShellRoute (Phase 2.1)

원본 React 앱의 `activeTab` state를 그대로 옮기지 않은 이유는 어제 글에서 적었다. Flutter에서는 **`StatefulShellRoute.indexedStack`**이 BottomNav 셸의 정석이다. 각 탭이 자체 네비게이션 스택을 갖고, 탭을 바꿔도 스크롤 위치와 라우트 히스토리가 유지된다.

```dart
final router = GoRouter(
  initialLocation: '/dashboard',
  observers: [LoggerNavigatorObserver()], // Phase 5.1에서 추가 예정
  routes: [
    StatefulShellRoute.indexedStack(
      builder: (context, state, navShell) => AppShell(navigationShell: navShell),
      branches: [
        StatefulShellBranch(routes: [GoRoute(path: '/dashboard', builder: ...)]),
        StatefulShellBranch(routes: [GoRoute(path: '/diet', builder: ...)]),
        StatefulShellBranch(routes: [GoRoute(path: '/exercise', builder: ...)]),
        StatefulShellBranch(routes: [GoRoute(path: '/my', builder: ...)]),
      ],
    ),
  ],
);
```

`AppShell`은 `Scaffold` + `BottomNavigationBar` + `navigationShell` 슬롯으로만 구성된 얇은 컨테이너다. 디자인 시스템과 결합되는 진짜 BottomNav는 Stage 8에서 React 원본과 픽셀 단위 정렬하면서 다시 짤 거라, 지금은 Material 기본 위젯 그대로 둔다.

## 2. Riverpod ProviderScope + Logger DI (Phase 2.2)

`ProviderScope`는 `main()`에서 직접 띄우면 테스트 환경에서 갈아끼우기가 불편해서, **`bootstrap()`** 함수로 분리했다.

```dart
// lib/app/bootstrap.dart
Future<void> bootstrap() async {
  WidgetsFlutterBinding.ensureInitialized();

  final logger = Logger(printer: PrettyPrinter(methodCount: 0));

  runApp(
    ProviderScope(
      observers: [_AppProviderObserver(logger)],
      overrides: [loggerProvider.overrideWithValue(logger)],
      child: const OncareApp(),
    ),
  );
}
```

ProviderObserver는 Stage 2 시점에서는 단순히 모든 provider의 라이프사이클(add/update/dispose)을 logger로 흘리는 역할만 한다. 나중에 디버그 환경에서 어떤 provider가 의도치 않게 rebuild되는지 추적할 때 쓸 거다.

## 3. 토큰화된 테마 — color/typo/spacing/radius (Phase 2.3)

여기서 한 번 결정이 갈렸다. Material 3의 `ColorScheme.fromSeed()`로 동적 컬러를 뽑을지, 아니면 디자인 토큰을 명시적으로 박을지. 결국 **명시적 토큰 + ThemeData 매핑** 쪽으로 갔다. 이유는 Stage 8에서 React 원본 shadcn 테마(`default_shadcn_theme.css`)와 정렬해야 하는데, 동적 컬러로 시작하면 그때 가서 토큰 표 전체를 다시 갈아엎어야 하기 때문이다.

```dart
// lib/design_system/tokens/colors.dart
abstract class AppColors {
  static const primary = Color(0xFF22C55E);
  static const primaryFg = Color(0xFFFFFFFF);
  static const background = Color(0xFFFFFFFF);
  static const muted = Color(0xFFF5F5F5);
  static const border = Color(0xFFE5E7EB);
  // ...
}

// lib/design_system/tokens/spacing.dart
abstract class AppSpacing {
  static const xs = 4.0;
  static const sm = 8.0;
  static const md = 16.0;
  static const lg = 24.0;
  static const xl = 32.0;
}
```

`AppTheme.light` / `AppTheme.dark`에서 이 토큰들을 `ThemeData`의 `ColorScheme`, `TextTheme`, `CardTheme` 등에 매핑한다. dark는 Stage 5에서 다시 정밀화할 거라 지금은 light의 보색 정도로만 임시 채워뒀다.

## 4. dio 클라이언트 + 인터셉터 3종 (Phase 2.4)

retrofit이 빠진 상태라 dio만으로 클라이언트를 구성했다. **인터셉터 3개**를 등록한 게 핵심이다.

- **LoggingInterceptor** — 요청/응답을 `logger`로 흘림. `pretty_dio_logger` 패키지 그대로 사용.
- **AuthInterceptor** — `secure_storage`에서 토큰을 읽어 `Authorization: Bearer …` 헤더 부착. 토큰이 없으면 헤더 생략.
- **MockInterceptor** — `--dart-define=APP_ENV=dev`일 때만 활성화. 특정 path에 대해 JSON 응답을 짧은 지연(50ms) 후 반환. Stage 4에서 각 feature의 mock 응답을 여기에 등록할 예정.

```dart
final dio = Dio(BaseOptions(
  baseUrl: AppConfig.instance.apiBaseUrl,
  connectTimeout: const Duration(seconds: 10),
  receiveTimeout: const Duration(seconds: 10),
));

dio.interceptors.addAll([
  LoggingInterceptor(logger),
  AuthInterceptor(secureStorage),
  if (AppConfig.instance.isDev) MockInterceptor(),
]);
```

테스트 페이지에서 `GET /ping` 더미 호출을 날리고 로그에 200 응답이 찍히는 것까지 확인했다.

## 5. drift 스키마 v1 + secure_storage + prefs 래퍼 (Phase 2.5)

drift는 처음 쓰는 거라 학습 비용을 좀 들였다. 핵심은 **`@DriftDatabase`** 어노테이션 + 빌드 러너로 `.g.dart`를 자동 생성하는 흐름이다.

```dart
@DriftDatabase(tables: [UsersTable])
class OncareDatabase extends _$OncareDatabase {
  OncareDatabase() : super(_openConnection());

  @override
  int get schemaVersion => 1;
}
```

지금은 `UsersTable` 하나만 정의해 두고 (id, name, locale, created_at), Stage 4에서 feature별로 테이블이 더 붙는 구조다. Stage 9에서는 5개 테이블(`diet_days`, `meal_entries`, `exercise_sessions`, `vital_logs`, `schedule_events`)이 추가되며 schema v2로 올라간다.

`secure_storage`와 `prefs`는 각각 얇은 래퍼만 만들어 둠 — `AuthStorage.readAccessToken()`, `SettingsRepository.getLocale()` 식.

## 6. 에러 모델 — AppError sealed + Result<T> (Phase 2.6)

Dart에는 코틀린의 sealed class와 Result 같은 게 일급으로 들어 있지 않아서, **freezed의 sealed union**으로 흉내냈다.

```dart
@freezed
sealed class AppError with _$AppError {
  const factory AppError.network(String message) = NetworkError;
  const factory AppError.notFound(String resource) = NotFoundError;
  const factory AppError.unauthorized() = UnauthorizedError;
  const factory AppError.unknown(Object error, StackTrace? trace) = UnknownError;
}

sealed class Result<T> {
  const Result();
}
class Ok<T> extends Result<T> { final T value; const Ok(this.value); }
class Err<T> extends Result<T> { final AppError error; const Err(this.error); }
```

이렇게 두면 controller에서 `switch (result)`로 패턴 매칭이 가능해진다. 예외를 throw로 통과시키지 않고 명시적으로 받는 패턴 — 안 그러면 UI 레이어에서 `try/catch`가 사방에 박힌다.

추가로 **`ErrorView`**와 **`EmptyState`** 위젯을 `shared/widgets/`에 두고, 모든 페이지에서 같은 빈 상태/에러 상태 UI를 쓰도록 했다.

## 7. ko + en 로케일 골격 (Phase 2.7)

`flutter_localizations + intl` 조합. `l10n.yaml`로 ARB 파일 위치를 박아 두고, `flutter gen-l10n`이 `lib/gen/l10n/`에 코드를 생성하도록 했다.

```yaml
# l10n.yaml
arb-dir: lib/l10n
template-arb-file: app_ko.arb
output-localization-file: app_localizations.dart
output-dir: lib/gen/l10n
synthetic-package: false
```

지금은 `appTitle`, `bottomNavDashboard` 같은 키 5~6개만 박아 두고, Stage 5에서 Dashboard 문자열을 전수 외부화할 때 본격적으로 채울 거다. 디바이스 로케일에 따라 자동 전환되며, `flutter run --dart-define=FORCE_LOCALE=en`으로 강제 토글도 된다.

## 8. 오늘의 회고와 내일 할 일

- **고민했던 것**:
  - 라우터 — `Navigator 2.0`을 직접 다룰지, go_router를 쓸지. go_router가 ShellRoute를 깔끔히 지원해서 결정에 큰 고민은 없었다.
  - 테마 — 동적 컬러(Material You) vs 명시적 토큰. Stage 8 정렬 작업을 미리 염두에 두고 토큰 쪽으로 갔다.
  - 에러 — Either 타입 패키지를 쓸지, freezed sealed로 직접 만들지. 학습 곡선 줄이려고 후자 선택.
- **결정한 것**: Core Infrastructure 7개 phase 모두 완료. BottomNav 셸에서 4개 빈 페이지 이동, 더미 dio 호출, drift 첫 마이그레이션, ko/en 토글 동작 확인.
- **남은 문제**:
  - **retrofit 재도입** — Stage 4 진입 시점에 다시 결정. 안 되면 dio로 라우트별 클라이언트를 직접 구현.
  - **dark theme 정밀화** — 지금은 light의 단순 반전. 디자인 토큰 표가 완성되면 다시 본다.
  - **테스트 — 코어 인프라 부분의 unit test가 비어 있다.** Stage 6에서 한 번에 채울 예정.

내일은 **Stage 3 — 디자인 시스템**에 들어간다. 토큰을 `AppColors`/`AppTypography`/`AppSpacing` 클래스로 한 번 더 정리하고, atom(Button/Card/Input/Badge/Avatar)과 molecule(MetricCard/ChartCard/SectionHeader)을 만든다. 동시에 `/dev/ui-catalog` 라우트를 띄워서 모든 위젯을 한 화면에서 확인할 수 있게 할 거다. 이 카탈로그가 없으면 Stage 4에서 디자인 시스템과 페이지를 동시에 디버깅하느라 시간이 두 배로 든다.

---

## 📌 요약

오늘은 **Stage 2 — 코어 인프라**를 끝냈다. go_router + StatefulShellRoute로 4탭 셸, Riverpod ProviderScope + observer, 토큰 기반 ThemeData, dio + 인터셉터 3종, drift 스키마 v1, freezed sealed로 만든 AppError와 Result<T>, ko/en l10n 골격까지 한 번에 깔았다. 이제 빈 화면 위에 디자인 시스템을 얹기만 하면 된다.
