---
title: "[코딩 자율학습 스프링 부트 3 자바 백엔드 개발 입문] 11장. HTTP와 REST 컨트롤러"
date: 2026-06-22 22:00:00 +0900
categories: [ECC, Team-Project-BE, springboot3]
tags: [dev, study, java, spring]
render_with_liquid: false
---

["코딩 자율학습 스프링 부트 3 자바 백엔드 개발 입문" 도서 바로가기](https://www.gilbut.co.kr/book/view?bookcode=BN003778)

# 11장. HTTP와 REST 컨트롤러

# 11.1 REST API의 동작 이해하기

REST API를 통해 클라이언트와 서버가 HTTP 요청과 응답을 JSON 데이터로 주고받는 구조는 다음과 같다.

- **HTTP 요청 메시지 구조:** 시작 라인인 '요청 라인(Request line)', 그 아래의 '헤더(Header)', 그리고 '본문(Body)'으로 구성된다. 요청 라인에는 HTTP 메서드, URL 경로, HTTP 버전 정보가 포함된다. 본문에는 전송할 데이터가 JSON 등의 형식으로 담긴다.
- **HTTP 응답 메시지 구조:** 시작 라인인 '상태 라인(Status line)', 그 아래에 '헤더(Header)'와 '본문(Body)'이 온다. 상태 라인에는 HTTP 버전과 HTTP 상태 코드가 포함된다.
- **HTTP 상태 코드:** 요청 성공 시 `200`, 데이터 생성 완료 시 `201`, 요청 자원 없음 시 `404`, 서버 내부 오류 발생 시 `500`을 반환한다. 상태 코드는 100~500번대까지 총 5개 그룹으로 나뉜다.
- **JSON 데이터 구조:** 키(Key)와 값(Value)의 쌍으로 속성을 표현한다. JSON의 값으로는 문자열, 숫자뿐만 아니라 또 다른 JSON 데이터나 배열(Array) 구조를 중첩하여 사용할 수 있다.

### REST와 API의 정의

- **REST(Representational State Transfer):** HTTP URL로 서버의 자원(Resource)을 명시하고, HTTP 메서드(POST, GET, PATCH/PUT, DELETE)를 통해 해당 자원에 대한 CRUD(생성, 조회, 수정, 삭제) 연산을 수행하는 설계 방식을 뜻한다.
- **API(Application Programming Interface):** 클라이언트가 서버의 자원을 요청할 수 있도록 서버에서 제공하는 인터페이스를 의미한다.
- **REST API:** REST 아키텍처 기반으로 구현한 API이다. 기기에 구애받지 않고 서버 자원을 재사용 및 확장할 수 있도록 지원한다.

---

# 11.2 REST API의 구현 과정

게시판 데이터(Article)를 처리하기 위한 REST API 주소 및 메서드는 다음과 같이 설계된다.

- **조회 요청:** `GET` `/api/articles` (전체 목록 조회) 또는 `GET` `/api/articles/{id}` (단일 항목 조회)
- **생성 요청:** `POST` `/api/articles`
- **수정 요청:** `PATCH` `/api/articles/{id}`
- **삭제 요청:** `DELETE` `/api/articles/{id}`

REST API는 일반 컨트롤러와 달리 요청을 받아 그 결과를 JSON 형태의 데이터로 반환하는 **REST 컨트롤러**를 사용하며, 응답 시 HTTP 상태 코드를 세부적으로 제어하기 위해 **ResponseEntity** 클래스를 활용한다.

---

# 11.3 REST API 구현하기

프로젝트 탐색기의 `com.example.firstproject` 하위에 `api` 패키지를 새로 생성하여 구현을 진행한다.

## 11.3.1 REST 컨트롤러 맛보기

1. `api` 패키지 하위에 `FirstApiController` 클래스를 생성한다.
2. 클래스 선언부 위에 REST API용 컨트롤러임을 선언하는 `@RestController` 어노테이션을 추가한다.
3. `localhost:8080/api/hello` 경로로 URL 요청이 들어왔을 때 문자열을 반환하는 `hello()` 메서드를 작성하고 `@GetMapping`을 선언한다.

    ```java
    @RestController
    public class FirstApiController{
        @GetMapping("/api/hello")
        public String hello(){
            return "hello world!";
        }
    }
    ```

4. 서버를 가동한 뒤 Talend API Tester에서 `GET` 메서드로 `http://localhost:8080/api/hello` 주소를 입력하고 [Send]를 클릭하여 상태 코드 `200`과 응답 본문(`BODY`)에 "hello world!" 데이터가 정상 반환되는지 확인한다.

> **💡 REST 컨트롤러와 일반 컨트롤러의 차이점**
일반 컨트롤러는 HTML 등의 **뷰 페이지**를 반환하지만, REST 컨트롤러는 JSON이나 텍스트 형태의 **데이터**를 직접 반환한다.
>

## 11.3.2 REST API: GET 구현하기

### 모든 게시글 조회하기

1. `api` 패키지에 `ArticleApiController` 클래스를 생성하고 `@RestController`를 선언한다.
2. 데이터베이스 조회를 위해 `@Autowired` 어노테이션을 사용하여 `ArticleRepository` 의존성을 주입한다.
3. `@GetMapping("/api/articles")` 요청을 처리하고 `List<Article>`을 반환하는 `index()` 메서드를 정의한다. 반환 구문에는 `articleRepository.findAll()`을 작성한다.

    ```java
    @GetMapping("/api/articles")
    public List<Article> index(){
        return articleRepository.findAll();
    }
    ```


### 단일 게시글 조회하기

1. 기존 `index()` 메서드 하단에 `@GetMapping("/api/articles/{id}")` 어노테이션을 사용하는 `show()` 메서드를 추가한다.
2. URL 경로에 명시된 가변인자 `id`를 메서드의 매개변수로 매핑하기 위해 매개변수 앞에 `@PathVariable` 어노테이션을 지정한다.
3. 반환 구문에 `articleRepository.findById(id).orElse(null)`을 선언하여 데이터가 존재하지 않을 경우 `null`을 반환하도록 설정한다.

    ```java
    @GetMapping("/api/articles/{id}")
    public Article show(@PathVariable Long id){
        return articleRepository.findById(id).orElse(null);
    }
    ```


## 11.3.3 REST API: POST 구현하기

- `@PostMapping("/api/articles")` 어노테이션을 선언하고 `Article` 객체를 반환하는 `create()` 메서드를 정의한다.
- 일반 웹 페이지 입력 폼 전송 방식과 달리, REST API의 본문(`BODY`)에 실려 오는 JSON 데이터를 DTO 매개변수로 정상 매핑하기 위해 매개변수 앞에 `@RequestBody` 어노테이션을 반드시 추가해야 한다.
- 전달받은 DTO 객체를 엔티티로 변환한 후, 리파지터리의 `save()` 메서드를 호출하여 데이터베이스에 저장하고 결과를 반환한다.

    ```java
    @PostMapping("/api/articles")
    public Article create(@RequestBody ArticleForm dto){
        Article article = dto.toEntity();
        return articleRepository.save(article);
    }
    ```


## 11.3.4 REST API: PATCH 구현하기

1. `@PatchMapping("/api/articles/{id}")` 어노테이션을 설정하고 데이터와 상태 코드를 함께 반환할 수 있도록 반환형을 `ResponseEntity<Article>`로 지정한 `update()` 메서드를 정의한다. URL의 id를 받기 위한 `@PathVariable`과 수정할 본문 데이터를 위한 `@RequestBody` 매개변수를 선언한다.
2. 메서드 본문 로직은 다음의 4단계 순서에 입각하여 분할 구현한다.
- **1단계 (DTO -> 엔티티 변환):** 전달받은 DTO를 엔티티 변환 메서드로 치환한다. 데이터 확인용 로그 출력을 위해 클래스 선언부 위에 롬복의 `@Slf4j` 어노테이션을 추가한다.
- **2단계 (타깃 조회):** 리파지터리의 `findById(id).orElse(null)` 메서드를 호출하여 수정할 기존 데이터를 데이터베이스에서 조회하여 변수에 저장한다.
- **3단계 (잘못된 요청 처리):** 데이터베이스에 수정 대상 데이터가 존재하지 않거나(`target == null`), 요청 URL의 id와 본문 데이터의 id가 일치하지 않을 경우(`id != article.getId()`) 조건문을 통해 에러 로그를 기록하고 HTTP 상태 코드 400(`HttpStatus.BAD_REQUEST`)을 반환한다.
- **4단계 (업데이트 및 정상 응답하기):** 대상 엔티티가 실존하는 정상적인 요청일 경우 데이터를 업데이트하고 상태 코드 200(`HttpStatus.OK`)과 함께 수정 완료된 데이터를 반환한다.

### 일부 데이터만 수정할 경우의 코드 보강 (엔티티 메서드 추가)

- 클라이언트가 일부 필드(예: title)를 제외하고 수정 요청을 보낼 경우, DTO 변환 과정에서 제외된 필드가 `null`로 인식되어 기존 데이터가 유실되는 문제가 발생한다.
- 이를 방어하기 위해 도메인 엔티티 클래스인 `Article.java` 파일 내부에 수정용 엔티티에 값이 존재할 때만 부분적으로 필드를 갱신하는 `patch()` 메서드를 정의한다.

    ```java
    // entity/Article.java 내부에 추가
    public void patch(Article article){
        if (article.title != null)
            this.title = article.title;
        if (article.content != null)
            this.content = article.content;
    }
    ```

- 컨트롤러의 4단계 로직을 해당 `patch()` 메서드를 반영하여 최종 완성한다.

    ```java
    // api/ArticleApiController.java 내부 4단계 구현 코드
    target.patch(article);
    Article updated = articleRepository.save(target);
    return ResponseEntity.status(HttpStatus.OK).body(updated);
    ```

![사진1](/assets/img/posts/springboot3/springboot3_chap11_1.png)

## 11.3.5 REST API: DELETE 구현하기

- **삭제 요청 수신 설정:** 메서드 매핑 어노테이션으로 `@DeleteMapping("/api/articles/{id}")`를 정의하고, 반환형을 `ResponseEntity<Article>`로 지정한 `delete()` 메서드를 작성한다.
- **삭제 비즈니스 로직 작성:**Java
  - **1단계 (대상 조회):** 삭제하려는 대상의 실존 여부를 판단하기 위해 `articleRepository.findById(id).orElse(null)` 메서드를 실행한다.
  - **2단계 (잘못된 요청 대응):** 조회된 데이터가 없을 경우(`target == null`), HTTP 상태 코드 400(`HttpStatus.BAD_REQUEST`)과 함께 빈 응답 값을 반환한다.
  - **3단계 (대상 소거 및 응답):** 대상이 존재하면 `articleRepository.delete(target);` 메서드로 해당 데이터를 삭제한다. 삭제 완료 후 상태 코드 200(`HttpStatus.OK`)을 전달하며, HTTP 응답의 body 영역이 없는 응답 객체를 만들기 위해 `build()` 메서드를 활용한다.

    ```java
    @DeleteMapping("/api/articles/{id}")
    public ResponseEntity<Article> delete(@PathVariable Long id){
        // 1. 대상 찾기
        Article target = articleRepository.findById(id).orElse(null);
    
        // 2. 잘못된 요청 처리하기
        if (target == null) {
            return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(null);
        }
    
        // 3. 대상 삭제하기
        articleRepository.delete(target);
        return ResponseEntity.status(HttpStatus.OK).build();
    }
    ```

![사진2](/assets/img/posts/springboot3/springboot3_chap11_2.png)
