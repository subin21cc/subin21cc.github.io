---
title: "[Capstone-Start] UI 카드 리스타일 & README 다이어그램"
date: 2026-05-24 17:30:00 +0900
categories: [Capstone, Start]
tags: [dev, capstone]
---

**📅 일시:** 2026. 05. 24

**💬 내용:** README ASCII → SVG 다이어그램, outlined 카드, robot 아이콘, 혈당 row 제거, 식단 데이트 헤더, 헤더 탭 와이어링

---

어제 일주일치 작업을 한꺼번에 푸시하고 나니, 화면을 한 발 뒤로 물러나서 보게 됐습니다. 동작은 다 되는데 **자잘한 디테일이 React 원본과 어긋나 있는 부분**이 여러 군데 남아 있었습니다. 그리고 README는 여전히 ASCII art 다이어그램 그대로라 캡스톤 발표 자료에 그대로 쓸 수가 없습니다.

오늘은 큰 기능을 더 짜기보다는 **두 개의 PR로 묶어서 정돈**하는 데 시간을 썼습니다.

1. **PR #12** — `feature/readme-diagrams-and-goldens` (오후 2시): README ASCII → SVG 다이어그램, golden 회귀 스냅샷 정리.
2. **PR #13** — `feature/ui-card-restyle-and-feedback` (오후 3~4시): 카드 outlined 변형, robot 아이콘 통일, 식단/운동 페이지 디테일 정리.
3. 끝물에 README 팀 테이블 정렬 자잘한 수정 1건.

## 1. README ASCII → SVG (PR #12)

기존 README는 시스템 아키텍처, 데이터 흐름, 화면 트리를 모두 **```...```로 감싼 ASCII art**로 그려두고 있었습니다. 코드 리뷰는 편하지만, 발표 슬라이드에 캡처해서 넣으면 가독성이 너무 떨어집니다.

문제 해결 방법은 두 가지였다.

- (A) Mermaid 차트로 옮기기 — GitHub README에서 자동 렌더링.
- (B) 직접 SVG로 그리기.

(A)가 빠르지만, 발표 슬라이드에 그대로 임베드하려면 결국 렌더링된 이미지를 추출해야 해서 작업이 두 번 들어갑니다. 그래서 **(B) SVG로 직접 그리기** 쪽으로 갔습니다. `assets/svg/` 디렉토리에 다이어그램 3장(시스템 아키텍처 / 데이터 흐름 / 화면 트리)을 두고, README는 `<img src="assets/svg/...svg" />` 태그로 임베드.

```markdown
## 시스템 아키텍처

<img src="assets/svg/system-architecture.svg" alt="On-Care 시스템 아키텍처" />
```

SVG는 다크/라이트 모두 OK이도록 색을 토큰 단위에서 컨트롤. README에서 GitHub의 자동 다크 모드 변환도 자연스럽게 적용됩니다.

### Golden 실패 스냅샷 커밋

같은 PR에 **golden test 실패 스냅샷**도 같이 박았습니다. Stage 6.3에서 `golden_toolkit`으로 atom 위젯들의 시각 회귀 가드를 깔아 뒀는데, 그 이후로 디자인 토큰 한두 개를 손대면서 일부 golden이 어긋난 상태였습니다. 평소엔 CI에서 차이만 출력하는데, **차이 PNG 자체를 레포에 같이 박아 두면** PR 리뷰어가 "어디가 어떻게 바뀌었나"를 한눈에 볼 수 있습니다.

`failures/` 디렉토리에 expected/actual/diff 3종 PNG를 넣고, 그 PR 안에서 master baseline을 갱신했습니다.

## 2. UI 카드 outlined 변형 (PR #13, 첫 커밋)

원본 React 프로토타입에서는 대시보드 상단 카드들이 **흰 배경 + 회색 보더** 스타일(`outlined`)이고, 콘텐츠 영역의 보조 카드는 **연한 muted 배경 + 보더 없음**(`filled`) 스타일이다. Flutter 쪽 `AppCard`는 Stage 3에서 filled 한 가지만 만들어 둬서, 메인 패널들이 어색하게 떠 있는 느낌이었다.

`AppCard`에 `variant` enum을 추가했다.

```dart
enum AppCardVariant { filled, outlined }

class AppCard extends StatelessWidget {
  const AppCard({
    super.key,
    required this.child,
    this.variant = AppCardVariant.filled,
    this.padding = const EdgeInsets.all(16),
  });
  // outlined: bg=white, border=AppColors.border, radius=12
  // filled: bg=AppColors.muted, no border, radius=12
}
```

