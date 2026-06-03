---
title: "[스프링 입문 - 코드로 배우는 스프링 부트, 웹 MVC, DB 접근 기술] 섹션 6. 회원 관리 예제 - 웹 MVC 개발"
date: 2026-05-09 12:00:00 +0900
categories: [ECC, Team-Project-BE, Spring_intro]
tags: [dev, study, java, spring]
---

["스프링 입문 - 코드로 배우는 스프링 부트, 웹 MVC, DB 접근 기술" 강의 바로가기](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EC%9E%85%EB%AC%B8-%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8)


# 섹션 6. 회원 관리 예제 - 웹 MVC 개발

- 회원 등록 및 조회를 위한 핵심 기능을 포함한 아주 단순한 형태의 웹 사이트를 Spring MVC 기반으로 구축해 보겠다.

# 17. 회원 웹 기능 - 홈 화면 추가

- `src/main/java/controller/HomeController`
  ![사진1](/assets/img/posts/spring_intro/spring_intro_sec6_1.png)
  ⇒ 웹 애플리케이션의 루트 경로(`/`)로 들어오는 HTTP GET 요청을 처리하는 컨트롤러다. "home"이라는 뷰 이름을 반환하여 홈 화면으로 연결한다.

- `src/main/resource/templates/home.html`
  ![사진2](/assets/img/posts/spring_intro/spring_intro_sec6_2.png)
  ⇒ Thymeleaf 템플릿 엔진을 사용해 제작된 홈 화면 마크업이다. 회원 가입과 회원 목록 조회를 위한 하이퍼링크 버튼을 포함하고 있다.

- 결과 보기 - `localhost:8080`
  ![사진3](/assets/img/posts/spring_intro/spring_intro_sec6_3.png)
  ⇒ 실제 브라우저에서 실행된 모습이다. "Hello Spring" 타이틀과 함께 기능 메뉴가 정상적으로 렌더링된 것을 확인할 수 있다.

- **우선순위 참고:** 스프링 부트는 요청이 들어오면 내장 톰캣 서버가 먼저 관련 **컨트롤러**가 있는지 탐색하고, 없을 경우에만 `static` 폴더의 정적 파일을 찾는다.

---

# 18. 회원 웹 기능 - 등록

### 회원 등록 폼 개발

- 회원 등록 폼 컨트롤러 - `src/main/java/hello.hello_spring/controller/MemerController`
  ![사진4](/assets/img/posts/spring_intro/spring_intro_sec6_4.png)
  ⇒ `/members/new` 경로로 들어오는 GET 요청을 처리하여 회원 등록 폼 화면을 반환하도록 설정된 컨트롤러 소스 코드이다.

- 회원 등록 폼 HTML - `src/main/resources/templates/members/createMemberForm.html`
  ![사진5](/assets/img/posts/spring_intro/spring_intro_sec6_5.png)
  ⇒ 사용자로부터 이름을 입력받기 위한 `<form>` 태그와 `input` 필드가 포함된 HTML 템플릿이다. `post` 방식으로 데이터를 전송하도록 설계되었다.

- 결과 보기 - `localhost:8080/members/new`
  ![사진6](/assets/img/posts/spring_intro/spring_intro_sec6_6.png)
  ⇒ 애플리케이션 실행 후 등록 페이지에 접속하면, 작성한 HTML 템플릿이 정상적으로 렌더링되어 사용자 입력을 대기하는 화면을 확인할 수 있다.

⇒ 브라우저를 통해 실제 구현된 회원 등록 폼 페이지다. '이름' 입력창과 등록 버튼이 포함된 UI가 정상적으로 출력된다.

### 회원 등록 컨트롤러

- 웹 등록 화면에서 데이터를 전달 받을 폼 객체 - `src/main/java/hello.hello_spring/controller/MemberForm`
  ![사진7](/assets/img/posts/spring_intro/spring_intro_sec6_7.png)
  ⇒ HTML의 `input` 태그 내 `name` 속성과 매핑되는 `name` 필드를 가진 DTO(Data Transfer Object) 클래스다. 클라이언트가 전송한 데이터를 객체 형태로 바인딩하는 역할을 한다.

- 회원 컨트롤러에서 회원을 실제 등록하는 기능 - `src/main/java/hello.hello_spring/controller/MemberController`
  - `create` 메서드 추가
    ![사진8](/assets/img/posts/spring_intro/spring_intro_sec6_8.png)
    ⇒ `@PostMapping`을 사용하여 `/members/new` 경로로 들어온 POST 요청을 처리한다. 전달받은 `MemberForm` 객체에서 이름을 추출해 서비스 계층에 가입을 요청하고, 처리가 완료되면 홈 화면(`/`)으로 리다이렉트한다.

