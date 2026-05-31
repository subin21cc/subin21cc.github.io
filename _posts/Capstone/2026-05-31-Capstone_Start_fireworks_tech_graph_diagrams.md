---
title: "[Capstone-Start] fireworks-tech-graph 스킬로 README 다이어그램 SVG화"
date: 2026-05-31 10:00:00 +0900
categories: [Capstone, Start]
tags: [dev, capstone]
---

**📅 일시:** 2026. 05. 31

**💬 내용:** Claude Code 스킬 `fireworks-tech-graph`로 자연어 → SVG 다이어그램을 만들고, 캡스톤 README의 ASCII 도표 3장을 일괄 교체한 작업 정리

---

이번 학기 Elevator Speech가 끝나고, 발표 자료와 README를 다시 한 번 정리하다가 가장 거슬렸던 부분이 **ASCII art 다이어그램**이었습니다. 시스템 아키텍처, Vision AI 파이프라인, RAG 파이프라인 — 세 개의 핵심 도표가 모두 ```...``` 블록 안의 텍스트 박스로 그려져 있었고, 발표 슬라이드에 캡처해 넣으면 가독성이 급격히 떨어졌습니다.

5/24에 PR #12에서 이 세 다이어그램을 모두 SVG로 교체했는데(commit `5fe2624`), 그때 사용한 도구가 **`yizhiyanhua-ai/fireworks-tech-graph`** Claude Code 스킬입니다. "자연어 설명을 받아 SVG + PNG로 출력하는 기술 다이어그램 생성기"라고 소개되어 있었는데, 직접 써 보니 캡스톤 발표 자료 수준에서 활용도가 꽤 높았습니다.

오늘은 이 스킬을 어떻게 알게 됐고, 실제 README에 어떻게 반영했는지, 그리고 자연어 → 다이어그램 도구의 **한계와 보완 방법**을 정리합니다. 비슷한 도구를 찾으시는 분께 검색에 잡히기 좋은 형태로 남깁니다.

## 1. fireworks-tech-graph는 무엇을 하는 스킬인가요

