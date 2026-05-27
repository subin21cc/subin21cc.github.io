---
title: "[Capstone-Start] Flutter MVP Day — Initial Commit"
date: 2026-05-23 23:30:00 +0900
categories: [Capstone, Start]
tags: [dev, capstone]
---

**📅 일시:** 2026. 05. 23

**💬 내용:** 일주일치 로컬 개발을 한 번에 push — Stage 4 ~ Stage 9 + 운동 탭 완성

---

토요일 아침까지 일주일치 작업이 전부 **로컬 브랜치**에만 쌓여 있었다. PLAN 문서, 부트스트랩, 코어 인프라, 디자인 시스템까지 — 매일 단계별로 진척이 있었지만 GitHub 레포를 어떻게 구성할지가 아직 정해지지 않아 push를 미뤘다. 오늘 **`Initial commit`**을 푸시했다. 이 시점부터 PLAN의 Stage×Phase 단위로 커밋을 끊으면서, 동시에 토요일이 끝나기 전에 가능한 단계까지 밀어붙이는 게 오늘 목표였다.

원격에 푸시한 커밋만 70개가 넘는다. 단계별로 회고를 정리한다.

## 1. Stage 0 — Discovery & Decisions (`docs:`)

PLAN.md (367줄), STRUCTURE.md (246줄), DESIGN_TOKENS.md, CONTRIBUTING_NOTES.md를 한 번에 커밋했다. PLAN §9 결정 이력에 Q1~Q12 모두를 박았고, `Co-Authored-By: Claude …` trailer 금지 규칙을 CONTRIBUTING_NOTES에 명시했다.

## 2. Stage 1 — Bootstrap (`feat(bootstrap):`, `ci(...)`, `chore:`)

- `chore: bootstrap flutter project (phase 1.1)`
- `chore: add lib/ skeleton & assets dirs (phase 1.2)`
- `chore(deps): add core dependencies (phase 1.3)` — Riverpod, go_router, dio, drift, fl_chart 등 한 번에.
- `chore: harden analyzer & lint rules (phase 1.4)` — `flutter_lints` + `riverpod_lint`(custom_lint).
- `feat(bootstrap): wire AppConfig, single main + dart-define (phase 1.5)`
- `ci: add format/analyze/test workflow (phase 1.6)`
- `ci(web): deploy to GitHub Pages on main push (phase 1.7)`
- `ci(web): auto-provision GitHub Pages site on first run (stage 1 hotfix)` — `actions/configure-pages@v5`로 첫 실행 시 Pages 사이트가 자동 활성화되도록 박아 둠.

## 3. Stage 2 — Core Infrastructure (PR #3, 7 phases)

`feature/fe-flutter-bootstrap` 브랜치를 머지하자마자 `feature/fe-flutter-core-infra` 브랜치를 열고 7개 phase를 차례대로 쌓았다.

- 2.1 `feat(router):` StatefulShellRoute + BottomNav 셸.
- 2.2 `feat(riverpod):` ProviderObserver + Logger DI.
- 2.3 `feat(theme):` tokenized colors/typo/spacing/radius + AppTheme.
- 2.4 `feat(network):` dio + 인터셉터 3종(logging / auth / mock).
- 2.5 `feat(storage):` drift + secure_storage + prefs.
- 2.6 `feat(errors):` AppError sealed + Result<T> + ErrorView/EmptyState.
- 2.7 `feat(l10n):` ko + en ARB 골격.

## 4. Stage 3 — Design System (PR #4, 5 phases)

- 3.1 `feat(catalog):` `/dev/ui-catalog` (dev-only).
- 3.2 `feat(design):` atom 5종 — Button/Card/Input/Badge/Avatar.
- 3.3 `feat(design):` molecule 3종 — MetricCard/ChartCard/SectionHeader.
- 3.4 `feat(responsive):` breakpoints + ResponsiveBuilder.
- 3.5 `feat(charts):` fl_chart 래퍼 — line/bar/donut.

여기까지가 어제까지 로컬에서 진행했던 분량이다. PR #4가 머지되면서 진짜 본격적인 오늘의 작업이 시작됐다.

## 5. Stage 4 — Feature MVPs (PR #5)

8개 피처를 mock 데이터 기반으로 한 번에 찍어냈다. 각 피처는 동일 패턴 — **mock 시드 → repository → controller(Riverpod) → page/widget**.

