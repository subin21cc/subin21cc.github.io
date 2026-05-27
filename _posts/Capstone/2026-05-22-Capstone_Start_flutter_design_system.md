---
title: "[Capstone-Start] Flutter 디자인 시스템"
date: 2026-05-22 22:30:00 +0900
categories: [Capstone, Start]
tags: [dev, capstone]
---

**📅 일시:** 2026. 05. 22

**💬 내용:** UI 카탈로그 · atom · molecule · 반응형 헬퍼 · fl_chart 래퍼 구축

---

오늘 목표는 **`/dev/ui-catalog` 라우트에서 모든 토큰과 위젯을 한 화면에서 시연 가능**한 상태까지 만드는 것이다. 카탈로그가 없으면 Stage 4에서 페이지를 짜면서 동시에 위젯을 디버깅하느라 시간이 두 배로 든다. 그래서 카탈로그를 phase 3.1로 가장 먼저 둔다.

## 1. Dev-only UI 카탈로그 (Phase 3.1)

`/dev/ui-catalog`는 prod 빌드에서는 접근 불가하도록 `AppConfig.isDev`로 라우트 자체를 가드했다. 카탈로그 페이지는 사실상 한 페이지짜리 스크롤뷰 — 토큰 섹션부터 atom, molecule, charts까지 위에서 아래로 흐른다.

```dart
if (AppConfig.instance.isDev)
  GoRoute(
    path: '/dev/ui-catalog',
    builder: (_, __) => const UiCatalogPage(),
  ),
```

페이지는 섹션 헤더(`SectionHeader` molecule을 dogfooding) + 그 아래 위젯 데모가 반복되는 구조다. 카탈로그 자체가 디자인 시스템의 첫 번째 사용자가 되는 셈이라, 여기서 어색한 부분이 보이면 그게 곧 atom/molecule API의 결함 신호다.

## 2. Atom 위젯 — Button, Card, Input, Badge, Avatar (Phase 3.2)

원본 React 코드의 shadcn/ui 컴포넌트들을 1:1로 옮기지는 않았다. shadcn의 variant 시스템을 Dart enum + sealed로 재해석했다.

### AppButton

shadcn의 `default`/`destructive`/`outline`/`secondary`/`ghost`/`link` 6개 variant 중, 캡스톤 화면에서 실제로 쓰는 건 4개라 그것만 만들었다.

```dart
enum AppButtonVariant { primary, secondary, outline, ghost }
enum AppButtonSize { sm, md, lg }

class AppButton extends StatelessWidget {
  const AppButton({
    super.key,
    required this.label,
    required this.onPressed,
    this.variant = AppButtonVariant.primary,
    this.size = AppButtonSize.md,
    this.leading,
    this.loading = false,
  });
  // ...
}
```

`loading` 플래그가 들어 있는 이유는, 식단 분석/저장 같은 비동기 액션에서 버튼이 두 번 눌리는 걸 막기 위해서다. `loading == true`이면 `onPressed`를 null로 무시하고, child 위치에 `CircularProgressIndicator(strokeWidth: 2)`를 띄운다.

### AppCard

shadcn `Card`는 header/content/footer 슬롯이 있는데, Flutter에서는 그냥 child 한 개만 받고 패딩과 라운드만 박는 형태로 단순화했다. Stage 8에서 React 원본과 픽셀 단위 정렬할 때 `variant: outlined`를 추가하게 되지만, 지금은 elevated 한 가지뿐이다.

### AppInput

`TextField`를 한 번 감싸서 라벨/에러/헬퍼 텍스트 슬롯을 표준화. `keyboardType`을 외부에서 지정 가능하게 둬서 식단 입력(체중 등 숫자), 검색(텍스트), 시간(스피너)에 모두 쓸 수 있도록 만들었다.

### AppBadge, AppAvatar

각각 한 화면짜리 위젯이라 따로 적을 게 별로 없다. AppAvatar는 이미지 url이 null이면 이니셜을 표시하도록 fallback을 두었다.

## 3. Molecule — MetricCard, ChartCard, SectionHeader (Phase 3.3)

