---
title: "[스프링 입문 - 코드로 배우는 스프링 부트, 웹 MVC, DB 접근 기술] 섹션 2. 프로젝트 환경설정"
date: 2026-05-02 20:00:00 +0900
categories: [ECC, Team-Project, Spring_intro]
tags: [dev, study, java, spring]
---

["스프링 입문 - 코드로 배우는 스프링 부트, 웹 MVC, DB 접근 기술" 강의 바로가기](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EC%9E%85%EB%AC%B8-%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8)

# 섹션 2. 프로젝트 환경설정

## 3. 프로젝트 생성

- 사전 준비물
  - Java 11 설치
  - IDE: IntelliJ 또는 Eclipse 설치 → IntelliJ 권장

### **스프링** **부트** **스타터** **사이트로** **이동해서** **스프링** **프로젝트** **생성**

https://start.spring.io
![사진1](/assets/img/posts/spring_intro/spring_intro_sec2_1.png)

- Project: Gradel - Groovy
- Language: Java
- Spring Boot: 3.5.14
- Project Metadata
  - Group: hello
  - Artifact: hello-spring
- Dependencies: Spring Web, Thymeleaf

![사진2](/assets/img/posts/spring_intro/spring_intro_sec2_2.png)
![사진3](/assets/img/posts/spring_intro/spring_intro_sec2_3.png)

- src에서 보통 main이랑 test 폴더 나눔
- resource폴더: 실제 자바 코드를 제외한 xml, properties, html 등 설정 파일 포함
- build.gradle

![사진4](/assets/img/posts/spring_intro/spring_intro_sec2_4.png)
- mavenCentral(): 라이브러리 다운받는 사이트 설정
- .gitignore: 소스코드 관리 → git에는 필요한 소스코드만 올리고 나머지 빌드된 결과물 등은 안 올리도록 한다.

### Gradle 빌드 실패 할 때?
![사진5](/assets/img/posts/spring_intro/spring_intro_sec2_5.png)
- “File” - “Project structure…” - “Project” -”SDK”를 JAVA17로 설정한다. (17인 이유는 위의 스프링부트 스타터에서 17로 설정했기 때문이다. 설정한 버전에 맞추면 된다.)
- “Gradle Settings”에서도 Gradle JDK를 17로 맞춰준다.
- “Gradle “탭을 클릭하고 “Reload All Gradle Projects (새로고침 아이콘)”를 누르면 설정 완료된다.

### 톰캣 서버 구동 확인 및 localhost:8080 접속
![사진6](/assets/img/posts/spring_intro/spring_intro_sec2_6.png)
- 서버 로그에 Tomcat initialized with port **8080**이 뜨고 실행도 성공했지만, 브라우저에서 localhost:8080 접속 시 “**White Label Error Page**”나 "**연결할 수 없음**"이 뜬다.
- 원인: 톰캣 서버 엔진은 정상적으로 구동되었으나, 사용자의 요청(URL)을 받아 처리할 **컨트롤러(Controller)**나 보여줄 **파일(Index.html 등)**이 프로젝트에 구현되어 있지 않기 때문이다.

⇒ 여기까지 하면, 프로젝트 환경설정에 성공한 것이다!

### 번외) 더 빠른 개발을 위한 빌드 및 실행 환경 최적화
![사진7](/assets/img/posts/spring_intro/spring_intro_sec2_7.png)
- **설정 경로:** Settings → Build, Execution, Deployment → Build Tools → Gradle
- **Build and run using**과 **Run tests using** 항목을 Gradle(Default)에서 IntelliJ IDEA로 변경
- 설정 바꾸는 이유: 기본적으로 IntelliJ는 빌드와 실행을 Gradle을 거쳐서 수행한다. 이 과정을 IntelliJ가 직접 처리하도록 변경하면 실행 속도를 훨씬 높일 수 있다.

---

## 4. 라이브러리 살펴보기
![사진8](/assets/img/posts/spring_intro/spring_intro_sec2_8.png)
⇒ 스프링 부트 프로젝트에서 **External Libraries** 목록이 방대한 이유는 Gradle이 라이브러리 간의 의존 관계를 파악해 필요한 파일을 자동으로 다운로드하기 때문이다.

### 1. 스프링 부트 핵심 라이브러리

- **spring-boot-starter-web**: 웹 애플리케이션 구축을 위한 핵심 스타터다. 내장 WAS인 Tomcat(톰캣)과 **Spring Web MVC**를 포함하고 있어 별도의 서버 설정 없이도 웹 서비스 구동이 가능하다.
- **spring-boot-starter-thymeleaf**: 서버 사이드 템플릿 엔진인 타임리프(Thymeleaf)를 사용하여 동적인 HTML 페이지(View)를 렌더링할 수 있게 돕는다.
- **spring-boot-starter(공통)**: 스프링 부트의 가장 기본이 되는 모듈이다. **스프링 코어**와 더불어 **Logback, slf4j** 같은 로깅 라이브러리를 포함하고 있어 애플리케이션의 뼈대와 기록을 담당한다.