- 4.1 `feat(dashboard):` real summary content with mock repository.
- 4.2 `feat(diet):` diet record MVP with mock 3-meal day.
- 4.3 `feat(exercise):` weekly exercise MVP.
- 4.4 `feat(my_health):` vitals MVP with mock 7-day history.
- 4.5 `feat(ai_coach):` mock suggestions MVP.
- 4.6 `feat(notification):` in-app panel + simulated push.
- 4.7 `feat(place):` nearby places MVP w/ Maps placeholder.
- 4.8 `feat(auth):` mock social sign-in MVP.

여기서 한 가지 결정이 또 갈렸다. **AI Coach 응답**을 진짜 외부 API로 붙일지 mock으로 갈지. PLAN Q7 결정대로 **mock**으로 갔다 — `features/ai_coach/data/`에 5~6개 시나리오 JSON을 시드해 두고, 사용자 입력에 따라 키워드 매칭으로 시나리오 하나를 골라 반환하는 단순한 로직. 캡스톤 일정 안에서 prompt engineering까지 들어가면 시간이 안 맞는다고 봤다.

**알림(`notification`)**은 푸시 시뮬레이션이 까다로웠다. FCM을 쓰지 않기로 했기 때문에(Q9), `NotificationCenter` 컨트롤러가 30초 타이머로 mock 알림을 큐에서 하나씩 꺼내 in-app panel에 추가하는 방식으로 구현했다.

**`Place`** 화면의 지도(google_maps_flutter)는 Stage 4에서는 placeholder만 두고, 실제 지도 임베드는 시연 직전에 API 키와 함께 활성화하는 쪽으로 미뤘다. 카드 리스트는 mock 6개로 표시.

## 6. Stage 5 — Integration & Polish (PR #6)

- 5.1 `feat(nav):` `LoggerNavigatorObserver` + flutter `import` 정리.
- 5.2 `feat(dashboard):` staggered fade + slide on summary cards (`flutter_animate`).
- 5.3 `feat(a11y):` MetricCard exposes a single Semantics label — TalkBack/VoiceOver가 카드를 하나의 단위로 읽도록.
- 5.4 `feat(responsive):` two-column dashboard for tablet/desktop.
- 5.5 `feat(i18n):` externalise dashboard copy — 모든 문자열을 ARB로 이동.

5.3 a11y에서 한 번 막혔다. MetricCard는 child가 여러 `Text` 위젯을 갖는데, 기본적으로 스크린리더는 각각을 따로 읽는다. "오늘 칼로리 350kcal 1800 중"이 별개로 3번 읽혀서 정신 사납다. `Semantics(container: true, label: '...')` 한 번에 묶고, child `Text`들을 `ExcludeSemantics`로 가리니까 한 줄로 깔끔히 읽힌다.

## 7. Stage 6 — Quality (PR #7)

테스트 카테고리 4종을 한 번에 채웠다.

- 6.1 `test:` model invariants / edge cases — `DietDay`, `ExerciseWeek`, `VitalLog` 같은 모델의 잘못된 인자 거부 케이스.
- 6.2 `test(dashboard):` page-level widget tests for data & error paths — happy path 1개 + 에러/로딩 상태 2개.
- 6.3 `test(golden):` infra + AppButton variants snapshot — `golden_toolkit`으로 atom의 시각 회귀 가드.
- 6.4 `test(integration):` app smoke — `integration_test/`에 4탭 전환 시나리오 1개.

`flutter test`가 그린이고, golden은 처음 한 번 생성 후 변경 없음을 확인했다. 커버리지는 70%+를 목표로 했지만 실제로는 63%선. Stage 8 이후로 한 번 더 채워 넣을 거다.

## 8. Stage 7 — Release Scaffolding (PR #8)

- `chore(release):` `RELEASE.md` + `CHANGELOG.md` — v0.1.0+1 초안.
- 버전 정책: `--build-name`/`--build-number`로 빌드 시 주입. `pubspec.yaml`의 `version:`은 기본값으로만 둠.

여기까지가 14시 30분쯤. 시간이 남아서 **Stage 8(UX alignment)**까지 같은 날 밀어붙이기로 했다.

## 9. Stage 8 — UX Alignment with React Original (PR #9)

원본 React 프로토타입과 픽셀 단위로 정렬하는 단계. 이때부터 디자인 결정의 사소한 어긋남이 한 번에 쏟아져 나왔다.

