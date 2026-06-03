---
title: "[스프링 입문 - 코드로 배우는 스프링 부트, 웹 MVC, DB 접근 기술] 섹션 3. 스프링 웹 개발 기초"
date: 2026-05-02 21:00:00 +0900
categories: [ECC, Team-Project-BE, Spring_intro]
tags: [dev, study, java, spring]
---

["스프링 입문 - 코드로 배우는 스프링 부트, 웹 MVC, DB 접근 기술" 강의 바로가기](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EC%9E%85%EB%AC%B8-%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8)

# 섹션 3. 스프링 웹 개발 기초

## 스프링 웹 개발 기초

- 정적 컨텐츠
  - **개념**: 서버에서 별도의 처리 없이 파일을 웹 브라우저에 그대로 내려주는 방식이다.
  - **특징**: `static` 폴더에 `index.html` 등을 올려두면 스프링 부트가 이를 그대로 제공한다. 사용자는 고정된 화면만 볼 수 있다.
- MVC와 템플릿 엔진
  - **개념**: 서버에서 프로그래밍을 통해 HTML을 동적으로 변경하여 내려주는 방식이다.
  - **특징**:
    - **Model**: 데이터 관리
    - **View**: 화면 렌더링 (Thymeleaf 등 사용)
    - **Controller**: 로직 처리 및 데이터 전달
  - 가장 전통적인 웹 개발 방식으로, 서버에서 HTML을 완성해서 브라우저에 전달한다.
- API
  - **개념**: HTML을 내려주는 대신, 데이터를(객체 등) JSON 형식으로 직접 전달하는 방식이다.
  - **특징**:
    - 최근의 리액트(React), 뷰(Vue) 같은 프론트엔드 프레임워크와 통신할 때 주로 사용한다.
    - 안드로이드나 iOS 앱 개발 시 서버와의 데이터 통신 표준으로 활용된다.
    - `@ResponseBody`를 사용하여 데이터를 직접 반환한다.

---

# 7. 정적 컨텐츠

- 스프링 부트 정적 컨텐츠 기능
- https://docs.spring.io/spring-boot/docs/2.3.1.RELEASE/reference/html/spring-boot-features.html#boot-features-spring-mvc-static-content
![사진1](/assets/img/posts/spring_intro/spring_intro_sec3_1.png)

- `src/resources/static/hello-static.html`
![사진2](/assets/img/posts/spring_intro/spring_intro_sec3_2.png)

- http://localhost:8080/hello-static.html
![사진3](/assets/img/posts/spring_intro/spring_intro_sec3_3.png)

### 정적 컨텐츠 이미지
![사진4](/assets/img/posts/spring_intro/spring_intro_sec3_4.png)
[1. 처리 프로세스]

1. **요청**: 웹 브라우저에서 `localhost:8080/hello-static.html`을 요청합니다.
2. **컨트롤러 확인 (우선순위 1)**: 내장 톰캣 서버가 요청을 받으면 먼저 스프링 컨테이너에서 해당 요청을 처리할 **컨트롤러**가 있는지 찾습니다.
3. **정적 리소스 조회 (우선순위 2)**: 매핑된 컨트롤러가 없다면, `resources: static/hello-static.html` 등 정적 리소스 경로에서 파일을 찾습니다.
4. **응답**: 파일을 찾으면 브라우저에 그대로 반환합니다.

[2. 핵심 요약]

- **우선순위**: 스프링은 항상 **컨트롤러**가 정적 리소스보다 우선순위를 가집니다. 특정 경로의 컨트롤러가 있다면 정적 컨텐츠는 무시됩니다.
- **경로**: 기본적으로 `src/main/resources/static` 위치에 있는 파일들이 정적 컨텐츠로 제공됩니다.
- **특징**: 서버에서 프로그래밍을 할 수 없으며, 작성된 파일 내용 그대로 화면에 출력됩니다.

---

# 8. MVC와 템플릿 엔진