GitHub: [yizhiyanhua-ai/fireworks-tech-graph](https://github.com/yizhiyanhua-ai/fireworks-tech-graph). 한 줄 설명을 그대로 옮기면 다음과 같습니다.

> Generate production-quality SVG+PNG technical diagrams from natural language. 7 styles, UML support, and AI/Agent workflow patterns.

핵심 기능을 정리하면:

| 항목 | 내용 |
| --- | --- |
| 입력 | 자연어 프롬프트 (한국어 / 영어 / 중국어) |
| 출력 | SVG 파일 + 1920px PNG 이미지 |
| 스타일 | **7종** (아래 표 참고) |
| 도표 종류 | UML 14종 (Class / Component / Deployment / Sequence / State Machine ...), AI/Agent 패턴 (RAG / Agentic Search / Mem0 / Multi-Agent / Tool Call), Architecture, data flow, flowchart, mind map |
| 검증 | 출력 SVG에 viewBox / 폰트 fallback / aria-label이 박혀 있어 GitHub 인라인 렌더링과 스크린리더 모두 OK |

### 7가지 스타일

| # | 스타일 이름 | 분위기 |
| --- | --- | --- |
| 1 | **Flat Icon** (default) | 흰 배경 + 부드러운 파스텔. 학술 자료 톤. |
| 2 | Dark Terminal | 검정 배경 + 네온 강조. 해커톤·CLI 발표용. |
| 3 | Blueprint | 진한 청색 + 격자. 시스템 설계 문서 톤. |
| 4 | Notion Clean | 미니멀 흰 배경. Notion 페이지 임베드용. |
| 5 | Glassmorphism | 어두운 그라데이션 + 반투명 카드. |
| 6 | Claude Official | 따뜻한 크림 배경. Claude 공식 톤. |
| 7 | OpenAI Official | 순백 + OpenAI 팔레트. |

캡스톤은 학교 발표 + GitHub README 임베드 + Notion 발표 자료에서 동일하게 보여야 했기 때문에, 가장 중립적인 **Style 1 · Flat Icon**으로 통일했습니다.

## 2. 설치와 사전 준비

### 스킬 설치

```bash
npx skills add yizhiyanhua-ai/fireworks-tech-graph
```

`npx skills add`는 Claude Code의 스킬 등록 명령입니다. 한 번 등록해 두면 그 다음부터는 채팅창에서 자연어로 호출이 됩니다.

### PNG 렌더러 설치 (필요 시)

스킬이 SVG에서 PNG를 뽑을 때 외부 렌더러가 필요합니다. 캡스톤 환경(macOS)에서는 `cairosvg`를 깔았습니다.

```bash
# 권장 (Python)
pip install cairosvg

# 대안 (macOS 네이티브)
brew install librsvg
```

GitHub README는 **SVG를 인라인 렌더링**하므로, PNG가 꼭 필요한 건 발표 슬라이드용입니다. 캡스톤에서는 둘 다 같이 받아 뒀습니다.

## 3. 실제 사용 — 캡스톤 README 도표 3장 일괄 생성

스킬 사용법은 채팅에 자연어로 "이런 도표 그려줘"라고 부탁하는 것입니다. 다이어그램별로 어떤 프롬프트를 썼는지 옮겨 둡니다.

### (1) Vision AI 파이프라인

자연어 설명만으로 어떻게 도표가 만들어지는지가 가장 잘 보이는 예입니다.

**프롬프트 (요약):**

> 세로 5단으로 흐르는 Vision AI 파이프라인을 그려주세요. 입력은 "사용자 식단 사진 업로드"이고, 1단계 YOLOv8 음식 필터, 2단계 Gemini Vision API 영양 분석, 3단계 공공데이터 식품영양성분 DB 매핑, 4단계 MySQL RDS 저장으로 흘러갑니다. YOLOv8 단계에서 음식이 아닌 경우 오른쪽으로 빠지는 "비음식 이미지 반려" 분기가 있어야 합니다. Flat Icon 스타일로, 한국어 라벨로 부탁해요.

**출력:** `docs/diagrams/vision-pipeline.svg` — viewBox `0 0 960 880`, 세로 5단 + 비음식 분기 박스. 박스 컬러는 단계별로 라이트 블루(입력) / 화이트(처리) / 그린(외부 데이터) / 레드(반려)로 자동 배색됐습니다.

### (2) RAG 파이프라인

가로 4단 흐름 + 컨텍스트 검색 분기를 요청했습니다.

**프롬프트 (요약):**

> 가로 4단으로 진행되는 RAG 코칭 파이프라인을 그려주세요. 단계는 (a) 사용자 질문 "오늘 식단 괜찮아?", (b) text-embedding-3-small 임베딩, (c) Pinecone 시맨틱 검색 (유사도 top-k), (d) GPT-4o가 컨텍스트 주입 후 답변 생성입니다. 하단에 별도로 "사용자 건강 이력 Vector DB"가 Pinecone으로 화살표 연결돼야 합니다. Flat Icon 스타일.

**출력:** `docs/diagrams/rag-pipeline.svg` — viewBox `0 0 1100 620`, 가로 4단 + Vector DB 보조 노드. 화살표 색이 단계마다 자동으로 다르게 칠해진 점이 인상적이었습니다(파랑·보라·초록 순으로 흐름이 시각화됨).

### (3) System Architecture

가장 정보가 많았던 도표라 프롬프트가 가장 길어졌습니다.

**프롬프트 (요약):**

> 4-tier 레이어드 시스템 아키텍처를 그려주세요. 최상단은 Flutter Mobile App (iOS/Android), 그 아래는 FastAPI Backend (Docker/AWS EC2). 다음 레이어에 세 박스가 가로로 배치 — MySQL RDS (사용자·식단·운동·포인트), Vision AI Pipeline (YOLOv8 → Gemini Vision), RAG Pipeline (LangChain + Pinecone + GPT-4o). 최하단에 External APIs 칩 5개 — 카카오맵, 공공데이터 식품영양성분, Google AI, OpenAI, Firebase Cloud Messaging. Flat Icon 스타일.

**출력:** `docs/diagrams/system-architecture.svg` — 4-tier 레이어 + External API 5개 칩. ASCII 시절에는 도표 폭이 80자를 넘어 모바일 GitHub 뷰에서 가로 스크롤이 생겼는데, SVG 임베드 후 모바일에서도 깨지지 않게 됐습니다.

## 4. 출력 SVG 살펴보기 — 무엇이 들어 있나요

스킬이 만들어 준 SVG 파일 하나를 까 보면, 검증 친화적인 디테일이 잘 박혀 있습니다.

```svg
<svg xmlns="http://www.w3.org/2000/svg"
     viewBox="0 0 1100 620" width="1100" height="620"
     role="img" aria-label="RAG 기반 AI 코칭 파이프라인">
  <style>
    text { font-family: 'Helvetica Neue', Helvetica, Arial,
           'Apple SD Gothic Neo', 'Noto Sans KR', 'Malgun Gothic',
           'PingFang SC', 'Microsoft YaHei', sans-serif; }
  </style>
  <defs>
    <marker id="arr-blue"  markerWidth="10" markerHeight="7"
            refX="9" refY="3.5" orient="auto">
      <polygon points="0 0, 10 3.5, 0 7" fill="#2563eb"/>
    </marker>
    <!-- ... -->
    <filter id="soft" x="-10%" y="-10%" width="120%" height="120%">
      <feDropShadow dx="0" dy="1" stdDeviation="1.5" flood-opacity="0.08"/>
    </filter>
  </defs>
  <!-- 박스 / 화살표 / 텍스트 ... -->
</svg>
```

다음 디테일이 일관되게 들어 있었습니다.

- `viewBox`와 `width`/`height` 동시 지정 — 반응형 임베드 대비.
- `role="img"` + `aria-label` — 스크린리더 접근성.
- **한국어 폰트 fallback 체인** — Apple SD Gothic Neo · Noto Sans KR · Malgun Gothic · PingFang SC · Microsoft YaHei까지. 어느 OS·뷰어에서 열어도 한국어가 깨지지 않습니다.
- `<marker>` 정의로 화살표 머리 종류별 컬러 분리 — 분기별 시각 구분.
- `<filter>`로 부드러운 그림자 — 평면적인 박스가 살짝 떠 보이는 효과.

처음 자연어 도구를 쓸 때 가장 걱정한 게 **출력의 일관성**이었는데, 세 다이어그램이 모두 같은 마커·폰트·그림자 정의를 공유하도록 만들어 줘서 README에 같이 박았을 때도 톤이 안 깨졌습니다.

## 5. 검증 자동화 — `scripts/validate-svg.sh`

SVG가 GitHub 인라인 렌더링에 잘 호환되는지 확인하기 위해 간단한 검증 스크립트를 따로 작성해 박았습니다. 커밋 메시지에 "validated via scripts/validate-svg.sh"라고 적은 이유입니다.

스크립트가 검사하는 항목은 다음 4가지입니다.

1. `viewBox` 속성 누락 여부 — 누락 시 모바일에서 깨질 수 있음.
2. `width`/`height` 속성 존재 — GitHub README 임베드 시 폭 지정용.
3. **한국어 폰트 fallback 체인**이 포함돼 있는지 — Noto Sans KR 또는 Apple SD Gothic Neo 키워드 검색.
4. `<text>` 요소 안에 깨진 인코딩(`â??` 같은 mojibake)이 없는지.

이 스크립트를 PR 머지 전에 돌려서, 외부 도구 산출물이 우리 컨벤션과 어긋나면 즉시 알 수 있도록 두었습니다. 자연어 도구는 출력이 100% 결정적이지 않기 때문에, **자동 검증 단계를 함께 두는 게 안전**합니다.

## 6. README ASCII vs SVG — Before / After

같은 정보가 표현 방식에 따라 얼마나 달라지는지가 가장 큰 학습 포인트였습니다.

### Before — ASCII art (5/24 이전)

```
┌──────────────────────────────────────────────────────────────────┐
│               Flutter Mobile App (iOS / Android)                 │
│        카메라 · 식단 · RAG 챗봇 · 운동 · 헬스장 · 포인트 · 캘린더          │
└──────────────────────────┬───────────────────────────────────────┘
                           │  HTTPS REST API
┌──────────────────────────▼───────────────────────────────────────┐
│              FastAPI Backend  (Docker / AWS EC2)                 │
│   ...                                                            │
```

- 폭이 80자 가까이 돼서 모바일 GitHub 앱에서 가로 스크롤 발생.
- 발표 슬라이드에 캡처해 넣으면 코드 글꼴이라 가독성 ↓.
- 한국어 라벨과 박스 정렬을 손으로 맞추느라 수정이 너무 어려움.

### After — SVG 임베드 (5/24 이후)

```markdown
<p align="center">
  <img src="docs/diagrams/system-architecture.svg"
       alt="On-Care 시스템 아키텍처" width="960"/>
</p>
```

- GitHub가 SVG를 인라인 렌더링해서 별도 PNG 없이 모든 뷰어에서 정상 표시.
- 다이어그램 1장당 README 본문 라인 수가 30+줄 → 3줄로 압축.
- 발표 슬라이드 / Notion 페이지 / 캡스톤 보고서에 동일 SVG를 그대로 재사용 가능.

5/24 PR 한 번에 README가 **290줄 → 210줄로 30% 줄어들었고**, 그 줄어든 분량은 모두 ASCII 박스였습니다.

## 7. 직접 써 보고 알게 된 한계

스킬이 완벽하진 않았습니다. 다음 세 가지 한계를 부딪쳤습니다.

### (1) 자연어 → 도표 매핑이 결정적이지 않음

같은 프롬프트를 두 번 부르면 박스 위치나 색이 미묘하게 달라집니다. 첫 번째 시도가 마음에 들면 그대로 커밋하는 게 답이고, 같은 컨셉을 여러 번 회생산하는 용도로는 부적합합니다.

### (2) 세밀한 위치 조정은 결국 손으로

박스 한 개의 폭이나 화살표 곡률을 1~2px 단위로 조정하려면 결국 SVG 파일을 직접 열어 수정해야 합니다. 스킬은 **첫 80% 작성**에 강하고, 마지막 20% 다듬기는 사람의 몫이었습니다. 다행히 출력 SVG가 사람이 읽기 좋게 정리돼 있어서 수정 자체는 어렵지 않았습니다.

### (3) 한국어 라벨 길이가 길면 박스 밖으로 흘러나옴

긴 한국어 라벨(예: "위치 기반 헬스장 검색·예약·트레이너 인앱 채팅")을 박스 안에 강제로 넣으면 자동 줄바꿈이 안 됩니다. 줄을 직접 끊거나(`<tspan>` 추가) 라벨 길이를 짧게 줄이는 식으로 우회했습니다.

이 세 가지는 **자연어 도구의 본질적인 한계**라 봐야 할 듯합니다. 정확도가 100%인 도구를 기대하기보다는, "초안 빠른 생성 → 사람이 마무리" 워크플로우의 첫 단계로 두는 게 현실적입니다.

## 8. 워크플로우 정리 — 다음에 또 쓴다면

이번 경험을 토대로, **다음에 비슷한 도표를 그릴 때 따를 워크플로우**를 정리했습니다.

1. **목표 다이어그램을 한 문단으로 묘사** — 단계 수, 분기, 라벨, 외부 노드까지.
2. **스타일 번호 명시** — `style 1` 같이 명시적으로 적어 줘야 톤이 일관됨.
3. **출력 SVG를 `docs/diagrams/`에 저장** — 다른 자료에서 재사용하기 위해.
4. **`scripts/validate-svg.sh`로 검증** — viewBox / 폰트 / 인코딩 게이트 통과 확인.
5. **README에는 `<img src=... width=...>` 임베드** — `<p align="center">`로 감싸 가운데 정렬.
6. **PR 단위로 분리** — 다이어그램 변경은 commit diff가 거의 안 보이므로, 다이어그램 전용 PR로 묶어 리뷰어가 SVG만 보면 되도록.

5/24 작업이 위 워크플로우의 정확한 적용 사례입니다.

## 9. 회고

자연어 다이어그램 도구를 처음 써 봤고, 결과적으로 **README 가독성·발표 자료 재사용성·작성 시간** 세 가지 측면에서 모두 이득이었습니다. 정량적으로는 다음과 같습니다.

- 작성 시간 — 다이어그램 3장 + 검증 스크립트 작성까지 약 90분. ASCII art로 처음부터 그렸다면 다이어그램 1장당 30~40분 + 정렬 디버깅 시간이 들었을 겁니다.
- README 본문 길이 — 290줄 → 210줄로 30% 축소.
- 다이어그램 재사용성 — 발표 슬라이드·보고서·Notion 페이지에 동일 SVG/PNG를 그대로 임베드.

다만 위에서 정리한 세 가지 한계 때문에 **사람이 마지막 20%를 다듬는 단계**는 계속 필요합니다. 자연어 도구가 무서운 점은 "결과가 그럴듯해 보여서 검수 없이 그대로 커밋하기 쉽다"는 점인데, 검증 스크립트 한 줄로 이걸 막을 수 있다는 게 가장 큰 학습이었습니다. 외부 도구를 도입할 때 **자동 검증 게이트를 함께 도입**하는 패턴은 다음 캡스톤·인턴십에서도 유지하려 합니다.

---

## 📌 요약

`yizhiyanhua-ai/fireworks-tech-graph`는 자연어 프롬프트를 받아 production-quality SVG + PNG 기술 다이어그램을 출력하는 Claude Code 스킬입니다. 7가지 시각 스타일(Flat Icon / Dark Terminal / Blueprint / Notion Clean / Glassmorphism / Claude Official / OpenAI Official), UML 14종, AI/Agent 패턴(RAG / Multi-Agent / Tool Call 등)을 지원합니다. 캡스톤에서는 README의 ASCII 도표 3장(Vision AI Pipeline / RAG Pipeline / System Architecture)을 Flat Icon 스타일 SVG로 일괄 교체하면서, README 길이가 290줄 → 210줄로 줄고 모바일 GitHub 뷰에서도 깨지지 않게 됐습니다. 다만 자연어 → 다이어그램 매핑이 결정적이지 않고 세밀 조정은 손으로 해야 한다는 한계가 있어, `scripts/validate-svg.sh` 같은 **자동 검증 게이트와 함께** 도입하는 워크플로우를 권장합니다.
