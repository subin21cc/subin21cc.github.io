---
title: "[Capstone-Start] Flutter 프로젝트 부트스트랩"
date: 2026-05-19 22:00:00 +0900
categories: [Capstone, Start]
tags: [dev, capstone]
---

**📅 일시:** 2026. 05. 19

**💬 내용:** `flutter create`부터 GitHub Pages 자동 배포까지 — 부트스트랩 단계 완료

---

전날 정리한 Q1~Q12에 대해 결정하였습니다. 핵심만 옮기면 이렇습니다.

| # | 결정 | 영향 |
| --- | --- | --- |
| Q1 | **자체 REST 서버** | `core/network`에 dio + retrofit 기반 클라이언트, `API_BASE_URL`을 `--dart-define`으로 주입. |
| Q2 | **Riverpod v2 + riverpod_generator** | 제안 채택. `ProviderScope`는 `app/bootstrap.dart`에서 설치. |
| Q3 | **drift(SQLite) + secure_storage + shared_preferences** | 헬스 시계열은 SQL이 유리. |
| Q4 | **소셜 4종** — Apple / Google / Kakao / Naver | `features/auth/`에서 4개 provider 추상화. |
| Q5 | **ko + en** | 기본 로케일은 ko. |
| Q10 | **Hash URL** | `setUrlStrategy(const HashUrlStrategy())`로 GitHub Pages 404 fallback 회피. |
| Q11 | **`/oncare-flutter/` 베이스 경로** | 라이브 URL은 `https://<user>.github.io/oncare-flutter/`. |
| Q12 | **Web만 CI 자동화** | iOS·Android 릴리즈 빌드는 수동. |

이제 부트스트랩에 들어가도 되겠다고 판단했습니다. 오늘 목표는 **빈 앱이 Android·iOS·Web 모두 빌드되고, main 브랜치 푸시 시 GitHub Pages가 자동으로 갱신되는 상태**까지 끌어올리는 것이다.

## 1. `flutter create` — 한 줄이지만 무겁다

`flutter create .`은 한 번 잘못 치면 org 식별자가 박혀버리고, 그러면 iOS 번들 ID와 Android applicationId가 같이 망가져서 되돌리기가 번거롭다. 그래서 옵션 4개를 다 박아서 한 번에 실행했다.

- `--project-name oncare`: 패키지 이름. `pubspec.yaml`의 `name:` 필드. import 경로에 그대로 들어가기 때문에 다시 바꾸기 어렵다.
- `--platforms=android,ios,web`: 세 플랫폼 폴더만 생성. macOS/Linux/Windows는 캡스톤 일정에서 불필요해 뺐다.

생성된 `android/`, `ios/`, `web/`은 Flutter가 알아서 관리하는 폴더라 직접 수정은 최소화하기로 했다.

## 2. lib/ 골격과 자산 디렉토리

PLAN §3에서 설계한 수직 슬라이스 구조를 그대로 옮겼다. 빈 `.gitkeep` 파일로 폴더만 박아두고, 실제 코드는 Stage 2부터 들어간다.

```
lib/
├─ main.dart
├─ app/
├─ core/
├─ shared/
├─ design_system/
├─ features/
│  ├─ dashboard/
│  ├─ diet/
│  ├─ exercise/
│  ├─ my_health/
│  ├─ ai_coach/
│  ├─ notification/
│  └─ place/
├─ l10n/
└─ gen/

assets/
├─ icons/
└─ images/
```

## 3. `pubspec.yaml` 의존성 한 번에 박기

캡스톤은 시간이 한정돼 있어서, 의존성을 한 번에 추가하고 lock을 한 번에 떠두는 게 가장 깔끔하다. 카테고리별로 그룹지어 정리했다.