- MVC: Model, View, Controller
  - **Model**: 내부 비즈니스 로직 및 데이터를 처리한다.
  - **View**: 사용자에게 보여줄 화면(UI)을 렌더링한다.
  - **Controller**: 요청을 받고 로직을 실행한 뒤, 결과를 모델에 담아 뷰로 전달한다.

### Controller

![사진5](/assets/img/posts/spring_intro/spring_intro_sec3_5.png)

### View

`src/resources/templates/hello-template.html`
![사진6](/assets/img/posts/spring_intro/spring_intro_sec3_6.png)

### **실행**

http://localhost:8080/hello-mvc?name=spring!!!
![사진7](/assets/img/posts/spring_intro/spring_intro_sec3_7.png)

### MVC, 템플릿 엔진 이미지
![사진8](/assets/img/posts/spring_intro/spring_intro_sec3_8.png)
- 동작 프로세스
1. **요청**: 브라우저가 `/hello-mvc`를 호출하면 내장 톰캣 서버가 이를 컨트롤러로 전달한다.
2. **처리**: 컨트롤러는 로직을 수행한 뒤 **Model**에 데이터를 담고, 화면 이름(예: `hello-template`)을 리턴한다.
3. **매핑**: **viewResolver**가 리턴된 이름과 매칭되는 HTML 파일을 `resources/templates`에서 찾는다.
4. **변환**: 템플릿 엔진(Thymeleaf)이 모델의 데이터를 반영해 변환된 HTML을 브라우저에 전달한다.

---

# 9. API

### @ResponseBody
![사진9](/assets/img/posts/spring_intro/spring_intro_sec3_9.png)
- `@ResponseBody` 를 사용하면 뷰 리졸버(`viewResolver` )를 사용하지 않는다.
- 대신에 HTTP의 BODY에 문자 내용을 직접 반환한다. (HTML BODY TAG를 말하는 것이 아님)

### @ResponseBody 객체 반환
![사진10](/assets/img/posts/spring_intro/spring_intro_sec3_10.png)
- `@ResponseBody` 를 사용하고, 객체를 반환하면 객체가 JSON으로 변환된다.
  - JSON: JavaScript Object Notation은 데이터를 저장하거나 교환하기 위해 사용하는, 사람이 읽기 쉽고 기계가 분석하기 용이한 가벼운 텍스트 기반의 데이터 형식이다.

### @ResponseBody 사용 원리
![사진11](/assets/img/posts/spring_intro/spring_intro_sec3_11.png)
- @ResponseBody란?
  - **직접 반환**: HTTP의 BODY에 문자 내용이나 객체를 직접 반환하는 역할을 한다.
  - **View 생략**: 이 애노테이션이 붙으면 `viewResolver` 대신 **HttpMessageConverter**가 동작하여 화면을 찾는 대신 데이터를 전송한다.
- HttpMessageConverter의 역할
  - 컨트롤러가 반환하는 데이터의 타입에 따라 적절한 컨버터가 선택된다.
  - **문자 처리**: `StringHttpMessageConverter`가 동작하여 문자열을 그대로 내보낸다.
  - **객체 처리**: `MappingJackson2HttpMessageConverter`가 동작하여 객체를 **JSON** 형식으로 변환하여 반환한다.
  - **기타**: byte 처리 등 여러 형태의 컨버터가 기본으로 등록되어 상황에 맞게 동작한다.
- 동작 프로세스
  1. **요청**: 웹 브라우저가 `localhost:8080/hello-api`를 호출한다.
  2. **확인**: 스프링은 컨트롤러에 `@ResponseBody`가 붙어 있는 것을 확인한다.
  3. **변환**: 반환 데이터가 객체라면 **JsonConverter**를 통해 `{name: spring}`과 같은 JSON 데이터로 바꾼다.
  4. **응답**: 변환된 데이터를 HTTP 응답 바디에 실어 브라우저로 보낸다.
- 참고
  - 클라이언트의 HTTP Accept 헤더와 서버의 컨트롤러 반환 타입 정보를 조합해 어떤 `HttpMessageConverter`를 사용할지 결정하게 된다.
