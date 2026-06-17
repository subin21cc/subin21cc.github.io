---
title: "[Capstone-Start] GitHub Pages 배포 & 설정 모달"
date: 2026-05-25 22:30:00 +0900
categories: [Capstone, Start]
tags: [dev, capstone]
---

**📅 일시:** 2026. 05. 25

**💬 내용:** GitHub Pages 배포 자동화, 식단 카메라 플로우, FAB 통일, 건강 지표 추이 모달, 설정 모달 4종, 차트 Y축 클램핑

---

## 1. README 브랜드 배너 & 로고 (PR #14)

README 상단에 **브랜드 배너 + 로고**를 박았다. On-Care 타이틀 옆에 작은 로고를 둬서 발표 자료 캡처 시 시각적 무게중심이 잡히도록.
디자인 툴을 잘 몰라서 직접 그리느라 애를 먹었다. 로고와 배너를 팀원들 간 상의 후, 고화질 사진이 없어 직접 그려서 만들었다.

```
docs(readme): add brand banner at top + logo beside On-Care title
```

이어서 README 본문 구조를 한 번 더 정돈 — HealthMate AI 설명의 markdown 포맷팅 미세 수정 등.

## 2. GitHub Pages 배포가 안 됨

**GitHub Pages 자동 배포 실패**가 있어 디버깅 사이클을 4번 돌았다.

### PR #15 — 배포 워크플로우 자체 추가

알고 보니 어제까지 깔린 `deploy-web.yml`은 **Flutter 프로젝트가 루트에 있다고 가정**한 워크플로우였다. 실제 우리 레포는 **`frontend/flutter/`** 하위에 있어서 `flutter pub get`이 실행조차 안 되고 있었다. 워크플로우 자체를 다시 짜서 푸시.

```
Add GitHub Pages deployment workflows
```

### PR #16 — Flutter 프로젝트 path fix

PR #15를 머지하니까 워크플로우는 도는데, 빌드 단계에서 `pubspec.yaml not found`. `working-directory: frontend/flutter`를 빌드 step에 박지 않아서였다.

```yaml
- name: Build web
  working-directory: frontend/flutter
  run: flutter build web --release --base-href "/oncare-flutter/"
```

```
Fix Flutter project path in GitHub Pages deploy workflow
```

### PR #17 — base-href를 `/frontend/`로

빌드는 통과하는데 라이브 URL에서 흰 화면. 콘솔을 까 보니 JS chunk가 모두 404다. 원인은 **레포 이름이 `oncare-flutter`가 아니라 `sudo-capstone-project`고, GitHub Pages가 `/frontend/` 경로에서 호스팅된다**는 사실이었다. PLAN Q11 결정 당시의 가정이 어긋난 것.

base-href를 `/frontend/`로 교체.

```yaml
flutter build web --release --base-href "/frontend/"
```

```
Use /frontend/ base-href to match custom domain
```

### PR #18 — drift WASM 다운로드 자동화

화면은 떴는데, 첫 진입에서 **drift 초기화가 실패**. Stage 9에서 깔아 둔 `LocalApiInterceptor`가 drift에 의존하고, drift는 web에서 `sqlite3.wasm` + `drift_worker.js`를 필요로 한다. 로컬 빌드에서는 `tool/fetch_drift_wasm.sh`로 미리 받아 뒀지만, **CI 배포 시에는 그 스크립트가 안 돌고 있었다**.

배포 워크플로우에 wasm 다운로드 단계 추가.

```yaml
- name: Download drift WASM assets
  working-directory: frontend/flutter
  run: bash tool/fetch_drift_wasm.sh
```

```
Download drift WASM assets in deploy workflow
```

여기까지 와서야 라이브 URL이 정상 동작했다. 어제부터 짧게 보면 5번의 배포 fail이 있었고, 4개 PR로 잡아냈다. 이런 종류의 함정(레포 위치, base-href, WASM 페어링)은 처음 부트스트랩 시점에 미리 시뮬레이션해두는 게 가장 좋지만, 한 번씩 다 겪어야 학습이 되는 것 같다.

## 3. 저녁 — 식단 카메라 + 분석 로딩 플로우 (PR #19)

원본 React 프로토타입에는 **식단 페이지의 카메라 FAB → 사진 → 분석 로딩 → 결과**가 가장 임팩트 있는 시연 흐름이다. Flutter 쪽은 FAB만 있고 그 뒤가 비어 있었다.

오늘은 **UI 셸까지만** 채워 넣기로 했다. 실제 카메라 권한·촬영·이미지 분석은 캡스톤 다음 학기에 들어가지만, 발표 시연에서는 다음 흐름이 보여야 한다.