```yaml
dependencies:
  flutter:
    sdk: flutter
  flutter_localizations:
    sdk: flutter

  # --- State management & DI ---
  flutter_riverpod: ^2.6.1
  riverpod_annotation: ^2.6.1
  hooks_riverpod: ^2.6.1
  flutter_hooks: ^0.21.2

  # --- Routing ---
  go_router: ^14.6.2

  # --- Networking ---
  dio: ^5.7.0
  pretty_dio_logger: ^1.4.0

  # --- Local persistence ---
  drift: ^2.20.3
  drift_flutter: ^0.2.4
  sqlite3_flutter_libs: ^0.5.24
  flutter_secure_storage: ^9.2.2
  shared_preferences: ^2.3.2

  # --- Data modeling ---
  freezed_annotation: ^3.0.0
  json_annotation: ^4.9.0

  # --- UI / charts / animation ---
  fl_chart: ^0.69.2
  flutter_animate: ^4.5.0

  # --- i18n / utilities ---
  intl: any
  logger: ^2.4.0
  equatable: ^2.0.7
```

**여기서 한 번 막혔다.** `retrofit_generator` 9.7.0이 retrofit 4.9.2와 호환되지 않아 `build_runner build`가 깨졌다. 분석해 보니 9.7.0에서 `Parser.DartMappable` 케이스가 추가됐는데, retrofit 4.9.2의 `switch` 문이 비-망라적(non-exhaustive)이라 코드 생성이 실패하는 패턴이었다. 일단 retrofit은 Stage 4 진입 전까지 빼두고, dio만으로 진행하기로 했다. `pubspec.yaml`에 NOTE 코멘트로 이유를 박아 두었다 — 나중에 누가 보더라도 왜 retrofit이 빠졌는지 알 수 있게.

## 4. 린트 정책 강화

`flutter_lints` 기본 룰만으로는 캡스톤 코드 리뷰에서 잡고 싶은 것들이 다 안 잡힌다. `analysis_options.yaml`에서 다음 항목을 추가로 켰다.

- `prefer_single_quotes`: 따옴표 일관성.
- `always_declare_return_types`: 반환 타입 누락 방지.
- `unawaited_futures`: 비동기 호출 누락 방지.
- `avoid_print`: `print()` 대신 `logger`만 쓰도록 강제.
- `riverpod_lint`: `custom_lint` 플러그인으로 Riverpod 안티패턴 자동 검출.

`flutter analyze`가 0건으로 통과해야 CI를 통과하도록 게이트를 둘 거라, 처음부터 룰을 빡세게 잡아두는 게 결과적으로 손이 덜 간다.

## 5. 환경별 엔트리포인트 — `--dart-define` 방식

flavors를 쓰는 안과 `--dart-define`을 쓰는 안 두 가지가 있는데, flavors는 Android/iOS 양쪽 다 별도 설정이 필요해서 캡스톤 일정 안에 부담이었다. 그래서 **`--dart-define`만으로 dev/prod를 가르고**, main 진입 시 `AppConfig.fromEnvironment()`로 한 번 읽어 들이는 방식을 택했다.

```dart
// lib/main.dart
void main() {
  useHashUrlStrategy();          // GitHub Pages 안전장치
  bootstrap();                   // ProviderScope + Logger + App 설치
}
```

```bash
# 개발
flutter run -d chrome --dart-define=APP_ENV=dev \
  --dart-define=API_BASE_URL=http://localhost:8080

# 프로덕션 빌드
flutter build web --release \
  --base-href "/oncare-flutter/" \
  --dart-define=APP_ENV=prod \
  --dart-define=API_BASE_URL=https://api.oncare.example.com
```

## 6. GitHub Actions — `ci.yml`

PR/푸시 단위로 검사 게이트를 둔다. **format → analyze → test** 순서로 실패 시 빠르게 떨어지도록 단계를 분리했다.

```yaml
# .github/workflows/ci.yml (요약)
name: CI
on: [push, pull_request]
jobs:
  verify:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: subosito/flutter-action@v2
        with:
          channel: stable
      - run: flutter pub get
      - run: dart format --output=none --set-exit-if-changed .
      - run: flutter analyze
      - run: flutter test
```