---

# 19. 회원 웹 기능 - 조회

- 회원 컨트롤러에서 조회 기능 - `src/main/java/hello.hello_spring/controller/MemberController`
  - `list` 메서드 추가
    ![사진9](/assets/img/posts/spring_intro/spring_intro_sec6_9.png)
    ⇒ `/members` 경로로 들어오는 GET 요청을 처리한다. `memberService.findMembers()`를 통해 전체 회원 리스트를 조회하고, 이를 `Model` 객체에 `"members"`라는 이름으로 담아 `memberList` 뷰로 넘겨준다.

- 회원 리스트 HTML - `src/main/java/resources/templates/members/memberList.html`
  ![사진10](/assets/img/posts/spring_intro/spring_intro_sec6_10.png)
  ⇒ Thymeleaf의 `th:each` 문법을 사용하여 전달받은 회원 리스트를 반복 루프로 처리한다. 각 회원의 ID와 이름을 테이블(Table) 형식으로 동적으로 렌더링한다.

### 결과 보기

- **초기 목록:** 등록된 회원이 없는 상태의 빈 목록 화면이다.
  ![사진11](/assets/img/posts/spring_intro/spring_intro_sec6_11.png)

- **데이터 입력:** 등록 폼(`members/new`)에서 'spring1', 'spring2' 등의 데이터를 순차적으로 입력하고 등록 버튼을 클릭한다.
  ![사진12](/assets/img/posts/spring_intro/spring_intro_sec6_12.png)
  ![사진13](/assets/img/posts/spring_intro/spring_intro_sec6_13.png)

- **최종 조회 결과:** 등록된 회원들이 고유 번호(ID)와 함께 목록에 실시간으로 반영된 것을 확인할 수 있다. 데이터는 메모리상에 저장되어 있으므로 서버 재시작 전까지 유지된다.
  ![사진14](/assets/img/posts/spring_intro/spring_intro_sec6_14.png)


---


## 📝 요약 및 정리

이번 섹션에서는 스프링 빈과 의존관계를 바탕으로 실제 웹 MVC를 구현하며 회원 등록 및 조회 기능을 완성했다.

### 1. 홈 화면 추가 및 이동

- **HomeController 생성**: 애플리케이션의 루트 경로(`/`)로 들어오는 GET 요청을 처리하는 컨트롤러를 작성했다.

- **홈 화면 연결**: "home"이라는 뷰 이름을 반환하여 `src/main/resource/templates/home.html`과 연결했다.

- **Thymeleaf 활용**: 홈 화면에는 회원 가입과 회원 목록 조회를 위한 하이퍼링크 버튼을 포함했다.

- **컨트롤러 우선순위**: 스프링 부트는 요청이 오면 내장 톰캣 서버에서 관련 컨트롤러를 먼저 찾고, 없을 때만 정적 컨텐츠(`static`)를 찾는다.



### 2. 회원 등록 기능 구현

- **등록 폼 컨트롤러**: `MemberController`에서 `/members/new` 경로의 GET 요청을 받아 회원 등록 폼 화면을 반환한다.

- **HTML 폼 구성**: `createMemberForm.html`을 생성하여 사용자로부터 이름을 입력받는 `<form>` 태그와 `POST` 방식의 데이터 전송을 설계했다.

- **데이터 바인딩 (DTO)**: 클라이언트가 전송한 데이터를 객체 형태로 받기 위해 `MemberForm` 클래스를 생성하고, HTML의 `name` 속성과 매핑했다.

- **등록 로직**: 컨트롤러의 `create` 메서드에서 전달받은 `MemberForm` 객체를 이용해 회원을 실제 등록한다.



### 3. 회원 목록 조회 기능 구현

- **회원 리스트 컨트롤러**: `/members` 경로로 들어오는 GET 요청을 처리하며, `memberService`를 통해 전체 회원을 조회한다.

- **Model 객체 활용**: 조회된 회원 리스트를 `Model` 객체에 담아 뷰(`memberList`)로 전달한다.

- **동적 렌더링**: `memberList.html`에서 Thymeleaf의 `th:each` 문법을 사용하여 회원 정보를 테이블 형식으로 반복 출력한다.


### 4. 실행 결과 및 특징

- **실시간 반영**: 등록 폼에서 데이터를 입력하면 목록 조회 화면에 고유 번호(ID)와 이름이 실시간으로 반영된다.

- **메모리 저장소**: 현재 데이터는 메모리에 저장되므로, 서버를 재시작하면 데이터가 초기화된다.


### **핵심 키워드:** 
`Controller`, `Thymeleaf`, `POST/GET`, `DTO(MemberForm)`, `Model`
