---
title: "[Capstone-Start] Flutter 마이그레이션 계획"
date: 2026-05-17 21:30:00 +0900
categories: [Capstone, Start]
tags: [dev, capstone]
---

**📅 일시:** 2026. 05. 17

**💬 내용:** React 프로토타입을 Flutter로 재구성하기 위한 기술 스택 의사결정

---

피그마 1차 프로토타입을 마치고, 다음 단계로 실제 동작하는 프로토타입을 만들기로 했다. 이미 React + TypeScript + Vite 기반으로 정적 프로토타입을 만들어둔 상황이라 이를 참고하기로 했다.

## 1. Flutter로 가기로 한 이유

처음에는 기존 React 코드를 그대로 살리고 React Native로 가는 방식을 고민했다. 그런데 다음 이유로 Flutter 쪽으로 무게가 기울었다.

- **단일 코드베이스에서 웹까지 커버**: React Native는 RN-Web을 끼워야 웹 빌드가 되는데, 우리는 GitHub Pages에 정적 배포해 교수님께 시연할 수 있는 라이브 데모가 필수였다. Flutter Web은 공식 지원이라 별도의 호환 레이어가 필요 없다.
- **UI 일관성**: 식단 카드, 차트, 모달이 많은 헬스케어 앱 특성상 플랫폼별 네이티브 위젯 편차를 줄이는 게 중요했다. Flutter는 자체 렌더링 엔진(Impeller/Skia)을 쓰기 때문에 디자인 시안 그대로 픽셀이 떨어진다.
- **차트 라이브러리 성숙도**: Recharts를 fl_chart로 치환 가능하다는 점을 미리 확인했다. 라인/바/도넛 모두 커스텀 툴팁이 가능하다.
- **개발 도구의 안정성**: Flutter DevTools, Hot Reload, 그리고 `flutter analyze`가 한 번에 묶여 있어 캡스톤 일정 내에서 디버깅 비용이 가장 낮다고 판단했다.

물론 단점도 분명했다. **Dart라는 학습 곡선**, **Material 3와 디자인 시안의 미묘한 어긋남**, **웹 렌더러 선택(canvaskit vs html vs WASM)** 같은 결정이 남는다는 점이 부담이었지만, 이건 개발 과정에서 부딪쳐 가며 해결하기로 했다.

## 2. 화면 트리 매핑

원본 React 앱은 SPA 단일 라우트에서 `activeTab` state로 4개 메인 화면을 스위치하는 구조였다. 이걸 Flutter의 라우팅 모델에 맞게 재해석할 필요가 있었다.

```
원본 화면 트리
├─ BottomNav (4 tabs)
│   ├─ Dashboard         ← 홈
│   ├─ DietRecord        ← 식단 기록
│   ├─ Exercise          ← 운동
│   └─ MyHealth          ← 내 건강
└─ 보조 화면/컴포넌트
    ├─ AICoach           ← AI 코칭 (모달 또는 별도 진입)
    ├─ NotificationPanel ← 알림 패널
    ├─ Place             ← 장소 추천
    └─ Header            ← 상단 헤더
```

`activeTab` state를 그대로 옮기면 URL 공유가 안 되고 뒤로가기도 깨지기 때문에, **go_router의 StatefulShellRoute**로 각 탭에 자체 네비게이션 스택을 부여하는 쪽으로 가닥을 잡았다. AI 코치와 알림 패널, 일정 추가 같은 보조 화면은 라우트가 아니라 **오버레이/모달 시트**로 다루기로 했다.

## 3. 기술 스택 1차 제안

PLAN 문서에 채택 안과 대안을 같이 적어 비교가 쉽도록 정리했다.

| 영역 | 채택안 | 비교 / 이유 |
| --- | --- | --- |
| 상태관리 | **Riverpod v2 + riverpod_generator** | 컴파일타임 안전성, DI 통합, 테스트 용이. Bloc은 보일러플레이트, Provider는 기능 부족. |
| 라우팅 | **go_router + StatefulShellRoute** | Web URL/딥링크/BottomNav 셸 라우트 1급 지원. |
| 모델/직렬화 | **freezed + json_serializable** | 불변 데이터 클래스, sealed union 표현이 깔끔. |
| 네트워크 | **dio**(+ 추후 retrofit) | 인터셉터·캐싱, mock-first 진행에 유리. |
| 로컬 저장소 | **drift**(시계열) + **flutter_secure_storage**(토큰) + **shared_preferences**(설정) | 헬스 시계열은 SQL이 적합. |
| 차트 | **fl_chart** | Recharts 대체, 활성 메인테인, 커스텀 범례·툴팁 OK. |
| 다국어 | **flutter_localizations + intl** | MVP는 intl, 본격화되면 slang 검토. |
| 테스트 | flutter_test + **mocktail** + **golden_toolkit** | mockito 대비 codegen 불필요. |

## 4. 사용자 확인이 필요한 결정 사항 (Q1 ~ Q12)