`dart format --set-exit-if-changed`로 포매팅 어긋남도 CI에서 떨어뜨린다. 나중에 PR이 늘었을 때 리뷰어가 포맷 코멘트로 시간을 쓰지 않도록 하기 위함이다.

## 7. GitHub Pages 자동 배포 — `deploy-web.yml`

main 브랜치에 푸시되면 `flutter build web`을 돌려 산출물을 `gh-pages` 브랜치로 푸시한다. **base-href를 빌드 인자로 박는 게 핵심**이다.

```yaml
- run: |
    flutter build web --release \
      --base-href "/oncare-flutter/" \
      --dart-define=APP_ENV=prod
- uses: peaceiris/actions-gh-pages@v4
  with:
    github_token: ${{ secrets.GITHUB_TOKEN }}
    publish_dir: ./build/web
```

여기서 한 가지 함정에 빠졌다. **GitHub Pages 사이트가 처음에는 활성화돼 있지 않다**. 워크플로우는 `gh-pages` 브랜치를 만들었지만, Pages 사이트 자체는 `Settings > Pages`에서 한 번 수동 설정해야 라이브 URL이 뜬다. 그래서 첫 푸시 후 한참동안 라이브 URL이 404를 뱉었다. 해결책으로 **stage 1 hotfix**로 `actions/configure-pages@v5`를 워크플로우에 끼워 넣어, 첫 실행 시 자동으로 Pages 사이트를 프로비저닝하도록 했다.

```yaml
- uses: actions/configure-pages@v5
```

이걸 박고 나니 처음부터 라이브 URL이 자동으로 노출됐다.

## 8. 오늘의 회고와 내일 할 일

- **고민했던 것**: retrofit 호환성 이슈에 시간을 꽤 썼다. 처음에는 generator 버전을 다운그레이드할까 했지만, Stage 4 진입 시점에 다시 재검토하기로 하고 일단 dio만으로 가는 결정을 내렸다. 캡스톤 초기에 호환성 문제로 멈춰 있는 건 시간 낭비라고 판단했다.
- **결정한 것**: **부트스트랩 완료**. 빈 앱이 Android·iOS·Web 빌드 통과, CI 그린, GitHub Pages 자동 배포까지 모두 동작한다. PLAN 기준 Stage 1 Exit 조건 충족.
- **남은 문제**:
  - **retrofit 재도입 시점** — 실제 API 클라이언트가 필요해지는 Stage 4 진입 전까지 generator 호환 버전 페어를 찾아야 한다.
  - **iOS/Android 릴리즈 자동화** — 우선 웹만 자동화했지만, 시연 직전에는 `.aab`/`.ipa`도 필요해질 수 있다. Stage 7에서 다시 본다.
  - **Pages base-href와 커스텀 도메인** — 추후 커스텀 도메인을 붙이면 base-href를 `/`로 바꿔야 한다. 이건 Q11 후속 결정.

내일은 **Stage 2 — Core Infrastructure**에 들어간다. 라우터(go_router + StatefulShellRoute) 셸을 짜고, Riverpod ProviderScope를 설치하고, 디자인 토큰을 ThemeData로 매핑하는 게 첫날 분량이다. 빈 4탭 BottomNav 셸을 보여주는 게 내일의 종료 조건이다.

---

## 📌 요약

오늘은 **Stage 1 부트스트랩**을 완료했다. `flutter create`로 프로젝트 골격을 깔고, 의존성을 한 번에 박고, 린트 룰을 빡세게 잡고, CI/CD 워크플로우 두 개(`ci.yml`, `deploy-web.yml`)를 띄웠다. GitHub Pages 첫 프로비저닝 함정도 hotfix로 해결했다. 다음 단계는 라우팅·상태관리·테마를 갖춘 코어 인프라 구축이다.