### 2. 테스트 라이브러리 (spring-boot-starter-test)

성공적인 개발을 위한 검증 도구들이 한곳에 모여 있다.

- **JUnit**: 자바 테스트의 표준 프레임워크로, 최신 버전인 **Jupiter** 엔진을 통해 효율적인 테스트 실행을 지원한다.
- **Mockito**: 가짜 객체(Mock)를 생성하여 의존 관계가 복잡한 비즈니스 로직을 독립적으로 테스트할 수 있게 해준다.
- **AssertJ**: `assertThat()` 문법을 통해 테스트 코드를 마치 영어 문장처럼 읽기 쉽게 작성하도록 돕는 강력한 단언문 라이브러리다.
- **Spring Test**: 스프링 컨테이너와 연동하여 실제 환경과 유사한 통합 테스트 설정을 지원한다.

### **💡 정리하며**

우리가 `build.gradle`에 추가한 것은 몇 줄의 스타터뿐이지만, 실제로는 이처럼 수많은 라이브러리가 유기적으로 연결되어 프로젝트를 지탱하고 있다. `External Libraries`의 목록은 바로 이러한 **의존성 관리의 결과물**이다.

---

## 5. View 환경설정

### 스프링 부트가 제공하는 Welcome Page 기능

스프링 부트 프로젝트를 실행했을 때 도메인 접속 시 가장 먼저 보여주는 **Welcome Page(첫 화면)** 설정 방법이다.

1. Welcome Page 설정 방법

- **파일 경로:** `src/main/resources/static/index.html` 파일을 생성한다.
- **기능:** 스프링 부트는 해당 경로에 `index.html` 파일이 있으면 이를 자동으로 인식하여 애플리케이션의 **Welcome page**로 사용한다.
![사진9](/assets/img/posts/spring_intro/spring_intro_sec2_9.png)
  - 위와 같이 ‘index.html’을 작성하면 아래와 같이 표시된다. (⇒ welcome 페이지 작성된 것)
![사진10](/assets/img/posts/spring_intro/spring_intro_sec2_10.png)
2. 작동 원리

- 스프링 부트는 별도의 컨트롤러 설정이 없더라도, `static` 혹은 `public` 같은 정적 리소스 경로에서 `index.html`을 먼저 찾는다.
- 공식 문서에 따르면, 이는 스프링 MVC가 제공하는 기본 정적 리소스 핸들링 기능의 일부다.

3. 공식 문서 참고

더 자세한 설정이나 옵션은 스프링 부트 공식 레퍼런스 가이드에서 확인할 수 있다.

- https://docs.spring.io/spring-boot/docs/2.3.1.RELEASE/reference/html/spring-boot-features.html#boot-features-spring-mvc-welcome-page

![사진11](/assets/img/posts/spring_intro/spring_intro_sec2_11.png)

### thymeleaf **템플릿** **엔진**

- `src/main/java/hello.hello_spring/hello.hello_spring.controller` 패키지 생성
- `src/main/java/hello.hello_spring/controller/HelloController` 클래스 생성

![사진12](/assets/img/posts/spring_intro/spring_intro_sec2_12.png)

- `src/main/resources/templates/index.html`

![사진13](/assets/img/posts/spring_intro/spring_intro_sec2_13.png)

- thymeleaf **템플릿엔진** **동작** **확인**

![사진14](/assets/img/posts/spring_intro/spring_intro_sec2_14.png)

- 동작 환경 그림

![사진15](/assets/img/posts/spring_intro/spring_intro_sec2_15.png)

**(1) 웹 요청 처리 흐름**

1. **요청(Request)**: 웹 브라우저에서 `localhost:8080/hello`로 접속하면 내장 톰캣 서버가 요청을 받는다.
2. **컨트롤러 호출**: 스프링 컨테이너 안의 `helloController`가 실행된다.
3. **데이터 및 뷰 전달**: 컨트롤러는 모델에 데이터를 담고(`data:hello!!`), 화면의 이름인 "hello"를 문자열로 리턴한다.
4. **화면 찾기**: 뷰 리졸버(viewResolver)가 리턴된 문자열을 받아 실제 화면 파일을 찾아 처리한다.

**(2) 뷰 리졸버(viewResolver)의 매핑 규칙**

스프링 부트 템플릿 엔진은 기본적으로 다음 경로에서 뷰 파일을 찾도록 설정되어 있다.

- **기본 매핑 경로**: `resources/templates/` + **{ViewName}** + `.html`
- 예를 들어 컨트롤러가 `"hello"`를 반환하면, `resources/templates/hello.html`이 실행된다.

---

## 6. 빌드하고 실행하기
![사진16](/assets/img/posts/spring_intro/spring_intro_sec2_16.png)
- 콘솔로 이동
1. `./gradlew build`
2. `cd build/libs`
3. `java -jar hello-spring-0.0.1-SNAPSHOT.jar`
4. 실행 확인

(cf. **Windows 사용자**: `./gradlew` 대신 `gradlew.bat build` 명령어를 사용)