| # | 결정 항목 | 옵션 | 영향 |
| --- | --- | --- | --- |
| Q1 | 백엔드 전략 | Mock / Firebase / 자체 REST / 추후 | 데이터 모델·인증·푸시 설계 |
| Q2 | 상태관리 | Riverpod / Bloc / Provider | 학습 곡선, 보일러플레이트 |
| Q3 | 로컬 DB | drift / isar / prefs만 | 모델 복잡도, 마이그레이션 |
| Q4 | 인증 | 없음 / 이메일 / 소셜(Apple·Google·Kakao·Naver) | UI/UX, 정책 |
| Q5 | 다국어 | 한국어만 / ko + en | 텍스트 외부화 부하 |
| Q6 | 디자인 충실도 | 1:1 픽셀 퍼펙트 / Material 3 + Oncare 토큰 | 개발 속도 / 디자인 정합성 |
| Q7 | AI Coach 백엔드 | Mock / 외부 API 직결 / 자체 프록시 | 키 관리·비용 |
| Q8 | 지도 | 카드 리스트만 / Google Maps / Naver·Kakao | 라이선스, 웹 호환 |
| Q9 | 푸시 | 인앱 패널만 / FCM 통합 | 백엔드 결정과 연동 |
| Q10 | Web URL 전략 | Hash / Path + 404 fallback | 배포 안정성 vs URL 미관 |
| Q11 | 레포 위치 | 어느 GitHub 계정/조직 | base-href, Pages 도메인 |
| Q12 | CI 자동화 범위 | 웹만 / 전체 | 시크릿 셋업 부하 |

특히 **Q1(백엔드)**, **Q8(지도)**, **Q10(URL 전략)** 세 가지가 캡스톤 일정에 가장 큰 영향을 준다. Q10 같은 경우 GitHub Pages가 SPA fallback을 기본 지원하지 않아서, Hash URL로 가지 않으면 새로고침 시 404가 떨어지는 함정이 있다는 걸 미리 확인해뒀다.

## 5. 디렉토리 구조 — 수직 슬라이스

원본 React 코드는 `src/app/components/`에 모든 컴포넌트를 한 폴더에 넣어둔 평면 구조였다. 화면이 7~8개 정도라 지금은 괜찮지만, 모달이나 도메인 로직이 더 붙으면 금방 더러워질 거라 판단했다. 그래서 Flutter 쪽은 **수직 슬라이스(feature-first)** 구조로 가기로 했다.

```
lib/
├─ main.dart            # 엔트리 (환경별 분기 가능)
├─ app/                 # 앱 셸 (App 위젯, 라우터, 테마)
├─ core/                # 설정·네트워크·저장소·에러·유틸
├─ shared/              # 도메인 무관 공용 위젯/모델/서비스
├─ design_system/       # 토큰·테마·atom 위젯 (자체 UI 키트)
├─ features/            # 피처 모듈 (수직 슬라이스)
│  ├─ dashboard/
│  ├─ diet/
│  ├─ exercise/
│  ├─ my_health/
│  ├─ ai_coach/
│  ├─ notification/
│  └─ place/
├─ l10n/                # ARB 파일
└─ gen/                 # 자동 생성(.g.dart, .freezed.dart, l10n)
```

각 `features/<name>/` 하위는 **Clean Architecture 슬림 버전** — `data / domain / presentation` 3개 계층만 사용하기로 했다. UseCase 계층은 도메인 로직이 단순한 동안에는 생략하고, controller(Riverpod notifier)가 repository를 직접 호출하는 방식으로 시작한다. 나중에 로직이 복잡해지면 그때 UseCase를 도입할 여지를 남겨둔 셈이다.

## 6. 오늘의 회고와 내일 할 일

- **고민했던 것**: React 프로토타입을 그대로 유지하면서 모바일만 따로 만들지(2-track), 아니면 Flutter로 통일할지가 가장 큰 고민이었다. 결국 캡스톤 일정 안에서 두 코드베이스를 동시에 유지하는 건 무리라고 판단했다.
- **결정한 것**: Flutter 단일 코드베이스로 가되, React 코드는 **참조용 원본**으로 보관. 화면별 디자인 토큰과 레이아웃을 Flutter로 옮길 때 비교 기준점으로 활용한다.
- **남은 문제**:
  - **Q1~Q12 사용자 확정** — 특히 인증 4종 소셜(Apple/Google/Kakao/Naver)을 모두 지원할지가 Stage 4 진입 전에 결정돼야 한다.
  - **디자인 토큰 추출** — 피그마에서 color/spacing/radius/typo를 어떻게 뽑아 코드 토큰으로 옮길지 표 작업이 남았다.
  - **base-href와 GitHub 레포 위치** — Pages 배포 도메인이 정해져야 `flutter build web --base-href` 옵션을 박을 수 있다.

내일은 Stage 0 마무리(디자인 토큰 표, 백엔드 전략 결정 기록)에 시간을 더 쓰고, **Stage 1(부트스트랩)** 진입을 목표로 한다. `flutter create .` 한 줄이지만, org 식별자와 platforms 플래그를 잘못 넣으면 되돌리기 귀찮으니 한 번에 가야 한다.

---

## 📌 요약

오늘은 코드를 짜는 대신 **Flutter 마이그레이션 로드맵**을 세웠다. React Native가 아닌 Flutter를 택한 이유, 화면 트리 매핑, 12가지 결정 사항(Q1~Q12), 수직 슬라이스 디렉토리 구조를 PLAN 문서로 정리했다. 캡스톤 일정 안에서 가장 큰 리스크는 결정이 늦어지는 것이라, 다음 작업 시작 전에 Q1~Q12 답을 받아오는 게 가장 시급하다.
