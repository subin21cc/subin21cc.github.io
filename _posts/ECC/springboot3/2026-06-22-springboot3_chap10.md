---
title: "[코딩 자율학습 스프링 부트 3 자바 백엔드 개발 입문] 10장. REST API와 JSON"
date: 2026-06-22 14:00:00 +0900
categories: [ECC, Team-Project-BE, springboot3]
tags: [dev, study, java, spring]
render_with_liquid: false
---

["코딩 자율학습 스프링 부트 3 자바 백엔드 개발 입문" 도서 바로가기](https://www.gilbut.co.kr/book/view?bookcode=BN003778)

# 10장. REST API와 JSON

# 10.1 REST API와 JSON의 등장 배경

웹 서비스를 사용하는 클라이언트에는 웹 브라우저뿐만 아니라 스마트폰, 스마트워치, 태블릿, CCTV, 각종 센서 등 다양한 IT 기기가 존재한다. 기기의 발전에 따라 수많은 종류의 클라이언트가 지속적으로 만들어지고 있다.

서버는 이러한 모든 클라이언트의 요청에 응답해야 한다. 그러나 웹 브라우저를 포함한 수많은 기기 각각에 맞춰 서로 다른 뷰(View) 페이지를 일일이 대응하여 응답하는 방식은 효율적이지 않다. 이에 대한 해결책으로 **REST API**를 사용한다.

REST API(Representational State Transfer API)는 서버의 자원을 클라이언트에 구애받지 않고 사용할 수 있게 하는 설계 방식이다. REST API 방식에서는 HTTP 요청에 대한 응답으로 특정 기기에 종속되지 않고 모든 기기에서 통용될 수 있는 데이터를 반환한다. 즉, 서버는 클라이언트의 요청에 대해 화면(View)이 아닌 데이터(Data)를 전송한다. 이때 주로 통용되는 응답 데이터 형식은 JSON(JavaScript Object Notation)이다. 과거에는 응답 데이터로 XML을 주로 사용했으나, 최근에는 JSON으로 통일되는 추세이다.

> **💡 API(Application Programming Interface)의 정의**애플리케이션을 간편히 사용할 수 있게 하는 미리 정해진 일종의 약속이며, 사용자와 프로그램 간 상호 작용을 돕는다. '윈도우 API'는 윈도우 개발을 위해 미리 정해진 약속이고, '자바 API'는 자바 개발을 위해 미리 만들어진 약속이다. 이 장에서 다루는 REST API는 클라이언트와 서버 사이의 상호 작용, 즉 HTTP 요청에 따른 JSON 응답에 대한 약속을 의미한다.
>

### XML과 JSON의 데이터 형식 비교

- **XML:** 사용자 정의형 HTML 형태로 데이터를 표현한다.
- **JSON:** 자바스크립트 방식을 차용한 객체 표현식이다. 키(Key)와 값(Value)으로 구성된 속성의 집합이다. 키는 문자열이므로 항상 큰따옴표(`""`)로 감싸고, 값은 문자열인 경우에만 큰따옴표(`""`)로 감싼다.

---

# 10.2 REST API 동작 살펴보기

연습용 REST API 서버를 제공하는 {JSON} Placeholder 사이트에 접속하여 HTTP 요청과 응답을 실습할 수 있다.

## 10.2.1 {JSON} Placeholder 사이트 둘러보기

- **사이트 사용 방법:** 'Try it' 영역에 HTTP 요청 코드가 작성되어 있다. [Run script] 버튼을 클릭하면 그에 대한 응답으로 반환되는 JSON 데이터를 확인할 수 있다.
- **사이트에서 제공하는 자원(Resources):** 게시글(posts) 100개, 댓글(comments) 500개, 앨범(albums) 100개, 사진(photos) 5,000개, 할 일 목록(todos) 200개, 사용자(users) 10명을 가짜 더미 데이터로 제공한다.
- **HTTP 메서드와 URL 경로(Routes):** REST API는 모든 HTTP 메서드를 지원하며, 메서드별 요청 URL 경로는 다음과 같이 정의되어 있다.
  - `GET` `/posts`: 모든 게시글 조회
  - `GET` `/posts/1`: 1번 게시글 조회
  - `GET` `/posts/1/comments`: 1번 게시글의 모든 댓글 조회
  - `POST` `/posts`: 게시글 생성
  - `PUT` `/posts/1`: 1번 게시글 수정(전체 변경)
  - `PATCH` `/posts/1`: 1번 게시글 수정(일부 변경)
  - `DELETE` `/posts/1`: 1번 게시글 삭제
- **REST API 사용 예시 설명**
  - **단일 데이터 조회:** `https://.../posts/1` URL 경로를 호출하면 1번 게시글의 데이터가 JSON 형태로 응답된다. 데이터 조회를 요청할 때 `method` 속성의 기본값이 `GET`이므로 구문에서 생략이 가능하다.
  - **모든 데이터 조회:** `https://.../posts` 경로로 요청을 보내며, 1번부터 100번까지의 모든 게시글이 JSON 데이터 형식으로 반환된다.
  - **데이터 생성:** 데이터 생성을 요청하므로 `method` 속성 값은 `POST`가 된다. 요청의 `body` 부분에 JSON 데이터로 새 게시글의 속성 값들을 실어 보내면, 지정된 URL로 요청이 전송되어 자원이 생성되고 결과가 JSON 형태로 반환된다.
  - **데이터 수정:** 데이터 수정 요청이므로 `method` 속성 값으로 `PUT`과 `PATCH`를 사용한다. `PUT`은 기존 데이터를 전부 새 내용으로 변경하며 기존 데이터가 없으면 새로 생성하는 동작을 수행하고, `PATCH`는 기존 데이터 중에서 일부 속성만 새 내용으로 변경한다.
  - **데이터 삭제:** 데이터 삭제 요청이므로 `method` 속성 값으로 `DELETE`를 사용하며, 특정 게시글 경로를 타겟으로 지정하여 삭제를 요청한다.

## 10.2.2 Talend API Tester 설치하기

크롬 웹 브라우저에서 HTTP 요청과 응답을 테스트할 수 있는 Talend API Tester 확장 프로그램을 설치한다.

1. 구글 검색창에 'talend api 확장 프로그램'을 검색하고 크롬 웹 스토어의 Talend API Tester 링크를 클릭한다.
2. 설치 화면에서 [Chrome에 추가] 버튼을 클릭한 후, 확인 창이 발생하면 [확장 프로그램 추가] 버튼을 클릭한다.
3. 크롬 브라우저 우측 상단의 퍼즐 모양 아이콘을 선택하여 Talend API Tester 프로그램의 핀 아이콘을 활성화하고 바로 가기를 생성한다.
4. 바로 가기 아이콘을 클릭한 뒤 화면 중앙의 [Use Talend API Tester - Free Edition] 버튼을 클릭하여 프로그램을 실행한다.

![사진1](/assets/img/posts/springboot3/springboot3_chap10_1.png)

## 10.2.3 GET 요청하고 응답받기

1. **모든 게시글 조회:** 메서드를 `GET`으로 선택하고 URL란에 `https://jsonplaceholder.typicode.com/posts`를 입력한 후 [Send] 버튼을 클릭한다.
2. 요청이 성공적으로 처리되었음을 의미하는 상태 코드 `200`이 응답으로 오며, `BODY` 영역에서 1번부터 100번까지 게시글의 JSON 데이터를 확인할 수 있다.
3. **1번 게시글 조회:** `GET` 요청 상태에서 URL 마지막에 `/1`을 추가하여 `https://jsonplaceholder.typicode.com/posts/1`로 수정한 후 [Send]를 클릭하면 상태 코드 `200`과 함께 1번 게시글의 JSON 데이터가 `BODY`에 출력된다.
4. **조회 요청 실패 확인:** 제공되는 게시글 범위를 초과하는 101번을 타겟으로 정하여 URL을 `https://jsonplaceholder.typicode.com/posts/101`로 수정하고 [Send]를 클릭하면, 요청한 자원을 찾을 수 없음을 뜻하는 상태 코드 `404` 에러가 반환되며 `BODY` 영역은 빈 상태(`{}`)가 된다.
- **HTTP 상태 코드 분류:** 상태 코드는 클라이언트가 보낸 요청의 처리 결과를 알려주는 코드이다. 1XX(정보), 2XX(성공), 3XX(리다이렉션 메시지), 4XX(클라이언트 요청 오류), 5XX(서버 응답 오류) 그룹으로 나뉜다.
- **HTTP 요청 메시지와 응답 메시지 구조:** 실제 요청과 응답은 HTTP 메시지에 실려 전송되며, 시작 라인(Start line), 헤더(Header), 빈 라인(Blank line), 본문(Body)으로 구성된다.
  - **시작 라인:** HTTP 요청 종류(Method) 및 URL 경로, 사용하는 HTTP 버전 정보 또는 응답 상태 코드가 위치한다.
  - **헤더:** 호스트 주소, 응답 날짜, 데이터 형식 등 전송에 필요한 부가 정보(Metadata)가 담긴다.
  - **빈 라인:** 헤더의 끝을 알리는 공백 줄이다.
  - **본문:** 실제 전송하고 주고받는 데이터가 담기는 영역이다.

![사진2](/assets/img/posts/springboot3/springboot3_chap10_2.png)

## 10.2.4 POST 요청하고 응답받기

1. 메서드를 `POST`로 선택하고 URL을 `https://jsonplaceholder.typicode.com/posts`로 설정한다. `BODY` 영역에 JSON 형식에 맞추어 생성할 데이터 구조(`"title"`, `"body"`)를 입력한 후 [Send] 버튼을 클릭한다.
2. 데이터가 정상적으로 생성되었음을 의미하는 상태 코드 `201`이 반환되며, `BODY` 부분에 자동으로 `id: 101` 번호가 할당되어 새로 생성된 데이터 내용이 JSON 형식으로 출력된다.
3. **생성 요청 실패 확인:** 문법 오류 발생 시의 응답을 확인하기 위해 `BODY` 영역의 JSON 데이터에서 `title`과 `body` 속성의 큰따옴표를 강제로 제거하여 규격을 손상시킨 후 [Send] 버튼을 클릭한다.
4. 서버 내부에 에러가 발생했음을 알리는 상태 코드 `500`이 반환되며, 구문 오류가 일어났음을 나타내는 `SyntaxError` 메시지가 `BODY` 영역에 출력된다.

![사진3](/assets/img/posts/springboot3/springboot3_chap10_3.png)

## 10.2.5 PATCH 요청하고 응답받기

1. 데이터 수정 요청을 보내기 위해 메서드를 `PATCH`로 선택한다. 1번 게시글 수정을 목적으로 URL을 `https://jsonplaceholder.typicode.com/posts/1`로 지정하고, `BODY` 영역에 수정할 데이터 내용을 JSON 규격에 맞춰 작성한 후 [Send] 버튼을 클릭한다.
2. 정상 처리 상태 코드인 `200`이 반환되며, `BODY` 영역을 통해 수정한 내용이 최종 반영되어 데이터가 변경되었음을 확인한다.

![사진4](/assets/img/posts/springboot3/springboot3_chap10_4.png)

## 10.2.6 DELETE 요청하고 응답받기

1. 메서드를 `DELETE`로 변경하고 10번 게시글 삭제 처리를 지시하기 위해 URL을 `https://jsonplaceholder.typicode.com/posts/10`으로 수정한 뒤 [Send] 버튼을 클릭한다.
2. 성공적으로 자원이 소멸되었음을 의미하는 상태 코드 `200`이 응답으로 오며, 본문(`BODY`) 영역의 JSON 데이터 데이터셋이 완전하게 소거되어 빈 상태(`{}`)로 반환된다.

![사진5](/assets/img/posts/springboot3/springboot3_chap10_5.png)