대시보드 상단(오늘의 건강 요약, 일정, 주간 점수), 식단의 영양소 카드, 운동의 통계 카드, MyHealth의 프로필/지표 카드를 모두 `outlined`로 전환. 시각적으로 한층 정돈됐다.

## 3. 온이의 피드백 — robot 아이콘으로 (PR #13)

원본 React에서 "온이의 피드백" 카드는 로봇 모양 아이콘을 쓰는데, Flutter 쪽은 임시로 `lightbulb_outlined`를 박아 둔 상태였다. 빠르게 `smart_toy_outlined`로 교체.

```dart
const Icon(Icons.smart_toy_outlined, size: 24, color: AppColors.primary)
```

`smart_toy_outlined`는 Material Symbols에서 정확히 React 원본의 로봇 톤과 매칭된다. AI 코치 진입 카드도 같이 이 아이콘으로 통일.

## 4. 대시보드 건강 요약에서 혈당 row 제거 (PR #13)

원본 프로토타입에서는 대시보드 "오늘의 건강 요약" 카드에 칼로리/나트륨/혈당 3행이 들어 있었지만, **사용자 인터뷰 결과 혈당은 매일 측정하는 사용자가 거의 없다**는 피드백이 반복됐다. 같은 정보가 MyHealth 탭에 별도로 있고, 대시보드에서는 칼로리/나트륨 2행만 보여주는 게 정보 밀도 면에서 더 깔끔하다.

대시보드 controller(`DashboardController`)의 `indicators` 리스트에서 혈당을 제거. 동시에 컨트롤러 테스트가 3행 가정으로 짜여 있어서 한 커밋 더 추가했다.

```
test(dashboard): align controller test with 3-row indicator list
```

테스트도 2행 가정으로 맞춤. 이건 위젯 테스트가 일찍 깔려 있어서 회귀가 즉시 잡힌 좋은 사례였다.

## 5. 나트륨 경고 배너도 제거 + 통합 피드백 카드로 (PR #13)

같은 맥락으로, "오늘의 건강 요약" 카드 안에 있던 나트륨 경고 배너도 제거했다. 카드 한 장 안에 정보 종류가 3가지(지표 / 진행률 / 경고)를 다 담는 건 너무 빽빽하다.

대신 **"온이의 피드백" 카드를 통합 일일 피드백 카드로 격상**시켰다. robot avatar + 한 줄 톤 메시지 + 액션 칩 1~2개. 칼로리/나트륨 진행률 기준이 임계 영역에 들어오면 여기에 알림이 뜨도록 controller에서 메시지를 합성.

```
feat(dashboard): combined daily-feedback card with robot avatar
```

## 6. 식단 페이지 — 날짜 헤더 + 주간 스트립 (PR #13)

식단 페이지 상단에 **"오늘 날짜 + 주간 스트립(일~토)"**을 박았다. 원본 React에서는 디폴트로 일요일부터 토요일까지 한 줄로 7개 셀이 보이고, 현재 선택된 날짜가 강조된다.

Flutter에서 동등하게 옮기면서 한 가지 함정에 빠졌다. `DateTime.weekday`는 **월요일이 1, 일요일이 7**인 ISO 기준이다. 그런데 우리 디자인은 **일요일이 0(첫 번째 열)** 가정. 한 번 잘못 박으니 강조된 셀이 한 칸 오른쪽으로 밀렸다. `weekday % 7` 같은 작은 보정으로 해결.

```dart
int sundayIndexOf(DateTime d) => d.weekday % 7;
```

```
feat(diet): date header + Sunday-to-Saturday week strip
```

## 7. 운동 코치 메시지 — '오늘' 표현 (PR #13)

운동 페이지의 AI 코치 카드 메시지가 **"일요일에는 가벼운 스트레칭이 좋아요"**처럼 요일을 직접 박는 mock이었는데, 사용자가 일요일에 봤을 때 어색했다. (이미 일요일인데 "일요일에는…"이라고 적힌 게 부자연스럽다.) 오늘 날짜와 메시지의 요일이 일치하면 **'오늘'로 치환**하도록 controller에서 후처리.

```
feat(exercise): coach message says '오늘' instead of '일요일'
```

이런 식의 자잘한 자연어 처리가 mock UX 품질을 결정한다.

## 8. 헤더 — 캘린더/알림 탭 와이어링 (PR #13)

대시보드에서는 헤더의 캘린더 / 알림 아이콘이 모달과 패널을 열도록 이미 와이어돼 있었는데, **Diet / Exercise / My 페이지에서는 같은 아이콘이 데드 링크**였다. (Stage 8.2에서 OncareHeader를 만들 때 dashboard만 우선 와이어한 흔적.)