atom 5종이 끝나니 molecule 3종이 자연스럽게 따라왔다.

### MetricCard

대시보드의 "오늘의 건강 요약"에서 칼로리/나트륨/혈당을 보여주는 카드. **시각적 톤 4종**(neutral / good / warning / danger)을 enum으로 받아서 색상과 배경을 매핑한다. 진행률 바가 100%를 넘으면 자동으로 danger로 바뀌는 로직을 내부에 둘지 외부에서 결정해 줄지 잠깐 고민했는데, **결정은 controller에 두고 MetricCard는 그냥 받은 톤대로 그리는 dumb widget**으로 두기로 했다. UI 위젯에서 비즈니스 임계값을 들고 있으면 테스트하기가 어렵다.

### ChartCard

차트 래퍼를 감싸는 카드. 헤더(제목 + 보조 액션) + 차트 영역 + 푸터(범례) 슬롯. Stage 3.5에서 fl_chart 래퍼가 들어가면 ChartCard가 호스트가 되는 구조다.

### SectionHeader

페이지에서 섹션을 구분하는 큰 타이틀. 카탈로그를 도그푸딩하는 첫 번째 위젯이다 — 카탈로그 자체가 "Tokens 섹션", "Atoms 섹션" 등의 SectionHeader로 구분돼 있다.

## 4. 반응형 헬퍼 — Breakpoints + ResponsiveBuilder (Phase 3.4)

웹 빌드까지 커버해야 해서, 모바일 1열 / 태블릿 2열 / 데스크탑 2~3열 전환이 필요하다. Tailwind 식 breakpoint 상수를 박고, `LayoutBuilder`를 감싼 헬퍼를 만들었다.

```dart
abstract class AppBreakpoints {
  static const mobile = 600.0;
  static const tablet = 900.0;
  static const desktop = 1200.0;
}

class ResponsiveBuilder extends StatelessWidget {
  final WidgetBuilder mobile;
  final WidgetBuilder? tablet;
  final WidgetBuilder? desktop;
  // LayoutBuilder로 maxWidth를 보고 분기.
}
```

지금은 카탈로그에서 demo만 보여주고, 실제 적용은 Stage 5.4(반응형 / 웹 최적화)에서 Dashboard에 두-컬럼 레이아웃을 박을 때 본격적으로 쓴다.

## 5. fl_chart 래퍼 — Line, Bar, Donut (Phase 3.5)

`fl_chart`는 기능은 풍부한데 API가 verbose해서, 페이지마다 `LineChartData(...)`를 직접 조립하면 코드가 금방 더러워진다. 그래서 **`AppLineChart` / `AppBarChart` / `AppDonutChart`** 세 가지 얇은 래퍼를 만들었다.

```dart
class AppLineChart extends StatelessWidget {
  const AppLineChart({
    super.key,
    required this.points,           // List<({double x, double y})>
    this.color = AppColors.primary,
    this.showDots = true,
    this.minY,
    this.maxY,
  });
  // ...
}
```

핵심은 **차트 입력 데이터를 도메인 그대로 받지 않는다**는 점이다. 차트는 `(x, y)` 튜플 리스트만 받고, 도메인 → 좌표 변환은 호출자(컨트롤러)가 책임진다. 이렇게 안 하면 차트 위젯이 도메인 모델을 직접 알게 돼서 재사용이 어려워진다.

데모용으로 카탈로그에 다음 3개를 박았다:
- **Line**: 7일 체중 추세 (clinically realistic한 70~75kg 범위).
- **Bar**: 주간 운동 시간 (`(요일, 분)` 7쌍).
- **Donut**: 오늘 영양소 비율 (탄/단/지 33/33/34).

Stage 8 이후에 **stacked bar**(운동 유형별 누적)와 **Y축 범위 클램핑**(혈압 등 임상 범위)이 추가되지만, 그건 그때 가서 한다.

## 6. Stage 3 Exit — 카탈로그 한 페이지로 검증

phase 3.1~3.5를 모두 끝낸 시점에 `/dev/ui-catalog`에서 다음이 모두 동작하는 것을 확인했다.

