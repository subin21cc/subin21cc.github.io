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
![사진1](/assets/img/posts/spring_intro_sec2_1.png)

- Project: Gradel - Groovy
- Language: Java
- Spring Boot: 3.5.14
- Project Metadata
  - Group: hello
  - Artifact: hello-spring
- Dependencies: Spring Web, Thymeleaf

![사진2](/assets/img/posts/spring_intro_sec2_2.png)
![사진3](/assets/img/posts/spring_intro_sec2_3.png)

- src에서 보통 main이랑 test 폴더 나눔
- resource폴더: 실제 자바 코드를 제외한 xml, properties, html 등 설정 파일 포함
- build.gradle

![사진4](/assets/img/posts/spring_intro_sec2_4.png)
- mavenCentral(): 라이브러리 다운받는 사이트 설정
- .gitignore: 소스코드 관리 → git에는 필요한 소스코드만 올리고 나머지 빌드된 결과물 등은 안 올리도록 한다.

### Gradle 빌드 실패 할 때?
![사진5](/assets/img/posts/spring_intro_sec2_5.png)
- “File” - “Project structure…” - “Project” -”SDK”를 JAVA17로 설정한다. (17인 이유는 위의 스프링부트 스타터에서 17로 설정했기 때문이다. 설정한 버전에 맞추면 된다.)
- “Gradle Settings”에서도 Gradle JDK를 17로 맞춰준다.
- “Gradle “탭을 클릭하고 “Reload All Gradle Projects (새로고침 아이콘)”를 누르면 설정 완료된다.

### 톰캣 서버 구동 확인 및 localhost:8080 접속
![사진6](/assets/img/posts/spring_intro_sec2_6.png)
- 서버 로그에 Tomcat initialized with port **8080**이 뜨고 실행도 성공했지만, 브라우저에서 localhost:8080 접속 시 “**White Label Error Page**”나 "**연결할 수 없음**"이 뜬다.
- 원인: 톰캣 서버 엔진은 정상적으로 구동되었으나, 사용자의 요청(URL)을 받아 처리할 **컨트롤러(Controller)**나 보여줄 **파일(Index.html 등)**이 프로젝트에 구현되어 있지 않기 때문이다.

⇒ 여기까지 하면, 프로젝트 환경설정에 성공한 것이다!

### 번외) 더 빠른 개발을 위한 빌드 및 실행 환경 최적화
![사진7](/assets/img/posts/spring_intro_sec2_7.png)
- **설정 경로:** Settings → Build, Execution, Deployment → Build Tools → Gradle
- **Build and run using**과 **Run tests using** 항목을 Gradle(Default)에서 IntelliJ IDEA로 변경
- 설정 바꾸는 이유: 기본적으로 IntelliJ는 빌드와 실행을 Gradle을 거쳐서 수행한다. 이 과정을 IntelliJ가 직접 처리하도록 변경하면 실행 속도를 훨씬 높일 수 있다.
