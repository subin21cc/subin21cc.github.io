---
title: "[Capstone-Start] Prototype - 1st Revision"
date: 2026-05-14 12:00:00 +0900
categories: [Capstone, Start]
tags: [dev, capstone]
---

**📅 일시:** 2026. 05. 14

**💬 내용:** 피그마 프로토타입 수정 - 1st Revision

---

서비스의 초기 아이디어를 시각화하기 위해 피그마를 활용해 프로토타입을 제작했습니다.
Figma Make를 활용했는데, 꽤 유용했습니다. 그러나 생각보다 수정이 필요한 부분도 많고, 추가할 기능이 많아 시간이 좀 걸렸습니다.

사용자의 건강 상태를 한눈에 파악하고 직관적으로 관리할 수 있는 사용자 경험(UX)을 구축하는 데 중점을 두었습니다.

그 첫 번째 결과물인 v1.0 프로토타입을 소개하려 합니다. 주요 화면 사진과 함께 On-Care가 제공하는 핵심 기능들을 상세히 설명하겠습니다.

우선, 탭은 '홈', '식단', '운동', 'My' 총 4가지로 구성됩니다. 챗봇을 탭으로 넣을까라는 고민도 했지만, 각 화면에서 들어갈 수 있도록 하는 것이 더 좋을 것 같아서 챗봇은 탭이 아닌 각 화면에서 접근할 수 있도록 했습니다.


---


# 홈

![홈 탭](/assets/img/posts/capstone/capstone-start-figma-home.png)

- 처음 서비스를 시작하면 뜨는 화면이다. 
- 현재 스트릭 대시보드에는 현재 스트릭 날짜와 오늘의 건강을 기록할 수 있는 버튼이 3개 있다.
- 오늘의 건강 요약 대시보드에는 오늘 내가 먹은 음식의 칼로리, 나트륨, 혈당과 식단&운동을 얼마나 했는지가 보여진다.
- 오늘의 일정 대시보드에는 오늘의 일정이 뜨고, 전체보기를 클릭하면 아래 사진과 같이 달력으로 일정을 한 눈에 보고 관리할 수 있다.
- 가장 아래에는 이번 주 건강 점수 대시보드가 있다. 초록/노랑/빨강 3가지 색상으로 직관적으로 건강 점수를 표현할 예정이다.

![홈 탭-전체일정](/assets/img/posts/capstone/capstone-start-figma-schedule_full.png)


---


# 식단

## 식단 - 메인

![식단 탭](/assets/img/posts/capstone/capstone-start-figma-diet.png)

- 상단에는 오늘 날짜와 옆으로 스크롤 할 수 있는 주간 달력으로 다른 날짜를 선택할 수 있다.
- 그 아래에는 오늘의 섭취량이 간단히 뜬다.
- 온이의 피드백은 1:1 맞춤 AI 식단 코치다.
- 오늘 먹은 음식이 정리되어 나타난다.

## 식단 - 온이의 피드백

![식단-피드백](/assets/img/posts/capstone/capstone-start-figma-diet_feedback.png)

- 오늘의 건강분석과 추천을 해준다.
- 빠른 질문과 직접 질문을 통해 맞춤 답변을 얻을 수 있다.

## 식단 - 추가

![식단-추가](/assets/img/posts/capstone/capstone-start-figma-diet_add.png)

- 사진을 찍으면 자동으로 식단이 기록된다.
- 혹은 수동으로 추가할 수도 있다.


---


# 운동

## 운동 - 메인

![운동-운동기록](/assets/img/posts/capstone/capstone-start-figma-exercise.png)

- 이번 주 운동 현황을 간단한 카드와 그래프로 한눈에 볼 수 있다.
- 하단에는 각 운동별로 자세히 알 수 있는 내역이 보인다.
- 온이의 피드백은 1:1 맞춤 운동 AI 코치다.

## 운동 - 온이의 피드백

![운동-피드백](/assets/img/posts/capstone/capstone-start-figma-exercise_feedback.png)

- 이번 주 운동을 분석과 추천을 해준다.
- 빠른 질문과 직접 질문을 통해 맞춤 답변을 얻을 수 있다.

## 운동 - 추가

![운동-추가](/assets/img/posts/capstone/capstone-start-figma-exercise_add.png)

- 운동 유형, 운동 시간, 운동 내용을 입력하여 운동 기록을 저장할 수 있다.

## 운동 - 헬스장

![운동-헬스장](/assets/img/posts/capstone/capstone-start-figma-exercise_gym.png)

- 상단의 헬스장 찾기를 클릭하면 헬스장을 검색할 수 있다.
- 아래에는 내 헬스장 목록이 보인다.
- 헬스장 정보를 볼 수 있고 1:1 상담을 할 수 있다.
- 1:1 상담은 건강 데이터 요약본을 자동 전송하여 개인 맞춤 상담이 가능하다.

## 운동 - 헬스장 찾기

![운동-헬스장찾기](/assets/img/posts/capstone/capstone-start-figma-exercise_gym_find.png)

- 주변 헬스장을 찾을 수 있다.
- 각 헬스장별로 거리, 평점, 강점을 한눈에 볼 수 있다.
- 상세보기를 클릭하면 더 자세한 내용을 볼 수 있다.
- 등록하기를 클릭하면 편하게 헬스장을 등록할 수 있다.


---


# My

## My - 메인

![My](/assets/img/posts/capstone/capstone-start-figma-my.png)

- 건강 지표(체중/혈압/혈당 변화)가 숫자와 그래프로 한눈에 보인다.
- 그 아래에는 누적 활동 포인트가 있고 포인트 랭킹을 볼 수도 있다.
- 추가로 개인정보 관리/건강 데이터 설정/알림 설정/고객 지원 기능도 제공한다.


---

현재 프로토타입은 프로젝트의 방향성을 설정한 초안(Draft) 단계이며, 아직 모든 기능이 확정된 것은 아닙니다. 
앞으로 본격적인 개발 과정을 거치며 사용자 피드백을 반영해 더욱 정교하고 완성도 높은 기능을 구현해 나갈 예정입니다.