- 8.1 `refactor(theme):` align tokens with original `default_shadcn_theme.css` — primary, muted, border 색을 React 원본 정확히 옮김.
- 8.2 `feat(shell):` align BottomNav with React original + `OncareHeader`.
- 8.3 `feat(dashboard):` five-card home layout matching the React original — 스트릭 / 오늘의 건강 / 일정 / 주간 점수 / AI 코치 5장.
- 8.4 `feat(diet):` nutrition summary + AI coach card + meal list.
- 8.5 `feat(exercise):` four stat cards + chart + AI coach + history.
- 8.6 `feat(my_health):` profile + risk + indicators + points + settings.
- 8.7 `feat(modals):` Calendar sheet + QuickInput + AddEvent dialogs.
- 8.8 `feat(panels):` right-slide Notification + AI Coach overlays.

여기서 자잘한 fixup 커밋이 다섯 개쯤 들어갔다 — `DietDay` 생성자 시그니처가 바뀌면서 unit test가 깨졌고, `MealCard` pill 색상이 어긋났고, nav-switching widget test가 더 이상 AppBar에 의존하지 못하게 됐고. 다 즉시 잡았다.

스타일 차원의 정돈도 같이 — `style: drop deprecated activeColor + use Dart 3 wildcards`, `style(my_health): drop unnecessary braces in '$rank위 랭킹'`. Dart 3의 `(_, _)` wildcard 패턴은 `separatorBuilder` 같은 콜백에서 미사용 인자를 강제로 명시할 때 깔끔하다.

CHANGELOG와 버전을 **v0.2.0+2**로 올림.

## 10. Stage 9 — Local Backend via drift + LocalApiInterceptor (PR #10)

이게 오늘 가장 만족스러웠던 부분이다. **백엔드 팀이 아직 API를 만들지 않은 상태**에서 프론트엔드만으로 풀스택처럼 동작하게 하는 트릭이다.

기본 아이디어는 이거다 — dio 인터셉터로 **request를 가로채서 drift DB에서 직접 응답을 만들어 반환**한다. 프론트엔드 코드는 평상시처럼 `GET /diet/days/today`를 부르고, network 레이어에서 진짜 API인 척 drift가 응답한다. 백엔드가 붙으면 인터셉터만 비활성화하면 코드 변경 없이 그대로 전환된다.

- 9.1 `feat(storage):` drift schema v2 — `diet_days`, `meal_entries`, `exercise_sessions`, `vital_logs`, `schedule_events` 5개 테이블 추가.
- 9.2 `feat(storage):` seed React mock data into drift on first run — `seeded_v1` 플래그로 한 번만.
- 9.3 `feat(network):` `LocalApiInterceptor` skeleton + snake_case mapper — 백엔드 컨벤션에 맞춰 응답을 snake_case로 흘림.
- 9.4 `feat(diet):` `GET /diet/days/today` via LocalApiInterceptor + `DioDietRepository`.
- 9.5 `feat(vitals):` `POST /vitals/{kind}` + `GET .../latest` with QuickInput wiring — 대시보드 QuickInput 모달에서 체중을 입력하면 drift에 저장되고, 다음 fetch에서 잡힌다.
- 9.6 `feat(exercise):` `GET /exercise/weeks/current` with weekly aggregation — drift 쿼리에서 weekday별 GROUP BY.
- 9.7 `feat(schedule+noti):` `/schedule/events` and `/notifications`.
- 9.8 `feat(dashboard):` `GET /dashboard/summary` aggregating diet+exercise+vital+schedule — 가장 복잡한 엔드포인트. 4개 도메인 데이터를 한 번에 묶어 응답.
- 9.9 `feat(misc):` AI Coach / users.me / MyHealth / Places.
- 9.10 `docs(plan):` Stage 9 decision log + e2e smoke test.

추가 docs도 같이 — `docs(backend): API_CATALOG + DUMMY_BACKEND options`, `docs: link API_CATALOG + DUMMY_BACKEND from README`. **`API_CATALOG.md`**에 prefix와 path 규약, request/response 페이로드 스키마를 백엔드와 공유할 수 있게 적었다. 백엔드 팀이 실제 API를 짤 때 이 카탈로그를 참고하면 프론트는 그대로 동작한다.

여기서 한 번 크게 막혔다. **Web 빌드가 drift WASM 없이 부팅이 안 됨**.

```
fix(web): bundle drift WASM (sqlite3.wasm + drift_worker.js)
fix(web): pair sqlite3.wasm with the resolved sqlite3 dart package + 4xx/5xx as DioException
```