- Color/Spacing/Typography 토큰 시각 데모.
- AppButton 4 variants × 3 sizes 그리드.
- AppCard / AppInput / AppBadge / AppAvatar 데모.
- MetricCard(4 톤) / ChartCard / SectionHeader 데모.
- ResponsiveBuilder로 너비별 레이아웃 변화 시연.
- Line / Bar / Donut 차트 3종.

여기서 발견한 자잘한 결함 두 개를 바로 고쳤다.
- AppButton의 `outline` variant가 dark theme에서 보더가 너무 흐려서, 토큰을 `border` 대신 `mutedForeground`로 매핑.
- MetricCard `warning` 톤의 노란 배경이 너무 진해서 60% 투명도로 톤다운.

카탈로그가 없었으면 이 두 가지 모두 Stage 4 페이지를 짜다가 발견해서 그제서야 디자인 시스템 PR을 다시 열어야 했을 거다.

## 7. Stage 4 진입을 위한 사전 작업

Stage 4(feature MVPs)는 phase 7개로 분량이 많은데, 토요일 하루에 다 끝낼 작정이라 사전 작업을 좀 해뒀다.

- **mock 데이터 JSON 시드** — 원본 React 앱의 `mockData.ts`에 있던 더미를 `core/storage/seed_data.dart`로 옮겨 두는 작업의 초안.
- **feature별 디렉토리 골격** — `features/{dashboard,diet,exercise,my_health,ai_coach,notification,place,auth}` 각각에 `data/`, `domain/`, `presentation/` 빈 폴더와 `.gitkeep`.
- **repository 인터페이스 시그니처** — `domain/repositories/<name>_repository.dart`에 메서드 시그니처만 박아 둠. 구현은 내일.

## 8. 오늘의 회고와 내일 할 일

- **고민했던 것**:
  - 카탈로그를 dev-only로 라우트 가드할지, prod에서도 노출할지. 결국 gauntlet으로 가드. 마케팅/시연용으로 노출해야 할 경우엔 별도의 페이지로 만들 거다.
  - shadcn 6 variants 다 옮길지, 캡스톤에서 쓰는 4개만 만들지. 후자 선택 — YAGNI.
  - MetricCard 임계값 — UI에서 결정 vs controller에서 결정. controller로 빼서 dumb widget 유지.
- **결정한 것**: 디자인 시스템 5 phase 모두 완료. 카탈로그에서 토큰부터 차트까지 모두 시연 OK.
- **남은 문제**:
  - **shadcn 테마와 픽셀 단위 정렬 안 됨** — Stage 8(UX alignment)에서 별도로 잡는다. 지금은 우리만의 토큰 기준.
  - **dark theme 검수 미흡** — light 위주로 시연했고, dark는 카탈로그에서 한 번씩만 토글해 봤다.
  - **golden test 부재** — 위젯 모양이 회귀하지 않게 막는 게 필요한데, Stage 6.3까지 미뤘다.

내일은 **Stage 4 — Feature MVPs**부터 시작해서, 흐름이 잘 풀리면 **Stage 5(Integration & Polish), Stage 6(Quality), Stage 7(Release)**까지 하루에 밀어볼 생각이다. 시간이 더 남으면 **Stage 8(UX alignment with React original)**과 **Stage 9(Local backend via drift)**까지 손대보고 싶다. 무리한 일정이지만, 토요일은 시간이 통째로 비어 있어서 한 번에 가는 게 컨텍스트 스위칭 비용을 줄이는 가장 좋은 방법이다.

---

## 📌 요약

오늘은 **Stage 3 — 디자인 시스템**을 끝냈다. dev-only `/dev/ui-catalog` 라우트 위에 atom 5종(Button/Card/Input/Badge/Avatar), molecule 3종(MetricCard/ChartCard/SectionHeader), 반응형 헬퍼, fl_chart 래퍼 3종(Line/Bar/Donut)을 모두 얹어서 한 화면에서 시연 가능한 상태가 됐다. 이제 화면별로 디자인 시스템을 조립해 MVP를 찍어내는 일만 남았다.
