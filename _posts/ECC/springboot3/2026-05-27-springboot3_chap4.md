---
title: "[코딩 자율학습 스프링 부트 3 자바 백엔드 개발 입문] 4장. 롬복과 리팩터링"
date: 2026-05-27 12:00:00 +0900
categories: [ECC, Team-Project-BE, springboot3]
tags: [dev, study, java, spring]
---

["코딩 자율학습 스프링 부트 3 자바 백엔드 개발 입문" 도서 바로가기](https://www.gilbut.co.kr/book/view?bookcode=BN003778)

# 4장. 롬복과 리팩터링

# 4.1 롬복이란

자바 개발을 하다 보면 Getter, Setter, 생성자 등 필수적이지만 반복적인 코드를 작성하느라 지칠 때가 많다. 롬복(Lombok)은 바로 이러한 불편함을 해결해 주는 **코드를 간소화해 주는 강력한 라이브러리**다.

### 롬복을 사용하면 무엇이 좋을까?

- **지루한 반복 코드 제거:** 어노테이션 몇 개만으로 필수적인 보일러플레이트 코드를 자동으로 생성해 준다. 덕분에 핵심 비즈니스 로직에만 온전히 집중할 수 있다.
- **더 우아한 로깅(Logging) 환경:** 디버깅을 위해 습관적으로 쓰던 `System.out.println()` 문을 더 효율적이고 체계적인 로깅 기능으로 개선할 수 있다.

> **💡 로깅(Logging)이란?**
프로그램이 실행되는 과정이나 시스템의 상태를 기록(로그)으로 남기는 것을 말한다. 서비스 운영 중 발생하는 에러를 추적하거나 성능을 분석할 때 빼놓을 수 없는 핵심 작업이다.
>

### 이번 장의 목표: 롬복으로 리팩터링(Refactoring)하기

이번 장에서는 기존에 작성된 코드를 롬복을 활용해 더 깔끔하고 직관적인 코드로 바꾸는 **리팩터링**을 진행해 보겠다.

- **리팩터링(Refactoring)이란?**
  코드가 고유하게 가진 **기능이나 결과물에는 변함이 없도록 유지**하면서, 내부 구조를 보기 좋게 고치거나 성능을 개선하는 작업을 말한다. 쉽게 말해 "겉은 똑같이 작동하지만, 속은 훨씬 더 건강하고 깨끗하게" 만드는 과정이다.

---

# 4.2 롬복을 활용해 리팩터링하기

이번 실습에서는 롬복(Lombok) 라이브러리를 활용하여 반복되는 코드를 간소화하고, `System.out.println()` 문을 로깅(Logging) 시스템으로 변환하는 리팩터링을 진행하겠다.

## 4.2.1 롬복 설치하기

1. **`build.gradle` 파일 열기:** `firstproject > src > build.gradle` 경로로 이동하여 파일을 연다.
2. **의존성(Dependencies) 추가:** `dependencies {}` 블록 내에 아래의 롬복 라이브러리 의존성 코드를 추가한다.

![사진1](/assets/img/posts/springboot3/springboot3_chap4_1.png)

3. **프로젝트 동기화:** 편집기 우측 상단에 표시되는 Gradle 새로고침 아이콘(코끼리 모양)을 클릭한다. 추가한 의존성을 감지하여 필요한 롬복 관련 라이브러리를 자동으로 다운로드한다.

## 4.2.2 DTO 리팩터링하기

`com.example.firstproject.dto.ArticleForm` 클래스를 열고 생성자와 `toString()` 메서드를 간소화한다.

1. **`ArticleForm()` 생성자 간소화**
  1. 기존에 작성된 `ArticleForm()` 생성자 코드 전체를 삭제한다.
  2.  `ArticleForm` 클래스 선언부 위에 `@AllArgsConstructor` 어노테이션을 추가한다. 해당 어노테이션을 사용하면 클래스 내부의 모든 필드를 매개변수로 하는 생성자가 자동으로 생성된다.

![사진2](/assets/img/posts/springboot3/springboot3_chap4_2.png)

1. **`toString()` 메서드 간소화**
  1. 기존의 `toString()` 메서드 코드 전체를 삭제한다.
  2. 클래스 선언부 위에 `@ToString` 어노테이션을 추가한다. 이를 통해 기존 `toString()` 메서드와 동일한 문자열 출력 효과를 얻을 수 있다.

![사진3](/assets/img/posts/springboot3/springboot3_chap4_3.png)

## 4.2.3 엔티티 리팩터링하기

1. **기존 코드 삭제:** `com.example.firstproject.entity.Article` 클래스를 연다. 엔티티 코드 내부에 명시되어 있는 `Article()` 생성자와 `toString()` 메서드를 완전히 삭제한다.
2. **대체 어노테이션 작성**
  1. `Article` 클래스 선언부 위에 `@AllArgsConstructor` 어노테이션을 추가한다. 이를 통해 클래스 안쪽의 모든 필드를 매개변수로 받는 생성자가 자동으로 생성된다.
  2. 클래스 선언부 위에 `@ToString` 어노테이션을 추가하여 삭제한 `toString()` 메서드의 기능을 대체한다.

![사진4](/assets/img/posts/springboot3/springboot3_chap4_4.png)

## 4.2.4 컨트롤러에 로그 남기기

1. **`@Slf4j` 어노테이션 추가:** `System.out.println()` 문으로 데이터를 출력하던 방식을 로깅 기능으로 대체하기 위해, `ArticleController` 클래스 선언부 위에 `@Slf4j` 어노테이션을 추가한다.
  1. `@Slf4j`는 *Simple Logging Facade for Java*의 약자로, 해당 클래스에서 로깅 기능을 사용할 수 있도록 지원한다.
2. **로깅 코드로 대체**
  1. 기존의 `System.out.println(form.toString());` 코드를 주석 처리하거나 삭제한다.
  2. `log.info(form.toString());` 문을 작성한다. `info()` 메서드의 괄호 안에는 로그로 출력하고자 하는 대상인 `form.toString()`을 전달한다.
3. **나머지 코드 적용** 동일한 방식으로 컨트롤러 내부에 남아 있는 다른 `System.out.println()` 문들을 모두 `log.info()`를 사용하는 로깅 코드로 대체한다.

![사진5](/assets/img/posts/springboot3/springboot3_chap4_5.png)