1. 식단 페이지 FAB 탭 → 풀스크린 카메라 UI(목업 — 셔터 / 갤러리 / 닫기 버튼만).
2. 셔터 탭 → "온이가 분석 중…" 로딩 화면 (lottie 대신 `flutter_animate`로 펄스 애니메이션).
3. 1.5초 후 → 분석 결과 모달 (mock 응답: 비빔밥, 칼로리 524kcal, 나트륨 1,212mg).
4. 저장 탭 → 모달 닫고, 식사 리스트에 추가됨.

```
Add diet add camera + analyzing loading flow (UI shell)
```

이 흐름이 **시연 데모의 클라이맥스**라 시간이 좀 들어갔다. 카메라 UI는 검정 배경 + 가운데 큰 흰 셔터 버튼 + 상단 X / 하단 갤러리 아이콘으로 단순화. mock 응답은 `assets/seed/diet_analyze_response.json`에서 읽어 들이도록 해서 추후 실제 API로 갈아 끼우기 쉽게.

### AI Coach mock에서 수면 제안 제거

같은 PR에 작은 fix 하나 — AI Coach mock 메시지 풀에 "수면 시간이 부족해요" 류 제안이 들어 있었는데, **수면 데이터를 입력받는 채널이 캡스톤 범위에 없어서** 사용자가 어떻게 입력했는지 추적이 안 된다는 모순. mock 풀에서 제거.

```
Remove sleep suggestion from AI Coach mock feedback
```

## 4. FAB 통일 (PR #20)

대시보드 / 식단 / 운동 페이지 모두 우측 하단에 FAB가 있는데, **3개 페이지의 FAB 모양이 미묘하게 다 달랐다.**

- 대시보드: 큰 둥근 사각형 + "+ 빠른 입력" 텍스트 라벨.
- 식단: 원형 + 카메라 아이콘만.
- 운동: 둥근 사각형 + "운동 기록 추가" 텍스트.

원본 React에서는 **모두 원형 아이콘-only FAB**로 통일돼 있었다. Flutter 쪽을 같은 모양으로 정리.

```dart
FloatingActionButton(
  onPressed: ...,
  shape: const CircleBorder(),
  child: const Icon(Icons.add),
)
```

```
Unify home/diet/exercise FABs as circular icon-only buttons
```

추가로 **나트륨 경고 robot 아이콘**도 "온이의 피드백"과 동일한 톤으로 정리(`smart_toy_outlined`).

```
Match sodium-warning robot icon to 온이의 피드백 style
```

## 5. 건강 지표 추이 모달 + 설정 모달 4종 (PR #21)

MyHealth 페이지에서 두 가지가 dead link 상태였다.

1. 체중/혈압/혈당 카드 탭 → 추이 그래프 모달.
2. 하단 설정 리스트 4행(개인 정보 / 건강 데이터 / 알림 설정 / 고객 지원) — 클릭해도 아무 반응 없음.

오늘 둘 다 정리.

### 건강 지표 추이 모달

체중 / 혈압 / 혈당 카드 탭 시 풀스크린 모달이 올라오고, 안에 **30일 추이 라인 차트** + 최근 입력값 7개 리스트. 차트는 Stage 3.5에서 만들어 둔 `AppLineChart`를 재사용.

```
Add 건강 지표 추이 modal for weight/BP/blood-sugar tiles
```

### 설정 모달 4종

각각 다른 콘텐츠를 가진 모달이라 4개 위젯을 따로 짰다.

- **개인 정보** — 이름 / 성별 / 키 / 생년월일 폼.
- **건강 데이터** — 목표 체중·혈압 입력 + 알림 임계값 슬라이더.
- **알림 설정** — 식사 알림, 운동 알림, 정기 점검 알림 토글 3개.
- **고객 지원** — FAQ 링크, 문의 폼 진입.

```
Add 4 settings modals (개인 정보 / 건강 데이터 / 알림 설정 / 고객 지원)
```

내부적으로는 `showModalBottomSheet`로 띄우고, 큰 모달의 경우 `isScrollControlled: true` + `DraggableScrollableSheet`로 풀스크린 가까이 올라오게 했다.

### 설정 행 이름 변경

"개인 정보 / 건강 데이터"는 어색한 표현인 것 같아 **"내 프로필 / 건강 목표"**로 변경.

```
Rename My settings rows: 개인 정보 → 내 프로필, 건강 데이터 → 건강 목표
```

## 6. 추이 데이터 mock 정리 (PR #22)