drift는 web에서 SQLite를 WASM으로 돌리는데, `sqlite3.wasm`과 `drift_worker.js`를 빌드 산출물에 같이 넣어줘야 한다. 처음에는 `web/` 디렉토리에 임의 버전을 박아 두니까 `sqlite3` Dart 패키지와 SHA가 안 맞아 부팅이 깨졌다. 정답은 **pubspec lock으로 묶인 `sqlite3` 패키지의 정확한 버전과 페어를 맞춰서** 다운로드하는 것. `tool/fetch_drift_wasm.sh` 스크립트로 자동화해 두었다 — CI 배포 워크플로우에서 이 스크립트를 부르도록 추후 박을 거다(내일 작업).

또 한 가지 짚었던 것 — dio가 기본적으로 4xx/5xx를 정상 응답으로 통과시키는 옵션이 있는데, LocalApiInterceptor가 에러 시 `DioException`을 던지지 않으면 controller에서 `try/catch`가 안 잡힌다. `validateStatus: (_) => true`를 끄고, 인터셉터에서 status code별로 `DioException` 정확히 throw하도록 보강했다.

CHANGELOG와 버전을 **v0.3.0+3**으로 올림. `chore: end of stage 9 — local backend via drift+LocalApiInterceptor`.

## 11. 운동 탭 — Stage 9 이후의 보너스 (PR #11)

PR #10 머지 후에도 시간이 좀 남았다. 운동 페이지에 **헬스장(Place) 탭 진입점**과 **운동 기록 추가 FAB**을 붙였다.

- `feat(exercise):` tab-based 운동 page with 헬스장 view + 운동 기록 추가 FAB.
- `feat(exercise):` seed multi-type sessions per day for stacked WeeklyActivity chart — 유산소 / 근력 / 스트레칭 3종이 한 막대 안에 누적되도록 stacked bar로 전환. `seeded_v1` → `seeded_v2`로 시드 버전을 올렸고, 업그레이드 시 단일 타입 데이터가 자동 정리되도록 마이그레이션 로직 추가.
- `feat(chart):` wider bars, always-on weekday labels, stacked tooltip — fl_chart 기본 툴팁은 한 번에 1개 시리즈만 보여줘서, 누적 막대 위에서 세 시리즈를 한 번에 보여주는 커스텀 툴팁으로 교체.

`ExerciseWeek.workoutCount`가 한 가지 헷갈리는 부분이었다 — 같은 날에 유산소 30분 + 근력 30분이 있으면 "운동 2회"인지 "운동 1회(2종)"인지. 원본 React에서는 "운동한 날 수"였기 때문에 collapse 로직을 추가해서 같은 날 여러 행을 1로 묶었다.

## 12. 오늘의 회고와 내일 할 일

- **고민했던 것**:
  - Stage 9 — 백엔드가 없는데 백엔드처럼 보이게 하는 게 과한 엔지니어링 아닌가? 결국 **백엔드와 인터페이스를 미리 합의해 둘 가치**가 있다고 결론. `API_CATALOG.md`가 그 합의의 산물이라 헛돈 게 아니다.
  - drift WASM 페어링 — `sqlite3` 패키지 lock 버전과 wasm 파일이 서로 SHA가 맞아야 한다는 사실을 깨닫는 데 한참 걸렸다.
- **결정한 것**: 일주일치 로컬 작업을 원격에 모두 푸시 완료. Stage 0~Stage 9 + 운동 탭. v0.3.0+3 태깅.
- **남은 문제**:
  - **CI 배포 워크플로우에서 drift WASM 자동 다운로드** — 지금은 로컬 빌드에서만 동작. 내일 `tool/fetch_drift_wasm.sh`를 워크플로우에 끼울 예정.
  - **README와 다이어그램** — 지금은 ASCII art뿐. 캡스톤 발표 자료로 쓰려면 SVG로 옮겨야 한다.
  - **UI 디테일** — 카드 보더, AI 코치 카드 아이콘, 식단/운동의 날짜 헤더 등 자잘하게 React 원본과 다른 부분이 5~6개 남아 있다.
  - **시연 환경 base-href** — 커스텀 도메인을 붙일 경우 `/oncare-flutter/` → `/`로 바꿔야 함.

내일은 PR 단위로 작게 끊으면서 **(1) README/다이어그램 정리, (2) UI 디테일 정렬**을 진행할 예정이다. 시연 D-day가 다가오기 때문에 큰 기능 추가보다는 만들어진 화면을 깔끔하게 다듬는 쪽으로 가는 게 맞다.

---

## 📌 요약

오늘은 일주일치 로컬 작업을 원격에 한꺼번에 푸시한 날이다. **PR #2~#11**까지 10개 PR을 머지하면서 **Stage 0(Discovery) → Stage 9(Local backend via drift+LocalApiInterceptor) + 운동 탭 stacked chart**까지 완성.