```
feat(header): wire calendar + notification taps on Diet / Exercise / My
```

각 페이지의 `OncareHeader`에 `onCalendarTap`/`onNotificationTap` 콜백을 받아서, 페이지 자체에 캘린더 시트와 알림 패널을 띄우는 형태로 연결.

여기서 한 가지 더 정리해 둘 만한 것: **모달 띄우기 책임을 어디에 둘지**. 원래는 `Header`가 직접 `showModalBottomSheet`를 부르도록 박아 두었는데, 그러면 같은 모달이 페이지 4개에서 4번 호출되면서 backdrop 중첩 같은 자잘한 문제가 생긴다. **Header는 콜백만 노출하고, 모달 띄우기는 페이지에서**로 책임을 분리. 이러면 각 페이지가 자기 컨텍스트에 맞는 옵션으로 모달을 띄울 수 있다.

## 9. 끝물에 — 팀 테이블 정렬 (`docs(readme):`)

PR #13 머지 후 README의 팀 소개 테이블이 멤버 이름 길이에 따라 컬럼 너비가 들쭉날쭉한 게 거슬렸다. GitHub의 마크다운 테이블은 컬럼 너비를 직접 지정할 방법이 없어서, **각 셀 최상단에 `1×1` 투명 PNG 스페이서**를 박는 트릭으로 동일 너비를 강제했다.

```html
<img src="assets/spacer.png" width="160" height="1" alt="" />
```

투명 1×1 PNG는 GitHub 캐시에서 친절하게 잡아주니까 페이지 로드 비용도 무시할 수준.

```
docs(readme): equalize Team table column widths with 1×1 transparent spacer
```

## 10. 오늘의 회고와 내일 할 일

- **고민했던 것**:
  - 다이어그램 Mermaid vs SVG — 발표 자료 재사용성 때문에 SVG 직접 그리기 선택.
  - 혈당 row 제거 — 정보를 빼는 결정은 늘 고민된다. MyHealth에 같은 정보가 있다는 redundancy 확인 후 결정.
  - 모달 띄우기 책임 — Header에서 직접 띄울지, 페이지에 위임할지. 후자.
- **결정한 것**:
  - SVG 다이어그램 3장으로 README 가독성 ↑.
  - `AppCard`에 `outlined` variant 추가.
  - "온이의 피드백" 카드를 통합 일일 피드백 카드로 격상.
  - Diet/Exercise/My 헤더 캘린더·알림 와이어링 완료.
- **남은 문제**:
  - **GitHub Pages 자동 배포가 아직 main 브랜치 푸시 시 동작하지 않음** — Stage 1.7에서 분명히 워크플로우는 깔았는데, 어딘가에서 path 또는 base-href가 안 맞는 듯하다. 내일 보고 잡는다.
  - **camera 식단 추가 플로우** — 원본에서는 식단 페이지 FAB → 카메라 → 분석 로딩 → 결과로 진행되는데, Flutter에서는 FAB만 있고 카메라 UI가 비어 있다.
  - **MyHealth 페이지의 지표 추이 모달** — 카드 탭 시 추이 그래프 모달이 뜨는 흐름이 아직 미구현.
  - **설정 모달 4종** — 개인 정보 / 건강 데이터 / 알림 설정 / 고객 지원 — MyHealth 하단 설정 리스트가 dead link 상태.

내일은 **(1) GitHub Pages 배포 실제 동작, (2) Diet 카메라 분석 로딩 플로우, (3) MyHealth 지표 추이/설정 모달 4종**까지 정리할 예정이다. 발표가 며칠 안 남아서 시연 시나리오가 매끄럽게 흐르는 게 가장 중요하다.

---

## 📌 요약

오늘은 두 개의 PR로 **README 가독성**과 **UI 디테일**을 정돈했다. ASCII 다이어그램을 SVG 3장으로 옮기고, golden 회귀 스냅샷을 레포에 박았다. `AppCard`에 `outlined` variant를 추가해 주요 패널을 흰 배경+회색 보더로 통일하고, 온이의 피드백 아이콘을 robot으로 교체, 대시보드 건강 요약에서 혈당 row 제거, 식단 페이지에 데이트 헤더+주간 스트립 추가, 운동 코치 메시지 자연어 보정, 헤더의 캘린더/알림 탭 와이어링까지. 발표 D-day가 임박해서 큰 기능 추가보다는 디테일 정렬에 집중하는 단계로 들어왔다.