- `Update health indicator trend mock data` — 30일치 체중/혈압/혈당 시드 추가.
- `Fix health indicator trend data` — 일부 행의 날짜 포맷이 어긋난 것 수정.

여기서 발견한 한 가지 시각적 문제 — **체중 차트가 70~75kg 사이에서만 움직이는데 Y축이 0부터 시작**해서 그래프가 거의 수평선처럼 보였다. 임상적으로 의미 있는 변화(70→72)가 화면상으로는 거의 안 보인다.

## 7. 차트 Y축 클램핑 (PR #23) — 마지막 작업

추이 차트의 Y축 범위를 **지표별 임상 의미 단위**로 박았다.

| 지표 | 범위 | 그리드 |
| --- | --- | --- |
| 체중 | 50 ~ 90 kg (사용자별 ±5 kg로 동적) | 5 kg |
| 혈압(수축기) | 90 ~ 160 mmHg | 10 mmHg |
| 혈압(이완기) | 60 ~ 110 mmHg | 10 mmHg |
| 혈당(공복) | 70 ~ 200 mg/dL | 20 mg/dL |

```dart
AppLineChart(
  points: state.weightPoints,
  minY: state.weightYMin,
  maxY: state.weightYMax,
  gridStep: 5,
)
```

```
Pin trend-chart Y-axis to clinically meaningful range per indicator
```

체중은 추가로 한 번 더 좁혔다. 70~75kg 범위에 있는 사용자의 경우 5kg 단위 그리드는 여전히 너무 듬성듬성하다.

```
Tighten 체중 chart range to 70-75 with 1 kg gridlines
```

`(maxY - minY) <= 6` 같은 조건이면 그리드 step을 1kg로 자동 축소하도록 `AppLineChart`에 가벼운 로직을 추가했다. 차트가 비로소 의미를 갖게 됐다.

## 8. 오늘의 회고와 내일 할 일

- **고민했던 것**:
  - 배포 4번 fail — 처음에 base-href와 레포 위치를 잘못 가정한 게 4단계로 누적된 결과. 부트스트랩 시점에 시뮬레이션해 보는 게 답인데, 그땐 그게 가능한 시각화가 없었다.
  - 카메라 플로우의 깊이 — 실제 권한·촬영까지 갈지 UI shell만 갈지. 캡스톤 범위 안에서 UI shell이 맞다고 판단.
  - Y축 클램핑 자동 단계 축소 — 임계값 6kg가 적당한지. 일단 박아 두고 사용자 시연 반응 보고 조정.
- **결정한 것**:
  - 라이브 시연 URL 동작 확인 — `https://<custom-domain>/frontend/` OK.
  - 식단 카메라 → 분석 → 저장 UI 시연 플로우 완성.
  - MyHealth 설정 4종 + 건강 지표 추이 모달 동작.
  - 차트가 임상 의미를 가진 범위에서만 그려지도록.
- **남은 문제**:
  - **카메라 실제 권한·촬영** — 다음 학기. UI shell은 됐지만 실 사용은 아직.
  - **AI Coach 응답을 진짜 API에 붙이는 작업** — Q7 결정대로 mock으로 시연하지만, 다음 단계에서 백엔드 프록시.
  - **실제 백엔드 연동** — Stage 9에서 깔아 둔 `API_CATALOG.md` 기준으로 백엔드 팀이 API 만들면 `LocalApiInterceptor` 비활성화만으로 전환 가능. 시연 시점에 결정.
  - **자동 e2e 테스트** — Stage 6.4에서 smoke 테스트 1개만 깔려 있고, 카메라 / 모달 / 설정 흐름은 아직 커버 안 됨.

이번 주 회고: 토요일에 일주일치 작업을 한꺼번에 푸시한 게 효율적이었던 동시에 디테일을 놓치기도 쉬웠다. 일/월 이틀에 걸쳐 작은 PR로 나눠 디테일을 잡는 게 발표 안정성 면에서 결과적으로 더 안전한 선택이었다. 다음 사이클에서는 **로컬에 너무 오래 쌓아 두지 말고**, 부분적으로라도 일찍 PR을 열어 두는 게 디버깅 부하를 줄일 듯하다.

---

## 📌 요약

README 배너 →  GitHub Pages 배포 디버깅 4단계(워크플로우 / path / base-href / WASM) → 식단 카메라+분석 UI 셸 + FAB 통일 + 건강 지표 추이 모달 + 설정 모달 4종 → 차트 Y축 임상 범위 클램핑. 라이브 시연 URL이 정상 동작하고, 발표 시나리오의 모든 흐름이 매끄럽게 흘러갈 수 있는 상태가 됐다.
